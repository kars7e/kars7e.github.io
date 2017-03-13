---
date: 2012-07-04T00:12:00Z
tags:
- antirants
- php
- programming
title: Bright sides of PHP - it evolved!
tumblr_url: http://blog.bigfun.eu/post/46111102206/bright-sides-of-php-it-evolved
url: /2012/07/04/bright-sides-of-php-it-evolved/
---

Today [Fabien Potencier](https://twitter.com/fabpot) published a [great post](http://fabien.potencier.org/php-is-much-better-than-you-think.html) about how much all the PHP rants (which are quite popular recently in the PHP blogosphere) are inadequate to the current situation of the language. It should be read by all developers, especially those who do not code in PHP day to day. Why? Because they have no idea how much this language evolved in the last years.

<!--more-->
I have to admit that I also made plenty of unkind comments about PHP itself and its ecosystem. However that was far before PHP 5.4 arrived. Its new functions like traits, short array syntax, and - what’s bothered me the most before - now you can dereference array index directly from the function that returns array (e.g. fun()[0] ). You can also access class members directly after instantiation (e.g. (new Foo())->bar() ). What’s more, in PHP5.4 you get built-in server, which is a killer feature for developers - seriously, installing big apache bundle for development… come on. And no, the fact that you develop plenty of sites is not an excuse for forcing standalone web server in development environments.
Changes in the language itself are not all good things - another big change, important as well, is how the PHP ecosystem works. Based on the power of git and Github, it embraces all the ease of development using these tools - Composer is great example.

## Still long way to go? (but not that long anymore)
Well, things went well for PHP recently, but there are still two main problems slowing down PHP progress - which of course would change what developers think of PHP, because the success on a software market is indisputable (look at statistics about PHP-based CMS usage provided by Fabien in his article).
First is the massive amount of legacy code hanging on servers all around the world. And when i’m talking legacy, i don’t have in mind bad designed code which can’t be developed because some poor coders had no idea about design patterns that time. I''n talking about old, pre-5.3 PHP code, without namespaces, sometimes even without classes, You can’t touch it. You can’t do anything. 
Second obstacle is hosting companies and their support for PHP. The main reason for PHP-based sites burst is the number of hostings where you can deploy your site (written in 2 days in PHP) in matter of minutes/hours. But these companies often still support PHP <= 5.3, mostly without SSH access (FTP only), with ridiculous strange pain-in-the-ass configuration which forces developer to be a magician. And you just can''t use all the great features which comes with new PHP and its environment.
