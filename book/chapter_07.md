---
layout: default
title: "Chapter 7: Your First Window"
nav_order: 1
parent: "Part 2: Making Games with Pygame"
---

# Chapter 7: Your First Window

## What You Will Learn
- What pygame is and why we use it
- How to open a window on screen with Python
- How the **game loop** works and why every game needs one
- How the coordinate system works in pygame
- How to draw shapes and colors on screen

Up until now, every program you have written has lived in the terminal. Text in, text out. That changes today. By the end of this chapter, you will have a colorful window on screen that *you* built from scratch using code.

We are going to use a library called **pygame**. Let's go.

## What is Pygame?

A **library** is a collection of code that someone else wrote so you don't have to. You have already used Python's built-in features like `print()` and `input()`. Pygame is a library that gives Python the power to open windows, draw graphics, play sounds, and read keyboard and mouse input -- everything you need to make a game.

Pygame is free and open source. Real indie game developers use it. If you have ever played *Frets on Fire* (a guitar hero clone) or *Angry Drunken Dwarves* -- those were made with pygame.

> Did you know? Pygame has been around since the year 2000. That makes it older than most of the people reading this book.

## Opening a Window

Create a new file called `first_window.py` and type in this code:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("My First Window")
```

Let's go through it line by line:

- `import pygame` -- this loads the pygame library so we can use it.
- `pygame.init()` -- this starts up all of pygame's systems (graphics, sound, etc.). You always need this line.
- `pygame.display.set_mode((600, 400))` -- this creates a window that is 600 pixels wide and 400 pixels tall. We store it in a variable called `screen`.
- `pygame.display.set_caption("My First Window")` -- this sets the text in the window's title bar.

Notice the double parentheses in `set_mode((600, 400))`. The outer parentheses are for calling the function. The inner parentheses create a **tuple** -- a pair of values grouped together. We will talk more about tuples soon.

Now run it:

```
python3 first_window.py
```

A window might flash on screen for a split second and then... it's gone. Or maybe nothing seems to happen at all.

What went wrong? Nothing, actually. Your code ran perfectly. It opened a window, set the title, and then the program ended. Python reached the last line, said "I'm done," and closed everything. The window disappeared because the program finished.

We need a way to keep the program running. We need a **loop**.

## The Game Loop

Every single game ever made has a loop at its heart. Tetris has one. Minecraft has one. Fortnite has one. It is called the **game loop**, and it does this:

```
+------------------------------------------+
|                                          |
|   1. Check for events (input)            |
|   2. Update the game state               |
|   3. Draw everything to the screen       |
|   4. Display the result                  |
|   5. Wait a tiny bit                     |
|   6. Go back to step 1                   |
|                                          |
+---->------>------>------>------>----+     |
|                                    |     |
+----<------<------<------<------<---+     |
|                                          |
+------------------------------------------+
```

This loop runs over and over, dozens of times per second. Each trip through the loop draws one **frame** -- like one page in a flipbook. Flip through the pages fast enough and it looks like smooth motion.

Let's build the game loop piece by piece.

### Step 1: A Loop That Runs Forever

Add a `while True` loop to your program:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("My First Window")

running = True
while running:
    pass
```

We use a variable called `running` to control the loop. As long as `running` is `True`, the loop keeps going. The `pass` keyword just means "do nothing" -- it is a placeholder.

If you run this now, you will get a window that stays open. Progress! But there is a problem: you cannot close it. The X button does not work. You will have to press `Ctrl+C` in the terminal to force it to stop. Go ahead and do that now.

The window ignores the close button because we are not *checking* for it. That brings us to events.

### Step 2: Handling Events

In pygame, an **event** is something that happens -- the user clicks the mouse, presses a key, or clicks the close button. Pygame collects these events into a list. Each trip through the loop, we need to check that list.

Replace `pass` with this:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
```

`pygame.event.get()` returns a list of all the events that happened since the last time we checked. We loop through each one. If any event has the type `pygame.QUIT` (which happens when someone clicks the close button), we set `running` to `False` and the while loop stops.

Run it now. The window opens, stays open, and when you click the X button, it closes cleanly.

### Step 3: Updating the Display

Add `pygame.display.flip()` inside the loop, after the event handling:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    pygame.display.flip()
```

Here is why we need this. Pygame uses a trick called **double buffering**. Think of it like this: you have two canvases. You draw on the hidden canvas behind the scenes, and when you are done, you "flip" it to the front so the player sees it all at once. Without `flip()`, nothing you draw will ever appear on screen.

### Step 4: Controlling Speed

Right now, the loop runs as fast as your computer can go. That wastes energy and makes the game run at different speeds on different computers. We need a speed limit.

Add a **Clock** before the loop and `clock.tick(60)` inside it:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("My First Window")

clock = pygame.time.Clock()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    pygame.display.flip()
    clock.tick(60)
```

`clock.tick(60)` tells pygame: "Wait long enough so that this loop runs at most 60 times per second." Each trip through the loop draws one frame, so this gives us 60 **frames per second** (FPS).

Think of it like a flipbook. If you flip the pages too fast, it is a blur. Too slow, it looks choppy. 60 pages per second is smooth and is what most games aim for.

After the loop ends, add one more line at the very bottom of your file (outside the loop, no indentation):

```python
pygame.quit()
```

This shuts down pygame cleanly when the window closes.

Run your program. You should see a window appear. That's YOUR window. You made that. It sits there, solid and stable, and it closes cleanly when you click the X. That is a working game loop.

---

## The Coordinate System

Before we draw anything, we need to understand *where* things go on screen. Pygame uses a **coordinate system** with two numbers: **x** (horizontal position) and **y** (vertical position).

Here is the important part: **y goes DOWN, not up**. In math class, y goes up. In pygame (and almost all computer graphics), y goes down. This trips up everyone at first, so read this carefully.

```
(0,0)-------- x increases --------> (600,0)
  |
  |
  y increases
  |
  |
  v
(0,400)                              (600,400)
```

- The **top-left** corner is position `(0, 0)`.
- Moving **right** increases x.
- Moving **down** increases y.
- The **bottom-right** corner of our 600x400 window is `(599, 399)`.

So if you want to put something near the top of the screen, give it a *small* y value. Near the bottom? A *large* y value. This is backwards from math class, and it will feel weird for a while. That's normal.

> Did you know? The reason y goes down in computer graphics is historical. Old TV screens drew the picture line by line from top to bottom, and the coordinate system was designed to match.

---

## Drawing Shapes and Colors

Now for the fun part. Let's put some color on screen.

### Colors as RGB Tuples

Computers make colors by mixing **Red**, **Green**, and **Blue** light. Each value goes from 0 (none) to 255 (full brightness). We write a color as three numbers in parentheses -- a **tuple**.

```
Red:    (255,   0,   0)    Full red, no green, no blue
Green:  (  0, 255,   0)    No red, full green, no blue
Blue:   (  0,   0, 255)    No red, no green, full blue
White:  (255, 255, 255)    All colors at full = white
Black:  (  0,   0,   0)    No colors at all = black
Yellow: (255, 255,   0)    Red + green = yellow
```

A **tuple** is like a list, but you use regular parentheses `()` instead of square brackets `[]`, and you cannot change the values after creating it. Tuples are perfect for things that always go together, like the three parts of a color.

Let's define some colors at the top of our program, right after `pygame.init()`:

```python
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
SKY_BLUE = (135, 206, 235)
GREEN = (34, 139, 34)
YELLOW = (255, 255, 0)
```

We write color names in ALL_CAPS because they are **constants** -- values we define once and never change.

### Filling the Background

Inside the game loop, *before* `pygame.display.flip()`, add this:

```python
screen.fill(SKY_BLUE)
```

This paints the entire window with our sky blue color. Run it -- you should see a bright blue window.

Why do we fill the screen every single frame? Imagine painting on a canvas. Each frame, you need to paint over the previous picture before drawing the new one. Otherwise, old drawings would pile up and make a mess. Filling the screen is like starting with a clean canvas each frame.

### Drawing Rectangles

`pygame.draw.rect()` draws a rectangle. It needs: the surface to draw on, the color, and a rectangle defined by `(x, y, width, height)`.

Add this line after `screen.fill(SKY_BLUE)` but before `pygame.display.flip()`:

```python
pygame.draw.rect(screen, GREEN, (0, 300, 600, 100))
```

This draws a green rectangle starting at position (0, 300), with a width of 600 and a height of 100. That covers the bottom strip of the window -- it looks like grass.

### Drawing More Shapes

Let's add a house and a sun. Add these lines after the grass rectangle:

```python
# House body
pygame.draw.rect(screen, RED, (150, 200, 120, 100))

# House roof
pygame.draw.polygon(screen, (139, 69, 19), [(140, 200), (210, 140), (280, 200)])

# Door
pygame.draw.rect(screen, (101, 67, 33), (190, 250, 40, 50))

# Sun
pygame.draw.circle(screen, YELLOW, (500, 80), 40)
```

Some new things here:

- `pygame.draw.polygon()` draws a shape with any number of corners. We give it a list of points and it connects them. Three points make a triangle -- a perfect roof.
- `pygame.draw.circle()` draws a circle. It needs: the surface, the color, the center position `(500, 80)`, and the radius `40`.
- We can write color tuples directly instead of using a variable name. `(139, 69, 19)` is a brown color for the roof.

### Drawing Lines

Let's add some sun rays. Add these after the sun:

```python
pygame.draw.line(screen, YELLOW, (500, 80), (500, 20), 2)
pygame.draw.line(screen, YELLOW, (500, 80), (560, 80), 2)
pygame.draw.line(screen, YELLOW, (500, 80), (540, 40), 2)
pygame.draw.line(screen, YELLOW, (500, 80), (460, 40), 2)
```

`pygame.draw.line()` takes: surface, color, start point, end point, and line thickness. The `2` at the end makes the lines 2 pixels wide.

Run your program. You should see a scene with a blue sky, green grass, a red house with a brown roof and door, and a yellow sun with rays.

---

## Checkpoint: Full Program

Your code should now look like this:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("My First Window")

clock = pygame.time.Clock()

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
SKY_BLUE = (135, 206, 235)
GREEN = (34, 139, 34)
YELLOW = (255, 255, 0)

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Background
    screen.fill(SKY_BLUE)

    # Ground
    pygame.draw.rect(screen, GREEN, (0, 300, 600, 100))

    # House body
    pygame.draw.rect(screen, RED, (150, 200, 120, 100))

    # House roof
    pygame.draw.polygon(screen, (139, 69, 19), [(140, 200), (210, 140), (280, 200)])

    # Door
    pygame.draw.rect(screen, (101, 67, 33), (190, 250, 40, 50))

    # Sun
    pygame.draw.circle(screen, YELLOW, (500, 80), 40)

    # Sun rays
    pygame.draw.line(screen, YELLOW, (500, 80), (500, 20), 2)
    pygame.draw.line(screen, YELLOW, (500, 80), (560, 80), 2)
    pygame.draw.line(screen, YELLOW, (500, 80), (540, 40), 2)
    pygame.draw.line(screen, YELLOW, (500, 80), (460, 40), 2)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Notice the `pygame.quit()` at the very end, after the loop. This shuts down pygame cleanly when the program exits. It is good practice to include it.

Type out the whole program, run it, and make sure it works. If you get an error, check your parentheses carefully -- pygame functions often have lots of nested parentheses, and it is easy to miss one.

## What You Learned
- **Pygame** is a library that gives Python the ability to make games with graphics and sound.
- `pygame.init()` starts pygame, and `pygame.display.set_mode()` opens a window.
- The **game loop** runs every frame: check events, update state, draw, display, wait.
- **Events** are things that happen (key presses, mouse clicks, close button).
- `pygame.display.flip()` shows what you have drawn on screen (double buffering).
- `clock.tick(60)` limits the game to 60 frames per second.
- In pygame's coordinate system, **(0, 0) is the top-left** and **y increases downward**.
- Colors are **(R, G, B) tuples** with values from 0 to 255.
- `screen.fill()` paints the whole screen -- we do this every frame to erase the old picture.
- `pygame.draw.rect()`, `pygame.draw.circle()`, `pygame.draw.line()`, and `pygame.draw.polygon()` draw shapes.

## Quick Quiz

1. What does `pygame.init()` do and why do we need it?
2. If the top-left of the screen is `(0, 0)` and the window is 600 pixels wide and 400 pixels tall, what are the coordinates of the bottom-right corner?
3. What color do you get with `(0, 0, 0)`? What about `(255, 255, 255)`?
4. Why do we call `screen.fill()` at the beginning of every frame, before drawing anything?

## Experiment

1. Change the window size to `(800, 600)`. What happens to the scene? Can you adjust the shape positions so the house is still centered and the ground still covers the bottom?
2. Change the background color to `(255, 165, 0)` (orange) and the sun color to `RED`. Now it looks like a sunset.
3. Add a second window (a small white rectangle) to the house. Then add two windows. Then add a chimney using `pygame.draw.rect()`.
4. Try making new colors. What does `(255, 0, 255)` look like? What about `(0, 255, 255)`?

## Chapter Challenge

Create a **night scene**. Fill the background with a dark color (try `(25, 25, 50)` for a deep navy). Draw a white or light yellow circle for the moon. Then draw at least 10 small white circles scattered around the sky for stars. Draw the ground in a dark green.

**Hint 1:** Stars are just small circles. A radius of 2 or 3 works well.

**Hint 2:** Spread the stars out by giving each one different x and y coordinates. Keep the y values small (between 20 and 200) so the stars stay in the sky, not on the ground.

**Hint 3:** If you want to get fancy, make some stars brighter than others by using slightly different shades of white, like `(200, 200, 200)` for a dimmer star.
