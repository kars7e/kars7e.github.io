---
author: karol_stepniewski
comments: true
date: 2016-03-01T09:05:45Z
tags:
- dajsiepoznac2016
- planning
title: '[DSP] Let''s do some planning'
url: /2016/03/01/dsp-lets-do-some-planning/
---

The competition I mentioned in my [previous post]({{< ref "2016-02-12-daj-sie-poznac-challange-accepted.md" >}}) has just [started](http://www.maciejaniserowicz.com/2016/03/01/daj-sie-poznac-2016-startujemy-i-przedluzamy-rejestracje/)!
So again, what is it about? For a period of 10 weeks (out of 13 available), work on a software project and blog about it minimum twice a week. My idea for this challenge is to write a game inspired by a game called [Exploding Kittens](http://www.explodingkittens.com/), using Elixir language and Phoenix framework.

And what's that planning I mentioned in the title of this post? I want to talk a bit about how I'm gonna tackle this problem from the time management perspective. After all, this competition has a certain set of rules which all of the participants must adhere to in order to remain in the competition. We have 10 weeks of work, 2 posts per week, which makes it 20 posts total (including this one). To make sure I'm on track, let's define some progress marks that I expect to have after every week. So the plan for the next 10 weeks is as follows:

### Week 1 - The Game
I will introduce basic game concepts and rules, and how I envision the game will look like at the end of this challenge.

### Week 2 - Hello Elixir
Week 2 will be about elixir - what is this language, what interesting features does it have, and how can we model the game in it.

### Week 3 - MVP
By the end of week 3 I'd like to have a running version of the game in pure Elixir, available to play through iex (Elixir REPL).

### Week 4 - We're going web-wide
I'll do an introduction to Phoenix, Elixir based high performance web framework.

### Week 5 - Work, work, work
I will spend this week on implementing the game in phoenix, and I will slowly transition to frontend.

### Week 6, 7 & 8 - Show me your pretty face
Well, we're talking games here, and it ain't [mud](https://en.wikipedia.org/wiki/MUD)-type, so we need to create some UI.
I honestly have no idea how it will work, what technologies will I use, or where will I take some decent graphics from. Fun ride, as usual.

### Week 9 & 10 - Let's scale it baby!
Elixir is really a [high performance language](http://www.phoenixframework.org/blog/the-road-to-2-million-websocket-connections), but at some point there is always a need to scale out and use more servers than just one. I'd like to spend last two weeks on adding a feature to the game server which will allow new servers to join existing ones and form a cluster where each node can communicate with each other. Although players of the same game will always talk to the same server (although I'm curious if this assumption can be changed), cluster will allow to display some global stats regardless of the server player is connected to (number of total games currently being played across all cluster nodes, etc.), and possibly some other interesting use cases.

As we're in week 1, it means that in the next post I will go into details of the game mechanics. Stay tuned!




