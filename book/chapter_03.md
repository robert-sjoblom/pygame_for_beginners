---
layout: default
title: "Chapter 3: Making Decisions"
nav_order: 3
parent: "Part 1: Learning Python"
---

# Chapter 3: Making Decisions

## What You Will Learn
- How to make your programs react differently depending on what happens
- How to compare values using comparison operators
- How to use `if`, `else`, and `elif` to choose between different paths
- How to combine conditions with `and`, `or`, and `not`
- How to build a number guessing game from scratch

## Programs That Think

Up until now, every program you have written does the same thing every time you run it. That is not how real programs work. A game needs to check: did the player press jump? Did they run into an enemy? Is their health at zero?

Programs make decisions using **conditions** -- statements that are either true or false. Python calls these **boolean values**, and there are exactly two of them: `True` and `False`.

Open a terminal and start Python:

```
python3
```

You will see a `>>>` prompt. This means Python is waiting for you to type something. When you see `>>>` in the examples below, do not type the `>>>` part -- just type what comes after it. To leave the interactive prompt and go back to the regular terminal, type `exit()` and press `Enter`.

Try typing these:

```python
>>> 5 > 3
True
>>> 2 > 10
False
>>> "hello" == "hello"
True
```

Python looked at each statement, decided whether it was true or false, and told you the answer. That is the foundation of every decision a program makes.

## Comparison Operators

A **comparison operator** compares two values and gives back `True` or `False`. Here are all six of them:

```
Operator    Meaning                  Example        Result
--------    -------                  -------        ------
==          equal to                 5 == 5         True
!=          not equal to             5 != 3         True
<           less than                3 < 10         True
>           greater than             10 > 3         True
<=          less than or equal to    5 <= 5         True
>=          greater than or equal to 7 >= 10        False
```

Watch out for `==` versus `=`. This trips up everyone at first:

- `=` means "put this value into this variable" (assignment)
- `==` means "are these two things equal?" (comparison)

Try a few in the Python prompt to get a feel for them:

```python
>>> age = 12
>>> age == 12
True
>>> age != 15
True
>>> age > 10
True
>>> age < 10
False
```

Type `exit()` to leave the Python prompt when you are done experimenting.

---

## Your First `if` Statement

Create a new file called `decisions.py`:

```python
age = int(input("How old are you? "))

if age >= 13:
    print("You are a teenager!")
```

Run it:

```
python3 decisions.py
```

If you type `13` or higher, you see the message. If you type `12` or lower, nothing happens. The program **skips** the indented line when the condition is false.

Let's look at the structure carefully:

```
if age >= 13:
    print("You are a teenager!")
^   ^
|   |
|   This line is INDENTED (4 spaces in from the left)
|
This line starts at the left edge
```

The **indentation** is not just for looks. Python uses indentation to know which lines belong inside the `if` block. The colon `:` at the end of the `if` line is also required -- it tells Python that an indented block is coming next.

### The Rules of Indentation

1. After a line that ends with `:`, the next line must be indented.
2. Use exactly **4 spaces** for each level of indentation. (Your text editor might insert them when you press `Tab`.)
3. Every line at the same indentation level is part of the same block.

Here is an `if` block with multiple lines inside it:

```python
age = int(input("How old are you? "))

if age >= 13:
    print("You are a teenager!")
    print("Welcome to the teen club.")
    print("Enjoy your stay.")

print("This line runs no matter what.")
```

The three `print` lines are all indented by 4 spaces, so they all belong to the `if` block. The last line is back at the left edge, so it runs no matter what.

If you type `15`:

```
You are a teenager!
Welcome to the teen club.
Enjoy your stay.
This line runs no matter what.
```

If you type `10`:

```
This line runs no matter what.
```

---

## Two Paths with `if` and `else`

Most decisions have two sides: "if this is true, do A; otherwise, do B." That is what `else` is for.

Update your `decisions.py` file:

```python
age = int(input("How old are you? "))

if age >= 13:
    print("You are a teenager!")
else:
    print("You are not a teenager yet.")

print("Thanks for telling me your age.")
```

Now something happens either way. If the condition is true, Python runs the `if` block. If the condition is false, Python runs the `else` block. It always picks exactly one.

```
How old are you? 10
You are not a teenager yet.
Thanks for telling me your age.
```

Think of it like a fork in a road:

```
        [Is age >= 13?]
          /         \
        YES          NO
        /             \
  "teenager!"    "not yet"
        \             /
         \           /
      "Thanks for telling me"
```

Both paths lead back to the same place after the decision.

---

## Multiple Paths with `elif`

Sometimes there are more than two options. That is where **`elif`** comes in. It is short for "else if."

Replace the contents of `decisions.py` with this:

```python
score = int(input("Enter your score (0-100): "))

if score >= 90:
    print("Grade: A")
elif score >= 80:
    print("Grade: B")
elif score >= 70:
    print("Grade: C")
elif score >= 60:
    print("Grade: D")
else:
    print("Grade: F")
```

Python checks each condition from top to bottom. The moment it finds one that is true, it runs that block and **skips the rest**. If none of the conditions are true, it runs the `else` block.

```
Enter your score (0-100): 85
Grade: B
```

```
Enter your score (0-100): 42
Grade: F
```

The order matters. Python stops checking as soon as it finds a match. A score of 95 is greater than 90, so it prints "Grade: A" and never even looks at the other conditions.

> Did you know? The `elif` keyword is a Python-specific shortening. Some other programming languages use `else if` (two words) instead. Python chose `elif` to keep the indentation clean.

---

## Combining Conditions: `and`, `or`, `not`

Sometimes you need to check more than one thing at once. Python gives you three **logical operators** for this.

### `and` -- Both Must Be True

```python
age = 12
has_ticket = True

if age >= 10 and has_ticket:
    print("You can ride the roller coaster!")
```

With `and`, the whole condition is only true when **both** sides are true:

```
  A      |  B      |  A and B
---------+---------+----------
  True   |  True   |  True
  True   |  False  |  False
  False  |  True   |  False
  False  |  False  |  False
```

### `or` -- At Least One Must Be True

```python
day = input("What day is it? ")

if day == "Saturday" or day == "Sunday":
    print("It's the weekend!")
```

With `or`, the whole condition is true when **at least one** side is true:

```
  A      |  B      |  A or B
---------+---------+---------
  True   |  True   |  True
  True   |  False  |  True
  False  |  True   |  True
  False  |  False  |  False
```

### `not` -- Flip It

`not` turns `True` into `False` and `False` into `True`.

```python
game_over = False

if not game_over:
    print("Keep playing!")
```

```
  A      |  not A
---------+---------
  True   |  False
  False  |  True
```

You can combine these operators. Parentheses help make things clear:

```python
if (age >= 10 and age <= 14) or has_vip_pass:
    print("You can enter.")
```

This reads as: "if age is between 10 and 14, OR they have a VIP pass."

---

## Nested `if` Statements

You can put an `if` statement inside another `if` statement. This is called **nesting**.

```python
has_key = True
door_locked = True

if has_key:
    if door_locked:
        print("You unlock the door and go through.")
    else:
        print("The door is already open. You go through.")
else:
    print("The door is locked and you don't have a key.")
```

Notice the indentation: the inner `if`/`else` is indented 4 spaces inside the outer `if`, and the code inside the inner blocks is indented another 4 spaces (8 total).

Nesting works, but do not go too deep. If you find yourself nesting three or four levels, there is usually a cleaner way to write it. For now, one level of nesting is plenty.

---

## Building a Number Guessing Game

Time to use everything you have learned. We are going to build a game where the computer picks a secret number and the player tries to guess it.

### Step 1: Think Before You Code

Before writing any Python, let's plan what the program needs to do. Let's write **pseudocode** again, like we did in Chapter 2 -- planning our steps in plain English before writing real code.

```
1. Pick a random number between 1 and 20
2. Ask the player for their guess
3. If the guess is too high, say "Too high!"
4. If the guess is too low, say "Too low!"
5. If the guess is correct, say "You got it!"
```

That is the whole game. Now we translate each line into Python.

### Step 2: Pick a Random Number

Python has a built-in **module** called `random` that can generate random numbers. A module is a collection of useful tools that someone else wrote. You load it with the `import` keyword.

Create a new file called `guess.py` and type this:

```python
import random

secret = random.randint(1, 20)
print("(Debug) The secret number is:", secret)
```

`random.randint(1, 20)` picks a random whole number between 1 and 20 (including both 1 and 20). The debug line is there so we can check that the program works. We will remove it later.

Run it a few times:

```
python3 guess.py
```

```
(Debug) The secret number is: 14
```

You should get a different number each time (or occasionally the same one -- that is how randomness works).

### Step 3: Get the Player's Guess

Add these lines after the existing code:

```python
import random

secret = random.randint(1, 20)
print("(Debug) The secret number is:", secret)

print("I'm thinking of a number between 1 and 20.")
guess = int(input("Your guess: "))
```

`int(input(...))` asks for input and converts it to a number, just like you did in Chapter 2.

### Step 4: Compare the Guess

Now for the decision-making part. Add the `if`/`elif`/`else` block:

```python
import random

secret = random.randint(1, 20)
print("(Debug) The secret number is:", secret)

print("I'm thinking of a number between 1 and 20.")
guess = int(input("Your guess: "))

if guess < secret:
    print("Too low!")
elif guess > secret:
    print("Too high!")
else:
    print("You got it!")
```

Run it and test all three outcomes. Use the debug line to verify the answers are correct.

```
(Debug) The secret number is: 12
I'm thinking of a number between 1 and 20.
Your guess: 5
Too low!
```

```
(Debug) The secret number is: 7
I'm thinking of a number between 1 and 20.
Your guess: 15
Too high!
```

```
(Debug) The secret number is: 3
I'm thinking of a number between 1 and 20.
Your guess: 3
You got it!
```

### Step 5: Clean Up

The game works. Remove the debug line so the player cannot cheat:

```python
import random

secret = random.randint(1, 20)

print("I'm thinking of a number between 1 and 20.")
guess = int(input("Your guess: "))

if guess < secret:
    print("Too low!")
elif guess > secret:
    print("Too high!")
else:
    print("You got it!")
```

Your code should now look exactly like the listing above. This is your finished guessing game for Chapter 3. Right now the player only gets one guess -- in the next chapter, you will learn how to let them keep guessing.

> Indentation can be confusing at first. If you get an `IndentationError`, check that your spaces line up. Every line inside an `if`, `elif`, or `else` block must be indented by exactly 4 spaces. Most text editors have a setting to show spaces as small dots, which helps a lot.

---

## What You Learned

- **Conditions** are expressions that evaluate to `True` or `False`.
- **Comparison operators** (`==`, `!=`, `<`, `>`, `<=`, `>=`) compare two values.
- **`if` statements** run a block of code only when a condition is true.
- **`else`** provides an alternative path when the condition is false.
- **`elif`** lets you check multiple conditions in order.
- **Logical operators** (`and`, `or`, `not`) combine conditions.
- **Nesting** means putting one `if` inside another.
- **`import random`** loads the random module, and `random.randint(a, b)` picks a random number between `a` and `b`.
- **Pseudocode** is a way to plan your program in plain language before writing real code.

## Quick Quiz

1. What is the difference between `=` and `==`?

2. What does this code print?

```python
x = 5
if x > 10:
    print("big")
elif x > 3:
    print("medium")
else:
    print("small")
```

3. Is this condition `True` or `False`?

```python
True and False
```

4. What is wrong with this code?

```python
name = "Alice"
if name == "Alice":
print("Hello, Alice!")
```

## Experiment

1. In the grading program, add a check at the top: if the score is less than 0 or greater than 100, print "That's not a valid score!" and skip the grading.

2. Change the guessing game to pick a number between 1 and 100 instead of 1 and 20. How does that change the difficulty?

3. Add an extra message to the guessing game: if the player's guess is within 2 of the secret number (but not exactly right), print "So close!" instead of just "Too high!" or "Too low!"

## Chapter Challenge

**Build a simple text adventure.**

The player is standing in front of a house. They can go `left` (to the garden), `right` (to the garage), or `inside` (through the front door). Each path should lead to a different description and a second choice. For example, inside the house they might find a friendly cat or a locked chest.

Use `input()` to get the player's choices and `if`/`elif`/`else` to respond.

**Hints (try before reading!):**

1. Start with `input("Do you go left, right, or inside? ")` and three `if`/`elif`/`else` branches.
2. Inside each branch, add another `input()` and another set of `if`/`elif`/`else` for the second choice.
3. Remember to use `==` to compare the player's input to the expected strings.
