---
layout: post
title:  "Modular Game Components"
date:   2026-5-11
categories: game foss
---

I tried [Godot](https://godotengine.org/) some years back and while it was an impressive piece of software, I found myself struggling with it. (Just to emphasize, I have nothing against Godot! Try it out! It really is a great piece of software that many people use to make great things!) Instead I found that I really liked the simplicity of [Love2D](https://www.love2d.org/). If you wanted to print "Hello, World!" to the screen, it really was as simple as:

```lua
function love.draw()
  love.graphics.print("Hello World!")
end
```

Saying that, Love2D might be too simple (well, for me). The simplicity of the `love.draw` loop exhbits challenges in *organization*. For instance, if there is only one `love.draw` function, how do you keep track of which scene (i.e. which level, menu screen, etc.) you are on? A naive approach to this might look like:

```lua
function love.load()
  scene = "menu"
end

function love.draw()
  if scene == "menu" then
    ...
  elseif scene == "level1" then
    ...
  ...
  end
end
```

but this becomes an unreeadable mess very quickly. Fortunately, [awesome-love2d](https://github.com/love2d-community/awesome-love2d) addresses this!

Modular Components
------------------

awesome-love2d keeps a list of libraries compatible with Love2D. In particular, I liked [HUMP](https://hump.readthedocs.io/en/latest/index.html). For the issue of scene management, HUMP provides a submodule [HUMP.gamestate](https://hump.readthedocs.io/en/latest/gamestate.html) which allows you to make (along with some other features) separate draw functions for each scene:

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

HUMP even has other features. [HUMP.timer](https://hump.readthedocs.io/en/latest/timer.html) (as the name suggests) allows for creating timers. It also has functionality for *tweening* values. That is, you can take one value and gradually change it to another value *over time*. I really love these two sublibraries. But even I found the HUMP.tween library a little *too* lacking. This gave me a predicament: should I use another tweening library, contribute to HUMP, modify HUMP source to accomidate my project, or make my own.

Making My Own
-------------

There was another problem I have yet to mention: I wanted to write games in Python. I know this is a rather weird choice given that Python is notoriously slow. Admittedly, I wanted to write in Python just because I was most familiar with Python. [Pygame](https://pyga.me/) seemed to be relatively similar to Love2D, but [awesome-pygame](https://github.com/kadir014/awesome-pygame) had fewer libraries listed. Even worse, after searching, I was not able to find much either. The good news, I could use this opportunity to help contribute to the Pygame ecosystem!

Learning From HUMP
------------------

I wanted a lot of HUMP functionality, but wanted to better enforce the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy). Thus, instead of making a HUMP equivalent for Pygame, I sought to make smaller libraries, of which, combining all of them results in something that resembles HUMP. As for scene management, it seems as though there is an equivalent to HUMP.gamestate called [PyScenes](https://github.com/treygilliland/PyScenes) but I could not find anything for tweening. So, I decided to start working on a tweening library.

[transytion](https://github.com/thyrgle/transytion) is a tweening library that originally started out similar to HUMP.timer, but it addressed some of the issues I had with HUMP.timer, namely:

- The callback mechanism was a bit cumbersome to use.
- Stopping tweens should have some sort of (optional) stop callback. Therefore, if something happens (for instance, an enemy dies) the tween can have a convenient to use fail mechanism and switch to doing something else.
- I wanted an extensive system for composing tweens together instead of glueing callbacks together.

Even later I realized that I could utilize Python decorators to simplify tween creation. A function can be associated with a tween (say, `move`), and by using some decorator magic, calling that function will not actually call the function immediately. Instead the function call runs `move` *then* calls the function. Taken from the documentation, this allows us to write:


```py
move = Tween(...)
@tween_then_call(move)
def say_something():
    print("Hello!")

say_something()
```

Thus, if one wants a character to move a bit, then say something, move a bit more, and say even more, we can keep things organized:

```py
say_something()
say_something2()
```

This keeps the focus of the code on the underlying logic and *not* the animations.

Beyond HUMP
-----------

As I continued working with Pygame, there seemed to be other core components missing:

- From my experience with [conjecscore.org](https://conjecscore.org/), I found it annoying to manage the layout of content on the screen compared to how it is done in web development. Layout ended up being much harder than I thought it would be. To (partially) fix this, I worked on [lpyout](https://github.com/thyrgle/lpyout) as a layout engine for games.
- The shortest path libraries I found were inconvenient to use (for games at least). For most of these libaries, the input required a graph representation (such as an adjacency matrix). I did not have an adjacency matrix of the world. Instead, I had a grid of the world. I made [gtravyl](https://github.com/thyrgle/gtravyl) that takes in a *grid representation* of the world and finds the shortest path. Although not as flexible, I believe this is much easier to use than the existing libraries out there while remaining flexible enough for *most* use cases.
- I feel (although potentially a *very* controversial opinion) that event listeners are underrated as an organizational tool. This includes games, so I developed [pypagate](https://github.com/thyrgle/pypagate) which can create formulas and (from formulas) create listeners. Formulas can describe many situations quite elegantly. Consider the following game scenario from the `README` of pypagate that creates event listeners for when a game should end:

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

My ultimate goal is not to create a game engine but to extend existing game frameworks, such as Pygame, so that we have the modularity of the Unix philosophy *with* the feature completeness of an engine. The result is a collectiion of libraries that anyone can use (Apache 2.0 License).
