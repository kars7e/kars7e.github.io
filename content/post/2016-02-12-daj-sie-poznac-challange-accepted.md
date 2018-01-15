---
author: karol_stepniewski
comments: true
date: 2016-02-12T09:05:45Z
tags:
- dajsiepoznac2016
- elixir
- phoenix
- programming
title: Daj Się poznać - challange accepted
tumblr_url: http://blog.bigfun.eu/post/139161106580/daj-sie-poznac-challange-accepted
url: /2016/02/12/daj-sie-poznac-challange-accepted/
---

To motivate myself a bit, I decided to take part in [Maciej](http://www.maciejaniserowicz.com/daj-sie-poznac/)’s contest and, for a period of 10 weeks, work on an open source project.

<!--more-->
I’ve recently started learning [Elixir language](elixir-lang.org), as well as [Phoenix framework](http://phoenixframework.org/), pretty new technologies based on the not so new Erlang VM. Elixir is a functional language created by Jose Valim, and Phoenix is a web framework created on top of it. Both of them have really neat features, and I encourage everyone to read more about them. The most compelling feature for me personally was the fact Elixir is based on erlang VM - a battle tested technology, used for distributed systems for >20 years. Many concepts, like actor model, which have been “discovered” in other languages recently, had been part of Erlang for many years. (It’s interesting how certain trends come and go, even in the area of technology, which you’d think should be driven by scientific method and well established best practices - that’s a topic for different post though).

I decided to write a game inspired by a card game called Exploding Kittens. It’s a game I’ve recently fallen in love with, so it’s kind of natural choice. However to achieve my goal I could probably choose any other multiplayer game (which does not require fancy 3D graphics, obviously). That’s because my goal is to write a distributed system that will:

- allow anonymous players to join the game easily, just by opening game’s website.
- handle many (and i really mean MANY, like ~1MLN) connections at the same time.

To achieve above, I want to:

- design this system in maximally stateless mode - game state will be held in memory, and each game instance will be handled by a separate ‘actor’ in the system
- System will be able to connect new nodes (physical/virtual servers) dynamically. This means that server can be scaled horizontally, based on the current load (number of players).

I’m aware those goals are quite ambitious given the time frame (10 weeks), however the beauty of this contest is that I don’t really have to finish the full project in time - most important is systematic work on the project, and that’s something I always had trouble with.
Wish me good luck! :-)

