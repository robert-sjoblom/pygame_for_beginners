---
layout: default
title: "Chapter 9: Snake"
nav_order: 3
parent: "Part 2: Making Games with Pygame"
---

# Chapter 9: Game 2 -- Snake

## What You Will Learn
- How to use a list to represent a snake that moves on a grid
- Grid-based movement: translating grid positions to pixel positions
- How to manage **game states** (menu, playing, game over)
- Collision detection with walls and the snake's own body

## A Classic Game

Snake has been around since the 1970s. Nokia made it famous on mobile phones in the late 1990s. Millions of people played it on tiny screens while waiting for the bus. The rules are simple: a snake moves around, eats food, and grows longer. If it hits a wall or its own body, the game is over.

You already built Pong in the last few chapters. Snake is a bigger project, but you know enough Python and pygame to handle it. We will build it step by step.

## Planning Before Coding

Before writing any code, let's think about what Snake actually is. This is called **planning**, and professional programmers do it every day.

Here is our plan in plain English (sometimes called **pseudocode** -- it looks like code but it is just organized thinking):

```
1. The snake is a LIST of body segments
2. Each segment sits on a GRID square
3. Every frame, the snake moves one square in its current direction
4. If the snake's head lands on food, the snake grows by one segment
5. If the snake's head hits a wall or its own body, the game ends
6. Food appears at a random empty grid square
```

### The Grid Concept

Our game window will be divided into a grid of equal-sized squares, like graph paper. Instead of thinking in pixels, we think in grid squares.

```
  Grid                         Pixels (cell_size = 30)

  0   1   2   3   4            0   30  60  90  120
  +---+---+---+---+            +---+---+---+---+
0 |   |   |   |   |          0 |   |   |   |   |
  +---+---+---+---+            +---+---+---+---+
1 |   | S |   |   |         30 |   | S |   |   |
  +---+---+---+---+            +---+---+---+---+
2 |   |   |   |   |         60 |   |   |   |   |
  +---+---+---+---+            +---+---+---+---+
3 |   |   | F |   |         90 |   |   | F |   |
  +---+---+---+---+            +---+---+---+---+

S = snake segment at grid (1, 1)
    pixel position = (1 * 30, 1 * 30) = (30, 30)

F = food at grid (2, 3)
    pixel position = (2 * 30, 3 * 30) = (60, 90)
```

The math is straightforward:

```
pixel_x = grid_x * cell_size
pixel_y = grid_y * cell_size
```

If each cell is 30 pixels wide, then grid position `(3, 2)` becomes pixel position `(90, 60)`. That is all there is to it.

## Setting Up the Window and Grid

Create a new file called `snake.py`. Let's start with the window and some constants.

```python
import pygame
import random

pygame.init()

CELL_SIZE = 30
GRID_WIDTH = 20
GRID_HEIGHT = 15
WINDOW_WIDTH = GRID_WIDTH * CELL_SIZE
WINDOW_HEIGHT = GRID_HEIGHT * CELL_SIZE

screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Snake")
clock = pygame.time.Clock()
```

We use UPPERCASE names for these values because they are **constants** -- values that never change while the program runs. This is a common convention in Python. It tells anyone reading the code: "This value is set once and stays the same."

Our grid is 20 cells wide and 15 cells tall. Each cell is 30 pixels, so the window is 600 by 450 pixels.

### Drawing the Grid (Optional)

It helps to see the grid while we build the game. Add this function:

```python
def draw_grid():
    for x in range(0, WINDOW_WIDTH, CELL_SIZE):
        pygame.draw.line(screen, (40, 40, 40), (x, 0), (x, WINDOW_HEIGHT))
    for y in range(0, WINDOW_HEIGHT, CELL_SIZE):
        pygame.draw.line(screen, (40, 40, 40), (0, y), (WINDOW_WIDTH, y))
```

This draws faint grey lines across the screen. You can remove it later once the game works.

---

## The Snake as a List

Remember lists from Chapter 6? The snake's body is a perfect use for a list. Each item in the list is an `[x, y]` pair representing one segment's grid position.

```python
snake = [[5, 7], [4, 7], [3, 7]]
```

This creates a snake three segments long. The **first** item in the list is the head. The rest are the body, stretching out behind it.

```
    0   1   2   3   4   5   6
    +---+---+---+---+---+---+---+
 7  |   |   |   | 2 | 1 | H |   |
    +---+---+---+---+---+---+---+

  H = head = snake[0] = [5, 7]
  1 = body = snake[1] = [4, 7]
  2 = tail = snake[2] = [3, 7]
```

We also need a direction for the snake to move in. Let's store that as another `[x, y]` pair:

```python
direction = [1, 0]
```

This means "move 1 cell to the right, 0 cells down." Here are all four directions:

```
Right: [1, 0]
Left:  [-1, 0]
Down:  [0, 1]
Up:    [0, -1]
```

### Drawing the Snake

Let's write a function to draw the snake on screen:

```python
GREEN = (0, 200, 0)
DARK_GREEN = (0, 150, 0)

def draw_snake(snake):
    for i, segment in enumerate(snake):
        x = segment[0] * CELL_SIZE
        y = segment[1] * CELL_SIZE
        rect = pygame.Rect(x, y, CELL_SIZE, CELL_SIZE)
        if i == 0:
            pygame.draw.rect(screen, GREEN, rect)
        else:
            pygame.draw.rect(screen, DARK_GREEN, rect)
```

The head is bright green, and the body is darker green. The `enumerate()` function gives us both the index number `i` and the segment, so we know which one is the head (index 0).

## Moving the Snake

This is the trickiest part of the whole game. Read this section carefully.

To move the snake, we do two things:
1. Add a new segment at the front (where the head is going)
2. Remove the last segment (the tail)

Watch what happens to the list:

```
Before moving right:

  snake = [[5,7], [4,7], [3,7]]

              [3,7]  [4,7]  [5,7]
                2      1      H     --> moving right

Step 1: Calculate new head position
  new_head = [5+1, 7+0] = [6, 7]

Step 2: Insert new head at the front of the list
  snake = [[6,7], [5,7], [4,7], [3,7]]

Step 3: Remove the last item (the tail)
  snake = [[6,7], [5,7], [4,7]]

After moving right:

                     [4,7]  [5,7]  [6,7]
                       2      1      H     --> still moving right
```

The snake appears to slide forward by one cell. The old tail disappears and a new head appears. Here is the code:

```python
def move_snake(snake, direction):
    head = snake[0]
    new_head = [head[0] + direction[0], head[1] + direction[1]]
    snake.insert(0, new_head)
    snake.pop()
```

`insert(0, new_head)` puts the new head at the start of the list. `pop()` removes the last item. Two lines of code, and the snake moves.

> If the snake moves strangely, add a `print(snake)` inside `move_snake()` to check what the body list looks like. Printing values to figure out what is going wrong is called **debugging**, and every programmer does it constantly.

---

## Handling Keyboard Input

The player uses the arrow keys to change the snake's direction. But there is one important rule: **the snake cannot reverse direction**. If the snake is going right, pressing left would make it run into its own body immediately. We must prevent that.

```python
def handle_input(direction):
    keys = pygame.key.get_pressed()

    if keys[pygame.K_UP] and direction != [0, 1]:
        return [0, -1]
    if keys[pygame.K_DOWN] and direction != [0, -1]:
        return [0, 1]
    if keys[pygame.K_LEFT] and direction != [1, 0]:
        return [-1, 0]
    if keys[pygame.K_RIGHT] and direction != [-1, 0]:
        return [1, 0]

    return direction
```

Look at the conditions: if the player presses Up, we only allow it if the snake is not currently going Down (`[0, 1]`). The same logic applies to all four directions.

In Python, comparing two lists with `==` or `!=` checks whether they contain the same values. So `[1, 0] == [1, 0]` is `True`.

## Spawning Food

Food appears at a random grid position. But it must not appear on the snake -- that would be unfair.

```python
RED = (200, 0, 0)

def spawn_food(snake):
    while True:
        food = [random.randint(0, GRID_WIDTH - 1),
                random.randint(0, GRID_HEIGHT - 1)]
        if food not in snake:
            return food
```

This uses a `while True` loop that keeps picking random positions until it finds one that the snake is not occupying. Then it returns that position.

To draw the food:

```python
def draw_food(food):
    x = food[0] * CELL_SIZE
    y = food[1] * CELL_SIZE
    rect = pygame.Rect(x, y, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(screen, RED, rect)
```

## Eating, Collisions, and Scoring

### Eating Food

When the snake's head is on the same grid square as the food, the snake eats it. The trick to making the snake grow is simple: **skip removing the tail for that frame**. Instead of calling `pop()`, we just leave the list alone. The snake is now one segment longer.

Go back to your `move_snake` function and replace it with this updated version:

```python
def move_snake(snake, direction, food):
    head = snake[0]
    new_head = [head[0] + direction[0], head[1] + direction[1]]
    snake.insert(0, new_head)

    if new_head == food:
        return True  # ate food, snake grows (no pop)
    else:
        snake.pop()  # normal move, remove tail
        return False
```

We changed `move_snake` to also take `food` and return `True` or `False` depending on whether the snake ate something.

### Collision with Walls

The snake hits a wall if its head goes outside the grid:

```python
def check_wall_collision(snake):
    head = snake[0]
    if head[0] < 0 or head[0] >= GRID_WIDTH:
        return True
    if head[1] < 0 or head[1] >= GRID_HEIGHT:
        return True
    return False
```

### Collision with Self

The snake hits itself if its head is at the same position as any other body segment:

```python
def check_self_collision(snake):
    head = snake[0]
    body = snake[1:]
    return head in body
```

`snake[1:]` is a **slice** -- it gives us every item in the list except the first one. So we check if the head's position matches any body segment.

---

## Game States

Up until now, our games just started and ran. But Snake needs three different screens:

1. A **start screen** that waits for the player to begin
2. The **playing** screen where the game runs
3. A **game over** screen that shows the final score

This idea is called **game state**. The game is always in exactly one state at a time. We store which state we are in using a simple variable:

```python
game_state = "menu"
```

Here is how the states connect:

```
  +--------+   press SPACE   +-----------+
  |  menu  | --------------> |  playing  |
  +--------+                 +-----------+
                                |     |
                                |     | hit wall or
                                |     | hit self
                                |     v
                                |  +-----------+
                                |  | game_over |
                                |  +-----------+
                                |     |
                                +-----+
                             press SPACE
                             (restart)
```

The game checks which state it is in and runs different code for each one. This pattern is used in almost every game, from simple phone games to big studio releases.

## Putting It All Together

Now we combine everything into one complete program. This is the full game. Type it into your `snake.py` file, replacing what you had before.

Notice that we left out the `draw_grid` function -- it was only needed while building the game. You can delete it from your file.

```python
import pygame
import random

pygame.init()

# Constants
CELL_SIZE = 30
GRID_WIDTH = 20
GRID_HEIGHT = 15
WINDOW_WIDTH = GRID_WIDTH * CELL_SIZE
WINDOW_HEIGHT = GRID_HEIGHT * CELL_SIZE
FPS = 10

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GREEN = (0, 200, 0)
DARK_GREEN = (0, 150, 0)
RED = (200, 0, 0)
GRAY = (100, 100, 100)

screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Snake")
clock = pygame.time.Clock()
font = pygame.font.Font(None, 36)
big_font = pygame.font.Font(None, 60)
```

This sets up everything we need. Notice `FPS = 10` -- Snake runs slower than Pong. At 10 frames per second, the snake moves 10 grid squares per second. This feels right for a grid-based game.

Next, add all the functions:

```python
def draw_snake(snake):
    for i, segment in enumerate(snake):
        x = segment[0] * CELL_SIZE
        y = segment[1] * CELL_SIZE
        rect = pygame.Rect(x, y, CELL_SIZE, CELL_SIZE)
        if i == 0:
            pygame.draw.rect(screen, GREEN, rect)
        else:
            pygame.draw.rect(screen, DARK_GREEN, rect)


def draw_food(food):
    x = food[0] * CELL_SIZE
    y = food[1] * CELL_SIZE
    rect = pygame.Rect(x, y, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(screen, RED, rect)
```

```python
def move_snake(snake, direction, food):
    head = snake[0]
    new_head = [head[0] + direction[0], head[1] + direction[1]]
    snake.insert(0, new_head)

    if new_head == food:
        return True
    else:
        snake.pop()
        return False


def handle_input(direction):
    keys = pygame.key.get_pressed()

    if keys[pygame.K_UP] and direction != [0, 1]:
        return [0, -1]
    if keys[pygame.K_DOWN] and direction != [0, -1]:
        return [0, 1]
    if keys[pygame.K_LEFT] and direction != [1, 0]:
        return [-1, 0]
    if keys[pygame.K_RIGHT] and direction != [-1, 0]:
        return [1, 0]

    return direction
```

```python
def spawn_food(snake):
    while True:
        food = [random.randint(0, GRID_WIDTH - 1),
                random.randint(0, GRID_HEIGHT - 1)]
        if food not in snake:
            return food


def check_wall_collision(snake):
    head = snake[0]
    if head[0] < 0 or head[0] >= GRID_WIDTH:
        return True
    if head[1] < 0 or head[1] >= GRID_HEIGHT:
        return True
    return False


def check_self_collision(snake):
    head = snake[0]
    body = snake[1:]
    return head in body
```

Now add a function to set up a new game. We will call this whenever the player starts or restarts:

```python
def new_game():
    snake = [[5, 7], [4, 7], [3, 7]]
    direction = [1, 0]
    food = spawn_food(snake)
    score = 0
    return snake, direction, food, score
```

Finally, the game loop. This is where the game state controls what happens. You will see a new kind of string in this code: `f"Score: {score}"`. The `f` before the quotation marks makes this an **f-string**. Python replaces anything inside curly braces `{}` with its value. So `f"Score: {score}"` becomes something like `"Score: 5"`. It is a clean way to put variables inside strings.

```python
game_state = "menu"
snake, direction, food, score = new_game()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if game_state == "menu" and event.key == pygame.K_SPACE:
                game_state = "playing"
                snake, direction, food, score = new_game()

            if game_state == "game_over" and event.key == pygame.K_SPACE:
                game_state = "playing"
                snake, direction, food, score = new_game()

    screen.fill(BLACK)

    if game_state == "menu":
        title = big_font.render("SNAKE", True, GREEN)
        prompt = font.render("Press SPACE to start", True, WHITE)
        screen.blit(title, (WINDOW_WIDTH // 2 - title.get_width() // 2,
                            WINDOW_HEIGHT // 3))
        screen.blit(prompt, (WINDOW_WIDTH // 2 - prompt.get_width() // 2,
                             WINDOW_HEIGHT // 2))

    elif game_state == "playing":
        direction = handle_input(direction)
        ate = move_snake(snake, direction, food)

        if ate:
            score += 1
            food = spawn_food(snake)

        if check_wall_collision(snake) or check_self_collision(snake):
            game_state = "game_over"

        draw_food(food)
        draw_snake(snake)

        score_text = font.render(f"Score: {score}", True, WHITE)
        screen.blit(score_text, (10, 10))

    elif game_state == "game_over":
        game_over_text = big_font.render("GAME OVER", True, RED)
        score_text = font.render(f"Score: {score}", True, WHITE)
        prompt = font.render("Press SPACE to play again", True, GRAY)
        screen.blit(game_over_text,
                     (WINDOW_WIDTH // 2 - game_over_text.get_width() // 2,
                      WINDOW_HEIGHT // 3))
        screen.blit(score_text,
                     (WINDOW_WIDTH // 2 - score_text.get_width() // 2,
                      WINDOW_HEIGHT // 2))
        screen.blit(prompt,
                     (WINDOW_WIDTH // 2 - prompt.get_width() // 2,
                      WINDOW_HEIGHT // 2 + 50))

    pygame.display.flip()
    clock.tick(FPS)

pygame.quit()
```

Save the file and run it:

```
python3 snake.py
```

Press `Space` to start. Use the arrow keys to steer the snake toward the red food squares. Each time you eat food, your score goes up and the snake grows longer. Try to beat your own high score.

---

## What You Learned

- A **grid system** simplifies movement: think in grid squares, convert to pixels only for drawing.
- The snake is a **list** of `[x, y]` positions. Moving means inserting a new head and removing the tail.
- Growing the snake means skipping the tail removal for one frame.
- **Game states** let you have different screens (menu, playing, game over) in the same program using a single variable.
- The `enumerate()` function gives you both the index and the value when looping through a list.
- **Slicing** (`snake[1:]`) creates a new list from part of an existing one.

## Quick Quiz

1. If `CELL_SIZE` is 30 and a snake segment is at grid position `[4, 2]`, what is its pixel position?

2. Why do we insert the new head at index 0 in the list instead of appending it at the end?

3. What happens if we never call `snake.pop()`?

4. The snake is moving to the right (`direction = [1, 0]`). The player presses the left arrow. What does `handle_input` return, and why?

## Experiment

1. **Change the speed.** Set `FPS` to 5. Then try 20. How does it feel? Find a speed you like.

2. **Bigger grid, smaller cells.** Change `CELL_SIZE` to 20 and `GRID_WIDTH` to 30 and `GRID_HEIGHT` to 22. The game should still work because everything uses the constants.

3. **Rainbow snake.** In `draw_snake`, use `i` (the segment index) to change the color of each segment. For example: `color = (0, 200 - i * 5, i * 5)`. What happens when the snake gets very long? If your snake gets very long, the color values might go below 0 or above 255, which causes an error. You can prevent this with `max(0, value)` and `min(255, value)` to keep numbers in the valid range.

## Chapter Challenge

**Add a high score to Snake.**

When the game ends, check if the player's score is higher than the stored high score. If it is, update the high score. Display the high score on both the playing screen and the game over screen.

Hints (try on your own first):

1. Create a variable `high_score = 0` before the game loop.
2. When `game_state` changes to `"game_over"`, compare `score` to `high_score`. If `score` is bigger, set `high_score = score`.
3. Render and blit the high score text the same way you display the current score.

Extra challenge: Can you make the high score survive between play sessions? Look up how to write a number to a file with `open()` and `write()`, then read it back with `open()` and `read()`.
