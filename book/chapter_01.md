---
layout: default
title: "Chapter 1: Getting Started"
nav_order: 1
parent: "Part 1: Learning Python"
---

# Chapter 1: Getting Started

## What You Will Learn
- What this book is about and what you will build
- How to use the terminal on your computer
- How to install Python and Pygame
- How to write and run your very first program

## Welcome to Making Games with Python and Pygame

You are about to learn how to make your own computer games. Real ones, with graphics and sound and things moving around on the screen. Along the way, you will learn programming — the skill of telling a computer exactly what to do.

Programming is like learning a new language, except the one listening is a machine. Machines are fast and powerful, but they are also very literal. They do exactly what you tell them, nothing more and nothing less. That is what makes programming both challenging and fun.

We will use the **Python** programming language because it is readable, widely used, and a great first language. We will also use a library called **Pygame**, which gives us the tools to draw graphics, play sounds, and handle keyboard and mouse input — everything we need to make games.

By the end of this book, you will have built several complete games, and you will understand the programming ideas behind them. Let's get started.

---

## The Terminal

The **terminal** is a program where you type commands to your computer. Instead of clicking icons and buttons, you type words and press `Enter`. Programmers use the terminal constantly.

To open the terminal on Debian Linux, press `Ctrl+Alt+T` on your keyboard. You can also search for "Terminal" in your applications menu.

When the terminal opens, you will see something like this:

```
dodobird@debian:~$
```

The part before the `$` sign tells you your username and computer name. The `~` means you are in your **home folder**. The `$` is the **prompt** — it means the terminal is waiting for you to type a command.

Try typing this and pressing `Enter`:

```
whoami
```

The terminal should print your username. That is a real command. You just told the computer to do something, and it did it.

> The terminal might look old-fashioned, but it is one of the most powerful tools on your computer. Professional programmers, system administrators, and hackers all use it every day.

---

## Installing Python

Python might already be installed on your Debian system. Let's check. Type this in the terminal:

```
python3 --version
```

If Python is installed, you will see something like:

```
Python 3.11.2
```

Your version number might be slightly different. That is fine. If you see an error message instead, we need to install Python. Type this:

```
sudo apt install python3 python3-pip
```

The `sudo` part means "run this as an administrator." Your computer will ask for your password. Type it in and press `Enter`. You will not see the letters as you type the password — that is normal. It is a security feature.

After it finishes, run `python3 --version` again to make sure it worked.

## Installing Pygame

**Pygame** is a library — a collection of code that someone else wrote so we don't have to. It handles the hard parts of making games, like drawing to the screen and playing sounds.

On Debian, the easiest way to install Pygame is through the system package manager. Type this:

```
sudo apt install python3-pygame
```

Let's make sure it worked. Type this:

```
python3 -c "import pygame; print(pygame.version.ver)"
```

You should see a version number. If you do, Pygame is ready to go.

---

## Installing VS Code

We need a program to write our code in. A **text editor** for programming is like a word processor, but for code. We will use **Visual Studio Code** (VS Code for short). It is free, popular, and works well on Linux.

Open a web browser and go to `https://code.visualstudio.com`. Download the `.deb` file for Linux. Once the download finishes, open the terminal and type:

```
sudo apt install ~/Downloads/code_*.deb
```

If your browser saved the file somewhere else, adjust the path. After it installs, you can open VS Code by typing `code` in the terminal or by finding it in your applications menu.

## Setting Up Your Project Folder

Let's create a folder where all your game projects will live. In the terminal, type:

```
mkdir -p ~/pygame_projects
```

The `mkdir` command means "make directory" (a directory is just another word for folder). The `-p` flag means "don't complain if it already exists."

Now open VS Code and point it at your new folder:

```
code ~/pygame_projects
```

VS Code will open with your project folder in the sidebar on the left. This is your workspace.

## Creating Your First File

In VS Code, look at the sidebar on the left. You should see the name of your folder. Hover over it and click the icon that looks like a file with a small `+` sign. Name the file `hello.py`.

The `.py` ending tells your computer (and VS Code) that this is a Python file.

---

## Your First Program

Programmers have been writing "Hello, World!" as their first program since the 1970s. It's a tradition. The idea is simple: make the computer print a message to prove that everything is set up and working.

Type this into your `hello.py` file:

```python
print("Hello, World!")
```

That's it. One line. Save the file with `Ctrl+S`.

Now open the terminal inside VS Code by pressing `` Ctrl+` `` (that is the backtick key, usually above `Tab` on your keyboard). Make sure you are in the right folder:

```
cd ~/pygame_projects
```

Run your program:

```
python3 hello.py
```

You should see:

```
Hello, World!
```

Congratulations. You just wrote and ran a program. The `print()` **function** takes whatever you put inside the parentheses and displays it on the screen.

## A More Interesting Program

One line is a bit boring. Let's make Python print something more fun. Create a new file called `rocket.py` and type this:

```python
print("    /\\")
print("   /  \\")
print("  /    \\")
print(" |      |")
print(" |      |")
print(" |      |")
print(" |______|")
print("  |    |")
print("  |    |")
print(" /|    |\\")
print("/_|____|_\\")
print("  ^^^^^^")
```

Save it and run it:

```
python3 rocket.py
```

You should see:

```
    /\
   /  \
  /    \
 |      |
 |      |
 |      |
 |______|
  |    |
  |    |
 /|    |\
/_|____|_\
  ^^^^^^
```

Every `print()` line outputs one row of text. Python runs your lines from top to bottom, one at a time. That is an important idea — programs execute **in order**, from the first line to the last.

> Notice the backslash `\` in the ASCII art. In Python, the backslash is a special character. In `print()` with regular strings it works fine for single backslashes like this, but keep it in mind for later. We will learn more about this in a future chapter.

## Running Programs from the VS Code Terminal

You already used the VS Code terminal to run `hello.py`. Here is the pattern you will use throughout this book:

1. Write or edit your code in the editor area of VS Code.
2. Save with `Ctrl+S`.
3. Click on the terminal panel at the bottom of VS Code.
4. Type `python3 filename.py` and press `Enter`.

You can also use the up arrow key in the terminal to bring back the last command you typed. That saves time when you are running the same program over and over while working on it.

---

## What You Learned

- The **terminal** is where you type commands to your computer. You open it with `Ctrl+Alt+T`.
- **Python** is the programming language we use in this book. You run Python files with `python3 filename.py`.
- **Pygame** is a library that gives us tools for making games.
- **VS Code** is the text editor where we write our code.
- The `print()` function displays text on the screen.
- Programs run from top to bottom, one line at a time.

## Quick Quiz

1. What keyboard shortcut opens the terminal on Debian Linux?
2. What command do you type to check which version of Python is installed?
3. What does the `print()` function do?
4. In what order does Python run the lines in your program?

## Experiment

- Change the message inside `print("Hello, World!")` to say something else. Run the program again.
- Add more `print()` lines to `rocket.py` to make the rocket taller or add flames coming out of the bottom.
- Try running `print("Hello, World!")` without the closing parenthesis. What error message do you get? Don't worry about breaking anything — you can't damage your computer by writing bad Python. The worst that happens is an error message.

## Chapter Challenge

Create a new file called `my_art.py`. Use `print()` to draw your own ASCII art picture. It could be a house, a face, a cat, a tree, or anything you want. Use at least eight `print()` lines.

**Hint 1:** Plan your picture on paper first. Count the spaces carefully.

**Hint 2:** Remember that each `print()` line outputs one row of text. Use spaces to position characters where you want them.

**Hint 3:** If you get stuck, look at how `rocket.py` works and follow the same pattern.
