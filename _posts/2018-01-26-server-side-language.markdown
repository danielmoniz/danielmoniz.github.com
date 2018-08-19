---
layout: post
title:  "Server-side Choices"
date:   2018-01-26 5:07:00 -0500
categories: td technology
---

The goal from the start has been to create an online game. In the previous article I discussed possibilities for the overall architectural plan. In short, the architectural options were:

* Monolithic
* Microservice

To summarize, the decision (for now) is to have one server handle requests for the general web app as well as handle the game itself. In other words, we're building one app that includes both a website and a game.


### Technology

This immediately leads to the rather important and controversial question:

> What language/framework do I use on the server?

It's all about picking the *best tool for the job*. This sometimes means "the tool I'm most familiar with" if you need to get something done quickly. In the case of breaking (mostly) new ground, it's good to do a survey of the options.

These are the options I considered and why:

* Ruby on Rails
  * Because I teach it at Bitmaker!
  * Could build the surrounding website very quickly.
* JavaScript/node
  * To show my friends that I'm cool.
  * I've spent a lot of time in JS-land in the past few years.
  * **The game is built in JavaScript on the front-end.**
* Python
  * I have a lot of Python experience on the web.

#### Ruby on Rails

There is a further benefit of choosing Ruby on Rails. I could potentially get friends or former students - anyone looking for experience - to pitch in to build the website itself! I find that pretty tempting because a) I'm more interested in building the game than the website, and b) it could be helpful to someone else looking to build their portfolio.

#### JavaScript

It would be nice to be cool. It's hard not to get caught up in the excitement of JavaScript. Everything moves so quickly. Hype surrounds every new framework. What a time to be alive!

The real reason, however, is that the game itself is being written in JS for the front-end. I'll explore this further below.

A major downside is that JS would not be my first (or second) choice to build a web app. I don't expect the web frameworks to be particularly mature or to provide a high level of productivity.

#### Python

Python is a wonderful language. [Try it for yourself][codecademy-python]! Unfortunately it doesn't come with either of the lovely benefits mentioned above.


### Choosing an Option

Looking at the given pros and cons, one stands out in particular, and not just because I bolded it.

**The game is built in JavaScript on the front-end.**

Why is this important? We're going to need the game to run on the back-end as well as the front-end to ensure that there is a single source of truth. Let's briefly explore why before returning to our choices.


>#### Single Source of Truth

> It's valuable to have one place that contains the true state of the game. In this case, we're talking about running the game itself on the server (as well as on each client). Here are some of the implications if we *don't* do that.

>##### The Benign Case
Users can sometimes take actions that should be impossible. Let's say players A and B are in-game. Player A places a tower, but due to network latency, Player B does not yet see the tower. Player B then places a tower in the same spot. This clearly cannot be allowed.

>There are a number of possible solutions to avoid this. For example, actions can have timestamps, where the earlier action always beats the later action. A simpler model, however, is to also have the game running on the server. Players A and B send their actions to the server. If the server cannot complete the action, it simply won't, and it can even inform the offending client that the action failed.

>##### The Malicious Case
Never trust the user. This is especially true in a browser, where hacking with JavaScript can be as easy as opening a developer console! If we were relying on (say) timestamps, a user could potentially send fake actions with old timestamps to the server or the other client. This could create an advantage for the user(s).

>If the server is the single source of truth, users can send any actions they want, but the server can simply ignore them if they are impossible. The server can then update the clients' games to entirely re-sync the games if they have become out of whack due to latency issues or cheating.

#### The Server Language

We know we need to run the game itself on the server, but we haven't actually picked a language yet. Given that it runs in JavaScript on the client side, the simple choice is to also run it on the server side.

If we picked a different back-end language for the game, we'd have to build the entire game twice - once in JavsScript and another time in Python/Ruby/whatever. You might argue that it would be easier the second time around - and you'd be right! But imagine the bugs?!

For simplicity, let's say we picked Python. Bugs can be introduced in the following ways:

* Any correct JavaScript code could be translated incorrectly into Python.
* Any incorrect JS can be translated correctly or incorrectly - both introduce bugs!
  * If translated correctly, we now have a bug on both the server and the client. The fix will have to happen in both places (and a fix might miss one of those places, then introducing a bug).
  * If translated incorrectly, either we have a new, separate bug, or we have correct game logic on the back-end and incorrect game logic on the front-end.
* Every test would have to be written twice, increasing the chances of bad tests and decreasing overall development speed.

Admittedly, this process could be used to actually find bugs, but I can't imagine that holding true over the long term. The idea of duplicating tests is not a happy thought, either - I'm sure I'd start making mistakes and testing the wrong things.

**JavaScript it is!**

To be clear, I would prefer to use another language on the back-end. The point of this exercise is that we pick our technology based on what we *need*, not what we *want*. It's time to leave the comfort zone!

### Web Framework

Lastly, the server-side component should be built in an existing framework to minimize effort. I briefly researched a number of JavaScript web frameworks. For the time being, however, I don't have need of anything too serious, and so [Express.js][expressjs] will do nicely. Many other web frameworks are actually built on top of Express, and so I can expect it to be solid (if minimal).



```
Three become just one
Creativity bursts forth
Strength comes from constraint
```

---


#### Updated tech stack:

* Front-end
  * HTML/CSS
  * JavaScript with MobX
* **Back-end**
  * **JavaScript with Express.js**


[expressjs]: https://expressjs.com/
[codecademy-python]: https://www.codecademy.com/learn/learn-python
