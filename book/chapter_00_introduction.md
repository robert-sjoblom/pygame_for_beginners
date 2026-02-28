---
layout: default
title: "Introduction"
nav_order: 4
permalink: /introduction
---

# Making Games with Python and Pygame

*A book for beginners who want to learn programming by building real games.*

---

## Table of Contents

### Part 1: Learning Python

| Chapter | Title | What You Will Build |
|---------|-------|---------------------|
| 1 | [Getting Started](#chapter-1-getting-started) | Your first program |
| 2 | [Variables and Values](#chapter-2-variables-and-values) | A calculator |
| 3 | [Making Decisions](#chapter-3-making-decisions) | A number guessing game |
| 4 | [Loops and Repetition](#chapter-4-loops-and-repetition) | An improved guessing game |
| 5 | [Functions](#chapter-5-functions) | A text adventure |
| 6 | [Lists and Dictionaries](#chapter-6-lists-and-dictionaries) | A quiz game |

### Part 2: Making Games with Pygame

| Chapter | Title | What You Will Build |
|---------|-------|---------------------|
| 7 | [Your First Window](#chapter-7-your-first-window) | A drawn scene with shapes |
| 8 | [Movement and Input -- Game 1: Pong](#chapter-8-pong) | Pong |
| 9 | [Game 2 -- Snake](#chapter-9-snake) | Snake |
| 10 | [Sprites and Art](#chapter-10-sprites-and-art) | Pong with real graphics |
| 11 | [Game 3 -- Flappy Bird](#chapter-11-flappy-bird) | Flappy Bird |
| 12 | [Classes and Objects](#chapter-12-classes-and-objects) | A bouncing balls demo |
| 13 | [Game 4 -- Point-and-Click Adventure](#chapter-13-adventure) | A space station adventure |
| 14 | [Game 5 -- Tower Defense](#chapter-14-tower-defense) | Tower Defense |
| 15 | [Polish, Sharing, and What Comes Next](#chapter-15-polish-and-sharing) | Sound, menus, and distribution |

---

## Introduction

### Who This Book Is For

This book is for you if you have never written a program before. You do not need to know anything about coding, and you do not need to be good at math. All you need is a computer running Debian Linux, some curiosity, and a willingness to type things in and see what happens.

By the end of this book, you will have built five complete games -- from classic Pong to a point-and-click space adventure -- and you will understand the ideas behind every line of code you wrote.

### Why Games?

Programming can be hard to learn when the only thing on your screen is text. Games give you something to see and interact with immediately. You type a few lines, run your program, and a window appears. You change a number, run it again, and something moves differently. That tight loop -- write, run, see -- is the fastest way to learn.

Games also happen to use almost every important programming concept: variables to track the score, conditions to check if the player won, loops to keep the game running, lists to manage enemies, and functions to organize your code. By the time you finish this book, you will have learned real programming -- not just game programming.

### What You Will Need

- **A computer running Debian Linux.** Other Linux distributions will work too, but the installation commands in this book are written for Debian.
- **An internet connection** for installing software and (optionally) downloading free game assets.
- **About 15-20 minutes at a time.** Every chapter is broken into sections with clear stopping points. You do not need to finish a whole chapter in one sitting.
- **Patience.** Programming is a skill, like playing an instrument or learning a sport. You will make mistakes. Your code will not work on the first try. That is completely normal and is part of the process.

### How This Book Works

The book is split into two parts.

**Part 1 (Chapters 1-6)** teaches you Python -- the programming language. You will work in the terminal, writing small programs that take input, make decisions, and print output. Each chapter introduces one or two big ideas and ends with a project that puts them to use. By the end of Part 1, you will be comfortable with variables, conditions, loops, functions, lists, and dictionaries.

**Part 2 (Chapters 7-15)** uses everything you learned to build games with Pygame -- a library that lets Python draw graphics, play sounds, and respond to keyboard and mouse input. Each game is more ambitious than the last, and each one teaches you new techniques.

### How to Read This Book

Read the chapters in order. Each one builds on the ones before it. If you skip ahead, you will run into concepts you have not seen yet.

**Type the code yourself.** Do not copy and paste. Typing forces you to read every character, and that is how the patterns stick in your memory. It is slower, but it works.

**Run your code often.** Do not type for twenty minutes and then run. Type a few lines, run, see what happens, then type a few more. If something breaks, you will know exactly which lines caused the problem.

**Read the error messages.** When your code does not work, Python tries to tell you what went wrong. Error messages look intimidating at first, but they are your most useful tool. The book will help you learn to read them.

**Do the exercises.** Every chapter ends with a Quick Quiz, some Experiments, and a Chapter Challenge. These are not optional filler -- they are where learning actually happens. The challenges have hints if you get stuck.

**Take breaks.** If you are frustrated or stuck, close your laptop and come back later. Programmers at every level do this. Sometimes the solution appears in your head while you are doing something else entirely.

### A Note About Mistakes

You are going to make mistakes. Lots of them. Every programmer does, no matter how experienced they are. A missing colon, a misspelled variable name, an indentation that is one space off -- these tiny errors can stop a program from running.

Early in the book, we will be very careful about guiding you step by step so that errors are rare. As you gain confidence, we will start showing you common mistakes on purpose and teaching you how to read error messages and fix problems on your own. By the end, you will be debugging like a real programmer.

### A Note About Git

Professional programmers use a tool called **git** to track every change they make to their code. It is like having an unlimited undo button for your entire project. We will not teach git in this book -- it is a big topic that deserves its own guide -- but it is good to know it exists. When you are ready, look it up. For now, we recommend saving your work often and keeping backup copies of files you are proud of.

### Conventions Used in This Book

Here is what different formatting means throughout the book:

- **Bold text** marks a new term when it is first introduced.
- `Backtick text` marks code, file names, commands, and keyboard keys.
- Code blocks look like this:

```python
print("Hello, World!")
```

- After code that produces output, you will see what to expect:

```
Hello, World!
```

- Tips and interesting facts appear in blockquotes:

> Did you know? Python is named after the comedy group Monty Python, not the snake.

- When you see a horizontal line (---) between sections, that is a good place to take a break if you need one.

### Let's Go

You have everything you need. Open Chapter 1, follow the instructions, and in a few minutes you will have written your first program.

Welcome to programming.
