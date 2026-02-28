---
layout: default
title: "Chapter 13: Point-and-Click Adventure"
nav_order: 7
parent: "Part 2: Making Games with Pygame"
---

# Chapter 13: Game 4 -- Point-and-Click Adventure

## What You Will Learn
- How to plan a larger game before writing code
- How to build a room system where the player clicks to move between locations
- How to handle mouse clicks and check what was clicked
- How to create an inventory system for picking up and using items
- How to display text messages on screen
- How to track puzzle state and build win conditions
- How to organize a bigger program using classes

This is the biggest program we have built so far. Take your time with it. We are going to break it into clear pieces, and there are four stopping points where you can save your work and come back later. Build one room first, make sure it works, then add the next. That is how professional game developers work too.

## The Genre: Point-and-Click Adventures

Point-and-click adventures were huge in the 1990s. Games like *Monkey Island* and *Myst* let players explore worlds by clicking on things -- doors, objects, characters. Lucasfilm Games (now LucasArts) pioneered this genre, making some of the most beloved games ever. The idea is simple: you see a scene, you click on things to interact with them, you collect items, and you solve puzzles to progress.

We are going to build our own. The theme is a space station. The power is out, doors are locked, and you need to find keycards and repair parts to fix the generator and restore power. No enemies, no combat -- just exploration and problem-solving.

> Did you know? Ron Gilbert, who created *Monkey Island* at Lucasfilm Games, had a rule: the player should never die or get permanently stuck. We are following that same rule in our game.

## Planning First

With a game this size, jumping straight into code is a mistake. We need a plan. Let's think about three things: rooms, items, and puzzles.

### The Room Map

Here is the layout of our space station. Six rooms, connected by doors:

```
                +-------------+
                |             |
                |  GENERATOR  |
                |   ROOM (6)  |
                |             |
                +------+------+
                       |
                +------+------+
                |             |
                |  STORAGE    |
                |   ROOM (5)  |
                |             |
                +------+------+
                       |
+-------------+--------+--------+-------------+
|             |                  |             |
|  CREW       |    MAIN         |  SCIENCE    |
|  QUARTERS(3)|    CORRIDOR (2) |  LAB (4)    |
|             |                  |             |
+-------------+--------+--------+-------------+
                       |
                +------+------+
                |             |
                |  AIRLOCK    |
                |  (START)(1) |
                |             |
                +-------------+
```

Room 1 (Airlock) is where the player starts. From there, they can go north to the Main Corridor. The corridor connects to Crew Quarters (west), Science Lab (east), and Storage Room (north). The Storage Room leads north to the Generator Room.

### The Items

We need four items:

1. **Blue keycard** -- found in the Crew Quarters. Opens the Science Lab door.
2. **Circuit board** -- found in the Science Lab. Needed to fix the generator.
3. **Wrench** -- found in the Storage Room. Needed to fix the generator.
4. **Red keycard** -- found in the Airlock. Opens the Storage Room door.

### The Puzzles

1. The Science Lab door is locked. You need the blue keycard (from the Crew Quarters) to open it.
2. The Storage Room door is locked. You need the red keycard (from the Airlock) to open it.
3. The generator is broken. You need both the circuit board (from the Science Lab) and the wrench (from the Storage Room) to fix it. Fixing the generator wins the game.

### The Design Summary

```
Rooms:  6
Items:  4 (blue keycard, red keycard, circuit board, wrench)
Puzzles: 3 (2 locked doors, 1 broken generator)
Goal:   Fix the generator to restore power
```

That is our whole plan. Now let's build it.

---

## Part 1: The Room System

Create a new file called `space_adventure.py`. We are going to start by getting one room on screen, then add more.

### The Room Class

Each room needs a few things: a name, a background color, some clickable areas (doors, items), and a way to draw itself. Let's make a `Room` class:

```python
import pygame

pygame.init()

WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Space Station Adventure")

clock = pygame.time.Clock()

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (100, 100, 100)
DARK_GRAY = (60, 60, 60)
LIGHT_GRAY = (180, 180, 180)
BLUE = (50, 100, 200)
RED = (200, 50, 50)
GREEN = (50, 200, 50)
YELLOW = (220, 220, 50)
DARK_BLUE = (20, 30, 60)
```

That is our setup. Now for the `Room` class. Add this below the colors:

```python
class Room:
    def __init__(self, name, bg_color, description):
        self.name = name
        self.bg_color = bg_color
        self.description = description
        self.clickable_areas = []
```

A room has a name, background color, and description. It also has `clickable_areas` (rectangles the player can click on).

### Clickable Areas

A **clickable area** is a rectangle on screen that does something when you click it. We need a simple way to define these. Add this class above the `Room` class:

```python
class ClickableArea:
    def __init__(self, rect, label, action, color=LIGHT_GRAY):
        self.rect = pygame.Rect(rect)
        self.label = label
        self.action = action
        self.color = color
```

Each clickable area has a rectangle (position and size), a label (what it says), an action (a string describing what happens), and a color. The `pygame.Rect` object is something pygame gives us -- it stores a rectangle and has useful methods for checking if a point is inside it.

### Drawing a Room

Add a `draw` method to the `Room` class:

```python
class Room:
    def __init__(self, name, bg_color, description):
        self.name = name
        self.bg_color = bg_color
        self.description = description
        self.clickable_areas = []

    def draw(self, screen, font):
        screen.fill(self.bg_color)

        # Draw room name at the top
        title = font.render(self.name, True, WHITE)
        screen.blit(title, (20, 10))

        # Draw clickable areas
        for area in self.clickable_areas:
            pygame.draw.rect(screen, area.color, area.rect)
            pygame.draw.rect(screen, WHITE, area.rect, 2)
            label = font.render(area.label, True, BLACK)
            label_x = area.rect.x + (area.rect.width - label.get_width()) // 2
            label_y = area.rect.y + (area.rect.height - label.get_height()) // 2
            screen.blit(label, (label_x, label_y))
```

This fills the screen with the room's background color, draws the room name at the top, and then draws each clickable area as a colored rectangle with a label centered inside it. We use `font.render()` to turn text into an image and `screen.blit()` to put that image on screen. You have seen this in earlier chapters.

### Setting Up the First Room

Now let's create the Airlock and get it on screen. Add this after the class definitions:

```python
font = pygame.font.Font(None, 28)
small_font = pygame.font.Font(None, 22)

# Create rooms
airlock = Room("Airlock", DARK_BLUE, "The airlock. Emergency lights cast a dim glow.")
airlock.clickable_areas.append(
    ClickableArea((325, 50, 150, 60), "North Door", "go_corridor")
)
airlock.clickable_areas.append(
    ClickableArea((600, 400, 120, 40), "Red Keycard", "pickup_red_keycard")
)

current_room = airlock
```

We created one room with two clickable areas: a door going north and a keycard to pick up. The `current_room` variable tracks which room the player is in.

### The Game Loop

Now add the game loop:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    current_room.draw(screen, font)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You should see a dark blue screen with "Airlock" at the top, a "North Door" rectangle, and a "Red Keycard" rectangle. Nothing happens when you click yet -- we will fix that next.

### Handling Mouse Clicks

We need to detect when the player clicks and check if they clicked inside a clickable area. Update the event handling inside the game loop:

```python
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.MOUSEBUTTONDOWN:
            mouse_pos = pygame.mouse.get_pos()
            for area in current_room.clickable_areas:
                if area.rect.collidepoint(mouse_pos):
                    print("Clicked:", area.action)

    current_room.draw(screen, font)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

`pygame.MOUSEBUTTONDOWN` fires when the player presses a mouse button. `pygame.mouse.get_pos()` returns the `(x, y)` position of the mouse cursor. Then we loop through every clickable area in the current room and check if the click was inside it using `collidepoint()`. For now, we just print the action to the terminal.

Run it and click on the rectangles. You should see messages like `Clicked: go_corridor` and `Clicked: pickup_red_keycard` in the terminal. That is our click system working.

This is a good stopping point. Save your work.

---

## Part 2: Multiple Rooms and Navigation

Now let's create all six rooms and let the player move between them. This is where the game starts to feel real.

### The Game Class

Managing rooms, inventory, and game state with loose variables will get messy. Let's put everything into a `Game` class. Replace everything after the color definitions (but keep the `ClickableArea` and `Room` classes) with this:

```python
class Game:
    def __init__(self):
        self.font = pygame.font.Font(None, 28)
        self.small_font = pygame.font.Font(None, 22)
        self.inventory = []
        self.message = ""
        self.message_timer = 0
        self.game_won = False
        self.rooms = {}
        self.current_room_id = 1
        self.create_rooms()
```

The `Game` class holds everything: fonts, inventory, a message to display, and all the rooms. The `message` and `message_timer` will let us show text to the player that fades away after a few seconds.

### Creating All the Rooms

Add the `create_rooms` method to the `Game` class. This is a longer method, but each room follows the same pattern:

```python
    def create_rooms(self):
        # Room 1: Airlock (start)
        room1 = Room("Airlock", DARK_BLUE,
                      "The airlock. Emergency lights cast a dim glow.")
        room1.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_2")
        )
        room1.clickable_areas.append(
            ClickableArea((600, 400, 120, 40), "Red Keycard", "pickup_red_keycard")
        )
        self.rooms[1] = room1
```

That is Room 1. Now add Room 2 right below it, still inside `create_rooms`:

```python
        # Room 2: Main Corridor
        room2 = Room("Main Corridor", DARK_GRAY,
                      "A wide corridor. Doors lead in every direction.")
        room2.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_1")
        )
        room2.clickable_areas.append(
            ClickableArea((20, 250, 150, 60), "West Door", "go_3")
        )
        room2.clickable_areas.append(
            ClickableArea((630, 250, 150, 60), "East Door", "go_4_locked")
        )
        room2.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_5_locked")
        )
        self.rooms[2] = room2
```

Notice `"go_4_locked"` and `"go_5_locked"`. These doors require keycards. We will handle that in the action processing. Continue adding the remaining rooms:

```python
        # Room 3: Crew Quarters
        room3 = Room("Crew Quarters", (40, 50, 70),
                      "Bunk beds line the walls. A locker hangs open.")
        room3.clickable_areas.append(
            ClickableArea((630, 250, 150, 60), "East Door", "go_2")
        )
        room3.clickable_areas.append(
            ClickableArea((200, 300, 140, 40), "Blue Keycard", "pickup_blue_keycard")
        )
        self.rooms[3] = room3

        # Room 4: Science Lab
        room4 = Room("Science Lab", (30, 60, 50),
                      "Beakers and screens everywhere. Something sparks.")
        room4.clickable_areas.append(
            ClickableArea((20, 250, 150, 60), "West Door", "go_2")
        )
        room4.clickable_areas.append(
            ClickableArea((500, 200, 140, 40), "Circuit Board", "pickup_circuit_board")
        )
        self.rooms[4] = room4

        # Room 5: Storage Room
        room5 = Room("Storage Room", (50, 45, 40),
                      "Crates and supplies. Dusty but useful.")
        room5.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_2")
        )
        room5.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_6")
        )
        room5.clickable_areas.append(
            ClickableArea((100, 300, 120, 40), "Wrench", "pickup_wrench")
        )
        self.rooms[5] = room5

        # Room 6: Generator Room
        room6 = Room("Generator Room", (40, 30, 30),
                      "A massive generator sits in the center, dark and silent.")
        room6.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_5")
        )
        room6.clickable_areas.append(
            ClickableArea((250, 150, 300, 200), "Generator", "fix_generator",
                          GRAY)
        )
        self.rooms[6] = room6
```

That is all six rooms. Each one has doors connecting to neighboring rooms and items or objects to interact with.

### Processing Actions

When the player clicks a clickable area, we get an action string like `"go_2"` or `"pickup_blue_keycard"`. We need a method that processes these actions. Add this to the `Game` class:

```python
    def process_action(self, action):
        # Movement
        if action.startswith("go_") and "locked" not in action:
            room_id = int(action.split("_")[1])
            self.current_room_id = room_id
            self.message = self.rooms[room_id].description
            self.message_timer = 180
            return
```

If the action starts with `"go_"` and is not locked, we extract the room number and move there. The `split("_")` breaks `"go_2"` into `["go", "2"]`, and we take the second part.

Now add handling for locked doors, still inside `process_action`:

```python
        # Locked doors
        if action == "go_4_locked":
            if "blue_keycard" in self.inventory:
                self.current_room_id = 4
                self.message = "You swipe the blue keycard. The door opens."
                self.message_timer = 180
            else:
                self.message = "This door is locked. You need a keycard."
                self.message_timer = 180
            return

        if action == "go_5_locked":
            if "red_keycard" in self.inventory:
                self.current_room_id = 5
                self.message = "You swipe the red keycard. The door opens."
                self.message_timer = 180
            else:
                self.message = "This door is locked. You need a keycard."
                self.message_timer = 180
            return
```

For locked doors, we check if the player has the right keycard in their inventory. If yes, they move through. If not, they get a message.

Now add item pickups:

```python
        # Item pickups
        if action == "pickup_red_keycard":
            if "red_keycard" not in self.inventory:
                self.inventory.append("red_keycard")
                self.message = "Picked up: Red Keycard"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_blue_keycard":
            if "blue_keycard" not in self.inventory:
                self.inventory.append("blue_keycard")
                self.message = "Picked up: Blue Keycard"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_circuit_board":
            if "circuit_board" not in self.inventory:
                self.inventory.append("circuit_board")
                self.message = "Picked up: Circuit Board"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_wrench":
            if "wrench" not in self.inventory:
                self.inventory.append("wrench")
                self.message = "Picked up: Wrench"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return
```

Each pickup checks if the player already has the item. If not, it adds it to the inventory list and shows a message.

> You might notice these four pickup blocks look very similar. In a larger game, you could write a single function that handles all pickups. For now, the repetition keeps things clear and easy to follow.

Finally, add the generator fix -- the win condition:

```python
        # Fix the generator (win condition)
        if action == "fix_generator":
            if "circuit_board" in self.inventory and "wrench" in self.inventory:
                self.game_won = True
                self.message = "You install the circuit board and tighten it with the wrench. The generator roars to life! Power is restored!"
                self.message_timer = 600
            elif "circuit_board" in self.inventory:
                self.message = "You have the circuit board, but you need a tool to install it."
                self.message_timer = 180
            elif "wrench" in self.inventory:
                self.message = "You have a wrench, but the circuit board is missing."
                self.message_timer = 180
            else:
                self.message = "The generator is broken. You need parts to fix it."
                self.message_timer = 180
```

The generator requires both the circuit board and the wrench. The game gives helpful hints depending on what the player has so far.

That is a lot of code. This is a good stopping point. Save your work and take a break if you need one.

---

## Part 3: Inventory, Messages, and Drawing

Now we need to see the inventory on screen, display messages, and tie the drawing together.

### Drawing the Inventory Bar

The inventory bar sits at the bottom of the screen. Add this method to the `Game` class:

```python
    def draw_inventory(self, screen):
        # Background bar
        inv_rect = pygame.Rect(0, HEIGHT - 80, WIDTH, 80)
        pygame.draw.rect(screen, (30, 30, 50), inv_rect)
        pygame.draw.line(screen, WHITE, (0, HEIGHT - 80), (WIDTH, HEIGHT - 80), 2)

        # Label
        label = self.small_font.render("Inventory:", True, WHITE)
        screen.blit(label, (10, HEIGHT - 72))

        # Draw each item
        item_names = {
            "red_keycard": ("Red Keycard", RED),
            "blue_keycard": ("Blue Keycard", BLUE),
            "circuit_board": ("Circuit Board", GREEN),
            "wrench": ("Wrench", LIGHT_GRAY),
        }

        x = 120
        for item in self.inventory:
            name, color = item_names[item]
            pygame.draw.rect(screen, color, (x, HEIGHT - 65, 100, 40))
            pygame.draw.rect(screen, WHITE, (x, HEIGHT - 65, 100, 40), 2)
            text = self.small_font.render(name, True, BLACK)
            screen.blit(text, (x + 5, HEIGHT - 55))
            x += 120
```

This draws a dark bar across the bottom, with a label and a colored rectangle for each item the player has collected. The `item_names` dictionary maps internal names to display names and colors.

### Drawing Messages

Messages appear in a text box above the inventory. Add this method:

```python
    def draw_message(self, screen):
        if self.message_timer > 0:
            self.message_timer -= 1

            # Message background
            msg_rect = pygame.Rect(20, HEIGHT - 130, WIDTH - 40, 40)
            pygame.draw.rect(screen, (20, 20, 40), msg_rect)
            pygame.draw.rect(screen, YELLOW, msg_rect, 2)

            # Message text
            text = self.small_font.render(self.message, True, YELLOW)
            screen.blit(text, (30, HEIGHT - 122))
```

The message shows when `message_timer` is above zero. Each frame, the timer counts down by one. At 60 frames per second, a timer of 180 lasts about 3 seconds.

### Drawing the Win Screen

When the player fixes the generator, we show a victory message:

```python
    def draw_win_screen(self, screen):
        screen.fill((10, 40, 10))
        title = self.font.render("POWER RESTORED!", True, GREEN)
        screen.blit(title, (WIDTH // 2 - title.get_width() // 2, 200))

        msg = self.font.render("You fixed the generator. The space station is saved!", True, WHITE)
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, 260))

        hint = self.small_font.render("Close the window to exit.", True, GRAY)
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 320))
```

### The Main Draw Method

Add one more method to `Game` that calls everything in the right order:

```python
    def draw(self, screen):
        if self.game_won:
            self.draw_win_screen(screen)
            return

        room = self.rooms[self.current_room_id]
        room.draw(screen, self.font)
        self.draw_inventory(screen)
        self.draw_message(screen)
```

### The Click Handler

We also need a method that handles clicks. Add this:

```python
    def handle_click(self, mouse_pos):
        if self.game_won:
            return

        room = self.rooms[self.current_room_id]
        for area in room.clickable_areas:
            if area.rect.collidepoint(mouse_pos):
                self.process_action(area.action)
                return
```

### The Updated Game Loop

Now replace the old game loop with this cleaner version that uses the `Game` class:

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

    game.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Run it. You should be able to click doors to move between rooms, pick up items, and see your inventory grow. Try to solve the puzzles: get both keycards, unlock both doors, collect the circuit board and wrench, and fix the generator.

This is a good stopping point. Save your work and test everything.

---

## Part 4: Polish and the Complete Game

The game works, but let's add a few touches to make it feel more finished.

### Drawing Room Decorations

Right now every room is just a colored background. Let's add some simple shape-based decorations to make each room look different. Add a `draw_decorations` method to the `Room` class:

```python
class Room:
    def __init__(self, name, bg_color, description):
        self.name = name
        self.bg_color = bg_color
        self.description = description
        self.clickable_areas = []
        self.decorations = []

    def draw(self, screen, font):
        screen.fill(self.bg_color)

        # Draw decorations
        for deco in self.decorations:
            deco_type, color, rect = deco
            if deco_type == "rect":
                pygame.draw.rect(screen, color, rect)
            elif deco_type == "circle":
                pygame.draw.circle(screen, color, (rect[0], rect[1]), rect[2])

        # Draw room name at the top
        title = font.render(self.name, True, WHITE)
        screen.blit(title, (20, 10))

        # Draw clickable areas
        for area in self.clickable_areas:
            pygame.draw.rect(screen, area.color, area.rect)
            pygame.draw.rect(screen, WHITE, area.rect, 2)
            label = font.render(area.label, True, BLACK)
            label_x = area.rect.x + (area.rect.width - label.get_width()) // 2
            label_y = area.rect.y + (area.rect.height - label.get_height()) // 2
            screen.blit(label, (label_x, label_y))
```

Now you can add decorations to rooms. For example, in `create_rooms`, after creating room 3:

```python
        room3.decorations.append(("rect", (70, 60, 80), (50, 150, 200, 120)))
        room3.decorations.append(("rect", (70, 60, 80), (50, 290, 200, 120)))
        room3.decorations.append(("rect", (80, 80, 90), (300, 200, 150, 180)))
```

These add rectangles that look like bunk beds and a locker. Add decorations to the other rooms too -- use rectangles for crates in the storage room, circles for control panels in the airlock, whatever you like. This is your game. Get creative.

### A Hint System

If the player seems stuck, a hint is nice. Add a clickable area in each room or a keyboard shortcut. Here is a simple version -- add this to the event handling in the game loop:

```python
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_h:
                game.show_hint()
```

And add the `show_hint` method to the `Game` class:

```python
    def show_hint(self):
        if self.game_won:
            return

        if len(self.inventory) == 0:
            self.message = "Hint: Look around the rooms for useful items."
        elif "red_keycard" not in self.inventory:
            self.message = "Hint: Check the Airlock for a keycard."
        elif "blue_keycard" not in self.inventory:
            self.message = "Hint: The Crew Quarters might have something."
        elif "circuit_board" not in self.inventory:
            self.message = "Hint: The Science Lab has electronics."
        elif "wrench" not in self.inventory:
            self.message = "Hint: Check the Storage Room for tools."
        else:
            self.message = "Hint: Head to the Generator Room."
        self.message_timer = 240
```

### Checkpoint: The Complete Game

Here is the full program. This is the longest listing in the book, but you built every piece of it step by step.

```python
import pygame

pygame.init()

WIDTH = 800
HEIGHT = 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Space Station Adventure")

clock = pygame.time.Clock()

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
GRAY = (100, 100, 100)
DARK_GRAY = (60, 60, 60)
LIGHT_GRAY = (180, 180, 180)
BLUE = (50, 100, 200)
RED = (200, 50, 50)
GREEN = (50, 200, 50)
YELLOW = (220, 220, 50)
DARK_BLUE = (20, 30, 60)


class ClickableArea:
    def __init__(self, rect, label, action, color=LIGHT_GRAY):
        self.rect = pygame.Rect(rect)
        self.label = label
        self.action = action
        self.color = color


class Room:
    def __init__(self, name, bg_color, description):
        self.name = name
        self.bg_color = bg_color
        self.description = description
        self.clickable_areas = []
        self.decorations = []

    def draw(self, screen, font):
        screen.fill(self.bg_color)

        # Draw decorations
        for deco in self.decorations:
            deco_type, color, rect = deco
            if deco_type == "rect":
                pygame.draw.rect(screen, color, rect)
            elif deco_type == "circle":
                pygame.draw.circle(screen, color, (rect[0], rect[1]), rect[2])

        # Draw room name at the top
        title = font.render(self.name, True, WHITE)
        screen.blit(title, (20, 10))

        # Draw clickable areas
        for area in self.clickable_areas:
            pygame.draw.rect(screen, area.color, area.rect)
            pygame.draw.rect(screen, WHITE, area.rect, 2)
            label = font.render(area.label, True, BLACK)
            label_x = area.rect.x + (area.rect.width - label.get_width()) // 2
            label_y = area.rect.y + (area.rect.height - label.get_height()) // 2
            screen.blit(label, (label_x, label_y))


class Game:
    def __init__(self):
        self.font = pygame.font.Font(None, 28)
        self.small_font = pygame.font.Font(None, 22)
        self.inventory = []
        self.message = "You wake up in the airlock. The station is dark. Find a way to restore power."
        self.message_timer = 300
        self.game_won = False
        self.rooms = {}
        self.current_room_id = 1
        self.create_rooms()

    def create_rooms(self):
        # Room 1: Airlock (start)
        room1 = Room("Airlock", DARK_BLUE,
                      "The airlock. Emergency lights cast a dim glow.")
        room1.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_2")
        )
        room1.clickable_areas.append(
            ClickableArea((600, 400, 120, 40), "Red Keycard", "pickup_red_keycard")
        )
        room1.decorations.append(("circle", GRAY, (100, 300, 30)))
        room1.decorations.append(("circle", GRAY, (150, 350, 20)))
        room1.decorations.append(("rect", DARK_GRAY, (650, 150, 80, 200)))
        self.rooms[1] = room1

        # Room 2: Main Corridor
        room2 = Room("Main Corridor", DARK_GRAY,
                      "A wide corridor. Doors lead in every direction.")
        room2.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_1")
        )
        room2.clickable_areas.append(
            ClickableArea((20, 250, 150, 60), "West Door", "go_3")
        )
        room2.clickable_areas.append(
            ClickableArea((630, 250, 150, 60), "East Door", "go_4_locked",
                          BLUE)
        )
        room2.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_5_locked",
                          RED)
        )
        room2.decorations.append(("rect", (80, 80, 80), (350, 200, 100, 10)))
        room2.decorations.append(("rect", (80, 80, 80), (350, 350, 100, 10)))
        self.rooms[2] = room2

        # Room 3: Crew Quarters
        room3 = Room("Crew Quarters", (40, 50, 70),
                      "Bunk beds line the walls. A locker hangs open.")
        room3.clickable_areas.append(
            ClickableArea((630, 250, 150, 60), "East Door", "go_2")
        )
        room3.clickable_areas.append(
            ClickableArea((200, 300, 140, 40), "Blue Keycard",
                          "pickup_blue_keycard")
        )
        room3.decorations.append(("rect", (70, 60, 80), (50, 150, 200, 120)))
        room3.decorations.append(("rect", (70, 60, 80), (50, 290, 200, 120)))
        room3.decorations.append(("rect", (80, 80, 90), (500, 200, 80, 180)))
        self.rooms[3] = room3

        # Room 4: Science Lab
        room4 = Room("Science Lab", (30, 60, 50),
                      "Beakers and screens everywhere. Something sparks.")
        room4.clickable_areas.append(
            ClickableArea((20, 250, 150, 60), "West Door", "go_2")
        )
        room4.clickable_areas.append(
            ClickableArea((500, 200, 140, 40), "Circuit Board",
                          "pickup_circuit_board")
        )
        room4.decorations.append(("rect", (50, 80, 60), (300, 100, 200, 80)))
        room4.decorations.append(("circle", GREEN, (350, 350, 15)))
        room4.decorations.append(("circle", RED, (420, 360, 10)))
        self.rooms[4] = room4

        # Room 5: Storage Room
        room5 = Room("Storage Room", (50, 45, 40),
                      "Crates and supplies. Dusty but useful.")
        room5.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_2")
        )
        room5.clickable_areas.append(
            ClickableArea((325, 50, 150, 60), "North Door", "go_6")
        )
        room5.clickable_areas.append(
            ClickableArea((100, 300, 120, 40), "Wrench", "pickup_wrench")
        )
        room5.decorations.append(("rect", (80, 70, 60), (500, 200, 100, 80)))
        room5.decorations.append(("rect", (80, 70, 60), (520, 300, 80, 80)))
        room5.decorations.append(("rect", (70, 60, 50), (200, 150, 120, 100)))
        self.rooms[5] = room5

        # Room 6: Generator Room
        room6 = Room("Generator Room", (40, 30, 30),
                      "A massive generator sits in the center, dark and silent.")
        room6.clickable_areas.append(
            ClickableArea((325, 450, 150, 60), "South Door", "go_5")
        )
        room6.clickable_areas.append(
            ClickableArea((250, 150, 300, 200), "Generator", "fix_generator",
                          GRAY)
        )
        room6.decorations.append(("rect", (60, 50, 50), (100, 100, 50, 300)))
        room6.decorations.append(("rect", (60, 50, 50), (650, 100, 50, 300)))
        self.rooms[6] = room6

    def process_action(self, action):
        # Movement to unlocked rooms
        if action.startswith("go_") and "locked" not in action:
            room_id = int(action.split("_")[1])
            self.current_room_id = room_id
            self.message = self.rooms[room_id].description
            self.message_timer = 180
            return

        # Locked door: Science Lab (needs blue keycard)
        if action == "go_4_locked":
            if "blue_keycard" in self.inventory:
                self.current_room_id = 4
                self.message = "You swipe the blue keycard. The door opens."
                self.message_timer = 180
            else:
                self.message = "This door is locked. You need a blue keycard."
                self.message_timer = 180
            return

        # Locked door: Storage Room (needs red keycard)
        if action == "go_5_locked":
            if "red_keycard" in self.inventory:
                self.current_room_id = 5
                self.message = "You swipe the red keycard. The door opens."
                self.message_timer = 180
            else:
                self.message = "This door is locked. You need a red keycard."
                self.message_timer = 180
            return

        # Item pickups
        if action == "pickup_red_keycard":
            if "red_keycard" not in self.inventory:
                self.inventory.append("red_keycard")
                self.message = "Picked up: Red Keycard"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_blue_keycard":
            if "blue_keycard" not in self.inventory:
                self.inventory.append("blue_keycard")
                self.message = "Picked up: Blue Keycard"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_circuit_board":
            if "circuit_board" not in self.inventory:
                self.inventory.append("circuit_board")
                self.message = "Picked up: Circuit Board"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        if action == "pickup_wrench":
            if "wrench" not in self.inventory:
                self.inventory.append("wrench")
                self.message = "Picked up: Wrench"
                self.message_timer = 180
            else:
                self.message = "You already have this."
                self.message_timer = 120
            return

        # Fix the generator (win condition)
        if action == "fix_generator":
            if "circuit_board" in self.inventory and "wrench" in self.inventory:
                self.game_won = True
                self.message = "The generator roars to life! Power restored!"
                self.message_timer = 600
            elif "circuit_board" in self.inventory:
                self.message = "You have the circuit board, but need a tool to install it."
                self.message_timer = 180
            elif "wrench" in self.inventory:
                self.message = "You have a wrench, but the circuit board is missing."
                self.message_timer = 180
            else:
                self.message = "The generator is broken. You need parts to fix it."
                self.message_timer = 180

    def handle_click(self, mouse_pos):
        if self.game_won:
            return

        room = self.rooms[self.current_room_id]
        for area in room.clickable_areas:
            if area.rect.collidepoint(mouse_pos):
                self.process_action(area.action)
                return

    def show_hint(self):
        if self.game_won:
            return

        if len(self.inventory) == 0:
            self.message = "Hint: Look around the rooms for useful items."
        elif "red_keycard" not in self.inventory:
            self.message = "Hint: Check the Airlock for a keycard."
        elif "blue_keycard" not in self.inventory:
            self.message = "Hint: The Crew Quarters might have something useful."
        elif "circuit_board" not in self.inventory:
            self.message = "Hint: The Science Lab has electronics you need."
        elif "wrench" not in self.inventory:
            self.message = "Hint: Check the Storage Room for tools."
        else:
            self.message = "Hint: You have everything. Head to the Generator Room."
        self.message_timer = 240

    def draw_inventory(self, screen):
        # Background bar
        inv_rect = pygame.Rect(0, HEIGHT - 80, WIDTH, 80)
        pygame.draw.rect(screen, (30, 30, 50), inv_rect)
        pygame.draw.line(screen, WHITE, (0, HEIGHT - 80), (WIDTH, HEIGHT - 80), 2)

        # Label
        label = self.small_font.render("Inventory:", True, WHITE)
        screen.blit(label, (10, HEIGHT - 72))

        # Draw each item
        item_names = {
            "red_keycard": ("Red Keycard", RED),
            "blue_keycard": ("Blue Keycard", BLUE),
            "circuit_board": ("Circuit Board", GREEN),
            "wrench": ("Wrench", LIGHT_GRAY),
        }

        x = 120
        for item in self.inventory:
            name, color = item_names[item]
            pygame.draw.rect(screen, color, (x, HEIGHT - 65, 100, 40))
            pygame.draw.rect(screen, WHITE, (x, HEIGHT - 65, 100, 40), 2)
            text = self.small_font.render(name, True, BLACK)
            screen.blit(text, (x + 5, HEIGHT - 55))
            x += 120

    def draw_message(self, screen):
        if self.message_timer > 0:
            self.message_timer -= 1

            # Message background
            msg_rect = pygame.Rect(20, HEIGHT - 130, WIDTH - 40, 40)
            pygame.draw.rect(screen, (20, 20, 40), msg_rect)
            pygame.draw.rect(screen, YELLOW, msg_rect, 2)

            # Message text
            text = self.small_font.render(self.message, True, YELLOW)
            screen.blit(text, (30, HEIGHT - 122))

    def draw_win_screen(self, screen):
        screen.fill((10, 40, 10))
        title = self.font.render("POWER RESTORED!", True, GREEN)
        screen.blit(title, (WIDTH // 2 - title.get_width() // 2, 200))

        msg = self.font.render(
            "You fixed the generator. The space station is saved!",
            True, WHITE)
        screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, 260))

        hint = self.small_font.render("Close the window to exit.", True, GRAY)
        screen.blit(hint, (WIDTH // 2 - hint.get_width() // 2, 320))

    def draw(self, screen):
        if self.game_won:
            self.draw_win_screen(screen)
            return

        room = self.rooms[self.current_room_id]
        room.draw(screen, self.font)
        self.draw_inventory(screen)
        self.draw_message(screen)


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

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_h:
                game.show_hint()

    game.draw(screen)
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Your code should now look like this. Compare it with what you have built so far and fix any differences. Run it. Play through the whole game. If something is not working, check for typos in the action strings -- they need to match exactly between `create_rooms` and `process_action`.

> Did you know? Professional game developers call the process of playing through your own game to find problems "playtesting." You are doing real playtesting right now.

## What You Learned
- **Planning matters.** Drawing a map and listing items and puzzles before coding saves time and headaches.
- **Clickable areas** use `pygame.Rect` and `collidepoint()` to detect mouse clicks.
- `pygame.MOUSEBUTTONDOWN` fires when the player clicks, and `pygame.mouse.get_pos()` gives the click position.
- A **room system** uses a dictionary of room objects and a `current_room_id` variable to track where the player is.
- An **inventory** is a list of strings. You check if an item is in the list with `in`.
- **Puzzle state** can be tracked by checking what is in the inventory and using boolean flags like `game_won`.
- A **message system** with a timer lets you display temporary text to the player.
- **Classes** like `Room`, `ClickableArea`, and `Game` keep a large program organized.
- Breaking a big project into small, testable pieces is the way professional programmers work.

## Quick Quiz

1. How do you check if the player clicked inside a rectangle in pygame?
2. If `self.inventory` is `["red_keycard", "wrench"]`, what does `"blue_keycard" in self.inventory` return?
3. Why do we use a `message_timer` instead of just always drawing the message?
4. In the room map, what two items do you need to fix the generator?

## Experiment

1. Add a seventh room to the space station. Connect it to an existing room and put something interesting in it -- maybe a window looking out into space, drawn with `pygame.draw.circle` for stars.
2. Change the theme entirely. Instead of a space station, make it a medieval castle, an underwater base, or a haunted library. Change the room names, colors, descriptions, and items to match.
3. Add a second use for one of the keycards. Maybe the blue keycard also opens a locker in the Crew Quarters that contains a bonus item.

## Chapter Challenge

Add a **map screen** that the player can open by pressing `M`. When the map is open, draw the ASCII room layout from the planning section as actual rectangles on screen. Highlight the room the player is currently in with a bright color. Press `M` again to close the map and go back to the game.

**Hint 1:** Add a `show_map` boolean to the `Game` class. Toggle it when `M` is pressed.

**Hint 2:** In the `draw` method, check `self.show_map`. If it is `True`, draw the map instead of the current room.

**Hint 3:** Each room on the map can be a small rectangle at a fixed position. Use `pygame.draw.rect()` for each one and `pygame.draw.line()` to connect them. Draw the current room's rectangle in a different color.
