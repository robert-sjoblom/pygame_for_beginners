---
layout: default
title: "Chapter 4: Loops and Repetition"
nav_order: 4
parent: "Part 1: Learning Python"
---

# Chapter 4: Loops and Repetition

## What You Will Learn
- How to make your programs repeat actions using `while` and `for` loops
- How `range()` works and why it is so useful
- How to escape a loop with `break` and skip ahead with `continue`
- How to improve the guessing game with loops
- A debugging technique: tracing through code with your finger

## Why Loops?

Imagine you want to print the numbers 1 through 10. Without loops, you would write this:

```python
print(1)
print(2)
print(3)
print(4)
print(5)
print(6)
print(7)
print(8)
print(9)
print(10)
```

That is ten lines for a simple task. What if you needed 1 to 1000? You would need a thousand lines. That is not practical, and it is exactly the kind of boring, repetitive work that computers are good at.

A **loop** tells Python: "repeat this block of code." There are two kinds of loops in Python: `while` loops and `for` loops.

## `while` Loops

A **`while` loop** keeps running as long as its condition is true. It works like an `if` statement that repeats.

Create a new file called `loops.py`:

```python
count = 1

while count <= 10:
    print(count)
    count = count + 1
```

Run it:

```
python3 loops.py
```

```
1
2
3
4
5
6
7
8
9
10
```

Ten lines of output, five lines of code. Here is how it flows:

```
     +------------------+
     |  count = 1       |
     +------------------+
              |
              v
     +------------------+
+--->| Is count <= 10?  |
|    +------------------+
|        |           |
|       YES          NO
|        |           |
|        v           v
|   +-----------+   DONE
|   | print     |
|   | count     |
|   +-----------+
|        |
|        v
|   +-----------+
|   | count =   |
|   | count + 1 |
|   +-----------+
|        |
+--------+
```

Each trip through the loop is called an **iteration**. This loop has 10 iterations. On the 11th check, `count` is 11, so `count <= 10` is `False`, and Python exits the loop.

### The Danger of Infinite Loops

What happens if the condition never becomes false? The loop runs forever. This is called an **infinite loop**.

```python
# WARNING: This loop never stops!
count = 1

while count <= 10:
    print(count)
    # Oops! We forgot count = count + 1
```

If you run this, your terminal fills with `1` over and over and never stops. To stop a runaway program, press `Ctrl+C` on your keyboard. You will see something like:

```
1
1
1
^CTraceback (most recent call last):
  ...
KeyboardInterrupt
```

This happens to every programmer, beginners and experts alike. It is not a disaster -- just press `Ctrl+C` and fix your code. The usual cause is forgetting to update the variable that the condition checks.

> Did you know? Some programs use infinite loops on purpose. A game's main loop runs forever (drawing the screen, checking for input, updating the world) until the player quits. You will build one of these when we get to Pygame.

---

## `for` Loops

A **`for` loop** runs a block of code a specific number of times. It is the most common loop in Python.

Replace the contents of `loops.py`:

```python
for i in range(5):
    print("Hello!")
```

```
Hello!
Hello!
Hello!
Hello!
Hello!
```

The variable `i` is the **loop variable**. Each time through the loop, `i` gets a new value. Let's see what `i` actually is:

```python
for i in range(5):
    print("i is", i)
```

```
i is 0
i is 1
i is 2
i is 3
i is 4
```

Notice: `range(5)` starts at 0 and ends at 4, not 5. It gives you 5 numbers, but they are 0 through 4. This is a common source of confusion, so take a moment to let it sink in.

## Understanding `range()`

The `range()` function has three forms:

**`range(stop)`** -- count from 0 up to (but not including) stop:

```python
for i in range(4):
    print(i)
```

```
0
1
2
3
```

**`range(start, stop)`** -- count from start up to (but not including) stop:

```python
for i in range(1, 6):
    print(i)
```

```
1
2
3
4
5
```

This is how you count from 1 to 5. The stop value is always excluded.

**`range(start, stop, step)`** -- count from start, going up by step each time:

```python
for i in range(0, 20, 5):
    print(i)
```

```
0
5
10
15
```

You can even count backwards with a negative step:

```python
for i in range(10, 0, -1):
    print(i)
```

```
10
9
8
7
6
5
4
3
2
1
```

Here is a quick reference:

```
range(5)          -->  0, 1, 2, 3, 4
range(1, 5)       -->  1, 2, 3, 4
range(2, 10, 2)   -->  2, 4, 6, 8
range(10, 0, -1)  -->  10, 9, 8, 7, 6, 5, 4, 3, 2, 1
```

---

## Looping Through Lists

Python has a data type called a **list** -- an ordered collection of items. You will learn all about lists in a later chapter, but they work so naturally with `for` loops that it is worth a quick preview.

A list looks like this:

```python
colors = ["red", "green", "blue", "yellow"]
```

You can loop through every item in a list:

```python
colors = ["red", "green", "blue", "yellow"]

for color in colors:
    print(color)
```

```
red
green
blue
yellow
```

The loop variable `color` takes on each value in the list, one at a time. You can name the loop variable anything, but it helps to pick a name that makes sense: `color` for a list of colors, `name` for a list of names, and so on.

```python
scores = [85, 92, 78, 95]

for score in scores:
    print("Score:", score)
```

```
Score: 85
Score: 92
Score: 78
Score: 95
```

---

## `break` and `continue`

Sometimes you need to leave a loop early, or skip one particular iteration.

### `break` -- Leave the Loop Immediately

`break` stops the loop right away, no matter what the condition says.

```python
for i in range(10):
    if i == 5:
        print("Found 5! Stopping.")
        break
    print(i)
```

```
0
1
2
3
4
Found 5! Stopping.
```

Once Python hits `break`, it jumps out of the loop entirely. The numbers 6 through 9 never print.

`break` is especially useful with `while True` loops. Instead of putting the condition in the `while` line, you check inside the loop and `break` when you are done:

```python
while True:
    answer = input("Type 'quit' to exit: ")
    if answer == "quit":
        break
    print("You typed:", answer)

print("Goodbye!")
```

```
Type 'quit' to exit: hello
You typed: hello
Type 'quit' to exit: test
You typed: test
Type 'quit' to exit: quit
Goodbye!
```

### `continue` -- Skip to the Next Iteration

`continue` skips the rest of the current iteration and jumps back to the top of the loop.

```python
for i in range(8):
    if i == 3 or i == 6:
        continue
    print(i)
```

```
0
1
2
4
5
7
```

Numbers 3 and 6 are missing because `continue` skipped them. Here is the flow:

```
     +------------------+
     |  Get next i      |
     +------------------+
              |
              v
     +------------------+
     | Is i == 3 or 6?  |
     +------------------+
        |           |
       YES          NO
        |           |
        |           v
        |     +-----------+
        |     | print(i)  |
        |     +-----------+
        |           |
        +-----<-----+
              |
              v
      (back to top of loop)
```

---

## Problem-Solving Technique: Trace Through Your Code

When a loop does not behave the way you expect, there is a powerful technique: **trace through it with your finger**.

Take this code:

```python
total = 0

for i in range(1, 4):
    total = total + i
    print("i:", i, "total:", total)
```

Put your finger on the first line. Move it down, one line at a time, and keep track of each variable's value on paper (or in your head):

```
Step  |  i  |  total  |  What happens
------+-----+---------+-----------------------------
  1   |  -  |    0    |  total starts at 0
  2   |  1  |    1    |  total = 0 + 1, print
  3   |  2  |    3    |  total = 1 + 2, print
  4   |  3  |    6    |  total = 3 + 3, print
```

Run the code to verify:

```
i: 1 total: 1
i: 2 total: 3
i: 3 total: 6
```

This technique works for any code, not just loops. When something goes wrong, slow down and trace through it step by step. Professional programmers do this all the time.

---

## Build-Up Exercise: Improving the Guessing Game

Remember the guessing game from Chapter 3? The player only got one guess. Let's fix that with loops.

### Step 1: Plan the Improvements

Here is what we want:

```
1. Pick a random number between 1 and 20
2. LOOP:
   a. Ask the player for a guess
   b. If correct, say "You got it!" and STOP the loop
   c. If too high, say "Too high!"
   d. If too low, say "Too low!"
3. Tell the player how many guesses it took
```

### Step 2: Add the Loop

Create a new file called `guess_v2.py`. Start with the code from Chapter 3 and wrap the guessing part in a `while True` loop:

```python
import random

secret = random.randint(1, 20)

print("I'm thinking of a number between 1 and 20.")

while True:
    guess = int(input("Your guess: "))

    if guess < secret:
        print("Too low!")
    elif guess > secret:
        print("Too high!")
    else:
        print("You got it!")
        break
```

The loop keeps asking for guesses until the player gets it right. When they do, `break` ends the loop.

Test it:

```
I'm thinking of a number between 1 and 20.
Your guess: 10
Too high!
Your guess: 5
Too low!
Your guess: 7
Too low!
Your guess: 8
You got it!
```

### Step 3: Add a Guess Counter

We want to tell the player how many guesses they needed. Create a variable before the loop and increase it each time:

```python
import random

secret = random.randint(1, 20)

print("I'm thinking of a number between 1 and 20.")

guesses = 0

while True:
    guess = int(input("Your guess: "))
    guesses = guesses + 1

    if guess < secret:
        print("Too low!")
    elif guess > secret:
        print("Too high!")
    else:
        print("You got it in", guesses, "guesses!")
        break
```

```
I'm thinking of a number between 1 and 20.
Your guess: 10
Too low!
Your guess: 15
Too high!
Your guess: 12
You got it in 3 guesses!
```

### Step 4: Add "Play Again?"

After the game ends, ask if the player wants to play again. This needs another loop around the entire game:

```python
import random

playing = True

while playing:
    secret = random.randint(1, 20)
    guesses = 0

    print("I'm thinking of a number between 1 and 20.")

    while True:
        guess = int(input("Your guess: "))
        guesses = guesses + 1

        if guess < secret:
            print("Too low!")
        elif guess > secret:
            print("Too high!")
        else:
            print("You got it in", guesses, "guesses!")
            break

    again = input("Play again? (yes/no): ")
    if again != "yes":
        playing = False

print("Thanks for playing!")
```

Your code should now look exactly like the listing above. Let's trace through the structure:

```
+---------------------------+
| while playing:            |
|                           |
|   Pick a new secret       |
|   Reset guesses to 0      |
|                           |
|   +---------------------+ |
|   | while True:         | |
|   |   Get a guess       | |
|   |   Count it          | |
|   |   Check it          | |
|   |   break if correct  | |
|   +---------------------+ |
|                           |
|   Ask "Play again?"      |
|   Set playing to False    |
|   if they say no          |
+---------------------------+
|
v
"Thanks for playing!"
```

The outer loop controls whether the whole game repeats. The inner loop controls the guessing within a single round.

Test it thoroughly. Play a full round, say "yes" to play again, play another round, then say "no."

```
I'm thinking of a number between 1 and 20.
Your guess: 10
Too high!
Your guess: 5
You got it in 2 guesses!
Play again? (yes/no): yes
I'm thinking of a number between 1 and 20.
Your guess: 15
Too low!
Your guess: 18
You got it in 2 guesses!
Play again? (yes/no): no
Thanks for playing!
```

---

## Nested Loops

Just like `if` statements can be nested, loops can be nested too. A **nested loop** is a loop inside another loop.

Here is a classic example -- a multiplication table:

```python
for row in range(1, 4):
    for col in range(1, 4):
        result = row * col
        print(result, end="\t")
    print()
```

The `end="\t"` tells `print` to put a tab character after the number instead of starting a new line. The empty `print()` at the end of the outer loop moves to a new line after each row.

```
1	2	3
2	4	6
3	6	9
```

Let's trace through the first few steps to understand it:

```
row=1, col=1  -->  1*1 = 1   (print, stay on same line)
row=1, col=2  -->  1*2 = 2   (print, stay on same line)
row=1, col=3  -->  1*3 = 3   (print, stay on same line)
                              (new line)
row=2, col=1  -->  2*1 = 2   (print, stay on same line)
row=2, col=2  -->  2*2 = 4   (print, stay on same line)
... and so on
```

For every single value of `row`, the entire inner loop runs from start to finish. If the outer loop runs 3 times and the inner loop runs 3 times, the code inside runs 3 x 3 = 9 times total.

This is tricky. Read it twice if you need to, and try tracing through it on paper.

---

## What You Learned

- **`while` loops** repeat as long as a condition is true.
- **Infinite loops** happen when the condition never becomes false. Press `Ctrl+C` to stop them.
- **`for` loops** repeat a set number of times, often using `range()`.
- **`range()`** generates sequences of numbers: `range(stop)`, `range(start, stop)`, and `range(start, stop, step)`.
- You can loop through **lists** with `for item in list`.
- **`break`** exits a loop immediately.
- **`continue`** skips to the next iteration.
- **Tracing** through code step by step is a reliable way to understand what it does and find bugs.
- **Nested loops** put one loop inside another. The inner loop runs completely for each iteration of the outer loop.

## Quick Quiz

1. What is the difference between a `while` loop and a `for` loop?

2. What does `range(3, 8)` produce?

3. What does this code print?

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

4. How do you stop a program that is stuck in an infinite loop?

## Experiment

1. Change the multiplication table to show rows and columns from 1 to 10. (Hint: change the `range()` values.)

2. Write a `while` loop that starts at 100 and counts down to 0 by 10. (100, 90, 80, ... 0.) Then do the same thing with a `for` loop and `range()`.

3. In the guessing game, add a maximum number of guesses (say, 6). If the player runs out of guesses, tell them the secret number and end the round. (Hint: you can change `while True` to `while guesses < 6`.)

## Chapter Challenge

**Build a number guessing game where the roles are reversed.**

This time the player thinks of a number between 1 and 100, and the computer tries to guess it. The player responds with "high", "low", or "correct" after each guess.

The computer should start by guessing 50 (the middle of the range), then adjust based on the player's feedback. If the player says "high", the computer should guess lower next time. If the player says "low", the computer should guess higher.

**Hints (try before reading!):**

1. Keep track of two variables: `low` (starts at 1) and `high` (starts at 100). The computer's guess is always the midpoint: `(low + high) // 2`.
2. If the player says "high", change `high` to `guess - 1`. If the player says "low", change `low` to `guess + 1`.
3. Use a `while True` loop that breaks when the player says "correct".
4. The `//` operator divides and rounds down to a whole number. `7 // 2` gives `3`.
