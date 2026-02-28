---
layout: default
title: "Chapter 11: Flappy Bird"
nav_order: 5
parent: "Part 2: Making Games with Pygame"
---

# Chapter 11: Game 3 -- Flappy Bird

## What You Will Learn
- How gravity works in games (velocity and acceleration)
- How to spawn and manage multiple objects using a list
- How to create pipes with gaps and scroll them across the screen
- How to detect collisions between the bird and pipes
- How to create a scrolling background
- How to put together a complete game with menus, scoring, and game over

## The Game That Took Over the World

Flappy Bird was a simple mobile game that became wildly popular in 2013. It proved that a great game doesn't need fancy graphics. The whole game boils down to one button: tap to flap. A bird falls due to gravity, and you tap the screen to make it jump up. Pipes scroll toward you, and you have to fly through the gaps. That is the entire game -- and millions of people could not stop playing it.

We are going to build our own version. By the end of this chapter, you will have a fully playable Flappy Bird clone.

## Planning the Game

Before we write a single line of code, let's think about what this game needs.

**The rules:**
- A bird falls due to gravity. Press `SPACE` to flap (jump up).
- Pipes scroll from right to left. Each pipe has a gap the bird must fly through.
- Your score increases each time you pass through a gap.
- Hit a pipe or the ground and it is game over.

Here is a rough layout of what the screen looks like during play:

```
+------------------------------------------+
|                                          |
|         ####                             |
|         ####  <-- top pipe               |
|         ####                             |
|                                          |
|    O>          <-- bird                  |
|                                          |
|         ####                             |
|         ####  <-- bottom pipe            |
|         ####                             |
|         ####                             |
|==========================================|
|/////////// ground //////////////////////|
+------------------------------------------+
```

The bird stays in roughly the same horizontal position. It is the pipes that move, not the bird (well, the bird moves up and down, but not left or right). This is a common trick in games -- the world moves past the player.

**What we need to build, step by step:**

```
1. Window and bird
2. Gravity (the bird falls)
3. Flapping (SPACE to jump)
4. Pipes that scroll across the screen
5. Collision detection
6. Scoring
7. Game states (menu, playing, game over)
8. Sprites and polish
```

Let's get started.

## Part 1: The Bird and Gravity

### Setting Up the Window

Flappy Bird uses a tall, narrow screen -- the original was a phone game, after all. Create a new file called `flappy.py`:

```python
import pygame

pygame.init()

WINDOW_WIDTH = 400
WINDOW_HEIGHT = 600
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()
FPS = 60
```

Nothing new here. We are using a 400 by 600 window, which is taller than it is wide.

Now add some colors and the bird's starting position:

```python
WHITE = (255, 255, 255)
SKY_BLUE = (135, 206, 235)
GREEN = (34, 139, 34)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)

bird_x = 80
bird_y = 300
bird_width = 34
bird_height = 24
```

The bird sits near the left side of the screen. It does not move left or right -- only up and down.

Add the game loop to draw the bird as a rectangle for now:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill(SKY_BLUE)
    bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
    pygame.draw.rect(screen, YELLOW, bird_rect)
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
```

Run it:

```
python3 flappy.py
```

You should see a yellow rectangle sitting in the middle of a sky-blue screen. Not very exciting yet, but that is our bird.

### Teaching the Bird to Fall: Gravity

Here is the big new concept for this chapter. In real life, gravity pulls things down, and they fall faster and faster the longer they fall. A dropped ball does not fall at a steady speed -- it **accelerates**.

We simulate this with two variables:

- **`bird_velocity`** -- how fast the bird is currently moving (positive means down, negative means up)
- **`gravity`** -- a small number we add to the velocity every frame

Every frame, we do two things:

```
bird_velocity += gravity      (velocity increases -- bird falls faster)
bird_y += bird_velocity       (bird moves by its current velocity)
```

Here is what happens over several frames if gravity is `0.5` and the bird starts with a velocity of `0`:

```
Frame    Velocity    bird_y    What's happening
-----    --------    ------    ----------------
  1        0.5        300.5    Just started falling
  2        1.0        301.5    Falling a bit faster
  3        1.5        303.0    Faster still
  4        2.0        305.0    Picking up speed
  5        2.5        307.5    Getting quick now
  6        3.0        310.5    Falling fast!
```

See how the bird moves a little bit in the first few frames, then more and more? That is acceleration. This is how real physics engines work, just more complex.

Add the gravity variables before the game loop:

```python
bird_velocity = 0
gravity = 0.5
```

Then update the bird's position inside the game loop, just before drawing:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    bird_velocity += gravity
    bird_y += bird_velocity

    screen.fill(SKY_BLUE)
    bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
    pygame.draw.rect(screen, YELLOW, bird_rect)
    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
```

Run it. The bird should fall off the bottom of the screen, slowly at first, then faster and faster. That is gravity working.

### Flapping

When the player presses `SPACE`, we set `bird_velocity` to a negative number. Negative velocity means the bird moves up. Gravity will immediately start pulling it back down, so you get that nice arc -- jump up, slow down, fall back.

Add a constant for the jump strength before the game loop:

```python
jump_strength = -8
```

The negative sign is important -- remember, negative means "up" in our coordinate system.

Now handle the `SPACE` key inside the event loop:

```python
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                bird_velocity = jump_strength
```

Run it again. Press `SPACE` repeatedly to keep the bird in the air. It should feel like the bird is flapping. If you stop pressing, it falls.

> Getting the gravity to feel right takes tweaking. Try changing the `gravity` and `jump_strength` values to see what feels good. Game developers call this "game feel" -- it is one of the most important parts of making a game fun, and it is more art than science.

---

## Part 2: Pipes

### How Pipes Work

Each obstacle is a **pair** of pipes: one hanging from the top and one rising from the bottom, with a gap between them. The bird must fly through the gap.

```
       |  |
       |  |  <-- top pipe
       |  |
       +--+
              <-- gap (bird flies here)
       +--+
       |  |
       |  |  <-- bottom pipe
       |  |
       |  |
```

We store each pipe pair as a **dictionary** with three values:

- `"x"` -- the horizontal position of the pipe (it starts off screen on the right and scrolls left)
- `"gap_y"` -- the vertical center of the gap
- `"passed"` -- whether the bird has already passed this pipe (for scoring)

Add these constants before the game loop:

```python
PIPE_WIDTH = 60
PIPE_GAP = 150
PIPE_SPEED = 3
PIPE_SPAWN_TIME = 1500
```

`PIPE_GAP` is the height of the opening the bird flies through. `PIPE_SPAWN_TIME` is how many milliseconds between new pipes appearing (1500 ms = 1.5 seconds).

Now add a list to hold the pipes and a variable to track when to spawn the next one:

```python
pipes = []
last_pipe_time = pygame.time.get_ticks()
```

### Spawning Pipes

We need a function that creates a new pipe pair. The gap position should be random so the game is different every time. Add this above the game loop:

```python
import random

def spawn_pipe():
    gap_y = random.randint(150, WINDOW_HEIGHT - 150)
    pipe = {"x": WINDOW_WIDTH, "gap_y": gap_y, "passed": False}
    return pipe
```

The `gap_y` is kept between 150 and 450 (the window height minus 150). This stops the gap from being right at the top or bottom edge, which would be unfair.

Add the `import random` line at the top of your file, next to `import pygame`.

### Moving and Drawing Pipes

Inside the game loop, after updating the bird, add this code to handle pipes:

```python
    current_time = pygame.time.get_ticks()
    if current_time - last_pipe_time > PIPE_SPAWN_TIME:
        pipes.append(spawn_pipe())
        last_pipe_time = current_time
```

This checks the clock every frame. If enough time has passed since the last pipe, it spawns a new one.

Now move the pipes to the left and remove any that have scrolled off screen:

```python
    for pipe in pipes:
        pipe["x"] -= PIPE_SPEED

    pipes = [p for p in pipes if p["x"] + PIPE_WIDTH > 0]
```

That second line is a **list comprehension** -- a short way to filter a list. It keeps only the pipes whose right edge is still on screen. You could also write this as a regular loop:

```python
    remaining = []
    for p in pipes:
        if p["x"] + PIPE_WIDTH > 0:
            remaining.append(p)
    pipes = remaining
```

Both do the same thing. Use whichever makes more sense to you.

Now draw the pipes. For each pipe pair, we draw two rectangles -- the top pipe and the bottom pipe:

```python
    for pipe in pipes:
        top_height = pipe["gap_y"] - PIPE_GAP // 2
        bottom_y = pipe["gap_y"] + PIPE_GAP // 2
        bottom_height = WINDOW_HEIGHT - bottom_y

        top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
        bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)

        pygame.draw.rect(screen, GREEN, top_rect)
        pygame.draw.rect(screen, GREEN, bottom_rect)
```

Make sure this drawing code comes after `screen.fill(SKY_BLUE)` and before `pygame.display.flip()`. Draw the pipes before the bird so the bird appears on top.

Run it. You should see green pipes scrolling in from the right, each with a gap at a random height. The bird can flap through them -- but nothing happens when you hit one. Let's fix that.

---

## Part 3: Collisions, Scoring, and Game States

### Collision Detection

We need to check if the bird's rectangle overlaps with any pipe rectangle or if it hits the ground. Add this after updating the bird and pipe positions, but before drawing:

```python
    bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)

    hit_pipe = False
    for pipe in pipes:
        top_height = pipe["gap_y"] - PIPE_GAP // 2
        bottom_y = pipe["gap_y"] + PIPE_GAP // 2
        bottom_height = WINDOW_HEIGHT - bottom_y

        top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
        bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)

        if bird_rect.colliderect(top_rect) or bird_rect.colliderect(bottom_rect):
            hit_pipe = True

    hit_ground = bird_y + bird_height >= WINDOW_HEIGHT - 20
    hit_ceiling = bird_y <= 0

    if hit_pipe or hit_ground:
        running = False
```

For now, hitting a pipe or the ground ends the game immediately. We will improve this with proper game states soon.

### Scoring

The score goes up by one each time the bird passes a pipe. We check if the bird's horizontal position has moved past a pipe that has not been counted yet:

```python
score = 0
```

Add this before the game loop. Then inside the loop, after moving the pipes:

```python
    for pipe in pipes:
        if not pipe["passed"] and pipe["x"] + PIPE_WIDTH < bird_x:
            pipe["passed"] = True
            score += 1
```

To display the score, add this after all the drawing but before `pygame.display.flip()`:

```python
    font = pygame.font.Font(None, 48)
    score_surface = font.render(str(score), True, WHITE)
    screen.blit(score_surface, (WINDOW_WIDTH // 2 - 10, 30))
```

> Creating the font object inside the loop works but is slightly wasteful. If you like, move the `font = pygame.font.Font(None, 48)` line to before the game loop. The game will still work either way.

### Adding Game States

Right now the game starts immediately and crashing ends the program. A real game has a menu screen and a game over screen. We covered game states in Chapter 9, so this should feel familiar.

Replace the simple `running` variable with a state system. Here are the states:

```
   +---------+     SPACE      +---------+    crash     +-----------+
   |  MENU   | ------------> | PLAYING  | ----------> | GAME_OVER  |
   +---------+               +---------+              +-----------+
       ^                                                    |
       |                      SPACE                         |
       +----------------------------------------------------+
```

Add a state variable before the game loop:

```python
state = "menu"
score = 0
```

Now we need to restructure the game loop so it behaves differently depending on the state. Here is the full updated game loop structure:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if state == "menu":
                    state = "playing"
                    bird_y = 300
                    bird_velocity = 0
                    pipes = []
                    score = 0
                    last_pipe_time = pygame.time.get_ticks()
                elif state == "playing":
                    bird_velocity = jump_strength
                elif state == "game_over":
                    state = "menu"

    screen.fill(SKY_BLUE)

    if state == "menu":
        font_big = pygame.font.Font(None, 56)
        font_small = pygame.font.Font(None, 32)
        title = font_big.render("Flappy Bird", True, WHITE)
        prompt = font_small.render("Press SPACE to start", True, WHITE)
        screen.blit(title, (WINDOW_WIDTH // 2 - title.get_width() // 2, 200))
        screen.blit(prompt, (WINDOW_WIDTH // 2 - prompt.get_width() // 2, 280))

    elif state == "playing":
        bird_velocity += gravity
        bird_y += bird_velocity

        current_time = pygame.time.get_ticks()
        if current_time - last_pipe_time > PIPE_SPAWN_TIME:
            pipes.append(spawn_pipe())
            last_pipe_time = current_time

        for pipe in pipes:
            pipe["x"] -= PIPE_SPEED

        pipes = [p for p in pipes if p["x"] + PIPE_WIDTH > 0]

        for pipe in pipes:
            if not pipe["passed"] and pipe["x"] + PIPE_WIDTH < bird_x:
                pipe["passed"] = True
                score += 1

        bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
        hit_pipe = False
        for pipe in pipes:
            top_height = pipe["gap_y"] - PIPE_GAP // 2
            bottom_y = pipe["gap_y"] + PIPE_GAP // 2
            bottom_height = WINDOW_HEIGHT - bottom_y
            top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
            bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)
            if bird_rect.colliderect(top_rect) or bird_rect.colliderect(bottom_rect):
                hit_pipe = True

        if hit_pipe or bird_y + bird_height >= WINDOW_HEIGHT - 20:
            state = "game_over"
        if bird_y < 0:
            bird_y = 0
            bird_velocity = 0

        for pipe in pipes:
            top_height = pipe["gap_y"] - PIPE_GAP // 2
            bottom_y = pipe["gap_y"] + PIPE_GAP // 2
            bottom_height = WINDOW_HEIGHT - bottom_y
            top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
            bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)
            pygame.draw.rect(screen, GREEN, top_rect)
            pygame.draw.rect(screen, GREEN, bottom_rect)

        pygame.draw.rect(screen, YELLOW, bird_rect)

        font = pygame.font.Font(None, 48)
        score_surface = font.render(str(score), True, WHITE)
        screen.blit(score_surface, (WINDOW_WIDTH // 2 - 10, 30))

    elif state == "game_over":
        font_big = pygame.font.Font(None, 56)
        font_small = pygame.font.Font(None, 32)
        title = font_big.render("Game Over", True, WHITE)
        score_text = font_small.render(f"Score: {score}", True, WHITE)
        prompt = font_small.render("Press SPACE to restart", True, WHITE)
        screen.blit(title, (WINDOW_WIDTH // 2 - title.get_width() // 2, 200))
        screen.blit(score_text, (WINDOW_WIDTH // 2 - score_text.get_width() // 2, 270))
        screen.blit(prompt, (WINDOW_WIDTH // 2 - prompt.get_width() // 2, 320))

    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
```

That is a lot of code, but every piece should be familiar from what we have done so far in this chapter or in earlier chapters. The key change is wrapping everything in `if`/`elif` blocks based on the current state.

Run it and play. You should have a working Flappy Bird with a menu, gameplay, scoring, and a game over screen.

---

## Part 4: Polish

### Using Sprites Instead of Rectangles

In Chapter 10, you learned how to load and display images. Let's replace the yellow rectangle with an actual bird image. If you have a bird sprite image (a small PNG file, around 34 by 24 pixels works well), load it like this:

```python
bird_image = pygame.image.load("images/bird.png").convert_alpha()
```

If you do not have an image, you can draw your own in any image editor and save it as `bird.png`, or keep using the rectangle. The game works either way.

To make the bird tilt based on its velocity (nose up when flapping, nose down when falling), use `pygame.transform.rotate`:

```python
    if state == "playing":
        angle = -bird_velocity * 3
        rotated_bird = pygame.transform.rotate(bird_image, angle)
        bird_draw_rect = rotated_bird.get_rect(center=bird_rect.center)
        screen.blit(rotated_bird, bird_draw_rect)
```

We multiply the velocity by `3` and negate it so the rotation looks right: negative velocity (going up) tilts the nose up, positive velocity (falling) tilts the nose down. The `get_rect(center=...)` call makes sure the rotated image stays centered on the bird's actual position.

Replace the `pygame.draw.rect(screen, YELLOW, bird_rect)` line in the playing state with the rotation code above. If you are still using a rectangle, leave it as is.

### Scrolling Background

A scrolling background makes the game feel more alive. The idea is simple: draw the background image twice, side by side, and scroll them both to the left. When the first copy moves off screen, wrap it back to the right.

```python
bg_x = 0
BG_SPEED = 1
```

Add this before the game loop. If you have a background image:

```python
bg_image = pygame.image.load("images/background.png").convert()
bg_image = pygame.transform.scale(bg_image, (WINDOW_WIDTH, WINDOW_HEIGHT))
```

Inside the playing state, instead of `screen.fill(SKY_BLUE)`, do:

```python
    bg_x -= BG_SPEED
    if bg_x <= -WINDOW_WIDTH:
        bg_x = 0
    screen.blit(bg_image, (bg_x, 0))
    screen.blit(bg_image, (bg_x + WINDOW_WIDTH, 0))
```

This draws two copies of the background next to each other, both moving left. When the first copy scrolls entirely off the left edge, the position resets and the loop is seamless.

> If you add the scrolling background, move the `screen.fill(SKY_BLUE)` line inside only the `menu` and `game_over` state blocks, so it does not paint over your background image during gameplay.

If you do not have a background image, you can skip this step or use `screen.fill(SKY_BLUE)` and draw some simple ground at the bottom:

```python
    ground_rect = pygame.Rect(0, WINDOW_HEIGHT - 20, WINDOW_WIDTH, 20)
    pygame.draw.rect(screen, (139, 119, 42), ground_rect)
```

### Optional: Increasing Difficulty

Want the game to get harder over time? You could make pipes spawn more frequently as the score increases:

```python
    spawn_delay = max(800, PIPE_SPAWN_TIME - score * 50)
    if current_time - last_pipe_time > spawn_delay:
        pipes.append(spawn_pipe())
        last_pipe_time = current_time
```

The `max(800, ...)` makes sure the delay never goes below 800 milliseconds, which would be extremely difficult but not impossible.

---

## Checkpoint: Complete Code Listing

Before we look at the complete code, there is one improvement. We have been calculating pipe rectangles in two places -- once for collision and once for drawing. Let's pull that into a helper function called `get_pipe_rects`:

```python
def get_pipe_rects(pipe):
    top_height = pipe["gap_y"] - PIPE_GAP // 2
    bottom_y = pipe["gap_y"] + PIPE_GAP // 2
    bottom_height = WINDOW_HEIGHT - bottom_y
    top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
    bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)
    return top_rect, bottom_rect
```

This function takes a pipe dictionary, builds the two `Rect` objects, and returns them as a pair. Now instead of repeating the rectangle math in multiple places, we just call `get_pipe_rects(pipe)` wherever we need the rectangles. You will see it used in the full listing below.

Your finished `flappy.py` should look like this:

```python
import pygame
import random

pygame.init()

WINDOW_WIDTH = 400
WINDOW_HEIGHT = 600
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()
FPS = 60

WHITE = (255, 255, 255)
SKY_BLUE = (135, 206, 235)
GREEN = (34, 139, 34)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)
BROWN = (139, 119, 42)

bird_x = 80
bird_y = 300
bird_width = 34
bird_height = 24
bird_velocity = 0
gravity = 0.5
jump_strength = -8

PIPE_WIDTH = 60
PIPE_GAP = 150
PIPE_SPEED = 3
PIPE_SPAWN_TIME = 1500

pipes = []
last_pipe_time = pygame.time.get_ticks()
score = 0
state = "menu"

font_big = pygame.font.Font(None, 56)
font_small = pygame.font.Font(None, 32)
font_score = pygame.font.Font(None, 48)


def spawn_pipe():
    gap_y = random.randint(150, WINDOW_HEIGHT - 150)
    pipe = {"x": WINDOW_WIDTH, "gap_y": gap_y, "passed": False}
    return pipe


def get_pipe_rects(pipe):
    top_height = pipe["gap_y"] - PIPE_GAP // 2
    bottom_y = pipe["gap_y"] + PIPE_GAP // 2
    bottom_height = WINDOW_HEIGHT - bottom_y
    top_rect = pygame.Rect(pipe["x"], 0, PIPE_WIDTH, top_height)
    bottom_rect = pygame.Rect(pipe["x"], bottom_y, PIPE_WIDTH, bottom_height)
    return top_rect, bottom_rect


running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if state == "menu":
                    state = "playing"
                    bird_y = 300
                    bird_velocity = 0
                    pipes = []
                    score = 0
                    last_pipe_time = pygame.time.get_ticks()
                elif state == "playing":
                    bird_velocity = jump_strength
                elif state == "game_over":
                    state = "menu"

    screen.fill(SKY_BLUE)

    if state == "menu":
        title = font_big.render("Flappy Bird", True, WHITE)
        prompt = font_small.render("Press SPACE to start", True, WHITE)
        screen.blit(title, (WINDOW_WIDTH // 2 - title.get_width() // 2, 200))
        screen.blit(prompt, (WINDOW_WIDTH // 2 - prompt.get_width() // 2, 280))

    elif state == "playing":
        bird_velocity += gravity
        bird_y += bird_velocity

        current_time = pygame.time.get_ticks()
        if current_time - last_pipe_time > PIPE_SPAWN_TIME:
            pipes.append(spawn_pipe())
            last_pipe_time = current_time

        for pipe in pipes:
            pipe["x"] -= PIPE_SPEED

        pipes = [p for p in pipes if p["x"] + PIPE_WIDTH > 0]

        for pipe in pipes:
            if not pipe["passed"] and pipe["x"] + PIPE_WIDTH < bird_x:
                pipe["passed"] = True
                score += 1

        bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
        hit_pipe = False
        for pipe in pipes:
            top_rect, bottom_rect = get_pipe_rects(pipe)
            if bird_rect.colliderect(top_rect) or bird_rect.colliderect(bottom_rect):
                hit_pipe = True

        if hit_pipe or bird_y + bird_height >= WINDOW_HEIGHT - 20:
            state = "game_over"
        if bird_y < 0:
            bird_y = 0
            bird_velocity = 0

        for pipe in pipes:
            top_rect, bottom_rect = get_pipe_rects(pipe)
            pygame.draw.rect(screen, GREEN, top_rect)
            pygame.draw.rect(screen, GREEN, bottom_rect)

        pygame.draw.rect(screen, YELLOW, bird_rect)

        ground_rect = pygame.Rect(0, WINDOW_HEIGHT - 20, WINDOW_WIDTH, 20)
        pygame.draw.rect(screen, BROWN, ground_rect)

        score_surface = font_score.render(str(score), True, WHITE)
        screen.blit(score_surface, (WINDOW_WIDTH // 2 - 10, 30))

    elif state == "game_over":
        title = font_big.render("Game Over", True, WHITE)
        score_text = font_small.render(f"Score: {score}", True, WHITE)
        prompt = font_small.render("Press SPACE to restart", True, WHITE)
        screen.blit(title, (WINDOW_WIDTH // 2 - title.get_width() // 2, 200))
        screen.blit(score_text, (WINDOW_WIDTH // 2 - score_text.get_width() // 2, 270))
        screen.blit(prompt, (WINDOW_WIDTH // 2 - prompt.get_width() // 2, 320))

    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
```

Run the complete listing and make sure everything works: menu screen, flapping, pipes, scoring, collision, and game over.

> Did you know? The original Flappy Bird was made by a single developer named Dong Nguyen from Vietnam. At its peak, it was earning $50,000 per day in ad revenue. Nguyen took the game down because he felt it was too addictive, but it inspired thousands of clones -- including yours.

---

## What You Learned

- **Gravity** in games is simulated by adding a small value to velocity each frame. Velocity then moves the object. This creates acceleration -- the longer something falls, the faster it goes.
- **Velocity** is the speed and direction an object is moving. Positive velocity moves down, negative moves up.
- **Spawning objects on a timer** using `pygame.time.get_ticks()` lets you create new obstacles at regular intervals.
- **Managing a list of objects** (the pipes list) is how games handle multiple enemies, bullets, coins, or anything else you need more than one of.
- **Removing off-screen objects** from the list prevents the game from slowing down over time.
- **Game feel** is the result of tweaking numbers like gravity, jump strength, pipe speed, and gap size. Small changes make a big difference.

## Quick Quiz

1. What happens if you set `gravity` to `0`? What about setting it to a negative number?

2. Why do we store pipes in a list instead of using separate variables like `pipe1_x`, `pipe2_x`, `pipe3_x`?

3. In our code, what does `pipe["passed"]` keep track of, and why do we need it?

4. If the bird's velocity is `4` and gravity is `0.5`, what will the velocity be after 3 more frames (assuming no flap)?

## Experiment

1. Change `gravity` to `0.3` and `jump_strength` to `-6`. How does the game feel different? Try `gravity = 0.8` and `jump_strength = -10`. Which combination feels best to you?

2. Change `PIPE_GAP` to `200` (easier) or `100` (harder). What gap size makes the game fun but challenging?

3. Add a high score that persists across game-over restarts within the same session. You will need a `high_score` variable that updates when `score > high_score` at game over, and display it on the game over screen.

## Chapter Challenge

**Add a ground collision effect and a "best score" display.**

When the bird hits the ground or a pipe, make it dramatic: change the background color to red briefly (for a few frames), then show the game over screen. Also, track the best score across all plays in the session and display it on both the game over screen and the menu screen.

**Hints (try before reading!):**

1. Add a `best_score` variable set to `0` before the game loop. When the game enters the `"game_over"` state, check if `score > best_score` and update it.

2. For the red flash effect, add a `flash_timer` variable. When a crash happens, set it to a number (like 10). Each frame in the game over state, count it down. While it is above 0, fill the screen with red instead of the normal background.

3. Display `best_score` on the menu and game over screens the same way you display the regular score -- render it with the font and blit it to the screen.
