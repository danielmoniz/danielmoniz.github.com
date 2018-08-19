---
layout: post
title:  "Server Architecture, Musings on"
date:   2018-01-14 5:33:00 -0500
categories: td technology
---

The goal from the start has been to create an online game. Although I have some experience creating a turn-based game, this one needs to be real-time. It's going to need to handle some number of concurrent players. I don't want to get carried away and say "millions", so I'll stick to "some number" for now.


### The Cloud

If it's online, I feel that usage of the cloud at this point is more or less mandatory. The cloud _should have_ (has? will?) revolutionized how companies scale their online games. I'm not convinced of that until I see Blizzard servers still running the day after releasing a game. Nevertheless, this is the direction the industry will go (see [Amazon Lumberyard][lumberyard]). Although I'm not predicting users on a massive scale, it probably still makes sense to utilize the cloud.

### Web App and Game

As a user-facing project, this has two main components - the web app and the game. I'm more interested in the game itself (and online connectivity, etc.), but the web app also to be built.

The question is: should these be separate? Should the same web server handle requests for both the web app and the game?

#### Idyllic Happy Land

Let's imagine the perfect system.

1. The user goes to www.futuretd.com (for example) and signs in.
1. They browse high scores, message a friend, and generally explore the ecosystem built around the game.
1. They click Play and start their own game.
1. The Magic happens. The server sees that they are starting a new game, and it provides a previously booted Amazon EC2 instance. It redirects them to game.futuretd.com/12345.
  * Even better, let's assume it uses Amazon Beanstalk and can scale as the game demands more resources.
1. All of their requests now go through a microservice - a cloud-based machine provisioned specifically for their game.

The above idea (thanks to [Devon Noel de Tilly][devon-github] for some of the ideas) sounds awesome. It would have a few major advantages.

1. The game server code could run in JavaScript, meaning that the game code could be reused.
1. The web app server could be built in anything at all!
1. It scales immensely.

Keeping in mind that I'm not a web architect, this is vastly out of scope for the time being. I would love for the game to go in this direction eventually, but I think I need a playable game before I start giving users (what users?) their own EC2 instances.

### Another Time

Sometimes problems aren't meant to be solved *now*. I'll solve this one when it becomes a little more relevant and focus on the basics until then.

My options are pretty limited if I push off this problem. One server providing both a web app and a game means I'm running the same language on both. On the bright side, logins will be easier if everything is running on the same server. That looks something like this:

1. The user goes to www.futuretd.com (for example) and signs in. They also play the game there and do everything else.

Yep, that's it. I'll have to give it some thought as to whether or not the drawbacks are a deal breaker.

```
Brow furled, frustrated
Rain falls, each drop a problem
Maybe tomorrow
```


[lumberyard]: https://aws.amazon.com/lumberyard/
[devon-github]: https://github.com/devonoel
