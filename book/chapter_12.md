---
layout: default
title: "Chapter 12: Classes and Objects"
nav_order: 6
parent: "Part 2: Making Games with Pygame"
---

# Chapter 12: Classes and Objects

## What You Will Learn
- What a class is and why programmers use them
- How to create your own class with `__init__` and `self`
- How to create objects from a class
- How to add methods (functions that belong to a class)
- How to manage multiple objects using a list of class instances
- A first look at inheritance (classes based on other classes)

## The Problem with Lots of Variables

Look at the Flappy Bird code from Chapter 11. It works, but it has a lot of separate variables floating around: `bird_x`, `bird_y`, `bird_velocity`, `bird_width`, `bird_height`. And that is just for one bird.

Now imagine you wanted a game with 10 different enemies. Would you write this?

```python
enemy1_x = 100
enemy1_y = 200
enemy1_speed = 3
enemy1_health = 5

enemy2_x = 300
enemy2_y = 150
enemy2_speed = 4
enemy2_health = 5

enemy3_x = 500
enemy3_y = 300
enemy3_speed = 2
enemy3_health = 5

# ... and so on for enemy4 through enemy10?
```

That is 40 variables just for 10 enemies. And if you want to add a new property -- say, `enemy_color` -- you have to add 10 more lines. If you want to move all the enemies, you need 10 separate lines of movement code. It gets messy fast.

There has to be a better way. And there is.

## What is a Class?

A **class** is a blueprint for creating things. It describes what properties something has and what it can do. An **object** is a specific thing built from that blueprint.

Think of it like a cookie cutter. The cookie cutter is the class -- it defines the shape. Each cookie you stamp out is an object. Every cookie has the same basic shape, but you can decorate each one differently (different icing, different sprinkles). The cookie cutter does not change.

```
    CLASS (the blueprint)              OBJECTS (built from the blueprint)
    +------------------+
    |     Enemy        |          enemy_a         enemy_b         enemy_c
    |                  |         +--------+      +--------+      +--------+
    |  has: x, y       | -----> |x = 100 |     |x = 300 |     |x = 500 |
    |  has: speed      |        |y = 200 |     |y = 150 |     |y = 300 |
    |  has: health     |        |speed= 3|     |speed= 4|     |speed= 2|
    |                  |        |health=5|     |health=5|     |health=5|
    |  can: move()     |        +--------+     +--------+     +--------+
    |  can: draw()     |
    +------------------+
```

One blueprint, many objects. Each object has its own values but shares the same structure.

You have actually been using objects already. Every time you write `pygame.Rect(10, 20, 50, 50)`, you are creating an object from the `Rect` class. The class `Rect` is the blueprint that says "a rectangle has an x, y, width, and height." Each specific rectangle you create is an object with its own values.

---

## Your First Class

Let's build a class from scratch. Create a new file called `my_class.py`:

```python
class Player:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 5
```

That is a complete class. It is small, but there is a lot to unpack. Let's go through it line by line.

**`class Player:`** -- This line starts a new class definition. The name `Player` is capitalized by convention. Class names in Python use **CamelCase** (each word starts with a capital letter, no underscores). This is different from variable names, which use `snake_case`.

**`def __init__(self, x, y):`** -- This is a special method called the **initializer** (some people call it the constructor). It runs automatically every time you create a new object from this class. The double underscores on each side are Python's way of marking it as special -- you do not call `__init__` yourself; Python calls it for you.

**`self`** -- This is the trickiest part. Every method in a class takes `self` as its first parameter. It refers to the specific object that the method is working on. When you write `self.x = x`, you are saying "this particular object's x value should be set to whatever was passed in." Think of `self` as the object saying "my" -- `self.x` means "my x position."

**`self.x = x`** -- This creates an **attribute** on the object. After this line, the object has a property called `x` that you can read and change later. The `x` on the right side is the parameter passed to `__init__`. The `self.x` on the left is the object's own stored value.

**`self.speed = 5`** -- This attribute is not passed in as a parameter. Every player starts with a speed of 5. You could change it later for a specific player if you wanted.

### Creating Objects

Now let's use the class. Add these lines below the class definition:

```python
player1 = Player(100, 200)
player2 = Player(300, 400)

print(player1.x, player1.y)
print(player2.x, player2.y)
print(player1.speed)
```

Run it:

```
python3 my_class.py
```

```
100 200
300 400
5
```

When you write `Player(100, 200)`, Python:

1. Creates a new, empty object.
2. Calls `__init__` on that object, passing `100` as `x` and `200` as `y`.
3. The `__init__` method sets `self.x = 100`, `self.y = 200`, and `self.speed = 5`.
4. Returns the finished object, which gets stored in `player1`.

`player1` and `player2` are separate objects. Changing one does not affect the other:

```python
player1.x = 999
print(player1.x)
print(player2.x)
```

```
999
300
```

Each object keeps its own copy of the attributes. That is the whole point.

---

## Methods: Functions That Belong to a Class

Right now our `Player` class just stores data. But classes can also have **methods** -- functions that belong to the class and can work with the object's data.

Update your `Player` class:

```python
class Player:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 5

    def move_right(self):
        self.x += self.speed

    def move_left(self):
        self.x -= self.speed

    def describe(self):
        print(f"Player at ({self.x}, {self.y}), speed: {self.speed}")
```

Each method takes `self` as its first parameter, just like `__init__`. This is how the method knows which object it is working on.

Replace the test code at the bottom of the file with this:

```python
player = Player(100, 200)
player.describe()

player.move_right()
player.move_right()
player.describe()

player.move_left()
player.describe()
```

Run it:

```
Player at (100, 200), speed: 5
Player at (110, 200), speed: 5
Player at (105, 200), speed: 5
```

Notice that when you call `player.move_right()`, you do not pass `self` -- Python handles that automatically. The object before the dot (`player`) becomes `self` inside the method.

### A Drawing Method

Here is where classes become really useful for games. Let's add a `draw` method that knows how to draw the player on screen. Create a new file called `class_demo.py`:

```python
import pygame

pygame.init()
screen = pygame.display.set_mode((600, 400))
clock = pygame.time.Clock()

class Player:
    def __init__(self, x, y, color):
        self.x = x
        self.y = y
        self.speed = 3
        self.color = color

    def handle_input(self, keys):
        if keys[pygame.K_LEFT]:
            self.x -= self.speed
        if keys[pygame.K_RIGHT]:
            self.x += self.speed
        if keys[pygame.K_UP]:
            self.y -= self.speed
        if keys[pygame.K_DOWN]:
            self.y += self.speed

    def draw(self, surface):
        pygame.draw.rect(surface, self.color, (self.x, self.y, 30, 30))
```

The `draw` method takes `surface` as a parameter (the screen to draw on) and uses `self.x`, `self.y`, and `self.color` to know where and how to draw. Everything the player needs is bundled together in one place.

Add the game loop below the class:

```python
player = Player(100, 200, (0, 150, 255))

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    player.handle_input(keys)

    screen.fill((30, 30, 30))
    player.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You can move the blue square around with the arrow keys. The game loop is clean and simple because all the player logic is inside the class.

---

## Why Classes Are Better: The Bouncing Balls Example

Let's see the real power of classes by comparing two approaches to the same program: three balls bouncing around the screen.

### The Messy Way (Without Classes)

Create a file called `messy_balls.py`:

```python
import pygame
import random

pygame.init()
screen = pygame.display.set_mode((600, 400))
clock = pygame.time.Clock()

ball1_x = random.randint(50, 550)
ball1_y = random.randint(50, 350)
ball1_dx = random.choice([-3, -2, 2, 3])
ball1_dy = random.choice([-3, -2, 2, 3])
ball1_color = (255, 0, 0)

ball2_x = random.randint(50, 550)
ball2_y = random.randint(50, 350)
ball2_dx = random.choice([-3, -2, 2, 3])
ball2_dy = random.choice([-3, -2, 2, 3])
ball2_color = (0, 255, 0)

ball3_x = random.randint(50, 550)
ball3_y = random.randint(50, 350)
ball3_dx = random.choice([-3, -2, 2, 3])
ball3_dy = random.choice([-3, -2, 2, 3])
ball3_color = (0, 0, 255)

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    ball1_x += ball1_dx
    ball1_y += ball1_dy
    if ball1_x <= 10 or ball1_x >= 590:
        ball1_dx = -ball1_dx
    if ball1_y <= 10 or ball1_y >= 390:
        ball1_dy = -ball1_dy

    ball2_x += ball2_dx
    ball2_y += ball2_dy
    if ball2_x <= 10 or ball2_x >= 590:
        ball2_dx = -ball2_dx
    if ball2_y <= 10 or ball2_y >= 390:
        ball2_dy = -ball2_dy

    ball3_x += ball3_dx
    ball3_y += ball3_dy
    if ball3_x <= 10 or ball3_x >= 590:
        ball3_dx = -ball3_dx
    if ball3_y <= 10 or ball3_y >= 390:
        ball3_dy = -ball3_dy

    screen.fill((0, 0, 0))
    pygame.draw.circle(screen, ball1_color, (ball1_x, ball1_y), 10)
    pygame.draw.circle(screen, ball2_color, (ball2_x, ball2_y), 10)
    pygame.draw.circle(screen, ball3_color, (ball3_x, ball3_y), 10)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. It works -- three colored balls bounce around. But look at all that repeated code. The movement logic is copied three times. The bouncing logic is copied three times. Adding a fourth ball means copying another block. Adding a new property (like size) means editing every single block.

### The Clean Way (With a Class)

Now create `clean_balls.py`:

```python
import pygame
import random

pygame.init()
screen = pygame.display.set_mode((600, 400))
clock = pygame.time.Clock()


class Ball:
    def __init__(self, color):
        self.x = random.randint(50, 550)
        self.y = random.randint(50, 350)
        self.dx = random.choice([-3, -2, 2, 3])
        self.dy = random.choice([-3, -2, 2, 3])
        self.color = color
        self.radius = 10

    def move(self):
        self.x += self.dx
        self.y += self.dy
        if self.x <= self.radius or self.x >= 600 - self.radius:
            self.dx = -self.dx
        if self.y <= self.radius or self.y >= 400 - self.radius:
            self.dy = -self.dy

    def draw(self, surface):
        pygame.draw.circle(surface, self.color, (self.x, self.y), self.radius)
```

The movement code and drawing code are written **once** inside the class. Now create the balls and the game loop:

```python
balls = [
    Ball((255, 0, 0)),
    Ball((0, 255, 0)),
    Ball((0, 0, 255)),
]

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    for ball in balls:
        ball.move()

    screen.fill((0, 0, 0))
    for ball in balls:
        ball.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. Same result -- three bouncing balls. But look at that game loop. It is tiny. And if you want 20 balls instead of 3, change the list:

```python
colors = [(255, 0, 0), (0, 255, 0), (0, 0, 255),
          (255, 255, 0), (255, 0, 255), (0, 255, 255)]
balls = [Ball(random.choice(colors)) for _ in range(20)]
```

That one line creates 20 balls, each with a random color from the list. The game loop does not change at all. Try it -- it is satisfying to see 20 balls bouncing around with so little code.

> Did you know? This style of programming, where you organize code into classes and objects, is called **object-oriented programming** (or OOP for short). It was invented in the 1960s and is now used in most large software projects. Video games use it heavily -- every character, item, and particle effect is usually an object.

---

## A Taste of Inheritance

There is one more concept worth knowing about, even though we will not use it much yet. **Inheritance** lets you create a new class based on an existing one. The new class gets everything the original has, plus whatever you add or change.

Here is a quick example. Say we have an `Enemy` class:

```python
class Enemy:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.speed = 2
        self.health = 3

    def move(self):
        self.x -= self.speed

    def describe(self):
        print(f"Enemy at ({self.x}, {self.y}), speed: {self.speed}")
```

Now we want a fast enemy that moves quicker but has less health. Instead of writing a whole new class, we **inherit** from `Enemy`:

```python
class FastEnemy(Enemy):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.speed = 5
        self.health = 1
```

The `(Enemy)` after `FastEnemy` means "this class is based on `Enemy`." The `super().__init__(x, y)` line calls the original `Enemy.__init__` to set up `x` and `y`, and then we override `speed` and `health` with different values.

`FastEnemy` gets the `move` and `describe` methods for free -- it inherits them from `Enemy`.

```python
regular = Enemy(100, 200)
fast = FastEnemy(100, 300)

regular.describe()
fast.describe()

regular.move()
fast.move()

regular.describe()
fast.describe()
```

```
Enemy at (100, 200), speed: 2
Enemy at (100, 300), speed: 5
Enemy at (98, 200), speed: 2
Enemy at (95, 300), speed: 5
```

Notice that the fast enemy says "Enemy at" because it inherited the `describe` method from `Enemy` without changes. If you wanted it to say "Fast Enemy at", you could override `describe` in `FastEnemy`.

Both share the same `move` method, but because `FastEnemy` has a higher `speed`, it moves further each time `move` is called.

Inheritance is powerful, but you do not need it for everything. For now, just know it exists. We will use it more in later chapters when our games get bigger.

## The Key Point

You do not *have* to use classes. The Flappy Bird game from Chapter 11 works perfectly without them. But for bigger games -- games with many enemies, multiple levels, power-ups, and particle effects -- classes make your code much easier to understand, change, and extend.

Classes can feel confusing at first. That is normal. The more you use them, the more natural they become. You have actually been using objects already -- `pygame.Rect` is a class, `pygame.Surface` is a class, and every time you called `.colliderect()` or `.blit()`, you were calling a method on an object.

Starting in the next chapter, we will use classes in our games. You will see how much cleaner the code becomes.

---

## Checkpoint: Complete Bouncing Balls Program

Here is the complete `clean_balls.py` listing:

```python
import pygame
import random

pygame.init()
screen = pygame.display.set_mode((600, 400))
clock = pygame.time.Clock()


class Ball:
    def __init__(self, color):
        self.x = random.randint(50, 550)
        self.y = random.randint(50, 350)
        self.dx = random.choice([-3, -2, 2, 3])
        self.dy = random.choice([-3, -2, 2, 3])
        self.color = color
        self.radius = 10

    def move(self):
        self.x += self.dx
        self.y += self.dy
        if self.x <= self.radius or self.x >= 600 - self.radius:
            self.dx = -self.dx
        if self.y <= self.radius or self.y >= 400 - self.radius:
            self.dy = -self.dy

    def draw(self, surface):
        pygame.draw.circle(surface, self.color, (self.x, self.y), self.radius)


colors = [
    (255, 0, 0),
    (0, 255, 0),
    (0, 0, 255),
    (255, 255, 0),
    (255, 0, 255),
    (0, 255, 255),
]

balls = [Ball(random.choice(colors)) for _ in range(10)]

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    for ball in balls:
        ball.move()

    screen.fill((0, 0, 0))
    for ball in balls:
        ball.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it and enjoy 10 colorful balls bouncing around the screen.

---

## What You Learned

- A **class** is a blueprint that defines what properties (attributes) and actions (methods) a type of object has.
- An **object** (also called an **instance**) is one specific thing created from a class.
- **`__init__`** is the initializer method that runs when you create a new object. It sets up the object's attributes.
- **`self`** refers to the specific object a method is working on. Think of it as the object saying "my" -- `self.x` means "my x."
- **Methods** are functions that belong to a class. They always take `self` as their first parameter.
- **Attributes** are variables that belong to an object, like `self.x` or `self.color`.
- Using a class and a list of objects is much cleaner than having dozens of separate variables.
- **Inheritance** lets you create a new class based on an existing one, reusing its code while changing or adding specific things.
- **Object-oriented programming** (OOP) is the name for this whole approach. Most large programs use it.

## Quick Quiz

1. What is the difference between a class and an object?

2. What does `self` refer to inside a method?

3. What is wrong with this code?

```python
class Cat:
    def __init__(self, name):
        self.name = name

    def speak():
        print(f"{self.name} says meow!")

my_cat = Cat("Whiskers")
my_cat.speak()
```

4. If you create two `Ball` objects and change the `color` of one, does the other one change too? Why or why not?

## Experiment

1. Add a `radius` parameter to the `Ball.__init__` method so each ball can have a different size. Create a mix of small and large balls.

2. Add a `speed_up` method to the `Ball` class that increases both `self.dx` and `self.dy` by 1 (keeping the sign). Call it when a ball bounces off a wall so balls get faster over time.

3. Go back to `class_demo.py` and create two `Player` objects with different colors and different key bindings (one uses arrow keys, the other uses `W`, `A`, `S`, `D`). You will need to change the `handle_input` method or pass the keys as parameters to `__init__`.

## Chapter Challenge

**Refactor the Flappy Bird pipe system to use a class.**

Create a `Pipe` class that stores its `x` position, `gap_y`, and `passed` state as attributes. Give it `move()`, `draw()`, and `get_rects()` methods. Then update the Flappy Bird game loop to use `Pipe` objects instead of dictionaries.

**Hints (try before reading!):**

1. Start with `class Pipe:` and an `__init__` that takes `gap_y` as a parameter. Set `self.x = WINDOW_WIDTH` (pipes always start on the right edge), `self.gap_y = gap_y`, and `self.passed = False`.

2. The `move` method should do `self.x -= PIPE_SPEED`. The `draw` method should calculate the top and bottom rectangles and draw them. The `get_rects` method should return the two rectangles for collision detection.

3. In the game loop, replace `pipes.append(spawn_pipe())` with `pipes.append(Pipe(random.randint(150, WINDOW_HEIGHT - 150)))`. Replace `pipe["x"]` with `pipe.x`, `pipe["gap_y"]` with `pipe.gap_y`, and so on.
