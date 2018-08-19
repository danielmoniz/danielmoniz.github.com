---
layout: post
title:  "Multiplayer Syncing - Timers"
date:   2018-06-07 7:15:00 -0500
categories: td technology
---

Now that the game has online connectivity using socket.io, multiple players can now submit actions to the server. The server can now pass these actions to the other players. The question is, will the game be perfectly synced?

Nope. In fact, players will never really be playing the same game, but we can get close. There are some major hurdles to get there. This article is the first in a series about online syncing and will focus on the problem of latency in multiplayer games.

## The Problem

Problems with syncing multiplayer games seem to boil down to one main idea:

**The players in a real-time online multiplayer game are not all playing the same game.**

There are two main pieces to this. Firstly, they are playing the game at different speeds. Secondly, there are almost always differences of state between their games.

The first point (speeds) seems strange. Why is this a problem? In reality, it is very difficult to get electronic hardware to produce an accurate timer. Compounding that issue, it is even more difficult to mass produce such devices. Building software on top of that to utilize the hardware timer in an identical fashion is, as far as I can tell, impossible.

Moreover, how is this possible? We've probably all played online multiplayer games before. When we do, everyone seems to be playing the same game at the same speed. This is really just a credit to the developers for building a game where players don't feel they have syncing issues.

The second point (state) seems equally strange, but it is primarily due to *latency*. Let's explore that now.



### Timers

As mentioned previously, timers are never identical between different computers. JavaScript also makes this trickier because of how timers work.








Ideas:

* JS timers don't guarantee precision *- this can be a separate article!*
  * they guarantee minimum time before being called!
  * How do you update the game to look smooth when actions happened in the past?
  * Can keep track of game 'ticks'
  * Probably need a separate timer for when to update the browser (ie. window.requestAnimationFrame)
* lag creates delays for specific players *- this can be a separate article!*
  * multiple strategies
    * slow them all down to match
    * let them be slow and update them when needed

* actions could be synced, but the state could be different on different machines
* timers - not the same on all devices
* when to sync the game
* update the clients fully, or only the changes?
* dealing with lag

A future article could simply list the issues and link to all of the posts on those topics.

---


```

```

---

#### Updated tech stack:

* Front-end
  * HTML/CSS
  * JavaScript with MobX and Socket.IO
* Back-end
  * JavaScript with Express.js






[battle-net-history]: https://www.pcgamer.com/the-story-of-battlenet/
[tcp-vs-udp]: https://gafferongames.com/post/udp_vs_tcp/
[client-server]: /tower-defense/td/technology/2018/01/26/server-side-language.html#single-source-of-truth
