---
layout: post
title:  "Today I Learned: Two things about user authentication security."
date:   2016-08-02 23:56:27 -0400
categories: today-i-learned
---

I am currently reading 'Programming Phoenix' book by Chris McCord, Bruce Tate and José Valim. I've already written few authentication solutions so I hadn't expected anything new, but there were two important security-related things that I didn't know and which I hadn't seen before (or didn't pay enought attention - for my defence, none of the solutions I wrote was used in production, it was all for learning purposes!) in any authentication tutorials out there.

# Session fixation attack

According to the [OWASP Website][session-fixation]:

> Session Fixation is an attack that permits an attacker to hijack a valid user session. The attack explores a limitation in the way the web application manages the session ID, more specifically the vulnerable web application. When authenticating a user, it doesn’t assign a new session ID, making it possible to use an existent session ID. The attack consists of obtaining a valid session ID (e.g. by connecting to the application), inducing a user to authenticate himself with that session ID, and then hijacking the user-validated session by the knowledge of the used session ID. The attacker has to provide a legitimate Web application session ID and try to make the victim's browser use it.

The fix for this is fortunately very simple if you're using Phoenix (or Plug, to be more precise).

{% highlight elixir %}
conn
|> put_session(:user_id, user.id)
|> configure_session(renew: true)
{% endhighlight %}

( [configure_session/2][configure_session] )

The `renew: true` option ensures that a new session id is generated for the cookie before sending it to the client.


# Timing attack

According to [Wikipedia][timing-attack]:

> In cryptography, a timing attack is a side channel attack in which the attacker attempts to compromise a cryptosystem by analyzing the time taken to execute cryptographic algorithms. Every logical operation in a computer takes time to execute, and the time can differ based on the input; with precise measurements of the time for each operation, an attacker can work backwards to the input.


Imagine the following code:

{% highlight elixir %}
import Comeonin.Bcrypt, only: [checkpw: 2]

def login(username, password) do
  user = get_user(username)

  cond do
    user && checkpw(password, user.hashed_password) ->
      {:ok, user}
    true ->
      {:error, :not_found}
  end
end
{% endhighlight %}

You can imagine that, given a large number of request, somebody could measure response times after providing random credentials. Even though they might not (and probably won't) guess a correct username/combination, knowing that for exising usernames, the checkpw function is executed, they can figure out if a user with a given username is in our database. This can be really dangerous. Fortunately, there's [Comeonin.dummy_checkpw/0][dummy_checkpw] function available. It simulates a check for a non-existing user in order to prevent timing attacks.

The improved code could look like this:

{% highlight elixir %}
import Comeonin.Bcrypt, only: [checkpw: 2, dummy_checkpw: 0]

def login(username, password) do
  user = get_user(username)

  cond do
    user && checkpw(password, user.hashed_password) ->
      {:ok, user}
    true ->
      dummy_checkpw()
      {:error, :not_found}
  end
end
{% endhighlight %}


# Conclusion

These two things made me realize that it's really important to get back to the basics from time to time - you never now what new things you can learn. Also, if you want to know more about the basics of security, it's worth to check out the [OWASP Top 10 Project][top-10]


[session-fixation]: https://www.owasp.org/index.php/Session_fixation
[configure_session]: https://hexdocs.pm/plug/Plug.Conn.html#configure_session/2
[dummy_checkpw]: https://hexdocs.pm/comeonin/Comeonin.Bcrypt.html#dummy_checkpw/0
[top-10]: https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project
