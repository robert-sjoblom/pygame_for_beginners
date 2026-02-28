---
layout: default
title: "Chapter 8: Pong"
nav_order: 2
parent: "Part 2: Making Games with Pygame"
---

# Chapter 8: Movement and Input -- Game 1: Pong

## What You Will Learn
- How to make objects move on screen
- How to read keyboard input to control a game
- How **collision detection** works with rectangles
- How to keep score and display text
- How to build a complete game from start to finish

You know how to open a window, draw shapes, and run a game loop. Now we are going to use all of that to build a real game: **Pong**.

Pong was one of the first video games ever made. It came out in 1972 -- over 50 years ago. Two paddles, one ball, and a simple rule: don't let the ball get past you. It is a perfect first game to build because it has movement, input, collision, and scoring, but nothing too complicated.

By the end of this chapter, you will have a two-player game you can actually play with a friend.

## Section 1: Planning and Setting Up

Before we write any code, let's think about what we need. Good programmers plan before they code.

### The Game Layout

Here is what our Pong game looks like:

```
+----------------------------------------------+
|                                              |
|    SCORE: 0               SCORE: 0          |
|                                              |
|   +--+                              +--+     |
|   |  |                              |  |     |
|   |  |            o                 |  |     |
|   |  |                              |  |     |
|   +--+                              +--+     |
|                                              |
|  LEFT                              RIGHT     |
| PADDLE                            PADDLE     |
+----------------------------------------------+
```

### What We Need

Let's list the pieces:

- **Two paddles:** rectangles on the left and right sides of the screen
- **One ball:** a small square that moves around
- **Two scores:** one for each player
- **Movement rules:** the ball bounces off the top wall, bottom wall, and both paddles
- **Scoring:** when the ball goes past a paddle, the other player gets a point
- **Controls:** W/S keys for the left paddle, Up/Down arrow keys for the right paddle

Now let's build it step by step.

### The Starting Template

Create a new file called `pong.py`. Start with the game loop template from Chapter 7:

```python
import pygame

pygame.init()

WIDTH = 600
HEIGHT = 400
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pong")

clock = pygame.time.Clock()

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill(BLACK)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Two things are different from before. First, we store the window width and height in variables called `WIDTH` and `HEIGHT`. We will use these numbers a lot, and it is much better to use a named variable than to type `600` and `400` over and over. Second, the background is black -- Pong traditionally has a black background.

Run it. You should see a black window. Good -- that is our empty Pong court.

### Drawing the Paddles

Now let's add the two paddles. Add these variables before the game loop, after the color definitions:

```python
# Paddle settings
PADDLE_WIDTH = 10
PADDLE_HEIGHT = 60
PADDLE_SPEED = 5

# Left paddle position
left_x = 20
left_y = HEIGHT // 2 - PADDLE_HEIGHT // 2

# Right paddle position
right_x = WIDTH - 20 - PADDLE_WIDTH
right_y = HEIGHT // 2 - PADDLE_HEIGHT // 2
```

Each paddle has an x and y position. The `//` operator is **integer division** -- it divides and rounds down to a whole number. `HEIGHT // 2 - PADDLE_HEIGHT // 2` puts the paddle right in the middle of the screen vertically.

Now draw the paddles inside the game loop, after `screen.fill(BLACK)`:

```python
pygame.draw.rect(screen, WHITE, (left_x, left_y, PADDLE_WIDTH, PADDLE_HEIGHT))
pygame.draw.rect(screen, WHITE, (right_x, right_y, PADDLE_WIDTH, PADDLE_HEIGHT))
```

Run it. Two white rectangles should appear on the left and right sides of the black window. They don't move yet -- that comes next.

### Drawing the Center Line

Real Pong has a dashed line down the middle. Let's add one. This is a nice touch. Add it after drawing the paddles:

```python
for i in range(0, HEIGHT, 20):
    pygame.draw.rect(screen, WHITE, (WIDTH // 2 - 1, i, 2, 10))
```

This loop starts at 0 and goes up to `HEIGHT`, jumping by 20 each time. At each step, it draws a small white rectangle. The result is a dashed line down the middle of the court.

---

## Section 2: Movement and the Ball

Now we make things move. This is where the game starts to feel real.

### Reading Keyboard Input

Pygame has a function called `pygame.key.get_pressed()` that returns the state of every key on the keyboard. It tells you which keys are being held down *right now*.

Add this inside the game loop, after the event handling but before the drawing code:

```python
keys = pygame.key.get_pressed()

if keys[pygame.K_w] and left_y > 0:
    left_y -= PADDLE_SPEED
if keys[pygame.K_s] and left_y < HEIGHT - PADDLE_HEIGHT:
    left_y += PADDLE_SPEED

if keys[pygame.K_UP] and right_y > 0:
    right_y -= PADDLE_SPEED
if keys[pygame.K_DOWN] and right_y < HEIGHT - PADDLE_HEIGHT:
    right_y += PADDLE_SPEED
```

Let's break this down:

- `keys[pygame.K_w]` is `True` if the `W` key is held down.
- `left_y -= PADDLE_SPEED` moves the paddle up (remember, smaller y means higher on screen).
- `left_y > 0` makes sure the paddle does not go above the top of the screen.
- `left_y < HEIGHT - PADDLE_HEIGHT` makes sure it does not go below the bottom.

The right paddle works the same way but uses the Up and Down arrow keys (`pygame.K_UP` and `pygame.K_DOWN`).

Run the program. Press `W` and `S` to move the left paddle. Press the Up and Down arrow keys to move the right paddle. Both paddles should slide up and down smoothly and stop at the edges of the screen.

### Adding the Ball

Now for the ball. Add these variables before the game loop, after the paddle positions:

```python
# Ball settings
BALL_SIZE = 10
ball_x = WIDTH // 2 - BALL_SIZE // 2
ball_y = HEIGHT // 2 - BALL_SIZE // 2
ball_speed_x = 3
ball_speed_y = 3
```

The ball has a position (`ball_x`, `ball_y`) and a speed in two directions (`ball_speed_x`, `ball_speed_y`). The speed tells us how many pixels the ball moves each frame.

Draw the ball inside the game loop, with the other drawing code:

```python
pygame.draw.rect(screen, WHITE, (ball_x, ball_y, BALL_SIZE, BALL_SIZE))
```

Yes, our "ball" is actually a small square. That is how the original Pong worked too.

### Making the Ball Move

This is the key idea: **each frame, we add the speed to the position**. Add this after the keyboard input code, before the drawing code:

```python
ball_x += ball_speed_x
ball_y += ball_speed_y
```

If `ball_speed_x` is 3, the ball moves 3 pixels to the right every frame. At 60 frames per second, that is 180 pixels per second. Smooth movement comes from making many small moves very quickly.

Run it. The ball should shoot across the screen and disappear off the edge. It does not bounce yet -- that is next.

### Bouncing Off the Top and Bottom Walls

When the ball hits the top or bottom edge, we want it to bounce. Bouncing means reversing the vertical direction. If the ball is going down (`ball_speed_y` is positive) and hits the bottom, it should start going up (`ball_speed_y` becomes negative).

Add this after the ball movement code:

```python
if ball_y <= 0 or ball_y >= HEIGHT - BALL_SIZE:
    ball_speed_y = -ball_speed_y
```

The `-` in front of `ball_speed_y` flips the sign. If it was `3`, it becomes `-3`. If it was `-3`, it becomes `3`. The ball reverses direction.

Run it. The ball should now bounce off the top and bottom walls. It still flies off the left and right sides though. We need paddle collisions for that.

---

## Section 3: Collisions, Scoring, and the Complete Game

This is the trickiest part of the chapter. Take your time with it.

### Collision Detection with Rectangles

**Collision detection** means figuring out when two objects overlap. In Pong, we need to know when the ball touches a paddle.

Pygame has a helpful tool for this: `pygame.Rect`. A **Rect** is a rectangle object that knows its position and size, and it has a method called `colliderect()` that checks if two rectangles overlap.

Here is what overlapping rectangles look like:

```
    NOT colliding:            COLLIDING:

    +----+                    +----+
    |    |    +----+          | +--|-+
    +----+    |    |          +-|--+ |
              +----+            +----+
```

Two rectangles collide when they overlap in *both* the horizontal and vertical directions. Checking this with math is fiddly, but `colliderect()` does it for us.

Add this collision code after the ball movement and wall bouncing code:

```python
left_paddle = pygame.Rect(left_x, left_y, PADDLE_WIDTH, PADDLE_HEIGHT)
right_paddle = pygame.Rect(right_x, right_y, PADDLE_WIDTH, PADDLE_HEIGHT)
ball_rect = pygame.Rect(ball_x, ball_y, BALL_SIZE, BALL_SIZE)

if ball_rect.colliderect(left_paddle) and ball_speed_x < 0:
    ball_speed_x = -ball_speed_x
if ball_rect.colliderect(right_paddle) and ball_speed_x > 0:
    ball_speed_x = -ball_speed_x
```

We create three `Rect` objects from our position variables, then check for collisions. When the ball hits the left paddle, we reverse its horizontal direction (it was going left, now it goes right). Same for the right paddle.

The extra checks `ball_speed_x < 0` and `ball_speed_x > 0` make sure we only bounce the ball when it is moving *toward* the paddle. Without this, the ball could get stuck inside a paddle and bounce back and forth forever.

> Collision detection is one of the trickiest parts of game programming. If it doesn't work right away, check your coordinates carefully. Print out the ball and paddle positions to see what is happening.

Run the game. The ball should now bounce off both paddles AND the top and bottom walls. You can play a basic rally! Try it with a friend -- one person on W/S, the other on the arrow keys.

### Scoring

Right now, when the ball gets past a paddle, it just flies off the screen forever. We need two things: a score system and a way to reset the ball.

Add score variables before the game loop:

```python
score_left = 0
score_right = 0
```

These two variables are our **game state** -- they remember what is happening in the game across frames. Every game has state: scores, lives, health, level, inventory. Game programming is really about managing state.

Now add scoring and ball reset code. Put this after the collision detection code:

```python
if ball_x < 0:
    score_right += 1
    ball_x = WIDTH // 2 - BALL_SIZE // 2
    ball_y = HEIGHT // 2 - BALL_SIZE // 2
    ball_speed_x = 3
    ball_speed_y = 3

if ball_x > WIDTH:
    score_left += 1
    ball_x = WIDTH // 2 - BALL_SIZE // 2
    ball_y = HEIGHT // 2 - BALL_SIZE // 2
    ball_speed_x = -3
    ball_speed_y = 3
```

When the ball goes past the left edge (`ball_x < 0`), the right player scores. When it goes past the right edge (`ball_x > WIDTH`), the left player scores. Either way, the ball resets to the center.

Notice that when the right player scores, the ball starts moving to the right (`ball_speed_x = 3`), and when the left player scores, the ball moves left (`ball_speed_x = -3`). The ball starts moving toward the player who just scored, giving the other player a moment to get ready.

### Displaying the Score

Scores are numbers, but we need to draw them as text on the screen. Pygame uses **fonts** for this. Add this before the game loop:

```python
font = pygame.font.Font(None, 48)
```

This creates a font object. `None` means "use the default font" and `48` is the size. Now add this to the drawing section, after `screen.fill(BLACK)` and before the other drawing code:

```python
left_text = font.render(str(score_left), True, WHITE)
right_text = font.render(str(score_right), True, WHITE)
screen.blit(left_text, (WIDTH // 4, 20))
screen.blit(right_text, (3 * WIDTH // 4, 20))
```

`font.render()` turns a string into an image. `str(score_left)` converts the number to a string. The `True` means we want smooth (anti-aliased) text. `screen.blit()` draws that image onto the screen at the given position.

**blit** is short for "block image transfer" -- a fancy way of saying "paste this image here." You will use `blit` a lot in pygame.

Run the game. You should see scores at the top of the screen. They go up when the ball gets past a paddle, and the ball resets to the center. You now have a fully working Pong game.

## Checkpoint: Complete Pong Code

Here is the full program. Read through it and make sure your code matches:

```python
import pygame

pygame.init()

WIDTH = 600
HEIGHT = 400
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pong")

clock = pygame.time.Clock()

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Paddle settings
PADDLE_WIDTH = 10
PADDLE_HEIGHT = 60
PADDLE_SPEED = 5

# Left paddle position
left_x = 20
left_y = HEIGHT // 2 - PADDLE_HEIGHT // 2

# Right paddle position
right_x = WIDTH - 20 - PADDLE_WIDTH
right_y = HEIGHT // 2 - PADDLE_HEIGHT // 2

# Ball settings
BALL_SIZE = 10
ball_x = WIDTH // 2 - BALL_SIZE // 2
ball_y = HEIGHT // 2 - BALL_SIZE // 2
ball_speed_x = 3
ball_speed_y = 3

# Score
score_left = 0
score_right = 0

# Font
font = pygame.font.Font(None, 48)

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Keyboard input
    keys = pygame.key.get_pressed()

    if keys[pygame.K_w] and left_y > 0:
        left_y -= PADDLE_SPEED
    if keys[pygame.K_s] and left_y < HEIGHT - PADDLE_HEIGHT:
        left_y += PADDLE_SPEED

    if keys[pygame.K_UP] and right_y > 0:
        right_y -= PADDLE_SPEED
    if keys[pygame.K_DOWN] and right_y < HEIGHT - PADDLE_HEIGHT:
        right_y += PADDLE_SPEED

    # Ball movement
    ball_x += ball_speed_x
    ball_y += ball_speed_y

    # Ball bouncing off top and bottom
    if ball_y <= 0 or ball_y >= HEIGHT - BALL_SIZE:
        ball_speed_y = -ball_speed_y

    # Ball bouncing off paddles
    left_paddle = pygame.Rect(left_x, left_y, PADDLE_WIDTH, PADDLE_HEIGHT)
    right_paddle = pygame.Rect(right_x, right_y, PADDLE_WIDTH, PADDLE_HEIGHT)
    ball_rect = pygame.Rect(ball_x, ball_y, BALL_SIZE, BALL_SIZE)

    if ball_rect.colliderect(left_paddle) and ball_speed_x < 0:
        ball_speed_x = -ball_speed_x
    if ball_rect.colliderect(right_paddle) and ball_speed_x > 0:
        ball_speed_x = -ball_speed_x

    # Scoring
    if ball_x < 0:
        score_right += 1
        ball_x = WIDTH // 2 - BALL_SIZE // 2
        ball_y = HEIGHT // 2 - BALL_SIZE // 2
        ball_speed_x = 3
        ball_speed_y = 3

    if ball_x > WIDTH:
        score_left += 1
        ball_x = WIDTH // 2 - BALL_SIZE // 2
        ball_y = HEIGHT // 2 - BALL_SIZE // 2
        ball_speed_x = -3
        ball_speed_y = 3

    # Drawing
    screen.fill(BLACK)

    # Score display
    left_text = font.render(str(score_left), True, WHITE)
    right_text = font.render(str(score_right), True, WHITE)
    screen.blit(left_text, (WIDTH // 4, 20))
    screen.blit(right_text, (3 * WIDTH // 4, 20))

    # Center line
    for i in range(0, HEIGHT, 20):
        pygame.draw.rect(screen, WHITE, (WIDTH // 2 - 1, i, 2, 10))

    # Paddles
    pygame.draw.rect(screen, WHITE, (left_x, left_y, PADDLE_WIDTH, PADDLE_HEIGHT))
    pygame.draw.rect(screen, WHITE, (right_x, right_y, PADDLE_WIDTH, PADDLE_HEIGHT))

    # Ball
    pygame.draw.rect(screen, WHITE, (ball_x, ball_y, BALL_SIZE, BALL_SIZE))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Type the whole thing out, run it, and play a few rounds. You just built a real game.

## What You Learned
- **Movement** works by changing an object's position a little bit each frame.
- `pygame.key.get_pressed()` tells you which keys are currently held down.
- **Collision detection** checks whether two objects overlap. `pygame.Rect` and `colliderect()` make this easy for rectangles.
- **Game state** is the collection of variables that describe what is happening in the game right now (scores, positions, speeds).
- `font.render()` turns text into an image, and `screen.blit()` draws that image on screen.
- **Bouncing** works by reversing a speed value: if the ball is going right at speed 3, flipping it to -3 sends it left.

## Quick Quiz

1. If `ball_speed_x` is `4` and we write `ball_speed_x = -ball_speed_x`, what is the new value?
2. Why do we check `ball_speed_x < 0` when detecting collision with the left paddle?
3. What does `pygame.key.get_pressed()` return? How is it different from the event system we used for the QUIT event?
4. What two things happen when a player scores a point in our Pong game?

## Experiment

1. Change `PADDLE_SPEED` to `10`. Now change it to `2`. How does the game feel different?
2. Change `ball_speed_x` and `ball_speed_y` to `5`. Is the game harder or easier? What about `1`?
3. Make the paddles taller by changing `PADDLE_HEIGHT` to `100`. Then try `30`. Which version is more fun?
4. Change the ball color to `(255, 100, 100)` -- a soft red. Change the paddle color to `(100, 255, 100)` -- green. Make it your own.

## Chapter Challenge

Add a **win condition**: when one player reaches 5 points, the game displays "LEFT WINS!" or "RIGHT WINS!" in big text in the center of the screen and stops the ball.

**Hint 1:** You need an `if` statement that checks whether `score_left >= 5` or `score_right >= 5`.

**Hint 2:** Create a variable called `game_over` that starts as `False`. Set it to `True` when someone reaches 5 points. When `game_over` is `True`, skip the ball movement and input code, and draw the winning message instead.

**Hint 3:** To draw the winning message, use `font.render()` the same way we drew the score. To center it, you can place it at roughly `(WIDTH // 2 - 80, HEIGHT // 2)`. Getting text perfectly centered takes some extra work, but close enough is fine for now.
