---
layout: post
title: Accessing PostgreSQL server through a SSH Tunnel
subtitle: (Port Forwarding)
tags:
- PostgreSQL
- SSH Tunnel
- Hacker
---

<b>Step 1:</b> Check the SSH connectivity with the server, verify username and password.

<b>Step 2:</b> Create the tunnel in your local system by executing the following command (It will prompt for password):
{% highlight bash %}
ssh -fNg -L 5555:localhost:5432 <user>@<server>
{% endhighlight %}

<b>Step 3:</b> Now, open your PostgreSQL client (eg, `pgAdmin 3` or `DBeaver` or `Postico` for OS X or `Terminal`) and fill in the connection details as usual. Check the image below.

<p><img class="img-responsive" src="{{ site.url }}/assets/images/postico-port-forwarding.png" alt="Postico DB connection" /></p>

<b>Note:</b> Yes, you'll have to use `'localhost'`.
