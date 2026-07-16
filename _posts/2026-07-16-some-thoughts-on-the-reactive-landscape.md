Some Thoughts on the Reactive Landscape
=======================================

I have now written three frameworks for [reactive programming](https://en.wikipedia.org/wiki/Reactive_programming):

- [pypagate](https://github.com/thyrgle/pypagate) for Python
- [forcamla](https://github.com/thyrgle/forcamla) for OCaml
- [formulis](https://github.com/thyrgle/formulis) for C++

Why? Because I do not like the current reactive programming landscape. Let me explain.

# The Problem

If you did not read Wikipedia article above you might not have the same idea in mind that I do for reactive programming. Even if you've used a reactive library such as the popular [RxJS](https://rxjs.dev/) you still may not have the same idea of reactive programming that I do.

Concisely put, I believe reactive program stems from treating the assignment operator as mathematical equality. But what does that mean? Let's look at the first Wikipedia example:

```javascript
var b = 1
var c = 2
var a = b + c
b = 10
console.log(a) // 3 (not 12 because "=" is not a reactive assignment operator)

// now imagine a special operator "$=" exists that changes the value of a variable (executes code on the right side of the operator and assigns result to left side variable) whenever explicitly initialized, and when referenced variables (on the right side of the operator) are changed
var b = 1
var c = 2
var a $= b + c
b = 10
console.log(a) // 12
```

The idea is, when `a` is assigned to `b + c` it remains always equal to `b + c` even if `b + c` change. For a more concrete example: Consider [Microsoft Excel](https://en.wikipedia.org/wiki/Microsoft_Excel), [Libreoffice Calc](https://www.libreoffice.org/), or any spreadsheet application. 

You've probably had to use a spreadsheet application at some point. There are essentially two types of cells in a spreadsheet:

- Cells with raw values.
- Cells with formulas.

Cells with raw values can be changed. Cells with formulae use other cells to compute values and when cells change value, the formula cells automatically update to reflect these changes.

Now let's look at [RxJS](https://rxjs.dev/). This is their first example:

```javascript
import { fromEvent } from 'rxjs';

fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```

This does not remind me of either the Wikipedia article or spreadsheets. There are no "events". What is the purpose of `subscribe`? Frankly, none of the examples in their [overview](https://rxjs.dev/guide/overview) strike me as resembling *the essence* of reactive programming. That is *not* to say there is no use case for RxJS, but it left me wondering:

> What if things were more like a spreadsheet?

# Solving the Problem

I started working on [pypagate](https://github.com/thyrgle/pypagate) trying to understand what I envisioned reactive programming to look like. Like I mentioned above, the core tenant is that *assignment becomes mathematical equality*. So, once `z = x + y`, `z` can change value depending on what value `x` and `y` are. Thus, I came up with the following scheme:

- Terms objects are like raw data cells in spreadsheets: They can be freely changed by the user/programmer.
- Formulae are compositions of terms/other formulae with binary operations and once created change only if their children terms change.

Furthermore, I wanted syntax to be convenient. This is something I feel like is lost with RxJS to be quite frank. I want to be able to write `z = x + y` and, ideally, nothing else. You can do this with operator overloading.

As an aside, core tenants of Python and C++ were broken making the reactive libraries. In Python, comparison operators like `==`, even when overriden, are expected to return a `bool`, but my library *intentionally* returns a `Formula` object. Why? Because you need to return something that has the ability to auto-update. A `bool` has no way of doing this. In C++, you should not modify `x` or `y` in the expression `x + y`, but you need to for reactivity. See, `x` and `y` have parent `formula` objects and when they change they must notify their parents. Thus, in `z = x + y` you add `z` as a parent to both `x` and `y`.

Despite all this, does this give you anything? Maybe a little, but we have to go just a *touch* beyond spreadsheets.

# Event Listeners

When I started pypagate, I liked the idea of spreadsheet programming, but it very quickly ran into the problem of

> What use case does this have?

I was hard pressed for ideas and realized I needed to add a bit more spice to this reactive programming idea, so, I added the ability *to make* event listeners.

I think this is best illustrated with an example, here is one from the README of pypagate:

```python
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

Consider how, in a traditional game you might need to check this each iteration of the game loop and have *yet another segment* of code in a monolithic update loop. This helps split things up, and furthermore, it does *not* execute every iteration of the game loop! In particular, with formulae we now have a method to refine our listening abilities, and with the combination of callbacks, we can *change* the (application) world at the appropriate time!

# Future Work

Unfortunately, I have tried thinking of good UI examples, but the convenience is not *quite* there.

For a second, pretend that [tkinter](https://docs.python.org/3/library/tkinter.html) is used for real world applications. Let's consider a [small (non-reactive) example of it:](https://tkdocs.com/tutorial/firstexample.html)

```python
from tkinter import *
from tkinter import ttk

def calculate(*args):
    try:
        value = float(feet.get())
        meters.set(round(0.3048 * value, 4))
    except ValueError:
        pass

root = Tk()
root.title("Feet to Meters")

mainframe = ttk.Frame(root, padding=(3, 3, 12, 12))
mainframe.grid(column=0, row=0, sticky=(N, W, E, S))

feet = StringVar()
feet_entry = ttk.Entry(mainframe, width=7, textvariable=feet)
feet_entry.grid(column=2, row=1, sticky=(W, E))

meters = StringVar()
ttk.Label(mainframe, textvariable=meters).grid(column=2, row=2, sticky=(W, E))

ttk.Button(mainframe, text="Calculate", command=calculate).grid(column=3, row=3, sticky=W)

ttk.Label(mainframe, text="feet").grid(column=3, row=1, sticky=W)
ttk.Label(mainframe, text="is equivalent to").grid(column=1, row=2, sticky=E)
ttk.Label(mainframe, text="meters").grid(column=3, row=2, sticky=W)

root.columnconfigure(0, weight=1)
root.rowconfigure(0, weight=1)
mainframe.columnconfigure(2, weight=1)
for child in mainframe.winfo_children(): 
    child.grid_configure(padx=5, pady=5)

feet_entry.focus()
root.bind("<Return>", calculate)

root.mainloop()
```

This example converts feet to meters. You input feet, click a button and then it calculates the meters.

Here is what I envision a reactive version of this *should* look like:

```python
from tkinter import *
from tkinter import ttk

root = Tk()
root.title("Feet to Meters")

mainframe = ttk.Frame(root, padding=(3, 3, 12, 12))
mainframe.grid(column=0, row=0, sticky=(N, W, E, S))

feet = Term("0")
feet_entry = ttk.Entry(mainframe, width=7, text=feet)
feet_entry.grid(column=2, row=1, sticky=(W, E))

meters = 0.3048 * as_float(feet)
ttk.Label(mainframe, text=meters).grid(column=2, row=2, sticky=(W, E))

ttk.Label(mainframe, text="feet").grid(column=3, row=1, sticky=W)
ttk.Label(mainframe, text="is equivalent to").grid(column=1, row=2, sticky=E)
ttk.Label(mainframe, text="meters").grid(column=3, row=2, sticky=W)

root.columnconfigure(0, weight=1)
root.rowconfigure(0, weight=1)
mainframe.columnconfigure(2, weight=1)
for child in mainframe.winfo_children(): 
    child.grid_configure(padx=5, pady=5)

feet_entry.focus()

root.mainloop()
```

In short:

- No button needed.
- No calculate needed. Just have `meters` be a formula in terms of the term `feet`.

Unfortunately, the world beyond my libraries are *not* reactive. So UI elements *do not* change when my formula/terms change. Hence, my future work will be making UI frameworks more reactive friendly to allow for this kind of programming.
