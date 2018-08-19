---
layout: post
title:  "Foundation"
date:   2018-01-07 11:24:00 -0500
categories: td technology
---

### Our requirements

My friend and I wanted to make an online Tower Defense game. I wanted to make that happen, immediately. This required a couple things:

1. Playable in browser
1. No crazy frameworks

The first one is a combination of obviousness and my own personal biases. Modern browsers can do so much for us that we take for granted. They scale images beautifully, allow for sharp typeface, and allow anyone who visits to experience your project without strings attached. I'm also primarily a web developer - living in JavaScript land on my most recent projects - and so browser-based technology it is. This means:

* HTML
* CSS
* JavaScript
* ...and WebGL (more on this in a later post)

JavaScript it is, then. Moreover, I specifically didn't want my friend to have to learn a hyper-specific framework. Yes, React, that means you're out. We needed the closest thing to vanilla JS without actually building a game framework in vanilla JS.

The goal, then, is to create a game as follows:

* Create a simple HTML skeleton for a game board with displays for lives, credits, and so on.
* Use CSS to make those look a little fancy.
* Use JavaScript to dynamically create and update divs and images. These are our units and towers.

Simple enough! This brings us to my one JavaScript library of choice to make this vastly easier.


### MobX

[MobX][mobx github] is beautiful. Most people see it in the context of [React][react] as an alternative to [Redux][redux github]. However, it's pretty useful on its own.

It isn't really a framework. It's a solution to the problem of mutability in React. In a way, it's the _opposite_ solution to Redux, which creates a framework designed to force the programmer to write functional code with immutable objects. MobX, on the other hand, makes object oriented programming cool again.

I'll try to talk more about MobX in a future post. What we need to know now is what it allows us to do.

* Set up certain pieces of data to be *observable*.
* Set up functions to be *observers* of those data.

And that's pretty much it (though there are more cool features of MobX). The functions that observe data will automatically be run when the data they rely on changes. We do this using the `autorun` function provided by MobX. For example, say we have a class called `Game`.

```javascript
import { observable, autorun, action } from 'mobx'

class Game {
  @observable lives = 5
}
```

We can set up a function to run automatically when `lives` changes because we have made `lives` *observable*.

```javascript
autorun(() => {
  const livesDisplay = document.querySelector('.lives-display')
  livesDisplay.innerHTML = game.lives
})
```

Just like that, we have a lives counter in our game which automatically keeps itself in sync - and it even does it synchronously. The above code inside `autorun` will run once off the bat, and then automatically trigger whenever `game.lives` changes. If we also referenced another observable, it would run when it's changed, too. Magic? Magic. But the good kind.

MobX allows us to put a constraint on ourselves so that we don't go changing `Game.lives` willy-nilly. If you use MobX's `useStrict` feature, you are not allowed to change an observable outside of an *action*. In order to change lives, then, we simply put a writer method inside of `Game`.

```javascript
  @action setLives(newLives) {
    this.lives = newLives
  }
```

`useStrict` prevents programmers from doing things like `game.lives = 5` outside of any reasonable context.

This allows the convention of keeping central/local all modifications to the citizens of a class. By my convention, if you _ever_ want to change lives, you have to call my `Game.setLives()` function, which of course lives inside of `Game`.


### A simple framework

In the span of about two hours, I was able to use MobX with otherwise-vanilla JavaScript/HTML/CSS to throw together the `Game` and `Unit` classes. The best part about this is that we are really just programming in JavaScript. This means that a newcomer, hesitant as they would rightly be about things like `@observable` and `@action`, can otherwise dive in and write regular old JS.

Although this was a proof of concept that I was prepared to throw away, it worked smashingly. We could then hammer away at giving units a target to reach, clicking a button to select a new tower to place, having that tower follow your cursor until you click, and so on. All of this is surprisingly easy with MobX.

A really cool corollary of the use of MobX is that these functions are only called when their relevant data actually changes. That makes this whole enterprise pretty efficient! We aren't re-rendering every item on every tick of the game.



```
A strong foundation
Thick roots into fertile soil
The future is up
```

---

When I introduce or change a technology, I'll try to provide an updated list of the tech being used and highlight the changes.

#### Tech stack:

* **Front-end**
  * **HTML/CSS**
  * **JavaScript with MobX**

[mobx github]: https://github.com/mobxjs/mobx
[redux github]: https://github.com/reactjs/redux
[react]: https://reactjs.org/
