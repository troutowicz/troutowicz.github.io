---
layout: post
title: My Apache web server DO NOTs
---

Here is my growing list of DO NOTs when administrating an Apache web server. Most of this advice is nothing new and has been blogged about many times. This is just my personal list from past/present experiences.

**DO NOT naively copy/paste**

Configuration ages and can become irrelevant and harmful. Browser problems from last decade are no longer relevant, and it is in your best interest to get to know your configuration. A perfect example is the MSIE SSL KeepAlive settings. If you still have CentOS 5 servers running with Apache web server, the default SSL virtual host recommends a workaround for an IE <= 5 problem. This old workaround will in fact apply to every version of IE, causing noticeable loss in performance. It should be removed.

```sh
SetEnvIf User-Agent ".*MSIE.*" \
nokeepalive ssl-unclean-shutdown \
downgrade-1.0 force-response-1.0
```

More reading:
[Link 1](http://blogs.msdn.com/b/ieinternals/archive/2011/03/26/https-and-connection-close-is-your-apache-modssl-server-configuration-set-to-slow.aspx)
[Link 2](http://newestindustry.org/2007/06/06/dear-apache-software-foundation-fix-the-msie-ssl-keepalive-settings/)

**DO NOT adjust prefork MPM willy-nilly**

Do not over optimize early on. Start with the defaults and adjust as you find bottlenecks. Raising `MaxClients` may seem harmless in the beginning, but if resources are limited and traffic picks up abruptly, or a bug in the app prevents connections from closing, the app will experience a larger negative impact in performance.

More reading:
[Link 1](https://servercheck.in/blog/3-small-tweaks-make-apache-fly)

**DO NOT fear SSL overhead**

SSL does require more computations and does increase network latency. However, neither of these things will cause *noticeable* overhead on the hardware available today. If sections of your app already use SSL, you should enforce SSL everywhere and take advantage of the added security. Depending on your configuration, this may simplify things by removing app specific HTTPS redirects. It should also be said that if you are moving an existing app to HTTPS by default, proper testing should be done in advance to make sure all app functionality works as expected.

More reading:
[Link 1](https://www.maxcdn.com/blog/ssl-performance-myth/)

**DO NOT use multiple instances (unless it makes sense)**

This one is up for debate. Configured correctly, with init scripts setup for each instance, this is acceptable. Configured with init scripts missing for instances is bothersome. No-one wants to manually restart services after a reboot. Unless your virtual hosts have diverse module requirements, avoid this extra work and complexity.

More reading:
[Link 1](https://wiki.apache.org/httpd/RunningMultipleApacheInstances)

**DO NOT quickly dismiss KeepAlive**

`KeepAlive` can provide significant page load speedup for apps of which the majority of requests target the origin server. Memory utilization does go up, but CPU utilization will go down. When building a public facing web app, the end user experience should be made as fast and enjoyable as possible.

More reading:
[Link 1](https://servercheck.in/blog/3-small-tweaks-make-apache-fly)

**DO NOT forget about requests targeting a server's IP**

Make sure to handle requests that contain the servers IP in the `Host` header. Not doing so could cause unexpected behavior and direct the user to an unexpected default virtual host. In some cases this can provide unintentional insight into the apps architectural design and harm your site's security. As an example, consider an Apache web server working as a reverse proxy for a Tomcat instance. If the request is not handled by a particular virtual host, then defaults to the first declared virtual host which passes requests to Tomcat, and that Tomcat instance does not handle IP based requests, you might be presented with the default Tomcat app which provides access to the Tomcat manager.

**DO NOT forget to DRY**

**D**on't **R**epeat **Y**ourself whenever possible. When managing a lot of virtual hosts, configuration can become unmanageable fast. Modularizing blocks of configuration and using the `Include` directive is one way to simplify configuration and avoid repeats. This can be especially elegant when managing many sub-domain virtual hosts that use the same wildcard SSL certificate. Placing the SSL conf into a separate file and `Include`ing it in the first Name-based virtual host will configure SSL for all virtual hosts belonging to that Name-based IP.

More reading:
[Link 1](http://httpd.apache.org/docs/2.4/mod/core.html#include) [Link 2](https://wiki.apache.org/httpd/NameBasedSSLVHosts)
