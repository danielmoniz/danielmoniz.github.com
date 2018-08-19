

Ideas:

* latency creates issues in real-time games
  * actions from the past need to be handled and the game state needs be rectified
  * Strategies: *- this can be a separate article!*
    * Optimistic: each client (read each players' computer) assumes they are doing the right thing until the server tells them otherwise. Creates fluid gameplay and jumpy conflicts of state.
    * Pessimistic: each client waits for instructions from the server as to what to do.
      * Not realistic in a real-time game
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
