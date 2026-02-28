---
layout: default
title: "Chapter 6: Lists and Dictionaries"
nav_order: 6
parent: "Part 1: Learning Python"
---

# Chapter 6: Lists and Dictionaries

## What You Will Learn
- What a **list** is and why you need one
- How to create, access, and modify lists
- Why Python starts counting at zero
- How to loop through a list with `for`
- What a **dictionary** is and how it pairs keys with values
- How to build a quiz game using lists and dictionaries together

## Lists: Keeping Things in Order

So far, every variable you have created holds a single value. One number. One string. One True or False. But what if you need to store a whole collection of things -- ten enemy names, a list of high scores, every item in a player's inventory?

You could do this:

```python
item1 = "sword"
item2 = "shield"
item3 = "potion"
item4 = "key"
```

But that is painful. What if you had 100 items? What if you wanted to loop through all of them? You would need 100 variable names and a very long program.

A **list** solves this. A list is a collection of items stored in a specific order, all inside a single variable.

```python
inventory = ["sword", "shield", "potion", "key"]
```

Think of it like a row of numbered boxes:

```
  inventory
  +--------+--------+--------+--------+
  | sword  | shield | potion |  key   |
  +--------+--------+--------+--------+
     [0]      [1]      [2]      [3]
```

Each box has a number called an **index**. You use the index to reach into the list and grab a specific item.

### Zero-Indexing

Here is the part that surprises almost everyone: Python starts counting at 0, not 1. The first item is at index `0`. The second item is at index `1`. The fourth item is at index `3`.

This is called **zero-indexing** and nearly every programming language does it. It feels strange at first, but you will get used to it.

Create a new file called `lists.py` in VS Code to try it out.

```python
inventory = ["sword", "shield", "potion", "key"]

print(inventory[0])
print(inventory[1])
print(inventory[2])
print(inventory[3])
```

```
sword
shield
potion
key
```

If you try to access an index that does not exist, Python gives an error:

```python
print(inventory[4])
```

```
IndexError: list index out of range
```

This error means you asked for a position that is past the end of the list. With four items, the valid indexes are 0, 1, 2, and 3.

> When you see `IndexError: list index out of range`, count the items in your list and check your index. Remember that the last valid index is always one less than the length of the list.

---

## Modifying Lists

Lists are not frozen. You can add, remove, and change items after creating them.

### Changing an Item

Use the index to replace an item:

```python
inventory = ["sword", "shield", "potion", "key"]
inventory[0] = "magic sword"
print(inventory)
```

```
['magic sword', 'shield', 'potion', 'key']
```

### Adding Items

`append` adds an item to the end of the list:

```python
inventory = ["sword", "shield"]
inventory.append("potion")
print(inventory)
```

```
['sword', 'shield', 'potion']
```

`insert` adds an item at a specific position. Everything after that position shifts over:

```python
inventory = ["sword", "shield", "potion"]
inventory.insert(1, "helmet")
print(inventory)
```

```
['sword', 'helmet', 'shield', 'potion']
```

`"helmet"` went into position 1, and `"shield"` moved from position 1 to position 2.

### Removing Items

`remove` deletes the first item that matches what you give it:

```python
inventory = ["sword", "shield", "potion"]
inventory.remove("shield")
print(inventory)
```

```
['sword', 'potion']
```

`pop` removes the item at a specific index and gives it back to you:

```python
inventory = ["sword", "shield", "potion"]
dropped = inventory.pop(1)
print(dropped)
print(inventory)
```

```
shield
['sword', 'potion']
```

If you call `pop()` with no index, it removes the last item.

### Slicing

You can grab a portion of a list using **slicing**. Write the start and end positions separated by a colon inside square brackets:

```python
colors = ["red", "green", "blue", "yellow", "purple"]
print(colors[1:3])
```

```
['green', 'blue']
```

Slicing gives you items from the start position up to (but not including) the end position. You can also leave out one side:

```python
print(colors[2:])    # From index 2 to the end
print(colors[:3])    # From the start up to index 3
```

```
['blue', 'yellow', 'purple']
['red', 'green', 'blue']
```

We will use slicing in later chapters when building games, so remember that `my_list[1:]` means "everything except the first item."

---

## Looping Through Lists

In Chapter 4 you learned `for` loops. They work beautifully with lists:

```python
inventory = ["sword", "shield", "potion", "key"]

for item in inventory:
    print("You have a " + item)
```

```
You have a sword
You have a shield
You have a potion
You have a key
```

On each pass through the loop, `item` takes the value of the next thing in the list. First `"sword"`, then `"shield"`, then `"potion"`, then `"key"`.

### List Length and Checking Membership

`len()` tells you how many items are in a list:

```python
inventory = ["sword", "shield", "potion"]
print(len(inventory))
```

```
3
```

The keyword `in` checks whether something exists in a list. It gives back `True` or `False`:

```python
inventory = ["sword", "shield", "potion"]

if "sword" in inventory:
    print("You are armed!")

if "armor" not in inventory:
    print("You have no armor. Be careful!")
```

```
You are armed!
You have no armor. Be careful!
```

This is very useful in games. Checking whether a player has a specific item, whether a name is in a high score list, whether a level has been completed -- all of these use `in`.

### Lists Can Hold Numbers Too

Lists are not just for strings. They can hold any type of value:

```python
high_scores = [1500, 1200, 900, 850, 600]
temperatures = [22.5, 23.1, 19.8, 21.0]
mixed = ["Alice", 100, True]
```

That last one holds a string, a number, and a boolean all in the same list. Python allows this, though in practice you usually keep one type of thing per list.

---

## Dictionaries: Looking Things Up

A list is great when you have items in order and you access them by position number. But sometimes you want to look things up by a **name** instead of a number.

Imagine a real dictionary. You do not look up a word by its page number. You look it up by the word itself, and it gives you the definition.

A Python **dictionary** works the same way. Instead of numbered indexes, it uses **keys** that map to **values**.

```
  player
  +----------+-----------+
  |   key    |   value   |
  +----------+-----------+
  | "name"   | "Alice"   |
  | "health" | 100       |
  | "level"  | 3         |
  +----------+-----------+
```

Here is how to create that in Python:

```python
player = {
    "name": "Alice",
    "health": 100,
    "level": 3
}
```

In VS Code, create a new file called `dicts.py`.

```python
player = {
    "name": "Alice",
    "health": 100,
    "level": 3
}

print(player["name"])
print(player["health"])
```

```
Alice
100
```

You access values using their key in square brackets, just like a list index, but with a string instead of a number.

### Adding and Changing Entries

To add a new key-value pair, just assign it:

```python
player = {"name": "Alice", "health": 100}
player["score"] = 0
print(player)
```

```
{'name': 'Alice', 'health': 100, 'score': 0}
```

To change an existing value, assign to an existing key:

```python
player = {"name": "Alice", "health": 100}
player["health"] = 80
print(player["health"])
```

```
80
```

This is perfect for game state. A player takes damage? Subtract from `player["health"]`. They level up? Increase `player["level"]`. They pick up an item? Add it to `player["inventory"]`.

### Looping Through Dictionaries

You can loop through a dictionary in a few ways. The most common is to loop through both keys and values together:

```python
player = {
    "name": "Alice",
    "health": 100,
    "level": 3
}

for key, value in player.items():
    print(key + ": " + str(value))
```

```
name: Alice
health: 100
level: 3
```

We use `str(value)` because some values are numbers and you cannot add a number to a string with `+`. The `str()` function converts the number to a string first.

You can also loop through just the keys:

```python
for key in player:
    print(key)
```

```
name
health
level
```

### Checking If a Key Exists

Just like lists, you can use `in` to check if a key is in a dictionary:

```python
player = {"name": "Alice", "health": 100}

if "health" in player:
    print("Health:", player["health"])

if "armor" not in player:
    print("No armor equipped.")
```

```
Health: 100
No armor equipped.
```

---

## Lists and Dictionaries Together

Here is where things get powerful. You can put dictionaries inside lists, or lists inside dictionaries. This lets you organize complex data.

A list of enemies, where each enemy has a name and a health value:

```python
enemies = [
    {"name": "Goblin", "health": 30},
    {"name": "Skeleton", "health": 50},
    {"name": "Dragon", "health": 200}
]
```

You can loop through the list and access each dictionary:

```python
enemies = [
    {"name": "Goblin", "health": 30},
    {"name": "Skeleton", "health": 50},
    {"name": "Dragon", "health": 200}
]

for enemy in enemies:
    print(enemy["name"] + " - HP: " + str(enemy["health"]))
```

```
Goblin - HP: 30
Skeleton - HP: 50
Dragon - HP: 200
```

Or a player with an inventory list inside a dictionary:

```python
player = {
    "name": "Alice",
    "health": 100,
    "inventory": ["sword", "shield", "potion"]
}

print(player["name"] + "'s items:")
for item in player["inventory"]:
    print("  - " + item)
```

```
Alice's items:
  - sword
  - shield
  - potion
```

Lists and dictionaries are used in every real program. Game inventories, high scores, player data, level maps, enemy spawns -- all of these use lists and dictionaries. Once you understand how to combine them, you can represent any kind of data your game needs.

---

## Build-up Exercise: A Quiz Game

Let's build a quiz game that stores questions in a list of dictionaries. This is a common pattern you will use again and again.

### Step 1: Plan the Data

Each question needs the question text and the correct answer. A dictionary is perfect for this:

```python
{"question": "What color is the sky?", "answer": "blue"}
```

A whole quiz is a list of these dictionaries:

```python
questions = [
    {"question": "What color is the sky?", "answer": "blue"},
    {"question": "How many legs does a spider have?", "answer": "8"},
    {"question": "What planet do we live on?", "answer": "earth"}
]
```

### Step 2: Loop Through and Ask

In VS Code, create a new file called `quiz.py`.

```python
questions = [
    {"question": "What color is the sky?", "answer": "blue"},
    {"question": "How many legs does a spider have?", "answer": "8"},
    {"question": "What planet do we live on?", "answer": "earth"}
]

score = 0

for q in questions:
    print(q["question"])
    player_answer = input("Your answer: ")
    if player_answer.lower() == q["answer"]:
        print("Correct!")
        score = score + 1
    else:
        print("Wrong! The answer was: " + q["answer"])
    print()

print("You got " + str(score) + " out of " + str(len(questions)))
```

Run it:

```
python3 quiz.py
```

```
What color is the sky?
Your answer: Blue
Correct!

How many legs does a spider have?
Your answer: 8
Correct!

What planet do we live on?
Your answer: Mars
Wrong! The answer was: earth

You got 2 out of 3
```

Let's walk through a few details:

- `q["question"]` gets the question text from the current dictionary.
- `player_answer.lower()` converts the player's answer to lowercase so that "Blue", "BLUE", and "blue" all match. `.lower()` is a **method** -- a function that belongs to a string. You call it with a dot after the variable. It returns a new version of the string with all letters in lowercase, so `"Blue".lower()` gives `"blue"`. This way, it does not matter if the player types `Blue`, `BLUE`, or `blue`.
- `score = score + 1` adds a point for each correct answer.
- `len(questions)` gives us the total number of questions without counting them ourselves.

### Step 3: Add More Questions

The beauty of this design is how easy it is to expand. Want more questions? Just add more dictionaries to the list:

```python
questions = [
    {"question": "What color is the sky?", "answer": "blue"},
    {"question": "How many legs does a spider have?", "answer": "8"},
    {"question": "What planet do we live on?", "answer": "earth"},
    {"question": "What language are we programming in?", "answer": "python"},
    {"question": "How many days are in a week?", "answer": "7"},
    {"question": "What is the capital of France?", "answer": "paris"}
]
```

The rest of the code does not change at all. The `for` loop handles any number of questions because it uses `len(questions)` and loops through the entire list. This is a good example of why lists are so useful.

### Step 4: Show the Final Score with a Message

Let's make the ending more interesting. Add this after the score printout:

Your complete file should now look like this:

```python
questions = [
    {"question": "What color is the sky?", "answer": "blue"},
    {"question": "How many legs does a spider have?", "answer": "8"},
    {"question": "What planet do we live on?", "answer": "earth"},
    {"question": "What language are we programming in?", "answer": "python"},
    {"question": "How many days are in a week?", "answer": "7"},
    {"question": "What is the capital of France?", "answer": "paris"}
]

score = 0

for q in questions:
    print(q["question"])
    player_answer = input("Your answer: ")
    if player_answer.lower() == q["answer"]:
        print("Correct!")
        score = score + 1
    else:
        print("Wrong! The answer was: " + q["answer"])
    print()

total = len(questions)
print("You got " + str(score) + " out of " + str(total))

if score == total:
    print("Perfect score!")
elif score >= total // 2:
    print("Good job!")
else:
    print("Keep practicing!")
```

The `//` operator is **integer division** -- it divides and drops any remainder. So `6 // 2` is `3`, and `5 // 2` is `2`. We use it here to find the halfway mark.

---

## What You Learned

- A **list** stores multiple items in order inside a single variable. You create one with square brackets: `["a", "b", "c"]`.
- Items are accessed by **index**, starting at 0. The first item is `my_list[0]`.
- Lists can be modified with `append`, `insert`, `remove`, and `pop`.
- A **dictionary** stores **key-value pairs**. You create one with curly braces: `{"key": "value"}`.
- You look up dictionary values using their key: `my_dict["key"]`.
- Both lists and dictionaries work with `for` loops, `len()`, and `in`.
- You can put dictionaries inside lists to represent collections of structured data -- like quiz questions, enemies, or player profiles.

## Quick Quiz

1. What index is the first item in a Python list?

2. What is the difference between `append` and `insert`?

3. What does this code print?

```python
animals = ["cat", "dog", "fish"]
animals.append("bird")
print(len(animals))
```

4. Given this dictionary, how would you print the player's health?

```python
player = {"name": "Bob", "health": 75, "level": 2}
```

## Experiment

1. Create a list of your five favorite foods. Print them using a `for` loop, numbered from 1 (not 0). Hint: you can use a variable that starts at 1 and increases by 1 each time through the loop.

2. Create a dictionary that represents a pet (name, species, age, color). Print all the information using a loop through `.items()`.

3. In the quiz game, try making questions have multiple valid answers. Change the `"answer"` value to a list of strings (for example, `["8", "eight"]`) and modify the checking code to handle this. Hint: you can use `in` to check if the player's answer is in the list.

## Chapter Challenge

Build an **inventory system** for a text adventure game.

The program should:
1. Start the player with an empty inventory (an empty list)
2. Show a menu: "pickup", "drop", "list", "quit"
3. If the player types "pickup", ask what item they want and add it to the inventory
4. If the player types "drop", ask what item and remove it (but tell the player if they do not have that item)
5. If the player types "list", show all items in the inventory with numbers starting from 1
6. If the player types "quit", end the program and show how many items they ended up with

**Hint 1:** Use a `while True` loop for the main menu.

**Hint 2:** Before removing an item, check if it is `in` the inventory list.

**Hint 3:** To show numbered items, you can keep a counter variable or use the `enumerate` function -- try searching "python enumerate" if you are curious.
