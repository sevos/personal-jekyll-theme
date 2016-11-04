---
layout: post
section-type: post
title: Failing PostgreSQL on OS X
---

Today there will be only quick tip. Have you ever come back to work on monday
and seen following error after lanunching your Rails development:

{% highlight bash %}
could not connect to server: Connection refused (PG::Error)
        Is the server running on host "localhost" (::1) and accepting
        TCP/IP connections on port 5432?
could not connect to server: Connection refused
        Is the server running on host "localhost" (127.0.0.1) and accepting
        TCP/IP connections on port 5432?
could not connect to server: Connection refused
        Is the server running on host "localhost" (fe80::1) and accepting
        TCP/IP connections on port 5432?
{% endhighlight %}


If you installed PostgreSQL via homebrew, you might want try removing postmaster.pid,
what helped me:

{% highlight bash %}
rm /usr/local/var/postgres/postmaster.pid
{% endhighlight %}
