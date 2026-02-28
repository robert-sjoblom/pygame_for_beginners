---
layout: default
title: "Chapter 10: Sprites and Art"
nav_order: 4
parent: "Part 2: Making Games with Pygame"
---

# Chapter 10: Sprites and Art

## What You Will Learn
- How to load images and draw them on screen instead of plain shapes
- What a **sprite** is and where the term comes from
- How to scale images and work with transparency
- How to create simple animation with multiple frames
- How to create your own pixel art with LibreSprite

## From Rectangles to Real Art

So far, every game we have built uses rectangles and circles. The paddles in Pong were white rectangles. The snake was a chain of green squares. That is fine for learning, but real games use images -- pictures of characters, backgrounds, and objects.

In this chapter, you will learn how to load image files into pygame and draw them on screen. By the end, you will be able to replace the plain shapes in your games with actual art.

## What Is a Sprite?

The word **sprite** comes from early video game hardware in the 1970s and 1980s. Machines like the Commodore 64 had special chips that could draw small images on screen and move them around independently from the background. Engineers called these small moving images "sprites."

Today, the word sprite means any 2D image used in a game -- a character, an enemy, a coin, a tree, a fireball. When someone says "I need to draw the player sprite," they mean "I need to draw the player's image on screen."

## Loading an Image

Pygame can load image files and turn them into something it can draw. Here is how:

```python
player_image = pygame.image.load("player.png")
```

That one line reads the file `player.png` from your computer and stores it as a pygame **Surface** (more on Surfaces soon).

### File Formats

Pygame supports several image formats, but **PNG** is the best choice. PNG files support **transparency** -- parts of the image can be see-through. This matters because game sprites are rarely perfect rectangles. A character has arms, legs, and space between them. Without transparency, that space would show up as a colored block.

```
  With transparency:       Without transparency:

  . . # # . .             X X # # X X
  . # # # # .             X # # # # X
  # # # # # #             # # # # # #
  . . # # . .             X X # # X X
  . # . . # .             X # X X # X

  . = see-through          X = ugly background color
  # = visible pixels        # = visible pixels
```

Use PNG for sprites. Avoid JPG -- it does not support transparency and it blurs pixel art.

### Where to Put Image Files

Keep your image files in the same folder as your Python file. If you have many images, create a subfolder called `images` and load from there:

```python
# If the image is in the same folder as your .py file:
player_image = pygame.image.load("player.png")

# If the image is in an "images" subfolder:
player_image = pygame.image.load("images/player.png")
```

Pick one approach and stick with it. For this book, we will put images in an `images` subfolder to keep things organized.

---

## Drawing an Image on Screen

To draw a loaded image, use `blit()`:

```python
screen.blit(player_image, (100, 200))
```

The word **blit** stands for **Block Image Transfer**. It is a term from the 1970s that means "copy a block of pixels from one place to another." When you blit an image onto the screen, you are copying its pixels onto the screen surface at the position you specify.

The position `(100, 200)` is where the **top-left corner** of the image will be placed.

```
  (0,0)
    +----------------------------------+
    |                                  |
    |        (100, 200)                |
    |          +--------+              |
    |          | player |              |
    |          | image  |              |
    |          +--------+              |
    |                                  |
    +----------------------------------+
```

### A Minimal Example

Let's write a small program to test loading and drawing an image. You will need an image file to use. If you do not have one yet, skip ahead to the LibreSprite section to create one, or download any small PNG image from the internet.

Create a file called `sprite_test.py`:

```python
import pygame

pygame.init()

screen = pygame.display.set_mode((600, 400))
pygame.display.set_caption("Sprite Test")
clock = pygame.time.Clock()

player_image = pygame.image.load("images/player.png")
player_image = player_image.convert_alpha()

player_x = 100
player_y = 150

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill((30, 30, 50))
    screen.blit(player_image, (player_x, player_y))
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

Notice the line `player_image = player_image.convert_alpha()`. The `convert_alpha()` method tells pygame to handle the image's transparency properly. Always call this right after loading a PNG that has transparent parts. If your image has no transparency, you can use `convert()` instead, which is slightly faster.

> If you see a `FileNotFoundError`, double-check the file name and make sure the image is in the right folder. The file name is case-sensitive on Linux: `Player.png` is not the same as `player.png`.

---

## Scaling Images

Sometimes your image is the wrong size. Maybe you drew a 16x16 pixel character but want it to appear bigger on screen. Pygame can resize images:

```python
player_image = pygame.image.load("images/player.png").convert_alpha()
player_image = pygame.transform.scale(player_image, (64, 64))
```

`pygame.transform.scale()` takes two arguments: the image and a tuple with the new width and height. This scales the 16x16 image up to 64x64 pixels.

A few things to keep in mind:

- Scaling **up** makes pixel art look blocky (which can actually look good for retro-style games).
- Scaling **down** can make images blurry or lose detail.
- It is best to create your art at the size you actually need.

You can also scale by a multiplier. If you want to double the size of an image:

```python
width = player_image.get_width() * 2
height = player_image.get_height() * 2
player_image = pygame.transform.scale(player_image, (width, height))
```

## Transparency in Practice

Let's see transparency in action. Change `sprite_test.py` to draw a colored rectangle behind the player, so you can see whether the transparency is working:

```python
    screen.fill((30, 30, 50))

    # Draw a bright rectangle behind the player
    pygame.draw.rect(screen, (200, 200, 0), (player_x - 10,
                     player_y - 10, 80, 80))

    screen.blit(player_image, (player_x, player_y))
    pygame.display.flip()
```

If transparency is working, you will see the yellow rectangle showing through the transparent parts of your sprite. If the transparent parts show up as a solid color instead, make sure you called `convert_alpha()` when loading the image.

---

## Simple Animation

Animation in games works the same way as a flipbook. You draw slightly different pictures and show them one after another, fast enough that it looks like movement.

```
  Frame 1      Frame 2      Frame 3      Frame 4

  . # # .      . # # .      . # # .      . # # .
  # # # #      # # # #      # # # #      # # # #
  . # # .      . # # .      . # # .      . # # .
  # .  . #     . # # .      . #  # .     # .  . #
  #    . #     # .  . #     #    . #     . # # .
  (standing)   (step 1)     (step 2)     (step 1)
```

In pygame, we store each frame as a separate image and switch between them on a timer.

### Loading Multiple Frames

```python
walk_frames = []
walk_frames.append(pygame.image.load("images/walk_0.png").convert_alpha())
walk_frames.append(pygame.image.load("images/walk_1.png").convert_alpha())
walk_frames.append(pygame.image.load("images/walk_2.png").convert_alpha())
walk_frames.append(pygame.image.load("images/walk_3.png").convert_alpha())
```

Or, more concisely using a loop:

```python
walk_frames = []
for i in range(4):
    image = pygame.image.load(f"images/walk_{i}.png").convert_alpha()
    walk_frames.append(image)
```

The `f"images/walk_{i}.png"` part is an **f-string**. The `{i}` gets replaced with the value of `i`, so it loads `walk_0.png`, `walk_1.png`, `walk_2.png`, and `walk_3.png`.

### Switching Frames with a Timer

We need two variables: one to track which frame we are showing, and one to track when to switch to the next frame.

```python
current_frame = 0
animation_timer = 0
animation_speed = 150  # milliseconds between frames
```

In the game loop, we update the timer and switch frames:

```python
    animation_timer += clock.get_time()
    if animation_timer >= animation_speed:
        animation_timer = 0
        current_frame = (current_frame + 1) % len(walk_frames)

    screen.blit(walk_frames[current_frame], (player_x, player_y))
```

`clock.get_time()` returns how many milliseconds passed since the last frame. We add that to our timer. When the timer reaches 150 milliseconds, we reset it and advance to the next frame.

The `%` operator (modulo) is doing something clever here. When `current_frame` reaches 4 (the length of our list), `4 % 4` gives 0, wrapping back to the first frame. This creates an endless loop through all the frames.

```
  current_frame:  0 -> 1 -> 2 -> 3 -> 0 -> 1 -> 2 -> 3 -> 0 ...
                  (wraps around because of %)
```

> This animation technique is the same one used in classic games like the original Super Mario Bros. Even games with hundreds of frames of animation still work this way -- they just have more images in their lists.

---

## Creating Sprites with LibreSprite

You do not need to be an artist to make game sprites. Plenty of professional games use simple pixel art. Let's create some using **LibreSprite**, a free pixel art editor.

### Installing LibreSprite

LibreSprite is not in Debian's default package list, so we install it using Flatpak:

```
flatpak install flathub com.github.libresprite.LibreSprite
```

If Flatpak is not set up on your computer, you can install it with `sudo apt install flatpak` first. Ask an adult for help if you get stuck.

To open LibreSprite after installation, type:

```
flatpak run com.github.libresprite.LibreSprite
```

### Making Your First Sprite

Open LibreSprite. You will see a canvas and a toolbar.

1. Go to **File > New** in the menu bar.
2. Set the width and height to **32 x 32** pixels. This is a common size for game sprites.
3. Click OK.

The canvas will look tiny. Use **View > Zoom In** (or scroll with `Ctrl` held down) to zoom in until you can see individual pixels clearly.

Now draw a simple character. Here are some pixel art tips:

- **Keep it small.** 32x32 pixels is plenty. Some classic games used 8x8 or 16x16.
- **Use few colors.** Pick 3-5 colors at most. A body color, an outline color, and one or two accent colors.
- **Start with the outline.** Draw the shape of your character first, then fill it in.
- **Symmetry helps.** Most characters look better if the left and right sides are roughly symmetrical.

There is no wrong way to draw a game sprite. Professional game developers started with simple pixel art too. The sprites in the first Legend of Zelda game were 16x16 pixels with three colors.

When you are happy with your sprite, save it:

1. Go to **File > Export** in the menu bar.
2. Choose PNG as the format.
3. Save it in your project's `images` folder.

> You can also find free game art online. Search for "free pixel art sprites" and you will find thousands of images released by artists for anyone to use. The website opengameart.org is a good place to start.

---

## Surfaces: The Big Picture

Everything you see on screen in pygame is a **Surface**. The screen itself is a Surface. Every image you load becomes a Surface. Even the text you render with fonts is a Surface.

A Surface is just a grid of pixels that pygame can work with. When you call `screen.blit(image, position)`, you are copying pixels from one Surface (the image) onto another Surface (the screen).

```
  player_image          screen
  (a Surface)           (also a Surface)

  +---------+           +-------------------------+
  | # . # . |           |                         |
  | . # . # |  blit ->  |     +---------+         |
  | # . # . |           |     | # . # . |         |
  | . # . # |           |     | . # . # |         |
  +---------+           |     | # . # . |         |
                        |     | . # . # |         |
                        |     +---------+         |
                        |                         |
                        +-------------------------+
```

You can create empty Surfaces too, which is useful for building images out of pieces:

```python
my_surface = pygame.Surface((100, 100))
my_surface.fill((0, 0, 255))
```

This creates a 100x100 blue square. You could blit other images onto it, then blit the whole thing onto the screen.

## Getting the Rect for Collisions

When we built Pong, we used `pygame.Rect` objects for collision detection. Images work with rects too. Every Surface has a `get_rect()` method:

```python
player_image = pygame.image.load("images/player.png").convert_alpha()
player_rect = player_image.get_rect()
```

By default, the rect is positioned at `(0, 0)`. You can place it wherever you want:

```python
player_rect = player_image.get_rect(topleft=(100, 200))
```

Or move it later:

```python
player_rect.x = 100
player_rect.y = 200
```

Now you can use `player_rect` for collision detection just like in Pong, and also use it for drawing:

```python
screen.blit(player_image, player_rect)
```

This blits the image at the rect's position. If you move the rect, the image moves with it.

---

## Putting It Together: Upgrading Pong

Let's take the Pong game from earlier and replace the shapes with images. This is a good exercise because you already understand how the game works -- now you are just changing how it looks.

You will need three images in your `images` folder:
- `paddle.png` -- a paddle sprite (around 20x80 pixels)
- `ball.png` -- a ball sprite (around 20x20 pixels)
- `background.png` -- a background image (600x400 or whatever your window size is)

If you do not have images yet, open LibreSprite and draw simple ones. A paddle can be a plain colored rectangle with a border. A ball can be a circle with shading. A background can be a dark color with a center line.

### Loading the Images

At the top of your Pong code, after creating the screen, load the images:

```python
paddle_image = pygame.image.load("images/paddle.png").convert_alpha()
ball_image = pygame.image.load("images/ball.png").convert_alpha()
background = pygame.image.load("images/background.png").convert()
```

Notice the background uses `convert()` instead of `convert_alpha()`. Backgrounds fill the entire screen, so they do not need transparency.

### Replacing the Drawing Code

Everywhere you used `pygame.draw.rect()` to draw a paddle, replace it with a blit:

```python
# Old:
pygame.draw.rect(screen, WHITE, paddle_left)

# New:
screen.blit(paddle_image, paddle_left)
```

Replace the ball drawing too:

```python
# Old:
pygame.draw.rect(screen, WHITE, ball)

# New:
screen.blit(ball_image, ball)
```

And replace `screen.fill()` with the background image:

```python
# Old:
screen.fill(BLACK)

# New:
screen.blit(background, (0, 0))
```

The collision detection and movement code stays exactly the same. The rects still handle all the math. We only changed what gets drawn on screen.

> This separation between game logic and visuals is an important idea in programming. The game does not care whether a paddle is a white rectangle or a detailed sprite. It only cares about positions and collisions. The visuals are a layer on top.

---

## What You Learned

- **Sprites** are 2D images used in games. The term dates back to early arcade hardware.
- `pygame.image.load()` loads an image file into a pygame Surface.
- `screen.blit(image, position)` copies an image onto the screen. Blit stands for Block Image Transfer.
- **PNG** is the best format for game sprites because it supports transparency.
- `convert_alpha()` enables proper transparency handling. Use `convert()` for images without transparency.
- `pygame.transform.scale()` resizes images.
- Animation works by cycling through a list of images on a timer, like a flipbook.
- A **Surface** is pygame's representation of any image or drawable area. The screen is a Surface, loaded images are Surfaces, and rendered text is a Surface.
- `image.get_rect()` gives you a Rect for positioning and collision detection.

## Quick Quiz

1. What does `blit` stand for?

2. Why should you use PNG instead of JPG for game sprites?

3. You load an image and draw it on screen, but the transparent parts show up as a black rectangle. What did you forget to do?

4. You have 6 animation frames in a list. The current frame is 5. What does `(5 + 1) % 6` equal, and why is that useful?

## Experiment

1. **Scale test.** Load a small image (16x16 or 32x32) and scale it to different sizes: 64x64, 128x128, 256x256. Notice how pixel art looks when scaled up. Some people like the blocky look.

2. **Animation speed.** If you built the walking animation, try changing `animation_speed` from 150 to 50 (very fast) and then to 500 (very slow). Find a speed that looks natural.

3. **Flip it.** Look up `pygame.transform.flip()` in the pygame documentation. Can you make a sprite face left when moving left and right when moving right?

## Chapter Challenge

**Animate the Snake.**

Go back to your Snake game from Chapter 9. Replace the plain rectangles with sprites:

1. Create (or find) three images: a head sprite, a body sprite, and a food sprite. Make them the same size as `CELL_SIZE` in your game (30x30 pixels).
2. Load the images at the top of `snake.py`.
3. Replace the `draw_snake` function so it blits the head image for `snake[0]` and the body image for every other segment.
4. Replace the `draw_food` function so it blits the food image.

Hints:

1. Load the images after `pygame.init()` and before the game loop.
2. In `draw_snake`, use `enumerate` just like before. When `i == 0`, blit the head image. Otherwise, blit the body image.
3. The position math is the same: `x = segment[0] * CELL_SIZE` and `y = segment[1] * CELL_SIZE`.

Extra challenge: Can you rotate the head sprite to face the direction the snake is moving? Look up `pygame.transform.rotate()`. You will need to check the `direction` variable: `[1, 0]` means facing right (0 degrees), `[0, -1]` means facing up (90 degrees), and so on.
