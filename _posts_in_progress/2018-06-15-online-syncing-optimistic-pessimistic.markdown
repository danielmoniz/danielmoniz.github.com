---
layout: post
title:  "Multiplayer Syncing - Timers"
date:   2018-06-07 7:15:00 -0500
categories: td technology
---
### Optimistic vs Pessimistic

If two players try to place a tower in the same place, it becomes a race to the server. The first action received (let's say from Player A) is the first action placed. The second tower (from Player B) therefore fails to place. Player A can continue to play as normal because they have no knowledge of Player B's issues. On Player B's side, the game needs to handle what to do about the failed placement of the tower. It should either be removed as soon as the issue is detected, or it should never be placed on B's screen. This is the difference between an optimistic or pessimistic update strategy.

The pessimistic strategy requires that players be more patient with their actions. Once an action has taken effect, they can be confident that it was successful. The optimistic strategy asks the players to forget about latency entirely until a conflict arises. This results in a smoother experience, followed by jumpy updates in state.

Having the game wait to place a tower until the server deems it acceptable seems like a poor user experience. I would much prefer to place a tower and only remove it if there are problems, ie. someone else has placed there already. The optimistic strategy seems therefore seems more viable - and, I think, is largely the future of the web.

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
