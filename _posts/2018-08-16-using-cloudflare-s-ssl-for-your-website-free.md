---
layout: post
title: Using Cloudflare's SSL for your website (FREE!)
tags:
- SSL
- Security
---

Many people who have static websites/blogs would say, they don't really need SSL for their website and I've said that too in past. SSL certificates are not cheap comparatively and we try to save that additional cost.

Internet Providers have been playing around with the web traffic and tampering it for a while now. Sniffing the traffic and content injection in web pages is common. These are some simple examples for why SSL is important.

Coming to the point, <a target="_blank" href="https://www.cloudflare.com/">Cloudflare</a> provides free SSL for your website, and it's super-easy to configure:
- Sign up on Cloudflare and enter your DNS. After that, they will automatically fetch all the DNS Records associated with your domain. You can cross-check from your domain provider to be sure.
- In the next step, they will provide few (usually 2) new Nameservers (something like blah.ns.cloudflare.com)
- You just need to replace the existing Nameservers with these new ones in your Domain provider's dashboard.
- Wait for DNS resolution to complete. Meanwhile, go to Cloudflare dashboard's Crypto menu, and Turn ON <b>Always use HTTPS</b> option.
- Once you see <b>Universal SSL Status: Active Certificate"</b> in the dashboard, your website will have working SSL. (happend for me within 5 minutes; but might take upto 24 hours)


<br>
<b>How does it work?</b>

Cloudflare has three ways in which you can use SSL.
- <b>Flexible SSL</b>: Traffic between User & Cloudflare is over HTTPS, but only HTTP between Cloudflare and your server.
- <b>Full SSL</b>: If your server is configured with SSL, then the traffic is end to end over HTTPS. In my case, my website is hosted on Github Pages, which is configured with SSL, so I chose this option.
- <b>Full SSL (Strict)</b>: Same as Full SSL, but additionally, the connection between Cloudflare & web server is authenticated for every request as well. This needs a signed certificate from a trusted authority.

You can <a target="_blank" href="https://support.cloudflare.com/hc/en-us/articles/200170416">read more</a> about it in Cloudflare's official documentation.

Cloudflare also provides a lot of other useful features like Caching, Firewall to name a few.

PS: I got to know about Cloudflare's free SSL plan today. Thank you Cloudflare.

<b>References:</b>
- https://support.cloudflare.com/hc/en-us/articles/200170416
- https://www.cloudflare.com/
