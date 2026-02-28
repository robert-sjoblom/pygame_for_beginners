---
layout: default
title: "Chapter 14: Tower Defense"
nav_order: 8
parent: "Part 2: Making Games with Pygame"
---

# Chapter 14: Game 5 -- Tower Defense

## What You Will Learn
- How to define a path using a list of waypoints
- How to move enemies along a path toward a goal
- How to calculate distance between two points
- How to make towers detect and shoot at enemies
- How to build a wave system that sends groups of enemies
- How to manage money, lives, and game state together
- How to balance a game so it is fun to play

This is the most complex game in the book. You have come a long way from "Hello, World!" in Chapter 1. Everything you have learned -- variables, loops, lists, functions, classes, sprites, mouse input, game states -- comes together here. Take your time, build it piece by piece, and test after every section.

## The Genre: Tower Defense

Tower defense games challenge you to stop waves of enemies by placing defensive towers along their path. The enemies march along a fixed route, and your towers shoot at them as they pass. If too many enemies reach the end, you lose. *Desktop Tower Defense* (2007) helped make the genre hugely popular in web browsers, and tower defense has been a staple of gaming ever since.

Our version keeps things straightforward: enemies follow a fixed path, towers go in pre-set positions, and you earn money for each enemy you stop. Simple rules, but the strategy comes from choosing where to place your towers and when to place them.

> Did you know? The tower defense genre traces its roots back to custom maps made in *StarCraft* and *Warcraft III*. Players built their own game modes inside existing games -- a tradition called "modding" that is still alive today.

## Planning the Game

Let's think through the design before writing any code.

### The Path

Enemies follow a fixed path across the screen. We define the path as a list of **waypoints** -- coordinates that the enemies walk toward, one after another.

```
Start                                                End
  *-------*                           *---------*
  |       |                           |         |
  |       *---------------------------*         |
  |                                             |
  *---------------------------------------------*

Waypoints: (50,400) -> (50,200) -> (200,200) -> (200,350) ->
           (600,350) -> (600,150) -> (750,150)
```

The enemies enter from the left and zigzag across the screen to the right. Each corner is a waypoint.

### Tower Positions

Instead of letting the player place towers anywhere (which is harder to program), we define specific positions where towers can go. These positions are shown as empty outlines on the screen. Click one to build a tower there.

```
Path:  *====*====*====*====*====*====*

Tower spots:
         [T1]      [T2]      [T3]
              [T4]      [T5]
```

### The Rules

- Each tower costs money to build.
- You start with enough money for 2-3 towers.
- Defeating enemies earns money for more towers.
- Enemies have health. Towers do damage.
- If an enemy reaches the end of the path, you lose a life.
- You start with 10 lives.
- Survive all waves to win. Lose all lives and it is game over.

### Waves

Each wave sends a group of enemies. Later waves have more enemies, and the enemies get tougher.

```
Wave 1: 5 basic enemies
Wave 2: 8 basic enemies
Wave 3: 6 fast enemies
Wave 4: 10 basic enemies + 4 fast enemies
Wave 5: 15 mixed enemies (the final wave)
```

That is our plan. Let's build it.

---

## Part 1: The Path and Enemies

Create a new file called `tower_defense.py` and start with the setup:

```python
import pygame
import math

pygame.init()

WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tower Defense")

clock = pygame.time.Clock()

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (100, 100, 100)
DARK_GREEN = (20, 80, 20)
BROWN = (139, 119, 101)
RED = (220, 50, 50)
BLUE = (50, 100, 220)
YELLOW = (220, 200, 50)
GREEN = (50, 200, 50)
ORANGE = (220, 150, 50)
DARK_GRAY = (50, 50, 50)
```

We import `math` because we will need it for distance calculations later.

### Defining the Path

The path is a list of `(x, y)` coordinates. Add this after the colors:

```python
# The path enemies follow (list of waypoints)
PATH = [
    (50, 450),
    (50, 200),
    (200, 200),
    (200, 400),
    (500, 400),
    (500, 150),
    (700, 150),
    (700, 450),
    (800, 450),
]
```

The enemies will start at the first waypoint and walk to each one in order. When they reach the last waypoint, they have made it through your defenses.

### Drawing the Path

Let's draw the path on screen so we can see it. Add a function:

```python
def draw_path(screen):
    for i in range(len(PATH) - 1):
        pygame.draw.line(screen, BROWN, PATH[i], PATH[i + 1], 30)

    # Draw waypoint dots
    for point in PATH:
        pygame.draw.circle(screen, YELLOW, point, 5)
```

This draws a thick brown line between each pair of waypoints, then draws small yellow dots at each waypoint so we can see the corners.

### A Quick Test

Add a temporary game loop to see the path:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill(DARK_GREEN)
    draw_path(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You should see a brown zigzag path on a dark green background. The path looks like a dirt road through grass. If the waypoints are not where you want them, adjust the numbers in the `PATH` list.

### The Enemy Class

Now let's make enemies that walk along the path. Add this class above the game loop:

```python
class Enemy:
    def __init__(self, health, speed, color=RED):
        self.x = PATH[0][0]
        self.y = PATH[0][1]
        self.health = health
        self.max_health = health
        self.speed = speed
        self.color = color
        self.waypoint_index = 0
        self.alive = True
        self.reached_end = False
        self.reward_given = False
        self.radius = 12
```

Each enemy starts at the first waypoint. It has health, speed, a color, and tracks which waypoint it is heading toward. The `alive` flag tells us if it has been defeated, and `reached_end` tells us if it made it through.

### Moving Toward a Waypoint

This is the trickiest part of the enemy code. The enemy needs to move toward its current target waypoint. When it gets close enough, it switches to the next one.

We need to calculate **distance** between two points. There are two ways to do this:

The simple way uses absolute values:

```python
distance = abs(x2 - x1) + abs(y2 - y1)
```

This is called **Manhattan distance** -- it measures distance as if you could only walk along grid lines, like the streets of Manhattan. It works for rough estimates.

The proper way uses the **Pythagorean theorem** that you may have seen in math class:

```python
distance = math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)
```

Or written another way:

```python
distance = ((x2 - x1) ** 2 + (y2 - y1) ** 2) ** 0.5
```

Both do the same thing. `** 0.5` means "square root," which is the same as `math.sqrt()`. This gives the real straight-line distance between two points.

```
Point A *-----------* Point B
         \         /
          \       /
           \     /     Straight-line distance =
            \   /      sqrt((x2-x1)^2 + (y2-y1)^2)
             \ /
              *
```

We will use the proper distance. Add an `update` method to the `Enemy` class:

```python
    def update(self):
        if not self.alive or self.reached_end:
            return

        # Target waypoint
        target_x, target_y = PATH[self.waypoint_index]

        # Calculate direction to target
        dx = target_x - self.x
        dy = target_y - self.y
        distance = math.sqrt(dx ** 2 + dy ** 2)

        # If close enough, move to next waypoint
        if distance < self.speed:
            self.x = target_x
            self.y = target_y
            self.waypoint_index += 1

            # Reached the end of the path?
            if self.waypoint_index >= len(PATH):
                self.reached_end = True
                return
        else:
            # Move toward the target
            self.x += (dx / distance) * self.speed
            self.y += (dy / distance) * self.speed
```

Let's break this down. We calculate `dx` and `dy` -- the difference in x and y between the enemy and its target. We find the distance. If the enemy is very close (closer than one step of its speed), we snap it to the waypoint and move on to the next one. Otherwise, we move the enemy toward the target.

The expression `(dx / distance) * self.speed` is a way to move at a constant speed in any direction. Dividing by the distance **normalizes** the direction -- it turns it into a unit of length 1 -- and then multiplying by speed gives us the right step size. This part is tricky. Read it twice if you need to.

### Drawing the Enemy

Add a `draw` method:

```python
    def draw(self, screen):
        if not self.alive:
            return

        # Body
        pygame.draw.circle(screen, self.color, (int(self.x), int(self.y)),
                           self.radius)

        # Health bar background
        bar_x = int(self.x) - 15
        bar_y = int(self.y) - 20
        pygame.draw.rect(screen, RED, (bar_x, bar_y, 30, 5))

        # Health bar fill
        health_width = int(30 * (self.health / self.max_health))
        pygame.draw.rect(screen, GREEN, (bar_x, bar_y, health_width, 5))
```

Each enemy is a colored circle with a tiny health bar floating above it. The health bar shrinks as the enemy takes damage. We use `int()` to convert the float positions to whole pixel numbers.

### Testing Enemies

Let's test with a few enemies. Replace your game loop with this:

```python
# Test: create some enemies
enemies = [Enemy(100, 2)]

frame_count = 0

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Spawn a new enemy every 60 frames (1 per second)
    frame_count += 1
    if frame_count % 60 == 0 and len(enemies) < 5:
        enemies.append(Enemy(100, 2))

    # Update enemies
    for enemy in enemies:
        enemy.update()

    # Draw
    screen.fill(DARK_GREEN)
    draw_path(screen)
    for enemy in enemies:
        enemy.draw(screen)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You should see red circles marching along the path, one spawning each second, up to five. Each one follows the waypoints, turning at each corner. Their green health bars sit above them at full health (since nothing is hurting them yet).

This is a good stopping point. Save your work.

---

## Part 2: Towers That Shoot

Now let's give the player a way to fight back.

### The Tower Class

A tower sits at a fixed position and shoots at enemies that come within range. Add this class:

```python
class Tower:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.range = 120
        self.damage = 25
        self.fire_rate = 30
        self.cooldown = 0
        self.target = None
        self.size = 20
```

Each tower has a position, a range (how far it can shoot), damage per shot, a fire rate (how many frames between shots), and a cooldown counter. The `target` will hold the enemy it is currently shooting at.

### Finding the Nearest Enemy

The tower needs to find the closest enemy within its range. Add this method:

```python
    def find_target(self, enemies):
        nearest = None
        nearest_distance = self.range + 1

        for enemy in enemies:
            if not enemy.alive or enemy.reached_end:
                continue

            dx = enemy.x - self.x
            dy = enemy.y - self.y
            distance = math.sqrt(dx ** 2 + dy ** 2)

            if distance <= self.range and distance < nearest_distance:
                nearest = enemy
                nearest_distance = distance

        self.target = nearest
```

We loop through all enemies, skip dead ones and ones that already reached the end, calculate the distance to each one, and keep track of the closest one. If the closest enemy is within range, that becomes our target.

### Shooting

Add an `update` method that handles shooting:

```python
    def update(self, enemies):
        # Cooldown timer counts down each frame
        if self.cooldown > 0:
            self.cooldown -= 1

        # Find a target
        self.find_target(enemies)

        # Shoot if we have a target and cooldown is done
        if self.target and self.cooldown <= 0:
            self.target.health -= self.damage
            if self.target.health <= 0:
                self.target.alive = False
            self.cooldown = self.fire_rate
```

Each frame, the cooldown ticks down. If there is a target and the cooldown has reached zero, the tower fires: it subtracts damage from the enemy's health. If health drops to zero or below, the enemy is marked as not alive. Then the cooldown resets.

### Drawing the Tower

Add a `draw` method:

```python
    def draw(self, screen):
        # Tower body (square)
        rect = pygame.Rect(self.x - self.size, self.y - self.size,
                           self.size * 2, self.size * 2)
        pygame.draw.rect(screen, BLUE, rect)
        pygame.draw.rect(screen, WHITE, rect, 2)

        # Range circle (faint)
        pygame.draw.circle(screen, (50, 50, 100), (self.x, self.y),
                           self.range, 1)

        # Laser to target
        if self.target and self.target.alive and self.cooldown > self.fire_rate - 5:
            pygame.draw.line(screen, YELLOW, (self.x, self.y),
                             (int(self.target.x), int(self.target.y)), 2)
```

The tower is a blue square with a white border. A faint circle shows its range. When it fires, a yellow line appears briefly between the tower and its target -- that is the laser shot. The `self.cooldown > self.fire_rate - 5` check means the line only appears for 5 frames after firing, so it looks like a quick flash.

### Tower Placement Spots

We need predefined spots where the player can build towers. Add these after the `PATH` definition:

```python
# Pre-defined tower placement positions
TOWER_SPOTS = [
    (130, 300),
    (350, 300),
    (350, 250),
    (130, 100),
    (600, 300),
    (600, 50),
]
```

These positions are near the path but not on it. The player will click on a spot to build a tower there.

### Drawing Tower Spots

Add a function to draw the available spots:

```python
def draw_tower_spots(screen, towers):
    occupied = [(t.x, t.y) for t in towers]
    for spot in TOWER_SPOTS:
        if spot not in occupied:
            rect = pygame.Rect(spot[0] - 20, spot[1] - 20, 40, 40)
            pygame.draw.rect(screen, DARK_GRAY, rect)
            pygame.draw.rect(screen, GRAY, rect, 2)
```

This draws each unoccupied spot as a dark gray square with a lighter border -- an invitation to click.

### Testing Towers

Let's test by placing a tower manually. Update the game loop (replace the old test code):

```python
enemies = []
towers = [Tower(130, 300)]
frame_count = 0

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Spawn enemies for testing
    frame_count += 1
    if frame_count % 60 == 0 and len(enemies) < 8:
        enemies.append(Enemy(100, 2))

    # Update
    for enemy in enemies:
        enemy.update()
    for tower in towers:
        tower.update(enemies)

    # Draw
    screen.fill(DARK_GREEN)
    draw_path(screen)
    draw_tower_spots(screen, towers)
    for tower in towers:
        tower.draw(screen)
    for enemy in enemies:
        enemy.draw(screen)

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You should see enemies marching along the path and a blue tower shooting at them when they come within range. The yellow laser flashes, the health bars shrink, and enemies disappear when their health runs out.

This is a good stopping point. Save your work.

---

## Part 3: Placing Towers, Money, and Lives

Time to let the player actually play the game.

### The Game Class

Let's organize everything into a `Game` class. Replace the loose variables and game loop with this structure:

```python
class Game:
    def __init__(self):
        self.font = pygame.font.Font(None, 32)
        self.small_font = pygame.font.Font(None, 24)
        self.enemies = []
        self.towers = []
        self.money = 100
        self.lives = 10
        self.tower_cost = 40
        self.wave_number = 0
        self.spawning = False
        self.spawn_timer = 0
        self.spawn_count = 0
        self.spawn_queue = []
        self.state = "playing"
        self.frame_count = 0
        self.wave_delay = 0
        self.waves = self.create_waves()
        self.start_next_wave()
```

The player starts with 100 money and 10 lives. Towers cost 40 each, so they can afford 2 to start.

### Defining Waves

Add the wave definitions:

```python
    def create_waves(self):
        waves = [
            {"enemies": [("basic", 5)]},
            {"enemies": [("basic", 8)]},
            {"enemies": [("fast", 6)]},
            {"enemies": [("basic", 10), ("fast", 4)]},
            {"enemies": [("basic", 8), ("fast", 5), ("tough", 3)]},
        ]
        return waves
```

Each wave is a dictionary with a list of `(type, count)` tuples. Wave 1 has 5 basic enemies. Wave 4 has 10 basic and 4 fast. Wave 5 is the hardest: a mix of basic, fast, and tough enemies.

### Starting a Wave

Add a method that prepares the next wave:

```python
    def start_next_wave(self):
        if self.wave_number >= len(self.waves):
            return

        wave = self.waves[self.wave_number]
        self.spawn_queue = []

        for enemy_type, count in wave["enemies"]:
            for _ in range(count):
                self.spawn_queue.append(enemy_type)

        self.spawning = True
        self.spawn_timer = 0
        self.spawn_count = 0
        self.wave_number += 1
```

This builds a `spawn_queue` -- a flat list of enemy types to spawn. For wave 4, the queue would be `["basic", "basic", ..., "fast", "fast", "fast", "fast"]`.

### Spawning Enemies

Add a method that spawns enemies from the queue one at a time:

```python
    def spawn_enemies(self):
        if not self.spawning:
            return

        self.spawn_timer += 1
        if self.spawn_timer >= 40:
            self.spawn_timer = 0

            if self.spawn_count < len(self.spawn_queue):
                enemy_type = self.spawn_queue[self.spawn_count]

                if enemy_type == "basic":
                    self.enemies.append(Enemy(100, 2, RED))
                elif enemy_type == "fast":
                    self.enemies.append(Enemy(60, 3.5, ORANGE))
                elif enemy_type == "tough":
                    self.enemies.append(Enemy(200, 1.5, (150, 50, 150)))

                self.spawn_count += 1
            else:
                self.spawning = False
```

Every 40 frames (roughly every 0.7 seconds), a new enemy appears. Basic enemies are red with 100 health and speed 2. Fast enemies are orange with less health but higher speed. Tough enemies are purple with lots of health but they are slow.

### Placing Towers with Clicks

Add a click handler:

```python
    def handle_click(self, mouse_pos):
        if self.state != "playing":
            return

        occupied = [(t.x, t.y) for t in self.towers]

        for spot in TOWER_SPOTS:
            if spot in occupied:
                continue

            rect = pygame.Rect(spot[0] - 20, spot[1] - 20, 40, 40)
            if rect.collidepoint(mouse_pos):
                if self.money >= self.tower_cost:
                    self.towers.append(Tower(spot[0], spot[1]))
                    self.money -= self.tower_cost
                return
```

When the player clicks, we check each unoccupied tower spot. If the click is inside a spot's rectangle and the player has enough money, we build a tower and subtract the cost.

### The Update Method

Add the main update logic:

```python
    def update(self):
        if self.state != "playing":
            return

        self.frame_count += 1

        # Spawn enemies from the current wave
        self.spawn_enemies()

        # Update enemies
        for enemy in self.enemies:
            enemy.update()

            # Enemy reached the end
            if enemy.reached_end and enemy.alive:
                enemy.alive = False
                self.lives -= 1

        # Earn money for defeated enemies
        for enemy in self.enemies:
            if not enemy.alive and not enemy.reached_end:
                if enemy.reward_given:
                    continue
                enemy.reward_given = True
                self.money += 10

        # Remove dead enemies
        self.enemies = [e for e in self.enemies if e.alive]

        # Update towers
        for tower in self.towers:
            tower.update(self.enemies)

        # Check for wave complete
        if not self.spawning and len(self.enemies) == 0:
            if self.wave_number >= len(self.waves):
                self.state = "won"
            else:
                self.wave_delay += 1
                if self.wave_delay >= 120:
                    self.wave_delay = 0
                    self.start_next_wave()

        # Check for game over
        if self.lives <= 0:
            self.state = "lost"
```

There is a lot happening here. Let's walk through it:

1. We spawn enemies from the queue.
2. We update each enemy's position.
3. If an enemy reached the end, it costs a life.
4. Dead enemies (defeated, not ones that reached the end) earn money. We check `reward_given` to make sure we only give the reward once.
5. We clean up the enemy list, keeping only living enemies.
6. Towers find targets and shoot.
7. When all enemies from a wave are gone, we wait 2 seconds (120 frames), then start the next wave.
8. If lives hit zero, the game is over.

### Drawing the UI

Add a method to draw the game information:

```python
    def draw_ui(self, screen):
        # Money
        money_text = self.font.render(f"Money: {self.money}", True, YELLOW)
        screen.blit(money_text, (10, 10))

        # Lives
        lives_text = self.font.render(f"Lives: {self.lives}", True, RED)
        screen.blit(lives_text, (10, 40))

        # Wave
        wave_text = self.font.render(
            f"Wave: {self.wave_number}/{len(self.waves)}", True, WHITE)
        screen.blit(wave_text, (10, 70))

        # Tower cost hint
        cost_text = self.small_font.render(
            f"Click a gray square to build a tower ({self.tower_cost} money)",
            True, GRAY)
        screen.blit(cost_text, (WIDTH - 420, 10))
```

### Drawing Win and Lose Screens

```python
    def draw_end_screen(self, screen):
        overlay = pygame.Surface((WIDTH, HEIGHT))
        overlay.set_alpha(180)
        overlay.fill(BLACK)
        screen.blit(overlay, (0, 0))

        if self.state == "won":
            title = self.font.render("YOU WIN!", True, GREEN)
            msg = self.small_font.render(
                "All waves defeated. The base is safe!", True, WHITE)
        else:
            title = self.font.render("GAME OVER", True, RED)
            msg = self.small_font.render(
                f"Reached wave {self.wave_number}. Try again!", True, WHITE)

        screen.blit(title, (WIDTH // 2 - title.get_width() // 2, 230))
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, 280))

        hint = self.small_font.render("Close the window to exit.", True, GRAY)
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 330))
```

We draw a semi-transparent overlay on top of the game, then the win or lose message. The `set_alpha(180)` makes the overlay see-through so the game is still faintly visible behind the text.

### The Main Draw Method

```python
    def draw(self, screen):
        screen.fill(DARK_GREEN)
        draw_path(screen)
        draw_tower_spots(screen, self.towers)

        for tower in self.towers:
            tower.draw(screen)
        for enemy in self.enemies:
            enemy.draw(screen)

        self.draw_ui(screen)

        if self.state in ("won", "lost"):
            self.draw_end_screen(screen)
```

### The Updated Game Loop

Replace the old game loop:

```python
game = Game()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.MOUSEBUTTONDOWN:
            mouse_pos = pygame.mouse.get_pos()
            game.handle_click(mouse_pos)

    game.update()
    game.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You can now click on the gray squares to place towers, watch enemies march along the path, and see your towers fight back. Try to survive all five waves.

This is a good stopping point. Save your work.

---

## Part 4: Balancing and the Complete Game

### Balancing Your Game

> Making a game fun to play means balancing the numbers. If towers are too strong, the game is boring -- the player barely has to try. If enemies are too tough, it is frustrating -- the player never feels like they have a chance. Good balance sits right in the middle: the player has to think, but they can win if they play smart.

Here are the numbers you can adjust:

```
Tower stats:
  range        How far the tower can shoot (120)
  damage       How much each shot hurts (25)
  fire_rate    Frames between shots (30 = about 2 shots/sec)
  cost         How much money to build (40)

Enemy stats:
  health       How much damage they can take (100 / 60 / 200)
  speed        How fast they move (2 / 3.5 / 1.5)

Economy:
  starting_money   How much you start with (100)
  kill_reward       Money per enemy defeated (10)
  lives             How many can get through (10)
```

Try changing these values. Make the enemies faster. Give the towers more range. Make towers cheaper but weaker. Every change shifts the balance. Professional game designers spend weeks tuning these numbers. You are doing the same thing, just on a smaller scale.

### Checkpoint: The Complete Game

Here is the full program. Your code should now look like this. Compare it with what you have built so far and fix any differences.

```python
import pygame
import math

pygame.init()

WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Tower Defense")

clock = pygame.time.Clock()

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (100, 100, 100)
DARK_GREEN = (20, 80, 20)
BROWN = (139, 119, 101)
RED = (220, 50, 50)
BLUE = (50, 100, 220)
YELLOW = (220, 200, 50)
GREEN = (50, 200, 50)
ORANGE = (220, 150, 50)
DARK_GRAY = (50, 50, 50)

# The path enemies follow (list of waypoints)
PATH = [
    (50, 450),
    (50, 200),
    (200, 200),
    (200, 400),
    (500, 400),
    (500, 150),
    (700, 150),
    (700, 450),
    (800, 450),
]

# Pre-defined tower placement positions
TOWER_SPOTS = [
    (130, 300),
    (350, 300),
    (350, 250),
    (130, 100),
    (600, 300),
    (600, 50),
]


def draw_path(screen):
    for i in range(len(PATH) - 1):
        pygame.draw.line(screen, BROWN, PATH[i], PATH[i + 1], 30)
    for point in PATH:
        pygame.draw.circle(screen, YELLOW, point, 5)


def draw_tower_spots(screen, towers):
    occupied = [(t.x, t.y) for t in towers]
    for spot in TOWER_SPOTS:
        if spot not in occupied:
            rect = pygame.Rect(spot[0] - 20, spot[1] - 20, 40, 40)
            pygame.draw.rect(screen, DARK_GRAY, rect)
            pygame.draw.rect(screen, GRAY, rect, 2)


class Enemy:
    def __init__(self, health, speed, color=RED):
        self.x = PATH[0][0]
        self.y = PATH[0][1]
        self.health = health
        self.max_health = health
        self.speed = speed
        self.color = color
        self.waypoint_index = 0
        self.alive = True
        self.reached_end = False
        self.reward_given = False
        self.radius = 12

    def update(self):
        if not self.alive or self.reached_end:
            return

        target_x, target_y = PATH[self.waypoint_index]
        dx = target_x - self.x
        dy = target_y - self.y
        distance = math.sqrt(dx ** 2 + dy ** 2)

        if distance < self.speed:
            self.x = target_x
            self.y = target_y
            self.waypoint_index += 1
            if self.waypoint_index >= len(PATH):
                self.reached_end = True
                return
        else:
            self.x += (dx / distance) * self.speed
            self.y += (dy / distance) * self.speed

    def draw(self, screen):
        if not self.alive:
            return

        pygame.draw.circle(screen, self.color, (int(self.x), int(self.y)),
                           self.radius)

        # Health bar
        bar_x = int(self.x) - 15
        bar_y = int(self.y) - 20
        pygame.draw.rect(screen, RED, (bar_x, bar_y, 30, 5))
        health_width = int(30 * (self.health / self.max_health))
        pygame.draw.rect(screen, GREEN, (bar_x, bar_y, health_width, 5))


class Tower:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.range = 120
        self.damage = 25
        self.fire_rate = 30
        self.cooldown = 0
        self.target = None
        self.size = 20

    def find_target(self, enemies):
        nearest = None
        nearest_distance = self.range + 1

        for enemy in enemies:
            if not enemy.alive or enemy.reached_end:
                continue

            dx = enemy.x - self.x
            dy = enemy.y - self.y
            distance = math.sqrt(dx ** 2 + dy ** 2)

            if distance <= self.range and distance < nearest_distance:
                nearest = enemy
                nearest_distance = distance

        self.target = nearest

    def update(self, enemies):
        if self.cooldown > 0:
            self.cooldown -= 1

        self.find_target(enemies)

        if self.target and self.cooldown <= 0:
            self.target.health -= self.damage
            if self.target.health <= 0:
                self.target.alive = False
            self.cooldown = self.fire_rate

    def draw(self, screen):
        rect = pygame.Rect(self.x - self.size, self.y - self.size,
                           self.size * 2, self.size * 2)
        pygame.draw.rect(screen, BLUE, rect)
        pygame.draw.rect(screen, WHITE, rect, 2)

        # Range circle
        pygame.draw.circle(screen, (50, 50, 100), (self.x, self.y),
                           self.range, 1)

        # Laser to target
        if self.target and self.target.alive and self.cooldown > self.fire_rate - 5:
            pygame.draw.line(screen, YELLOW, (self.x, self.y),
                             (int(self.target.x), int(self.target.y)), 2)


class Game:
    def __init__(self):
        self.font = pygame.font.Font(None, 32)
        self.small_font = pygame.font.Font(None, 24)
        self.enemies = []
        self.towers = []
        self.money = 100
        self.lives = 10
        self.tower_cost = 40
        self.wave_number = 0
        self.spawning = False
        self.spawn_timer = 0
        self.spawn_count = 0
        self.spawn_queue = []
        self.state = "playing"
        self.frame_count = 0
        self.wave_delay = 0
        self.waves = self.create_waves()
        self.start_next_wave()

    def create_waves(self):
        waves = [
            {"enemies": [("basic", 5)]},
            {"enemies": [("basic", 8)]},
            {"enemies": [("fast", 6)]},
            {"enemies": [("basic", 10), ("fast", 4)]},
            {"enemies": [("basic", 8), ("fast", 5), ("tough", 3)]},
        ]
        return waves

    def start_next_wave(self):
        if self.wave_number >= len(self.waves):
            return

        wave = self.waves[self.wave_number]
        self.spawn_queue = []

        for enemy_type, count in wave["enemies"]:
            for _ in range(count):
                self.spawn_queue.append(enemy_type)

        self.spawning = True
        self.spawn_timer = 0
        self.spawn_count = 0
        self.wave_number += 1

    def spawn_enemies(self):
        if not self.spawning:
            return

        self.spawn_timer += 1
        if self.spawn_timer >= 40:
            self.spawn_timer = 0

            if self.spawn_count < len(self.spawn_queue):
                enemy_type = self.spawn_queue[self.spawn_count]

                if enemy_type == "basic":
                    self.enemies.append(Enemy(100, 2, RED))
                elif enemy_type == "fast":
                    self.enemies.append(Enemy(60, 3.5, ORANGE))
                elif enemy_type == "tough":
                    self.enemies.append(Enemy(200, 1.5, (150, 50, 150)))

                self.spawn_count += 1
            else:
                self.spawning = False

    def handle_click(self, mouse_pos):
        if self.state != "playing":
            return

        occupied = [(t.x, t.y) for t in self.towers]

        for spot in TOWER_SPOTS:
            if spot in occupied:
                continue

            rect = pygame.Rect(spot[0] - 20, spot[1] - 20, 40, 40)
            if rect.collidepoint(mouse_pos):
                if self.money >= self.tower_cost:
                    self.towers.append(Tower(spot[0], spot[1]))
                    self.money -= self.tower_cost
                return

    def update(self):
        if self.state != "playing":
            return

        self.frame_count += 1
        self.spawn_enemies()

        for enemy in self.enemies:
            enemy.update()
            if enemy.reached_end and enemy.alive:
                enemy.alive = False
                self.lives -= 1

        for enemy in self.enemies:
            if not enemy.alive and not enemy.reached_end:
                if enemy.reward_given:
                    continue
                enemy.reward_given = True
                self.money += 10

        self.enemies = [e for e in self.enemies if e.alive]

        for tower in self.towers:
            tower.update(self.enemies)

        if not self.spawning and len(self.enemies) == 0:
            if self.wave_number >= len(self.waves):
                self.state = "won"
            else:
                self.wave_delay += 1
                if self.wave_delay >= 120:
                    self.wave_delay = 0
                    self.start_next_wave()

        if self.lives <= 0:
            self.state = "lost"

    def draw_ui(self, screen):
        money_text = self.font.render(f"Money: {self.money}", True, YELLOW)
        screen.blit(money_text, (10, 10))

        lives_text = self.font.render(f"Lives: {self.lives}", True, RED)
        screen.blit(lives_text, (10, 40))

        wave_text = self.font.render(
            f"Wave: {self.wave_number}/{len(self.waves)}", True, WHITE)
        screen.blit(wave_text, (10, 70))

        cost_text = self.small_font.render(
            f"Click a gray square to build a tower ({self.tower_cost} money)",
            True, GRAY)
        screen.blit(cost_text, (WIDTH - 420, 10))

    def draw_end_screen(self, screen):
        overlay = pygame.Surface((WIDTH, HEIGHT))
        overlay.set_alpha(180)
        overlay.fill(BLACK)
        screen.blit(overlay, (0, 0))

        if self.state == "won":
            title = self.font.render("YOU WIN!", True, GREEN)
            msg = self.small_font.render(
                "All waves defeated. The base is safe!", True, WHITE)
        else:
            title = self.font.render("GAME OVER", True, RED)
            msg = self.small_font.render(
                f"Reached wave {self.wave_number}. Try again!", True, WHITE)

        screen.blit(title, (WIDTH // 2 - title.get_width() // 2, 230))
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, 280))

        hint = self.small_font.render("Close the window to exit.", True, GRAY)
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 330))

    def draw(self, screen):
        screen.fill(DARK_GREEN)
        draw_path(screen)
        draw_tower_spots(screen, self.towers)

        for tower in self.towers:
            tower.draw(screen)
        for enemy in self.enemies:
            enemy.draw(screen)

        self.draw_ui(screen)

        if self.state in ("won", "lost"):
            self.draw_end_screen(screen)


# --- Main game loop ---
game = Game()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.MOUSEBUTTONDOWN:
            mouse_pos = pygame.mouse.get_pos()
            game.handle_click(mouse_pos)

    game.update()
    game.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. Play through all five waves. Where you place your first two towers matters a lot -- try spots near bends in the path where enemies spend the most time in range.

If the game is too hard or too easy, go back to the balancing section and tweak the numbers. That is not cheating -- that is game design.

## What You Learned
- A **path** is a list of waypoints. Enemies move from one waypoint to the next.
- **Distance calculation** uses the Pythagorean theorem: `math.sqrt(dx**2 + dy**2)`.
- **Normalizing** a direction (dividing by distance) lets you move at a constant speed in any direction.
- **Towers** find the nearest enemy in range and reduce its health on a timer.
- A **wave system** sends groups of enemies with delays between spawns.
- **Game economy** (money for kills, costs for towers) creates strategy.
- **Classes** organize complex game objects: `Enemy`, `Tower`, and `Game` each manage their own data and behavior.
- **Balancing** numbers is a core part of game design. Small changes to damage, speed, or cost can make the difference between boring, fun, and frustrating.
- You can build a complete, playable game by combining all the concepts from earlier chapters.

## Quick Quiz

1. Why do we divide `dx` and `dy` by the distance when moving an enemy toward a waypoint?
2. If a tower has `fire_rate = 60` and the game runs at 60 frames per second, how often does the tower shoot?
3. What happens if an enemy's health drops below zero? How does the code handle it?
4. Why do we check `enemy.reward_given` when awarding money?

## Experiment

1. Change the path. Add more waypoints, make it longer, add more zigzags. How does this affect the difficulty?
2. Add a new enemy type called `"boss"` with 500 health and speed 1. Make it appear only in wave 5. Give it a unique color.
3. Change the tower cost to 30 and the starting money to 150. Does the game become too easy? Find a balance that feels right.
4. Make towers draw differently based on whether they have a target -- for example, change the tower color from blue to bright cyan when actively shooting.

## Chapter Challenge

Add **tower upgrades**. When the player clicks on a tower that is already built, instead of doing nothing, offer to upgrade it. An upgraded tower should have more range, more damage, or a faster fire rate.

**Hint 1:** Add a `level` attribute to the `Tower` class, starting at 1. Each upgrade increases the level and improves the stats.

**Hint 2:** In `handle_click`, check if the clicked spot already has a tower. If it does and the player has enough money, call an `upgrade` method on that tower.

**Hint 3:** Make upgrades cost more than building a new tower (maybe 60 money). Cap the level at 3 so towers cannot become infinitely powerful. Change the tower's color based on level so the player can see which towers are upgraded.
