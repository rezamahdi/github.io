---
title: DnSpider
date: 2022-08-25
keywords: projects
author: Reza Mahdi
---

<pre>
 _     __             
| \._ (_ ._ o _| _ ._ 
|_/| |__)|_)|(_|(/_|  
         |            
</pre>

DnSpider is a high-performance and async DNS subdomain scanner.
The major benefit of DnSpider comapred to other scanners is that it uses heavy
optimization in design. For example main thread of process runs a *generator*, that
generates domain names and pass it to a lock-free queue. At other side of queue,
workers peek them (by reference not by value), make DNS queries asyncronousely and
mark them.

Submit patches, bug reports and discussion about DnSpider to
dnspider@freelists.org mailing list. This list is hosted at
[Freelists](http://freelists.org).

To subscribe to this mailing list, send a message to
dnspider-request@freelists.org with 'subscribe' as subject line.

In summorized, Use following info to interact with us:
 * List address: dnspider@freelists.org
 * Archive: https://www.freelists.org/archive/dnspider
 * Subscription: dnspider-request@freelists.org
 
In the case of bug report, prefix your email with '[BUG]' to be distinguishable
from patches and normal messages.

freelists.org lists prefix all messages with '[dnspider]' so filtering emails is
trivial.
