---
date: 2010-05-10T13:49:00Z
tags: []
title: How to make Linksys WRT54GL able to redirect port
tumblr_url: http://blog.bigfun.eu/post/586626994/how-to-make-linksys-wrt54gl-able-to-redirect-port
url: /2010/05/10/how-to-make-linksys-wrt54gl-able-to-redirect-port/
---

Today i wanted to make few of my computers visible from outside. I decided to set up some simple from WAN to LAN port redirections. After 0,5h of trying, i realized that’s just not possible. Yes, it’s possible to make port forwarding (e.g. from WAN:80 to SomeLanHost:80 ), but i could not make port redirection (e.g. from WAN:666 to LAN:80).
Fortunately, there is great software called dd-wrt.com, which makes your $60 worth router to a $600 worth one. It adds a lot of great features, also the one i needed most - port redirection. Simply download the package (version for WEB interface was good for me) and update the router firmware - it goes through easy web interface, few clicks - and it’s done! Just remember to stay away from power connector during the proccess.
After that, i got new handy web interface. Then i could simply redirect from different WAN ports to the same 80 port but on different hosts.
There are more supported routers, full list is available here.
