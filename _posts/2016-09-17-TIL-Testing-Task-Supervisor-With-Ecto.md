---
layout: post
title:  "Today I Learned: How to test Task.Supervisor.async_nolink with Ecto 2.0"
date:   2016-09-18 18:00:00
categories: today-i-learned
---

In the project I'm currently working on, I have a piece of code responsible for calculating some statistics and sending it via Phoenix Channels after changing adding or removing some records in the API call.
Since don't want to wait for this to render the response, I'm using `Task.Supervisor.async_nolink` to do this.

{% highlight elixir %}
Task.Supervisor.async_nolink(MyApp.TaskSupervisor, fn ->
  ## Do something with Repo
end)
{% endhighlight %}

There is one problem though when I run my tests. Since I'm using Ecto Repo in the async task, its process have to share a connection with the test process, which gives a following error:

{% highlight bash %}
[error] Postgrex.Protocol (#PID<0.XXX.0>) disconnected: **
(DBConnection.ConnectionError) owner #PID<0.XXX.0> exited while
client #PID<0.XXX.0> is still running with: shutdown
{% endhighlight %}

(If you want to find out more about connection ownership, you can read [this article][ownership])

Disabling the task in tests wasn't an option, so I figured out that I can solve this problem by running the task synchronously in tests.

I'm using the technique described [here][mocks] to replace Task.Supervisor while running tests.

{% highlight elixir %}
@task_supervisor Application.get_env(:app, :task_supervisor) || Task.Supervisor

## ...

@task_supervisor.async_nolink(TaskSupervisor, fn ->
  ## ...
end)
{% endhighlight %}

Then, I define TestTaskSupervisor to be used in tests:

{% highlight elixir %}
defmodule TestTaskSupervisor do
  def async_nolink(_, fun), do: fun.()
end
{% endhighlight %}

The last thing to do is to set:

{% highlight elixir %}
config :my_app, :task_supervisor, TestTaskSupervisor
{% endhighlight %}

in the `config/test.exs` file. Now, I'm sure that the task will run synchronously and finish before the test process without causing the `ConnectionError`.


[mocks]: http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/
[ownership]: https://medium.com/@qertoip/making-sense-of-ecto-2-sql-sandbox-and-connection-ownership-modes-b45c5337c6b7#.8zu5l91q2
