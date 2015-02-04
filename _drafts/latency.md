---
layout: post
title:  "Network latency"
date:   2015-02-04 21:00:00
categories: network latency web-performance
---

As web developers we all obsess over asset size and what we can do to reduce them. Switching minifiers, inlining ATF assets, using CDNs is some of tricks we use to reduce download speeds, but resource loading times is just part of the battle. The other half is network latency and the overhead of actually establishing a connection with your servers. In this blog post we will dive into the inner workings of all the black-magic between a user clicks a link and a file actually starts downloading.

# Cold start

To measure the overhead, let's see what the timings took for the Google logo image when performing a cold request.

![Checking Service Workers](/assets/posts/network-latency/cold_start.png)

So let's break down the numbers. DNS takes up 14ms (9%), TCP and SSL negotiation 79ms (50%) and server-time + a netwrok round trip another 56ms (35%). And as for the downloading of the actual content? 3ms or 2% of the total request time. That's right, all that image compression techniques amount to just 2% of the time it takes to actually download an image, from request to all bytes send and received.

The rest 98% is just network overhead, so it's understandable that modern browsers use various optimisation techniques to shorten network latency, and we'll look at some of them in this blog post.

I'll be using Chrome's Developer Tools for network analysis, including the [chrome://net-internals](chrome://net-internals) diagnosing page.

# DNS Resolution

As most of you are familiar, DNS is the process of converting a dns name (in our case www.googe.ro) to an ip address.

So let's take a look at what events Chrome emits when performing a DNS resolution.

![Checking Service Workers](/assets/posts/network-latency/dns_event.png)

The Chrome event is called HOST_RESOLVER_IMPL_JOB **(1)** and can be found by searching under the [chrome://net-internals/#events](chrome://net-internals/#events) tab. We performed a request to resolve `www.google.ro` and got 4 ips back which match the given DNS name **(2)**. So far so good.

The intersting bit comes when we observe that the time delta for the DNS resolution process is 14ms **(3)** and that the process involved spawning a socket and connecting through UDP to the nameserver. The delta is the time it takes to get a response from the nameserver through the socket we just opened.

Being such a popular domain, it was most likely cached by my local nameserver (my router) and the resolution time took only 14ms (just a single round-trip from my local browser to the name-server). This would be multiplied by each hop the resolution process makes from name-server to name-server.

It's only normal that Chrome would take some optimisation steps to reduce DNS resolution times, and one of them is DNS caching. You can check the current state of the cache by going to [chrome://net-internals/#dns](chrome://net-internals/#dns). In my case, after the initial request, I saw something like this:

![Checking Service Workers](/assets/posts/network-latency/dns_cache.png)

Chrome can keep up to 1000 resolved DNS names into it's internal cache, which gets adjusted over time using the internal Predictor. The browser performs pre-resolution on DNS hostnames when you type a query in the omnibox or when you hover over a link, and stores the result into it's internal cache. Considering these heuristics, the DNS name is mostly already resolved by the time you actually clicked a link.

But thankfully, there is a way to give hints to a browser that a particular DNS name will be needed in the near future, which will trigger a pre-resolve and reduce network latency. The hint is given though a `link` tag and looks like this:

{% highlight html %}
<link rel="dns-prefetch" href="//dns-name">
{% endhighlight %}

It's important to note that this is just a *hint* and does not guaranty that the browser will actually resolve the given DNS.

You can also measure the dns resolution time using navigation timing, if you need some actual Real User Metrics:

{% highlight javascript %}
performance.timing.domainLookupEnd - performance.timing.domainLookupStart;
// = 14
{% endhighlight %}

The domain cache also ties in with Domain Sharding, which is the process of using multiple domains to load resources to bypass the 6 concurrent connections per domain Chrome imposes. Although it does help with this limit, you will also incur a DNS resolution penalty for each unique domain the browser has to resolve. But this is a topic for a future blog post, so we won't go into details.

# Socket and TCP connection reuse

The bigest chunk of the network latency we saw when requesting our image was from the TCP and SSL negotiation. So let's step back and see what this process actually involves.



- SSL additional round trip
- TCP pre-connect and socket reuse
- SPDY
- handshake for each resource - cannot be cached like DNS

# Request Round trip + server time

# Further Reading

https://www.igvita.com/posa/high-performance-networking-in-google-chrome/
https://www.igvita.com/2012/06/04/chrome-networking-dns-prefetch-and-tcp-preconnect/
http://www.inetdaemon.com/tutorials/internet/tcp/3-way_handshake.shtml
