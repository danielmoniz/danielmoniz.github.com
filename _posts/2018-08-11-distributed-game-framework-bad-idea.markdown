---
layout: post
title:  "Distributed Game Framework - A Bad Idea"
date:   2018-08-11 10:00:00 -0500
categories: td technology mobx
---


This really seemed like a good idea at the time. Hint: it wasn't.

## The Idea

Early on I had the problem of how to render content in the game, and more importantly, how to update that content regularly - say, every 20ms. The rendering was easy enough - simply make `div`s that contain images and move them around the page with JavaScript and CSS. In order to repeatedly update the state of the game, you need a timer.

My first thought was that it would be inefficient (and distinctly uncool) to loop over each entity every 20ms and tell a renderer to have them update. Instead, I decided to take a distributed approach.

Each entity would, upon creation:

* initialize its own renderer
* initialize its own timer for updating state (eg. moving)

### Initialize a renderer

Upon calling `new Enemy()`, or perhaps `Enemy.create()`, a renderer would also be created. This aspect worked nicely (at first) because the entity can simply initialize the specific renderer it needs.

A 'renderer' here just means a JavaScript class that updates the displayed entity when its state changes. MobX's `autorun` is used to ensure that happens.

```JavaScript
import { autorun } from 'mobx'

autorun(() => {
  if (tower.target) {
    const angle = tower.getAngleToPoint(tower.target.xFloor, tower.target.yFloor)
    towerElement.style.transform = "rotate(" + angle + "deg)"
  }
})
```

where `tower` is an instance of a JS class, representing an entity in the game, and `towerElement` is its corresponding DOM element. The conversion happening in `tower.getAngleToPoint()` is from radians to degrees. CSS is then used to rotate the div as necessary.

Note that every time the value for `tower.target` changes (as long as the `.target` attribute is *observable*), that `autorun` function will trigger and update our entity via CSS. For example, using decorator syntax:

```JavaScript
import { observable } from 'mobx'

export default class Tower extends Unit {
  @observable target // changes to target trigger autorun!
  // ...
}
```

### Initialize a timer

Next, a timer is needed to actually make changes to the tower. MobX is already handling the rendering nicely when those changes occur.

The timer is needed to change the state of the tower itself. In other words, what actions will it take in the game? It needs to pick targets, to shoot, and so on. We therefore create a method called `Tower.act()`.

```JavaScript
class Tower() {
  // ...
  act() {
    // pick target if needed
    // shoot if possible
    // etc.
  }
}
```

This method now needs to be triggered every 20ms so that the tower's state can change, and in turn, the tower's image will rotate automatically when it picks a target.

```JavaScript
class Tower() {
  // ...
  setUpTimer() {
    setInterval(() => {
      this.act()
    }, 20)
  }
}
```

Now our `Tower.act()` method will be called every (approximately) 20ms. Done!


## The Problems

The main problems are:

* Performance
* Inability to swap out renderers

Say I want to use a different renderer. For example, at around this time I began considering using `PixiJS` for rendering in hopes that I could improve performance. This is more difficult if the rendering is distributed throughout the app. Whoops. I'll be covering that problem (and my solution) in a separate post.

### Performance

The biggest problem was trying to render many entities on the screen at the same time. I had hoped to render hundreds, if not thousands, of enemies for your towers to shoot. However, after hitting only a few hundred of them, the game slowed to a crawl.

Digging into it, the source of the problem should probably have been obvious: timers. Every single one of those entities had its own timer, all fighting to fire every 20ms. The solution to this is equally obvious: have a single timer for the game which iterates over every entity and triggers updates in their state. MobX can still handle the rendering using `autorun`.

I therefore refactored the entities to be managed by a single game loop. The game itself also needed to create a single `GameRenderer`, and to have this manage the entities separately from the rest of their state. This was a big job, but the payoff was immediate: I could render about 2x the number of units on screen.


## Conclusion

Every game needs a game loop. I tried to get away from it by having each entity provide its own loop, but this adds far more overhead to your game and reduces performance. Why have hundreds of timers when you can have one? Trying to avoid a single loop by adding timers didn't sense in hindsight.

I will describe my solutions to these problems in greater detail in a later post. In particular, it's worth exploring how to have a single access point to the renderer so that it can be easily swapped out for a better rendering library.



---

```
How intractable
Is a thunderstorm, writhing
Shelter splits the rain
```

---

#### Tech stack:

* Front-end
  * HTML/CSS
  * JavaScript with MobX and Socket.IO
* Back-end
  * JavaScript with Express.js
