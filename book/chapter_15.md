---
layout: default
title: "Chapter 15: Polish and Sharing"
nav_order: 9
parent: "Part 2: Making Games with Pygame"
---

# Chapter 15: Polish, Sharing, and What Comes Next

## What You Will Learn
- How to add sound effects and background music to your games
- How to build title screens, menus, and game over screens using game states
- How to save and load high scores using files
- How to add screen shake and particle effects to make your game feel alive
- How to share your games with friends, including building standalone executables
- Where to go next on your programming journey

## Welcome to the Final Chapter

You have built games. Multiple games. You went from opening the terminal for the first time to writing programs with collision detection, game states, and real-time graphics. That is a serious accomplishment.

This chapter is different from the others. Instead of building one big project, we are going to cover a collection of useful topics that apply to any game you make. Think of it as a toolbox of techniques you can reach for whenever you need them.

We will start by adding polish to make your games feel more professional. Then we will cover how to share your games with other people. Finally, we will talk about what comes next.

---

## Part 1: Adding Polish

Professional games feel good to play. The difference between a rough prototype and a polished game often has nothing to do with the core gameplay. It is the little things: a sound when you score a point, a menu screen, a screen shake when something explodes. These details make players feel like the game is responding to them.

### Sound Effects and Music

Games without sound feel empty. Even the simplest boop when a ball bounces off a paddle makes the game feel ten times better.

Pygame has two sound systems built in:

- `pygame.mixer` for **sound effects** (short sounds like beeps, explosions, and clicks)
- `pygame.mixer.music` for **background music** (longer audio that plays continuously)

#### Loading and Playing Sound Effects

Before using sounds, you need to make sure the mixer is initialized. If you are already calling `pygame.init()`, this happens automatically. Here is how to load and play a sound effect:

```python
pygame.init()

# Load a sound from a file
boop_sound = pygame.mixer.Sound("boop.wav")

# Play the sound
boop_sound.play()
```

That is it. Two lines: one to load, one to play. You can call `play()` as many times as you want, and pygame will layer the sounds if they overlap.

You can also control the volume:

```python
boop_sound.set_volume(0.5)  # 0.0 is silent, 1.0 is full volume
```

#### Background Music

Background music works differently because it streams from the file instead of loading the whole thing into memory. This makes it better for longer audio tracks.

```python
pygame.mixer.music.load("background_music.ogg")
pygame.mixer.music.set_volume(0.3)
pygame.mixer.music.play(-1)  # -1 means loop forever
```

The `-1` argument tells pygame to repeat the music endlessly. If you pass `0`, it plays once and stops. If you pass `3`, it plays four times total (once plus three repeats).

To stop the music:

```python
pygame.mixer.music.stop()
```

#### What Audio Formats Work?

Pygame works best with `.wav` files for sound effects and `.ogg` files for music. The `.mp3` format sometimes causes problems depending on your system, so stick to `.wav` and `.ogg` when you can.

> You can convert audio files between formats using a free program called Audacity, which is available on Debian. Install it with `sudo apt install audacity`.

#### Where to Find Free Game Sounds

You do not need to record your own sounds. Many websites offer free sound effects and music for game developers:

- **freesound.org** -- A huge library of free sound effects. You need to create an account, but it is free. Search for "beep", "explosion", "coin", or whatever you need.
- **opengameart.org** -- Free art and audio made specifically for games. Everything here is licensed for use in your projects.

When you use sounds from these sites, check the **license**. Most are free to use, but some ask you to credit the creator. It is good practice to always give credit.

#### Example: Adding Sound to Pong

Let's add sound effects to the Pong game from earlier chapters. You need two sound files: one for when the ball hits a paddle, and one for when a player scores. For now, you can create simple placeholder sounds or download them from freesound.org.

Assume you have two files in the same folder as your game: `paddle_hit.wav` and `score.wav`.

At the top of your Pong game, after `pygame.init()`:

```python
paddle_hit_sound = pygame.mixer.Sound("paddle_hit.wav")
paddle_hit_sound.set_volume(0.6)

score_sound = pygame.mixer.Sound("score.wav")
score_sound.set_volume(0.8)
```

Then, in the part of your code where the ball bounces off a paddle:

```python
if ball_rect.colliderect(left_paddle_rect):
    ball_speed_x = abs(ball_speed_x)
    paddle_hit_sound.play()
```

And where a player scores:

```python
if ball_x < 0:
    right_score += 1
    score_sound.play()
    reset_ball()
```

Run the game. You should hear a sound every time the ball hits a paddle and every time someone scores. It is a small change in code but a big change in how the game feels.

---

### Title Screens and Menus

You already built a simple menu in the Snake game using game states. Let's look at a more reusable version of that pattern.

The idea is the same: a variable called `game_state` controls which screen the game shows. But this time, we will add a **highlighted selection** so the player can choose between options.

```python
game_state = "menu"
menu_options = ["Play", "Quit"]
selected_option = 0
```

Here is how the menu state draws and handles input:

```python
if game_state == "menu":
    title = big_font.render("MY GAME", True, WHITE)
    screen.blit(title, (WIDTH // 2 - title.get_width() // 2,
                        100))

    for i, option in enumerate(menu_options):
        if i == selected_option:
            color = YELLOW
        else:
            color = GRAY
        text = font.render(option, True, color)
        screen.blit(text, (WIDTH // 2 - text.get_width() // 2,
                           250 + i * 50))
```

The currently selected option is drawn in yellow, and the others are grey. The player changes their selection with the arrow keys:

```python
if event.type == pygame.KEYDOWN:
    if game_state == "menu":
        if event.key == pygame.K_UP:
            selected_option -= 1
            if selected_option < 0:
                selected_option = len(menu_options) - 1

        if event.key == pygame.K_DOWN:
            selected_option += 1
            if selected_option >= len(menu_options):
                selected_option = 0

        if event.key == pygame.K_RETURN:
            if menu_options[selected_option] == "Play":
                game_state = "playing"
            elif menu_options[selected_option] == "Quit":
                running = False
```

Notice how we handle wrapping: if the player presses Up when the first item is selected, the selection wraps to the last item. If they press Down on the last item, it wraps to the first. This feels natural.

Press `Enter` (which pygame calls `K_RETURN`) to confirm the selection.

```
  +-------------------+
  |     MY GAME       |
  |                   |
  |   > Play          |  <-- yellow (selected)
  |     Quit          |  <-- grey
  |                   |
  +-------------------+
```

This pattern works for any menu. Add more options to the `menu_options` list and add more `elif` checks for each one.

---

### Game Over Screens

A good game over screen does three things: shows the player what happened (their score), lets them try again, and optionally tracks high scores.

You have already seen the basic pattern in Snake. Let's expand it with high score tracking that saves to a file, so the high score survives even after the player closes the game.

#### Saving a High Score to a File

Python can read and write files with the built-in `open()` function. This is called **file I/O** (input/output). Here is how to save a high score:

```python
def save_high_score(score):
    file = open("high_score.txt", "w")
    file.write(str(score))
    file.close()
```

The `"w"` means "write". This creates the file if it does not exist, or overwrites it if it does. We convert the score to a string with `str()` because `write()` only accepts text.

Loading the high score back:

```python
def load_high_score():
    try:
        file = open("high_score.txt", "r")
        score = int(file.read())
        file.close()
        return score
    except (FileNotFoundError, ValueError):
        return 0
```

The `"r"` means "read". We convert the text back to a number with `int()`. The `try` and `except` block handles the case where the file does not exist yet (the very first time the game runs) or where the file contains something that is not a valid number (a corrupt file). If either problem occurs, we return 0.

> The `try`/`except` pattern is called **exception handling**. It means "try this code, and if a specific error happens, do this other thing instead of crashing." You will use it a lot as you write more programs.

#### Putting It Together

At the start of your game, load the high score:

```python
high_score = load_high_score()
```

When the game ends, check if the player beat the high score:

```python
if game_state == "game_over":
    if score > high_score:
        high_score = score
        save_high_score(high_score)
```

On the game over screen, display both scores:

```python
if game_state == "game_over":
    game_over_text = big_font.render("GAME OVER", True, RED)
    score_text = font.render(f"Score: {score}", True, WHITE)
    high_text = font.render(f"High Score: {high_score}", True, YELLOW)
    prompt = font.render("Press SPACE to play again", True, GRAY)

    center_x = WIDTH // 2
    screen.blit(game_over_text,
                (center_x - game_over_text.get_width() // 2, 120))
    screen.blit(score_text,
                (center_x - score_text.get_width() // 2, 200))
    screen.blit(high_text,
                (center_x - high_text.get_width() // 2, 240))
    screen.blit(prompt,
                (center_x - prompt.get_width() // 2, 320))
```

Now when the player gets a new high score, it is saved to `high_score.txt` in the game folder. The next time they start the game, it loads back automatically.

---

### Screen Shake and Simple Effects

These are the kinds of details that make people say "this game feels amazing" without knowing exactly why. Professional game developers call this **game feel** or **juice**.

#### Screen Shake

Screen shake is surprisingly easy. When something dramatic happens (a collision, an explosion, a big score), you briefly offset everything you draw by a small random amount.

Here is a simple screen shake system:

```python
import random

shake_amount = 0
shake_x = 0
shake_y = 0
```

When you want to trigger a shake (for example, when the ball hits a wall):

```python
shake_amount = 10  # how many frames to shake
```

In your game loop, before drawing anything:

```python
if shake_amount > 0:
    shake_x = random.randint(-4, 4)
    shake_y = random.randint(-4, 4)
    shake_amount -= 1
else:
    shake_x = 0
    shake_y = 0
```

Then, when you draw things, add the offset:

```python
screen.blit(some_surface, (x + shake_x, y + shake_y))
pygame.draw.rect(screen, RED, (rect_x + shake_x, rect_y + shake_y,
                               width, height))
```

Everything on screen jumps around for a few frames, then settles back to normal. It is a small effect, but it makes impacts feel powerful.

#### Particle Effects

Particles are small dots or shapes that fly out from a point. They make events like scoring, eating food, or collisions feel more exciting.

Each particle is a small dictionary with a position, velocity, lifetime, and color:

```python
particles = []

def spawn_particles(x, y, color, count=10):
    for i in range(count):
        particle = {
            "x": x,
            "y": y,
            "vx": random.uniform(-3, 3),
            "vy": random.uniform(-3, 3),
            "life": random.randint(15, 30),
            "color": color,
        }
        particles.append(particle)
```

`random.uniform(-3, 3)` gives a random decimal number between -3 and 3. Each particle flies in a slightly different direction.

To update and draw the particles, add this to your game loop:

```python
def update_particles():
    for particle in particles[:]:
        particle["x"] += particle["vx"]
        particle["y"] += particle["vy"]
        particle["life"] -= 1
        if particle["life"] <= 0:
            particles.remove(particle)

def draw_particles():
    for particle in particles:
        size = max(1, particle["life"] // 5)
        pygame.draw.circle(screen, particle["color"],
                           (int(particle["x"]), int(particle["y"])),
                           size)
```

Notice `particles[:]` in the for loop. This creates a copy of the list to loop over, which lets us safely remove items from the original list during the loop. Without this, Python would skip items or crash.

The particles shrink as they age because we calculate their `size` from their remaining `life`. When a particle's life reaches zero, it is removed from the list.

To use the particles, call `spawn_particles()` when something interesting happens:

```python
# When the player scores
spawn_particles(ball_x, ball_y, YELLOW, 20)

# When the snake eats food
spawn_particles(food[0] * CELL_SIZE + 15, food[1] * CELL_SIZE + 15,
                RED, 15)
```

Then call `update_particles()` and `draw_particles()` in every frame of your game loop.

> Professional game developers spend a surprising amount of time on these small details. The game Vlambeer's Nuclear Throne is famous for its screen shake and particles. The core gameplay is simple, but the effects make every action feel incredible.

---

## Part 2: Sharing Your Games

You have made a game and you want your friends to play it. There are a few ways to do this, from simple to advanced.

### The Simple Way: Copy the Folder

The most straightforward way to share a Python game is to give someone the whole project folder. They need:

1. Python installed on their computer
2. Pygame installed (`pip install pygame`)
3. Your entire game folder, including all the code files, images, sounds, and fonts

Put everything in one folder. If your game loads `paddle_hit.wav`, that file needs to be right next to your `.py` file (or in a subfolder you reference correctly).

```
my_pong_game/
    pong.py
    paddle_hit.wav
    score.wav
    background_music.ogg
```

Zip the folder, send it to your friend, and tell them to unzip it, open a terminal in the folder, and run:

```
python3 pong.py
```

This works, but it requires your friend to install Python and pygame. That is fine for other programmers, but not ideal for someone who just wants to play your game.

---

### PyInstaller: Building an Executable

**PyInstaller** is a tool that bundles your Python game into a single **executable** file -- a program that runs without needing Python installed. You can give this file to anyone with the same operating system, and they can double-click it to play your game.

#### Installing PyInstaller

Open a terminal and run:

```
pip3 install pyinstaller
```

> On newer versions of Debian, you may need to add `--break-system-packages` to the command, or ask an adult to help you set up a virtual environment.

#### Building Your Game

Navigate to the folder containing your game and run:

```
pyinstaller --onefile my_game.py
```

PyInstaller will create a `dist` folder. Inside it, you will find a single file called `my_game` (on Linux) or `my_game.exe` (on Windows). That file is your complete game.

```
my_game_folder/
    my_game.py
    paddle_hit.wav
    score.wav
    dist/
        my_game        <-- this is the executable
    build/             <-- temporary files, you can ignore this
    my_game.spec       <-- configuration file, you can ignore this
```

#### Important: Assets Are Not Included Automatically

Here is the most common problem with PyInstaller: it bundles your Python code, but it does not automatically include your sound files, images, or other assets. You need to copy those files so they sit next to the executable:

```
give_to_friend/
    my_game
    paddle_hit.wav
    score.wav
    background_music.ogg
```

Zip this folder and send it. Your friend can unzip it and run the game without installing Python.

> There is a way to include assets inside the executable itself using PyInstaller's `--add-data` flag, but it requires changes to how your code loads files. The "copy assets next to the executable" approach is simpler and works just as well.

#### Troubleshooting

If the executable crashes or behaves strangely:

- **Missing files:** Make sure all your sound files, images, and fonts are next to the executable.
- **Different operating system:** A Linux executable will not work on Windows, and vice versa. You need to build on the same operating system your friend uses.
- **Run from terminal to see errors:** If the executable crashes immediately, run it from the terminal instead of double-clicking. Error messages will appear in the terminal.
- **Run from the game folder:** Make sure to open a terminal in your game folder and run the executable from there (for example, `./dist/my_game`), so the game can find its image and sound files.

> For the curious: it is also possible to create `.deb` packages (the same format that Debian uses for software installation). That is a more advanced topic and beyond the scope of this book, but it is good to know it exists.

---

## Part 3: Version Control

As your programs get bigger, you will eventually make a change that breaks something, and you will wish you could go back to the version that worked. Professional programmers solve this problem with a tool called **Git**.

### What Is Git?

Git is a **version control** system. It tracks every change you make to your code and lets you go back to any previous version at any time. Think of it as having unlimited undo for your entire project -- not just one file, but every file in the folder.

With Git, you can:

- Save snapshots of your project (called **commits**) at any point
- See exactly what changed between any two snapshots
- Undo changes that broke something
- Work on new features without risking the working version
- Collaborate with other programmers on the same project

Almost every professional software project uses Git. The code for Linux, Python, pygame, Firefox, and millions of other programs is all tracked with Git.

### Do You Need to Learn Git Right Now?

Not yet. Git is an important tool, but it is a topic for another book. When you are ready, here are some good places to start:

- **git-scm.com/book** -- The official Git book, free to read online
- **learngitbranching.js.org** -- An interactive visual tutorial that teaches Git through exercises
- **ohshitgit.com** -- A fun, practical guide to fixing Git mistakes (the name is rude, but the content is helpful)

For now, the simplest way to protect your work is to make copies of your project folder before you make big changes. Name them something like `snake_v1`, `snake_v2`, and so on. This is a rough version of what Git does, and it will keep you safe until you are ready to learn the real thing.

---

## Part 4: What Comes Next

You have finished this book. But you have not finished learning. Programming is one of those skills where there is always something new to discover, something harder to try, something more interesting to build.

Here are some directions you could go from here.

### More Pygame Games

You have built the foundations. Now push yourself. Here are some game ideas, roughly in order of difficulty:

- **Breakout / Arkanoid** -- A paddle at the bottom, rows of bricks at the top, and a bouncing ball. You already know how to bounce a ball and detect collisions.
- **A platformer** -- A character that runs and jumps across platforms. This requires gravity, which is just adding a small amount to the vertical speed every frame.
- **A top-down shooter** -- A spaceship that flies around and shoots at enemies. You will learn about spawning enemies in waves and managing lots of objects at once.
- **A racing game** -- A car that drives along a track. You could use a top-down view and experiment with simple physics.
- **An RPG** -- A character that walks around a map, talks to people, and fights monsters. This is a big project, but you can start small and keep adding to it.

Each of these will teach you something new. A platformer teaches you about gravity. A shooter teaches you about managing many objects. An RPG teaches you about data and story.

### Beyond Games

Python is used for far more than games. Here are a few other areas:

- **Web development** -- Frameworks like Flask and Django let you build websites and web applications with Python. If you have ever wondered how websites work behind the scenes, this is the next step.
- **Automation** -- Python can automate boring tasks: renaming hundreds of files, downloading data from the internet, organizing your music collection. If you find yourself doing the same task on the computer over and over, you can probably write a Python script to do it for you.
- **Data science** -- Python is the most popular language for analyzing data and creating visualizations. Libraries like pandas and matplotlib are used by scientists, journalists, and businesses every day.
- **Artificial intelligence** -- Many of the AI tools making headlines right now are built with Python. Libraries like PyTorch and TensorFlow let you build and train machine learning models.

### Other Game Engines

Pygame is a great learning tool, but there are more powerful engines for building bigger games:

- **Godot** -- A free, open-source game engine with a visual editor and its own scripting language (GDScript, which is similar to Python). Many indie game developers use Godot. It can make 2D and 3D games.
- **Unity** -- A widely-used game engine that works with C#. Many professional and indie games are made with Unity.
- **Unreal Engine** -- A powerful engine used for high-end 3D games. It uses C++ and a visual scripting system called Blueprints.

If you enjoy making games, try Godot next. Its scripting language will feel familiar because of your Python experience, and the visual editor makes it easier to build larger projects.

### Online Resources

Bookmark these for when you need help:

- **docs.python.org** -- The official Python documentation. It can be dense, but it is the definitive reference for the language.
- **pyga.me/docs** -- The official pygame documentation. Look up any function to see exactly what it does and what arguments it takes.
- **reddit.com/r/learnpython** -- A friendly community where beginners ask questions and get helpful answers.
- **stackoverflow.com** -- The biggest programming Q&A site. Almost every error message you will ever see has been asked about and answered here.

### The Most Important Advice

The most important thing: **keep building things**. Every programmer got good by writing lots of programs. Not by reading books. Not by watching videos. By writing code, running into problems, and figuring out solutions.

It does not matter what you build. It does not matter if it is ugly or buggy or half-finished. What matters is that you keep going. Every project teaches you something. Every bug you fix makes you a better programmer.

---

## What You Learned

This chapter covered a lot of ground:

- **Sound effects** are loaded with `pygame.mixer.Sound()` and played with `.play()`. Background music uses `pygame.mixer.music`.
- **Title screens and menus** use the game state pattern with a `selected_option` variable to highlight the current choice.
- **High scores** can be saved to a file with `open()` and `write()`, and loaded back with `open()` and `read()`. Use `try`/`except` to handle the case where the file does not exist yet.
- **Screen shake** is achieved by adding a random offset to draw positions for a few frames.
- **Particle effects** are lists of small objects with position, velocity, and a lifetime that counts down to zero.
- **Sharing games** can be as simple as zipping a folder or as polished as building an executable with PyInstaller.
- **Git** is a version control tool that professional programmers use to track changes. You will learn it when you are ready.
- There are many directions to go next: more pygame, web development, automation, data science, AI, and other game engines.

---

## Your Final Challenge

You do not need a quiz this time. You do not need an experiment. You need a challenge -- the biggest one yet.

**Build a game we did not cover in this book.**

Pick an idea. It could be one of the suggestions above, or something entirely your own. Plan it out on paper first. Think about what objects are on screen, how they move, what the player does, and what the win/lose conditions are.

Then build it. One function at a time. One feature at a time. Debug it when it breaks. Add polish when it works.

You have all the tools you need:

- Variables, lists, and dictionaries to store data
- Functions to organize your code
- Game loops to make things move
- Collision detection to make things interact
- Game states to manage different screens
- Sound, menus, and effects to make it feel real

This is not the end. This is the starting line.

---

## Closing

When you started this book, you did not know what a terminal was. You did not know what a variable was, or a function, or a game loop. You probably felt lost more than once.

And now here you are. You have written programs. You have built games from scratch -- real games with graphics and sound and things moving on the screen. You have debugged errors, fixed collision detection, managed game states, and handled keyboard input. You did all of that.

You are a programmer now. Not a future programmer. Not an almost-programmer. A programmer. That means you can build anything you can imagine, one line of code at a time.

Go build something.
