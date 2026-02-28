---
layout: default
title: "Chapter 5: Functions"
nav_order: 5
parent: "Part 1: Learning Python"
---

# Chapter 5: Functions

## What You Will Learn
- Why functions exist and how they help you write better programs
- How to create your own functions with `def`
- How to pass information into functions with **parameters**
- How to get information back out of functions with `return`
- Why `return` and `print` are completely different things
- How to build a text adventure game where each room is a function

## You Already Know Functions

Here is a secret: you have been using functions since Chapter 1. Every time you wrote `print()`, `input()`, `int()`, `range()`, or `randint()`, you were **calling a function** that someone else wrote.

A **function** is a named block of code that does a specific job. You call it by writing its name followed by parentheses.

```python
print("Hello")          # print is a function
name = input("Name? ")  # input is a function
age = int("25")         # int is a function
```

These functions were built into Python (or into a module like `random`). But what if you could write your own? That is exactly what this chapter is about.

## Your First Function

In VS Code, create a new file called `functions.py`. Type this:

```python
def greet():
    print("Hello, adventurer!")
    print("Welcome to the dungeon.")
```

The keyword `def` means "define a function." After `def` comes the function's name (`greet`), then parentheses `()`, then a colon `:`. Everything indented underneath belongs to the function.

Now here is the important part. Save the file and run it:

```
python3 functions.py
```

Nothing happens. No output at all. That is correct. **Defining a function does not run it.** You have only taught Python a new word. You still need to **call** the function. Add this line at the bottom of your file, with no indentation:

```python
def greet():
    print("Hello, adventurer!")
    print("Welcome to the dungeon.")

greet()
```

Run it again:

```
python3 functions.py
```

```
Hello, adventurer!
Welcome to the dungeon.
```

There it is. When Python hits `greet()`, it jumps up to the function definition, runs the code inside, then comes back down and continues.

The best part? You can call it as many times as you want:

```python
def greet():
    print("Hello, adventurer!")
    print("Welcome to the dungeon.")

greet()
greet()
greet()
```

```
Hello, adventurer!
Welcome to the dungeon.
Hello, adventurer!
Welcome to the dungeon.
Hello, adventurer!
Welcome to the dungeon.
```

Write it once, use it many times. That is one of the biggest ideas in programming.

---

## Functions With Parameters

Our `greet` function says the same thing every time. What if we want to greet different people by name? We can pass information into a function using a **parameter**.

Create a new file called `greet2.py` in VS Code.

```python
def greet(name):
    print("Hello, " + name + "!")
    print("Welcome to the dungeon.")

greet("Alice")
greet("Bob")
greet("Charlie")
```

```
Hello, Alice!
Welcome to the dungeon.
Hello, Bob!
Welcome to the dungeon.
Hello, Charlie!
Welcome to the dungeon.
```

The word `name` inside the parentheses is a **parameter**. It is a placeholder. When you call `greet("Alice")`, the value `"Alice"` gets placed into `name` for that one call. Next time you call `greet("Bob")`, `name` becomes `"Bob"`.

Think of it like a mail slot in a door. You slide a value through the slot, and the function uses it inside.

```
                    +---------+
  "Alice" -------> | name    |  <-- the parameter
                    |         |
                    | print() |  <-- the function body
                    +---------+
```

### Multiple Parameters

Functions can take more than one parameter. Separate them with commas:

```python
def greet(name, location):
    print("Hello, " + name + "!")
    print("Welcome to " + location + ".")

greet("Alice", "the dungeon")
greet("Bob", "the forest")
```

```
Hello, Alice!
Welcome to the dungeon.
Hello, Bob!
Welcome to the forest.
```

The order matters. The first value you pass goes into the first parameter, the second value into the second parameter.

### Default Parameter Values

Sometimes you want a parameter to have a **default value** -- a fallback if no value is given:

```python
def greet(name, location="the dungeon"):
    print("Hello, " + name + "!")
    print("Welcome to " + location + ".")

greet("Alice")
greet("Bob", "the forest")
```

```
Hello, Alice!
Welcome to the dungeon.
Hello, Bob!
Welcome to the forest.
```

Alice did not provide a location, so the default `"the dungeon"` was used. Bob provided `"the forest"`, so that replaced the default.

One rule: parameters with default values must come after parameters without them. `def greet(location="the dungeon", name)` would give you an error.

---

## Functions That Return Values

So far our functions have printed things. But sometimes you want a function to calculate something and give the answer back to you. That is what `return` does.

Create a new file called `calculate.py` in VS Code.

```python
def add(a, b):
    result = a + b
    return result

answer = add(3, 5)
print(answer)
```

```
8
```

When Python reaches `return result`, it sends the value of `result` back to wherever the function was called. In this case, `add(3, 5)` becomes `8`, and that `8` gets stored in `answer`.

Here is what happens step by step:

```
  answer = add(3, 5)
               |
               v
        def add(a, b):       a = 3, b = 5
            result = a + b   result = 8
            return result     sends 8 back
               |
               v
  answer = 8
```

### The Difference Between print and return

This is the part that confuses almost everyone when they first learn it. Read this section carefully. Read it twice if you need to.

`print` shows something on the screen. That is all it does. It does not send any value anywhere.

`return` sends a value back to the code that called the function. It does not show anything on the screen.

Let's see the difference:

```python
def add_with_print(a, b):
    print(a + b)

def add_with_return(a, b):
    return a + b
```

They look similar. But watch what happens when we try to use them:

```python
def add_with_print(a, b):
    print(a + b)

def add_with_return(a, b):
    return a + b

result1 = add_with_print(3, 5)
result2 = add_with_return(3, 5)

print("result1 is:", result1)
print("result2 is:", result2)
```

```
8
result1 is: None
result2 is: 8
```

Wait -- `result1` is `None`? What happened?

`add_with_print` displayed `8` on the screen (you can see it on the first line), but it did not return anything. When a function does not return a value, Python gives back a special value called `None`, meaning "nothing."

`add_with_return` did not display anything on its own, but it sent `8` back to the caller. Now `result2` holds `8` and we can use it for other calculations.

Here is a way to think about it:

- `print` is like shouting the answer out loud in a room. Everyone hears it, but nobody wrote it down.
- `return` is like writing the answer on a piece of paper and handing it to the person who asked. They can use it however they want.

If you need to **use the result later** (in another calculation, in an if-statement, stored in a variable), use `return`. If you just want to **show something to the user**, use `print`.

> This trips up even experienced beginners. If your function seems to work when you test it but later your code behaves strangely, check: are you using `print` when you should be using `return`?

---

## Scope: What Happens Inside Stays Inside

Variables created inside a function are separate from variables outside. This is called **scope**.

```python
def set_score():
    score = 100
    print("Inside the function:", score)

score = 0
set_score()
print("Outside the function:", score)
```

```
Inside the function: 100
Outside the function: 0
```

Wait -- we set `score = 100` inside the function, but the `score` outside is still `0`? Yes. The `score` inside the function is a different variable that just happens to have the same name. Think of it like boxes inside boxes:

```
+--------------------------------------+
|  Main program                        |
|  score = 0                           |
|                                      |
|    +------------------------------+  |
|    |  set_score() function        |  |
|    |  score = 100  (different!)   |  |
|    +------------------------------+  |
|                                      |
|  score is still 0 here               |
+--------------------------------------+
```

The function has its own box. Variables inside that box only exist while the function is running. Once the function finishes, those variables disappear.

This is actually a good thing. It means your functions cannot accidentally break variables in the rest of your program. Each function is its own little world.

If you need to get a value out of a function, use `return`:

```python
def calculate_score():
    score = 100
    return score

final_score = calculate_score()
print("Score:", final_score)
```

```
Score: 100
```

---

## Build-up Exercise: A Text Adventure

Now let's put everything together and build a real program: a text adventure game where each room is a function.

### Step 1: Plan the Map

Before writing any code, let's think about what we need. Our adventure will have a few rooms connected together:

```
                    +----------+
                    |  Start   |
                    |  Room    |
                    +----+-----+
                         |
                    +----+-----+
                    | Corridor |
                    +----+-----+
                    /           \
           +------+---+    +----+------+
           | Treasure |    |  Monster  |
           |  Room    |    |   Room    |
           +----------+    +-----------+
```

The player starts in the Start Room. From there they go to the Corridor. In the Corridor they choose to go left (Treasure Room) or right (Monster Room).

### Step 2: Pseudocode

Before we write Python, let's write our plan in plain English. This is called **pseudocode** -- it looks like code but is just describing what we want to happen.

```
function start_room:
    print description of the starting room
    ask the player to continue
    go to corridor

function corridor:
    print description of the corridor
    ask the player: left or right?
    if left: go to treasure_room
    if right: go to monster_room

function treasure_room:
    print "You found the treasure! You win!"

function monster_room:
    print "A monster appears!"
    ask: fight or run?
    if fight: print "You defeated the monster!"
    if run: go back to corridor
```

### Step 3: Write the Code

In VS Code, create a new file called `text_adventure.py`. We will build it function by function. Start with the simplest rooms at the bottom of the map, then work upward.

First, the two ending rooms:

```python
def treasure_room():
    print()
    print("=== Treasure Room ===")
    print("Gold coins cover the floor!")
    print("You fill your pockets.")
    print("Congratulations, you win!")

def monster_room():
    print()
    print("=== Monster Room ===")
    print("A giant spider drops from the ceiling!")
    choice = input("Do you fight or run? ")
    if choice == "fight":
        print("You swing your torch and scare it away!")
        print("Behind it, you find a hidden exit. You win!")
    elif choice == "run":
        print("You dash back the way you came!")
        corridor()
    else:
        print("The spider stares at you, confused.")
        monster_room()
```

> Do not run this yet -- we still need to add the `corridor` and `start_room` functions.

Notice that `monster_room` calls `corridor()` if the player runs. And if they type something invalid, it calls `monster_room()` again -- the function calls itself! This is fine for a small game like ours.

Now add the corridor above those two functions:

```python
def corridor():
    print()
    print("=== Corridor ===")
    print("A long hallway stretches before you.")
    print("To the left, you see a golden glow.")
    print("To the right, you hear strange noises.")
    choice = input("Do you go left or right? ")
    if choice == "left":
        treasure_room()
    elif choice == "right":
        monster_room()
    else:
        print("You need to choose left or right.")
        corridor()
```

And finally the start room and the code that actually starts the game. Here is the complete file:

```python
def start_room():
    print("=== The Dungeon ===")
    print("You wake up in a dark stone room.")
    print("A single torch flickers on the wall.")
    print("There is a doorway ahead.")
    input("Press Enter to continue... ")
    corridor()

def corridor():
    print()
    print("=== Corridor ===")
    print("A long hallway stretches before you.")
    print("To the left, you see a golden glow.")
    print("To the right, you hear strange noises.")
    choice = input("Do you go left or right? ")
    if choice == "left":
        treasure_room()
    elif choice == "right":
        monster_room()
    else:
        print("You need to choose left or right.")
        corridor()

def treasure_room():
    print()
    print("=== Treasure Room ===")
    print("Gold coins cover the floor!")
    print("You fill your pockets.")
    print("Congratulations, you win!")

def monster_room():
    print()
    print("=== Monster Room ===")
    print("A giant spider drops from the ceiling!")
    choice = input("Do you fight or run? ")
    if choice == "fight":
        print("You swing your torch and scare it away!")
        print("Behind it, you find a hidden exit. You win!")
    elif choice == "run":
        print("You dash back the way you came!")
        corridor()
    else:
        print("The spider stares at you, confused.")
        monster_room()

start_room()
```

The very last line, `start_room()`, is what actually starts the game. Everything above it is just definitions -- teaching Python the words. That last line says "go."

Run it:

```
python3 text_adventure.py
```

```
=== The Dungeon ===
You wake up in a dark stone room.
A single torch flickers on the wall.
There is a doorway ahead.
Press Enter to continue...
```

Play through the game a few times. Try every path. Try typing something unexpected and see what happens.

> This approach of calling functions from inside other functions works well for a small adventure like ours. In a bigger game with many rooms, you would use a different technique -- like a variable that tracks which room the player is in and a loop. You will see this approach when we build games with pygame.

### What Makes This Design Good?

Look at what we built. Each room is a self-contained function. If you want to add a new room, you write one new function and connect it by calling it from another room. You do not have to rewrite anything else.

This is the real power of functions: you break a big problem into small pieces, solve each piece separately, and connect them together.

> Early text adventure games like *Zork* (1977) were built using this same idea -- separate functions for separate locations. The games were enormous, but each piece was manageable on its own.

---

## What You Learned

- A **function** is a reusable block of code with a name. You define it with `def` and call it with its name followed by `()`.
- **Parameters** let you pass information into a function. **Default values** provide a fallback.
- `return` sends a value back to the caller. `print` just shows text on the screen. They are not the same thing.
- Variables inside a function are in their own **scope** -- they do not affect variables outside.
- Functions can call other functions. This lets you break a big program into small, manageable pieces.
- Write it once, use it many times.

## Quick Quiz

1. What keyword do you use to create a new function?

2. What is the difference between defining a function and calling it?

3. What does this code print?

```python
def double(n):
    return n * 2

result = double(7)
print(result)
```

4. What does this code print?

```python
def mystery():
    x = 10

x = 5
mystery()
print(x)
```

## Experiment

1. In the text adventure, add a new room called `puzzle_room` that asks a riddle. If the player answers correctly, they go to the treasure room. If not, they get sent back to the corridor.

2. Write a function called `shout` that takes a word as a parameter and returns that word in all uppercase with three exclamation marks. (Hint: the string method `.upper()` converts text to uppercase.)

3. Try giving `greet` a parameter with a default value that is a number instead of a string. What happens if you try to add it to a string with `+`? Can you fix it using `str()`?

## Chapter Challenge

Build a **calculator** using functions. Write four functions: `add(a, b)`, `subtract(a, b)`, `multiply(a, b)`, and `divide(a, b)`. Each should **return** the result (not print it).

Then write a main part of the program that:
1. Asks the user for a number
2. Asks for an operation (+, -, *, /)
3. Asks for a second number
4. Calls the right function and prints the result
5. Asks if the user wants to do another calculation

**Hint 1:** Use `float()` instead of `int()` so the calculator can handle decimal numbers.

**Hint 2:** Use a `while` loop to let the user keep calculating.

**Hint 3:** For division, what happens if the second number is 0? You could check for that inside your `divide` function and return a message like `"Cannot divide by zero"`.
