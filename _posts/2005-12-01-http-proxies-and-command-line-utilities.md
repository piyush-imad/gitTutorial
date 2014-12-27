---
author: slowe
comments: false
date: 2005-12-01 20:14:12+00:00
layout: post
slug: http-proxies-and-command-line-utilities
title: HTTP Proxies and Command-Line Utilities
wordpress_id: 131
categories:
- Linux
tags:
- Linux
- Networking
---

A few weeks ago, I locked down outbound HTTP traffic from my network. All outbound HTTP traffic must now pass through a caching HTTP proxy. (The proxy also performs some content filtering so that the kids don't access stuff they shouldn't.)

In my recent experiments with [Kubuntu](http://kubuntu.org/), I needed to download some small files using `wget`, but couldn't because of the proxy. As it turns out, there is a really simple fix.

Simply set an environment variable called `http_proxy` to the correct URL of the outbound proxy server, e.g., "http://proxy.company.com:8080" or whatever. In the bash shell, this can be easily accomplished with this command:

    export http_proxy=â€œhttp://proxy.domain.com:8080â€

Once this environment variable was set properly (I messed up a couple of times trying to set it, getting my bash syntax confused with my tcsh syntax), `wget` and `apt-get` worked like a charm. I haven't yet tried this with yum on Fedora Core 3, but I anticipate that it will work there as well.