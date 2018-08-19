---
layout: post
title:  "Latency with Multiplayer"
date:   2018-06-15 12:00:00 -0500
categories: td technology
---

Now that the game has online connectivity using socket.io, multiple players can submit actions to the server. The server can now pass these actions to the other players. The question is, will the game be perfectly synced?

Nope. In fact, players will never really be playing the same game, but we can get close. There are some major hurdles to get there. This article is the first in a series about online syncing and will focus on the problem of latency in multiplayer games.

## The Problem

Problems with syncing multiplayer games seem to boil down to one main idea:

**The players in a real-time online multiplayer game are not playing the same game.**

There are two main pieces to this. Firstly, they are playing the game at different speeds. Secondly, there are almost always differences of state between their games.

The first point (speeds) seems strange, and I'll cover it in a later post. The second point (state) seems equally strange and it is primarily due to *latency*. Let's explore that now.

## Latency

*Latency* (or *ping*) describes how long it takes your computer to communicate with the server and receive a response. In a turn-based game this is less of a problem because each player takes turns making actions or moves. You have to wait for your opponent(s) to take their turn anyway. In real-time games, any delay on one of your actions can result in a too-late response to your opponent. Imagine playing a boxing game, but the 'block' action was always delayed by 500ms. You'd get knocked out pretty quickly!

This problem stems from the players (and the server) not being in the same room. In the old days (eg. [Warcraft 2 before Battle.net][battle-net-history]), online gaming was often peer-to-peer. That is, each player connects directly to each other either over a local area network (LAN), or using their modems to connect to each other directly using [TCP/IP][tcp-vs-udp]. Playing over a LAN is usually not feasible, and there are major problems with the peer-to-peer approach (eg. cheating).

Modern online games tend to involve a server. This was discussed to some extent in [a previous post][client-server]. In short, the server is used as a single source of truth for the game, and each client (ie. each player's computer) respects any updates it gets from the server. This prevents many types of cheating. The clients all hope to replicate the state of the server as closely as possible. If they succeed in that goal, then all clients are sharing the same state - and therefore playing the same game.

As soon as we add in latency, the server receives player actions in a delayed fashion. This introduces a number of problems. Firstly, should (A) the server perform the action (eg. place a tower in a Tower Defense game) at the time it receives the action, or (B) find a way to do it when it was actually performed - in the past?

### Option B - perform it in the past

Option B sounds more complicated, and maybe even impossible. In some games, however, it is absolutely necessary. In [Rocket League][rocket-league], which is essentially a game of soccer (where the players are, hilariously, rocket-powered cars), players hit a ball around an arena in hopes of putting it in their opponents' net. Let's break down a specific example with latency.

A player hits the ball at time 0ms. The player has a latency of 40ms (there and back), so the server receives the action in about half of that - 20ms. The server then computes where the ball (and the player) should be now if the ball was actually hit 20ms ago (this calculation is non-trivial - more on this later). It lets the player know that, at time 20ms, it should be in a certain position. The client then receives this information 20ms later - at time 40ms. The player's machine computes what should have happened in the last 20ms because the server's state is now 20ms old.

In the best case this works beautifully. If the server had no problem with the player's action, then it doesn't require that the client change the state of their game. This allows for a smooth gameplay experience. More likely, however, there will be very small variances in state.

In such a case, would be a poor user experience to have the player jump to a slightly different position. Instead, the client has to calculate - much like the server did previously - whether the player's actions are *consistent* with the updated (now 20ms old) state of the game. If they are, the client need not make any changes to the state of the player's game, and can instead continue to update the server with more player actions (eg. accelerating). The player then notices no change in the position of their car.

Problems naturally occur when two players interact. In Rocket League, if two players are racing to the ball and one at least player has high latency, the ball will sometimes jump to a new position (called *rubberbanding*). This is because the server has decided that your client's interpretation of the game state is inconsistent with what happened on the server. For example, if you think you hit the ball first, your ball would have moved forward. If the server decides your opponent actually hit the ball (and you did not), your game will experience a major jump in the position of the ball. You will then see the ball *rubberband*, ie. snap to a different direction.

>In less extreme cases, games can slow down or speed up objects in order to have them sync more gracefully with the server. This is naturally more difficult to implement, but the result can be a far smoother user experience.

#### Hybrid Solutions

Given that I have only ever seen rubberbanding with the ball and have never experienced a jump in the player's car position, Rocket League seems to trust the clients' position of each player and only choose to update the ball. The game would likely feel unplayable otherwise. The developers therefore implement a kind of hybrid solution. I mention this only because these strategies do not appear to be all-or-nothing; there is room for each game to implement them based on their specific needs.


### Option A - perform it when received

We can imagine Rocket League if the server assumed an action only occurred when it received the action. Using the previous example, if you hit the ball at time 0ms, the server would assume the ball was actually hit when it received the action - at 20ms. You would therefore be constantly out of sync. The game would likely present this in the form of stutter. I'm sure some work could be done to make it smoother by speeding up or slowing down objects as mentioned above, but it would feel inconsistent if the physics were not acting as expected.

In other genres of real-time games such as Tower Defense, having some units jump forward or backward isn't such a big deal. If I place a tower and that tower officially gets placed 50ms later, it may not even be noticeable. Even considering 500ms, the game would likely still be very playable (if a bit annoying). Playing a game where the towers start shooting half a second after they are placed feels pretty reasonable, if not ideal.


## Which strategy to choose?

The idea of having the server ensure that all actions take effect immediately is alluring, and Option B seems to offer that. The problem is that to calculate the current state given a change in the past seems difficult. Remember, if a player takes an action at 0ms and the server receives it at 20ms, the server will have to rectify the fact that it has already moved on by 20ms. It will then have to recalculate its state before updating the clients.

Moreover, there will be many such actions from potentially many players. This means that, in the process of the server recalculating its state, it may have to restart the process when it receives another action. This will be, at the very least, very difficult to implement.

Performing the action when it's received (Option A) seems like a reasonable, if imperfect, option. Technically there will be more differences in state since every action taken by a player is officially enacted (say) 20ms from that time. However, each update will be very small. In the context of a Tower Defense game, I believe these updates will be unnoticeable.

Developing Option A is fairly trivial in that there is no additional work to do. The server receives an action, performs it, and updates the client as normal. A future blog post will cover how and when that updating occurs, as well as what kind of updates to make.

### Conclusion

I will be choosing **Option A** - namely, having the server perform actions as they are received. If a better user experience is required, Option B can be developed later.




---


```
An imperfect past
Interspersed introspection
Brings closure and peace
```








[battle-net-history]: https://www.pcgamer.com/the-story-of-battlenet/
[tcp-vs-udp]: https://gafferongames.com/post/udp_vs_tcp/
[client-server]: /tower-defense/td/technology/2018/01/26/server-side-language.html#single-source-of-truth
[rocket-league]: https://www.rocketleague.com/
