---
layout: post
title:  "Modular Game Components"
date:   2026-5-11
categories: game foss
---

I tried [Godot](https://godotengine.org/) some years back and while it was an impressive piece of software, I found myself struggling with it. (Just to emphasize, this is nothing against Godot. Try it out! It really is a great piece of software that many people use to make great things!) Instead I found that I really liked the simplicity of [Love2D](https://www.love2d.org/). If you wanted to print "Hello, World!" to the screen, it really was as simple as:

```lua
function love.draw()
  love.graphics.print("Hello World!")
end
```

Saying that, Love2D might be too simple (well, for me). The simple `love.draw` loop exhbits challenges in *organization*. For instance, if there is only one `love.draw` function, how do you keep track of which scene (i.e. which level, menu screen, etc.) you are on. A naive approach to this might look like:

```lua
function love.load()
  scene = "menu"
end

function love.draw()
  if scene == "menu" then
    ...
  else if scene == "level1" then
    ...
  ...
  end
```

but this becomes a large monolithic unreeadable mess very quickly. Fortunately, [awesome-love2d](https://github.com/love2d-community/awesome-love2d) addresses this!

Modular Components
------------------

awesome-love2d keeps a list of libraries compatible with Love2D. In particular, I liked [HUMP](https://hump.readthedocs.io/en/latest/index.html). For the issue of gamestate, HUMP provides a submodule [HUMP.gamestate](https://hump.readthedocs.io/en/latest/gamestate.html) which allows you to make (along with some other features) separate draw functions for each scene such as:

```lua
local menu = {}
local game = {}

function menu:draw()
    -- Do something for drawing menu.
end

function game:draw()
    -- Do something for drawing the game.
end
```

The organization I sought after was now found!

HUMP even has other features. [HUMP.timer](https://hump.readthedocs.io/en/latest/timer.html) allows for creating timers. It also has functionality for *tweening* values. That is, you can take one value and gradually change it to another value. I really loved these two sublibraries. But even I found the HUMP.tween library a little too lacking, but I did not want to use another tweening library, contribute to HUMP, modify HUMP source to accomidate my project, or make my own.

Making My Own
-------------

There was another problem: I wanted to write games in Python. I know this is a rather weird choice: It was purely because I was most familiar with Python. [Pygame](https://pyga.me/) seemed to be relatively similar to Love2D, but [awesome-pygame](https://github.com/kadir014/awesome-pygame) had fewer libraries listed. Even worse, after searching, I was not able to find much. The good news, I could use this opportunity to help contribute to the Pygame ecosystem!

Learning From HUMP
------------------

I wanted a lot of HUMP functionality, but wanted to enforce the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy). Thus, instead of making a HUMP equivalent for Pygame, I sought to make many small libraries, of which, combining all of them results in something that resembles HUMP. Fortunately, it seems as though there is an equivalent to HUMP.gamestate via [PyScenes](https://github.com/treygilliland/PyScenes). So, I decided to start working on tweening.

transytion
----------

[transytion](https://github.com/thyrgle/transytion) is a tweening library. That is all it is, and all it will be. Saying that, as I mentioned earlier, despite how much I love HUMP, I found there were some missing features.

- The callback mechanism was slightly cumbersome.
- Stopping tweens should have some sort of (optional) stop callback.
- An extensive system for composing tweens together.

Later I would realize that I could better utilize Python decorators to even further simplify tween creation. A function can be associated with a tween and by using some decorator magic, calling that function will not actually call the function immediately, but instead run a tween *then* call the function. Taken, from the documentation this allows us to write:

```py
move = Tween(...)
@tween_then_call(move)
def say_something():
    print("Hello!")

say_something()
```

Thus, if one wants a character that moves a bit, then says something, moves a bit again, and says more, we can keep things organized and simply write:

```py
say_something()
say_something2()
```

Letting us focus on the underlying logic and *not* the animations.

Beyond HUMP
-----------

As I continued working with Pygame, there seemed to be other core components missing:

- From my experience with [conjecscore.org](https://conjecscore.org/), I found it annoying to manage the layout of content on the screen compared to how it is done in web development. To (partially) fix this, I worked on a [lpyout](https://github.com/thyrgle/lpyout) as a layout engine for games.
- The shortest path libraries I found were inconvenient to use (for games at least). For most of these libaries, the input required a graph representation (such as an adjacency matrix). I did not have an adjacency matrix of the world, I had a grid of the world. So, I made [gtravyl](https://github.com/thyrgle/gtravyl) that focuses shortest paths for grid representations of a world. Although not as flexible, I believe this is much easier to use than the existing libraries out there and is still flexible enough for most use cases.
- I feel (although potentially a *very* controversial opinion) that event listeners are underrated as an organizational tool. This includes games, so I developed [pypagate](https://github.com/thyrgle/pypagate) which makes it so you can turn formulas into event listeners. Formulas end up being very flexible and can describe many situations quite elegantly. Consider the following game scenario from the README of pypagate that creates event listeners for when a game should end:

```py
>>> from pypagate import Term, fire_on
>>> class Player:
...     def __init__(self, health):
...         self.health = Term(health)
...
>>> player = Player(3)
>>> @fire_on(player.health == 0)
>>> def game_over():
...     print("Game over")
>>> player.health -= 1
>>> player.health -= 1
>>> player.health -= 1
Game over
```

Conclusion
----------

My ultimate goal is not to create an engine. My goal is to extend existing engines, such as Pygame, so that we have the modularity of the Unix philosophy with the feature completeness of the Godot engine. Thus, I am publishing many smaller libraries as a collective of game libraries anyone can use (Apache 2.0 License).
