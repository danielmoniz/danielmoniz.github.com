---
layout: post
title:  "Online Connectivity"
date:   2018-02-22 7:15:00 -0500
categories: td technology
---


In order to create an online multiplayer game, you need communication between the clients (ie. the players' computers) and the server. This naturally has to be asynchronous; we can't afford to have page loads happening in the middle of a game! These leads to only two real options: [Ajax][ajax-mdn] and [WebSockets][websockets-wikipedia].

### Ajax

A browser can request information from or send information to the server, continue doing what it's doing, and handle the response when it arrives. This collective set of technologies is called Ajax, and it allows for communication with the server without reloading the page. JavaScript comes with the XMLHttpRequest (XHR) object for this purpose. Many people use [jQuery's $.ajax()][jquery-ajax] method for an easy-to-use version of this. Modern browsers also support the [Fetch API][fetch-api] which allows for making Ajax calls without pulling in any additional libraries.

Regardless of our technology of choice, how would using Ajax work in a game? Keep in mind that the client/browser has to make a request to the server for information. Each player's browser would therefore have to make requests every interval (however short or long) to see if the game state has changed. If it hasn't changed, do nothing; if it has, update the game using the response.

Thinking about this in the context of a tower defense game, there are clearly times when this is awkward. For example, if any player places a tower, the ideal situation allows for all other players to see that tower on their screen near-instantaneously. This would force us to make new Ajax requests every, say, 30 milliseconds. Below are some reasons why this sounds like a nightmare (feel free to skip to the next section if you believe me).

>Requests to webservers are expensive. This is why servers are expensive, and why scaling up to allow for more traffic on Amazon's web services (AWS) costs money.

>It could get tricky if any one of those requests is a bit delayed. If two requests are made, `A` and `B`, in that order, and response to `B` arrives first, then we have to make sure we don't undo the intended change.

>* For example, if each request contained the entire state of the game, the late response to `A` (sent before the tower was placed) might not include the tower. This could cause the tower to appear, then disappear after `B`, then reappear again when a response to our next request `C` arrives.

> If we intend to avoid requesting the entire game state on every request, we have to ask for something specific. This means that we need X requests every 30ms, where X is the number of things that might need updating by the server.

Okay, so no Ajax requests.

### WebSockets

WebSockets are persistent connections made between clients and servers. Although holding such connections is more demanding on the server than simply listening for requests (versus regular HTTP, used by Ajax), it allows for two-way communication. That is, servers can push to clients and, conversely, clients to servers. Ajax, in comparison, uses regular [HTTP][http], and so the communication is always from client to server.

Using our previous example, if a player places a tower, the browser can send that information to the server. Nothing new yet. But the server can process that information and immediately notify all other players! Their respective browsers then process the information of the newly placed tower and place it in their game views.

Similarly, if the game running on the server decides that a new wave of enemies needs to be spawned, it can simply do so and immediately alert all players with the relevant information.

#### Socket.IO

Given the use of JavaScript on both the front and back-end, the choice of technology was pretty easy: [Socket.IO][socket-io]. It's something of a standard for implementing WebSockets using JavaScript. ([read here][why-not-socket-io] for an argument against Socket.IO).

>They have a tutorial consisting of a [simple chat application][socket-io-tutorial]. I highly recommend you try this out - it easily led to me building my own game server for a previous turn-based game project. A chat is not so very different from a game - both players can act and send information to the other player via the server.

Socket.IO comes with two libraries: client and server. We simply have to run both and ask them nicely to meet in the middle. The client side can be added in your HTML and will look something like this:

```html
<script type="text/javascript" src="/javascripts/socket.io.js"></script>
<script type="text/javascript">
  var socket = io();
</script>
```

My server-side kickoff point looks like this:
```js
var server = http.createServer(app); // create the app as normal using Express.js

var io = require('socket.io')(server); // initialize Socket.IO

var GameServer = require('../build/server/GameServer')
const gameServer = new GameServer(io) // pass the Socket.IO object to my game server
```

The `GameServer` then listens for a number of events, and emits events under certain conditions (the clients do the same). For example:

```js
socket.on('place tower', (tower) => {
  if (socket.gameManager && socket.gameManager.game) {
    socket.gameManager.game.placeTower(tower)
    socket.broadcast.to(socket.roomId).emit('place tower', tower)
  }
})
```

The above code does the following:
* Listen for the `'place tower'` event and accept the tower data that was sent
* If there is a game running, do the following:
  * Tell the server's game to place a tower
  * Broadcast to every client (except the one that emitted the `'place tower'` event) that they should place the tower.

This of course required setting up the `GameManager` on the socket - Socket.IO won't build a game for you!

Let's look at the client side:

```js
socket.on('place tower', (tower) => {
  game.placeTower(tower)
})
```

The client side is much simpler. It simply has to listen for the event `'place tower'`, accept the tower data, and pass it along to the game itself. This is because the receiving clients don't need to tell the others about the new tower - the server has done that already.

Something worth noting at this point is that the `.placeTower()` method on both the client and the server are identical (more or less - this may be a future blog entry). As mentioned in [a previous post][server-side-choices], this is the beauty of using the same language on both the front and back-end.

Lastly, the client that actually placed the tower and triggered the chain of events has a function as follows:

```js
placeTower(tower) {
  this.emit('place tower', tower)
}
```

This function simply gets called when a user manually places a tower. It emits the `'place tower'` event to the server (not the other clients). That's it!


### Cooperative play...for free?

I wish! There is still the task of actually keeping the games synced. This is a much bigger topic. However, if the game is short and simple, we have a co-op multiplayer game - one where all players are on the same team and have the same resources. In essence, they are (ideally) playing exactly the same game.


```
The power of speech
Ineffective, beautiful
Fuels democracy
```

---

#### Tech stack:

* Front-end
  * HTML/CSS
  * JavaScript with MobX **and Socket.IO**
* Back-end
  * JavaScript with Express.js







[websockets-wikipedia]: https://en.wikipedia.org/wiki/WebSocket
[jquery-ajax]: http://api.jquery.com/jquery.ajax/
[fetch-api]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
[ajax-wikipedia]: https://en.wikipedia.org/wiki/Ajax_(programming)
[ajax-mdn]: https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX
[socket-io]: https://socket.io/
[socket-io-tutorial]: https://socket.io/get-started/chat/
[why-not-socket-io]: https://codeburst.io/why-you-don-t-need-socket-io-6848f1c871cd
[http]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol

[server-side-choices]: /tower-defense/td/technology/2018/01/26/server-side-language.html
