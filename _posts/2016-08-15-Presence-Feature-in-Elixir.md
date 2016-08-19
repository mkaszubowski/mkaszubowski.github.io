---
layout: post
title:  "Building Presence feature in Elixir"
date:   2016-08-02 23:56:27 -0400
categories: elixir
---

# Introduction

Recently, I've been building a simple Google Docs clone in Elixir/Phoenix. One of the features I've implemented was a live list of users currently watching a document. It was based on the Phoenix channel sending messages to a GenServer when user connected/disconnected. I did a few tests and I was quite happy with it. Then I read about [Phoenix Presence][phoenix-presence] and realized that my system has a few flaws.

# Distributed systems

The main problem with my solution is that it only worked on a single node. After [starting phoenix in distributed enviroment][multiple-nodes], each node spawned its owned process for my GenServer, and they did not shared any state. That's not good, let's try to fix that.

# Back to the beggining

Before starting the task of synchronizing the nodes, let's talk about a basic solution. For the sake of simplicity, we'll completely forget about web interface and Phoenix and focus on pure OTP application:

{% highlight elixir %}
defmodule Presence.UserList do
  use GenServer

  @name :user_list

  ## Client API
  def start_link() do
    GenServer.start_link(__MODULE__, :ok ,name: @name)
  end

  def add_user(user), do: GenServer.cast(@name, {:add_user, user})
  def remove_user(%{id: id}), do: remove_user(id)
  def remove_user(user_id), do: GenServer.cast(@name, {:remove_user, user_id})
  def get_users(), do: GenServer.call(@name, :get_users)

  ## Server Callbacks
  def init(:ok) do
    {:ok, []}
  end

  def handle_call(:get_users, _from, users) do
    {:reply, users, users}
  end

  def handle_cast({:add_user, user}, users) do
    {:noreply, [user | users]}
  end

  def handle_cast({:remove_user, user_id}, users) do
    users = Enum.reject(users, fn(user) -> user.id == user_id end)
    {:noreply, users}
  end
end
{% endhighlight %}

We've got here two asynchronous casts to add and remove the user from the list and one synchronous call to get the list back. Nothing complicated right now.

To make things easier, we can add our task to the app's supervision tree:

{% highlight elixir %}
defmodule Presence do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      worker(Presence.UserList, [])
    ]

    opts = [strategy: :one_for_one, name: Presence.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
{% endhighlight %}

# Multiple nodes

OK, now's the time to think about synchronizing multiple nodes.

Following [Chris' post][multiple-nodes], we can add `sys.config` file ad the root of our project:

{% highlight erlang %}
[{kernel,
  [
    {sync_nodes_optional, ['n1@127.0.0.1', 'n2@127.0.0.1']},
    {sync_nodes_timeout, 10000}
  ]}
].
{% endhighlight %}

Then, in two terminal windows, we can start `iex` sessions on our nodes.

{% highlight bash %}
iex --name n1@127.0.0.1 --erl "-config sys.config" -S mix
iex --name n2@127.0.0.1 --erl "-config sys.config" -S mix
{% endhighlight %}

And verify that they are connected:

{% highlight iex %}
iex(n1@127.0.0.1)1> Node.list
[:"n2@127.0.0.1"]


iex(n2@127.0.0.1)1> Node.list
[:"n1@127.0.0.1"]
{% endhighlight %}

Notice that [Node.list/0][node-list] returns the list of connected nodes excluding the local node.

To synchronize the nodes we'll use asynchronous communication. Nodes will send messages using `cast` including their own users list and then handle that messages to update their state. Fortunately, instead of iterating through the list of the nodes, we can use [GenServer.abcast/3][abcast] to send the same message to the list of connected nodes with `GenServer.abcast(Node.list(), :user_list, message)`.



{% highlight elixir %}
defmodule Presence.UserList do
  use GenServer

  @name :user_list

  ## Client API
  def start_link() do
    GenServer.start_link(__MODULE__, :ok ,name: @name)
  end

  def add_user(user), do: GenServer.cast(@name, {:add_user, user})
  def remove_user(%{id: id}), do: remove_user(id)
  def remove_user(user_id), do: GenServer.cast(@name, {:remove_user, user_id})
  def get_users(), do: GenServer.call(@name, :get_users)

  ## Server Callbacks
  def init(:ok) do
    :timer.send_interval(3_000, :begin_sync)
    {:ok, %{users: [], nodes_state: %{}}}
  end

  def handle_call(:get_users, _from, %{users: users, nodes_state: nodes_state} = state) do
    nodes_users = nodes_state |> Map.values |> List.flatten
    all_users = Enum.uniq(users ++ nodes_users)

    {:reply, all_users, state}
  end

  def handle_cast({:add_user, user}, %{users: users} = state) do
    users = [user | users]
    state = %{state | users: users}
    {:noreply, state}
  end

  def handle_cast({:remove_user, user_id}, %{users: users} = state) do
    users = Enum.reject(users, fn(user) -> user.id == user_id end)
    state = %{state | users: users}
    {:noreply, state}
  end


  def handle_cast({:sync, node, node_users}, %{nodes_state: nodes_state} = state) do
    nodes_state = Map.merge(nodes_state, %{node => node_users})
    state = %{state | nodes_state: nodes_state}

    {:noreply, state}
  end

  def handle_info(:begin_sync, %{users: users} = state) do
    GenServer.abcast(Node.list(), @name, {:sync, Node.self(), users})

    {:noreply, state}
  end
end
{% endhighlight %}

There are a few changes. First of all, we've added `:timer.send_interval(3_000, :begin_sync)` to our `init` function. It will send `:begin_sync` message every 3 seconds. We handle it in the `handle_info` function and send local node's state to other nodes. This message is then handled by `handle_cast({:sync, node, node_users}, %{nodes_state: nodes_state} = state)`. Notice that we've changes server's state structure. We now hold the list of users connected to local node and the map with other nodes' lists. The `handle_call(:get_users, _from, %{users: users, nodes_state: nodes_state} = state)` function merges all this data when returning the users.

Ok, that's a step forward. When we now start two nodes and add different users in each one, after about 3 seconds, the nodes state should be synchronized.

{% highlight bash %}
iex(n1@127.0.0.1)1> UserList.add_user(%{id: 1, name: "José"})
:ok
iex(n1@127.0.0.1)2> UserList.get_users
[%{id: 1, name: "José"}, %{id: 2, name: "Chris"}]
{% endhighlight %}

{% highlight bash %}
iex(n2@127.0.0.1)1> UserList.add_user(%{id: 2, name: "Chris"})
:ok
iex(n2@127.0.0.1)2> UserList.get_users
[%{id: 2, name: "Chris"}, %{id: 1, name: "José"}]
{% endhighlight %}

## Handling a failure

One thing we can be sure while building a disctributed system is that there WILL be failures. We have to take care of handling them.
Elixir can provide us with `Node.monitor/2` function which takes a node and `true` as a second argument. It sets up the monitor which sends `{:nodedown, node}` message when the monitored node fails. We can handle that information in `handle_info`:

{% highlight elixir %}
def init(:ok) do
  :timer.send_interval(3_000, :begin_sync)
  {:ok, %{users: [], nodes_state: %{}, nodes: []}}
end

def handle_info({:nodedown, node},
  %{nodes: nodes, nodes_state: nodes_state} = state) do
  nodes = List.delete(nodes, node)
  nodes_state = Map.delete(nodes_state, node)

  new_state = %{state | nodes: nodes, nodes_state: nodes_state}
  {:noreply, new_state}
end
{% endhighlight %}

Notice that we've added `nodes` list to the server's state. To understand why, we have to know that each time `Node.monitor(node, true)` is called, a new monitor is created. We don't want that. That's why we will store a list of monitored nodes in the state.
We can set up the monitors in the existing handler for `:begin_sync`:

{% highlight elixir %}
def handle_info(:begin_sync, %{users: users, nodes: current_nodes} = state) do
  case Node.list() -- current_nodes do
    [] -> {:noreply, state}
    new_nodes ->
      for node <- new_nodes, do: Node.monitor(node, true)
      GenServer.abcast(new_nodes, @name, {:sync, Node.self(), users})

      nodes = new_nodes ++ current_nodes
      new_state = %{state | nodes: nodes}
      {:noreply, new_state}
  end
end
{% endhighlight %}

Here we're checking if there are any new nodes connected. If they are, we start to monitor them, send them the state of the local node and add the node to the state.
Now we will be notified if any other node goes down so `UserList.get_users` won't return the users from the failed nodes. That's what we wanted, but in the meantime, we broke the synchronization. We only send the state once to each node. In order to fix this, let's send `:sync` messages every time we add or remove the user:

{% highlight elixir %}
# Only updated fuctions are shown here

def handle_cast({:add_user, user}, %{users: users, nodes: nodes} = state) do
  users = [user | users]
  state = %{state | users: users}
  sync(nodes, users)

  {:noreply, state}
end

def handle_cast({:remove_user, user_id}, %{users: users, nodes: nodes} = state) do
  users = Enum.reject(users, fn(user) -> user.id == user_id end)
  state = %{state | users: users}
  sync(nodes, users)

  {:noreply, state}
end

def handle_info(:begin_sync, %{users: users, nodes: current_nodes} = state) do
  case Node.list() -- current_nodes do
    [] -> {:noreply, state}
    new_nodes ->
      for node <- new_nodes, do: Node.monitor(node, true)
      sync(new_nodes, users)

      nodes = new_nodes ++ current_nodes
      new_state = %{state | nodes: nodes}
      {:noreply, new_state}
  end
end

defp sync(nodes, users) do
  GenServer.abcast(nodes, @name, {:sync, Node.self(), users})
end
{% endhighlight %}

And that's it for now, if you now play with our server in the IEx, everything should work fine.

# Summary

The main purpose of this post was to document my learning process and experiments with OTP and distribution of elixir applications. That's why I'm pretty sure that there are some things that could be improved or maybe even entirely wrong. Nevertheles, I think I achieved my goal which was to improve my previous solution. I'm going to keep exploring this field and try to deal with the few issues I know about:

- There are a lot of communication going on. Each node is sending messages to every connected node.
- Lack of ordering of messages. I suppose that nodes' states might not be correct all the time because of the asynch
 messages.
- Too much information in each messages - sending the whole state is not necessary when we add/remove the user.
- Lack of tests - I have not found enough information about testing distributed systems in elixir yet.

And probably many more. In the future, I'll focus on implementing lighter messages between nodes (containing only information about user joining/leaving), adding a logical vector clock to keep the messages in order and adding some tests.

Of course, if you want to implement a presence feature in your system, [Phoenix Presence][phoenix-presence] is probably the best option.

The last thing: I would really appreciate all the comments, both positive and negative. You can contact me at maciej@mkaszubowski.pl. Thanks for reading!


[phoenix-presence]: https://www.google.com/?ion=1&espv=2#q=phoenix%20presence
[multiple-nodes]: https://dockyard.com/blog/2016/01/28/running-elixir-and-phoenix-projects-on-a-cluster-of-nodes
[node-list]: http://elixir-lang.org/docs/stable/elixir/Node.html#list/0
[abcast]: http://elixir-lang.org/docs/stable/elixir/GenServer.html#abcast/3
