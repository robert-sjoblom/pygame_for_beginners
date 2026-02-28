---
layout: default
title: "Chapter 2: Variables and Values"
nav_order: 2
parent: "Part 1: Learning Python"
---

# Chapter 2: Variables and Values

## What You Will Learn
- What variables are and how to use them
- The different types of values: numbers, text, and booleans
- How to get input from the user
- How to convert between types
- How to build a simple calculator program

## What Is a Variable?

A **variable** is a name that refers to a value. You can think of it like a labeled box. You put something in the box, and later you can look at the label to find it again.

```
  score          name            lives
 +-------+     +---------+     +-------+
 |  100  |     | "Alice" |     |   3   |
 +-------+     +---------+     +-------+
```

In Python, you create a variable by writing a name, an equals sign, and a value:

```python
score = 100
```

That line creates a variable called `score` and stores the number `100` in it. The `=` sign does not mean "is equal to" here — it means "store this value." Programmers call this **assignment**.

You can use the variable's name anywhere you would use the value itself:

```python
score = 100
print(score)
```

Run that, and you will see:

```
100
```

Notice that there are no quotation marks around `score` inside `print()`. If you wrote `print("score")`, Python would print the word "score" instead of the number 100. Without quotes, Python looks up the variable. With quotes, Python treats it as text.

You can change a variable by assigning it a new value:

```python
score = 100
print(score)
score = 200
print(score)
```

Output:

```
100
200
```

The old value is gone. The box now holds 200 instead of 100.

### Naming Rules

Variable names in Python follow a few rules:

- They can contain letters, numbers, and underscores (`_`).
- They cannot start with a number. `score1` is fine, but `1score` is not.
- They are case-sensitive. `Score` and `score` are two different variables.
- They cannot be Python **keywords** — special words that Python reserves for itself, like `if`, `for`, and `while`.

In this book, we use **snake_case**: all lowercase, with underscores between words. For example: `player_speed`, `high_score`, `enemy_count`. This is the standard style in Python.

> Choosing good variable names matters. A name like `x` tells you nothing. A name like `player_speed` tells you exactly what it holds. When your programs get longer, clear names will save you a lot of confusion.

---

## Numbers

Python has two main types of numbers.

An **integer** (or **int**) is a whole number with no decimal point:

```python
lives = 3
high_score = 10000
temperature = -5
```

A **float** is a number with a decimal point:

```python
speed = 2.5
gravity = 9.8
price = 0.99
```

### Arithmetic

Python can do math with numbers. Create a new file called `math_test.py` and type this:

```python
a = 10
b = 3

print(a + b)
print(a - b)
print(a * b)
print(a / b)
```

Run it:

```
13
7
30
3.3333333333333335
```

Notice that division with `/` always gives you a float, even when the numbers divide evenly. Try `print(10 / 2)` and you will get `5.0`, not `5`.

There are a few more operators worth knowing. Add these lines to your file:

```python
print(a // b)
print(a % b)
print(a ** b)
```

Run the whole file again. The new output at the end will be:

```
3
1
1000
```

Here is what those operators do:

```
 Operator   Name                 Example    Result
 --------   ----                 -------    ------
 +          Addition             10 + 3     13
 -          Subtraction          10 - 3     7
 *          Multiplication       10 * 3     30
 /          Division             10 / 3     3.333...
 //         Integer division     10 // 3    3
 %          Remainder (modulo)   10 % 3     1
 **         Power (exponent)     10 ** 3    1000
```

**Integer division** (`//`) divides and throws away the decimal part. The **remainder** or **modulo** (`%`) gives you what is left over after division. The **power** operator (`**`) raises a number to an exponent — `10 ** 3` means 10 times 10 times 10.

> The modulo operator `%` is surprisingly useful in games. For example, you can use it to check if a number is even or odd: if the remainder when dividing by 2 is 0, the number is even.

---

## Strings

A **string** is a piece of text. You create one by putting characters inside quotation marks:

```python
name = "Python"
greeting = 'Hello there'
```

You can use double quotes or single quotes — both work the same way. Pick one and stick with it. This book uses double quotes.

### Joining Strings Together

You can combine strings with the `+` operator. This is called **concatenation**:

```python
first = "Game"
second = "Over"
message = first + " " + second
print(message)
```

Output:

```
Game Over
```

We added a `" "` (a string containing just a space) between the two words. Without it, you would get `GameOver`.

### Useful String Tools

Strings come with built-in tools called **methods**. Here are a few:

```python
shout = "hello".upper()
print(shout)

whisper = "HELLO".lower()
print(whisper)

length = len("Python")
print(length)
```

Output:

```
HELLO
hello
6
```

The `.upper()` method returns a new string with all letters in uppercase. The `.lower()` method does the opposite. The `len()` function (not a method — notice it goes around the string, not after it) tells you how many characters are in a string.

### f-strings

Often you want to mix text and variables. The cleanest way to do this in Python is with an **f-string**. Put the letter `f` right before the opening quote, and then use curly braces `{}` to insert variables:

```python
player_name = "Alex"
player_score = 42
print(f"{player_name} scored {player_score} points!")
```

Output:

```
Alex scored 42 points!
```

The `f` stands for "format." Inside the curly braces, Python evaluates whatever you write and inserts the result into the string. You can even put math inside the braces:

```python
width = 8
height = 5
print(f"Area: {width * height}")
```

Output:

```
Area: 40
```

f-strings are one of the most useful features in Python. You will use them constantly.

---

## Booleans

A **boolean** is a value that is either `True` or `False`. Nothing else.

```python
game_over = False
has_key = True
print(game_over)
print(has_key)
```

Output:

```
False
True
```

Note the capital letters — `True` and `False`, not `true` and `false`. Python is picky about this.

Booleans become important when your program needs to make decisions. We will use them heavily starting in the next chapter. For now, just know they exist and that there are only two possible values.

---

## Getting Input from the User

So far, our programs just print things out. Let's make them interactive. The `input()` function pauses the program and waits for the user to type something:

```python
name = input("What is your name? ")
print(f"Hello, {name}!")
```

Run this program. You will see:

```
What is your name?
```

The program is waiting. Type your name and press `Enter`:

```
What is your name? Sam
Hello, Sam!
```

The string you pass to `input()` is the **prompt** — it tells the user what to type. The space at the end of `"What is your name? "` is there so the cursor doesn't bump right up against the question mark.

Here is the important thing: **`input()` always returns a string.** Even if the user types a number, Python treats it as text. This matters, and we will deal with it right now.

---

## Type Conversion

Create a file called `type_test.py` and type this:

```python
age = input("How old are you? ")
print(f"Next year you will be {age + 1}")
```

Run it and type a number. You will get an error:

```
TypeError: can only concatenate str (not "int") to str
```

Python is telling you that `age` is a string (because `input()` always returns a string), and you cannot add the number `1` to a string. You need to **convert** the string to an integer first.

Fix the program like this:

```python
age = input("How old are you? ")
age = int(age)
print(f"Next year you will be {age + 1}")
```

Now it works:

```
How old are you? 12
Next year you will be 13
```

The `int()` function converts a string to an integer. There is also `float()` for decimal numbers and `str()` for turning numbers back into strings.

```python
whole = int("42")       # string to integer
decimal = float("3.14") # string to float
text = str(100)          # integer to string
```

### When Conversion Fails

What happens if you try `int("hello")`? Python cannot turn the word "hello" into a number, so it gives you an error:

```
ValueError: invalid literal for int() with base 10: 'hello'
```

For now, just be careful to only convert strings that actually contain numbers. We will learn how to handle these errors properly in a later chapter.

---

## Building a Calculator

Let's put everything together and build something useful. We will make a calculator that asks for two numbers and an operation, then shows the result.

### Think Before You Code

Before writing any code, let's think about what our program needs to do. Programmers call this writing **pseudocode** — a plan written in plain language, not in Python.

```
CALCULATOR PLAN
---------------
1. Ask the user for the first number
2. Ask the user for the second number
3. Ask the user which operation they want (+, -, *, /)
4. Convert the numbers from strings to floats
5. Do the math based on the operation
6. Show the result
```

This is a good habit. When you plan first, the coding part goes faster because you already know what to write.

### Writing the Code

Create a new file called `calculator.py`. Let's build it step by step.

First, get the input from the user:

```python
print("=== Simple Calculator ===")
first = input("Enter the first number: ")
second = input("Enter the second number: ")
operation = input("Enter an operation (+, -, *, /): ")
```

Now convert the number inputs to floats so we can do math with them:

```python
first = float(first)
second = float(second)
```

Since we are showing all four operations at once, you can delete the `operation = input(...)` line -- we will not need it yet. We will use it in the Chapter Challenge when you add `if` statements.

Now do the calculation. We don't know `if` statements yet (that is next chapter), so we will calculate all four operations and show the one the user asked for. Actually, let's keep it simple and just do all four:

```python
print(f"{first} + {second} = {first + second}")
print(f"{first} - {second} = {first - second}")
print(f"{first} * {second} = {first * second}")
print(f"{first} / {second} = {first / second}")
```

Your complete `calculator.py` should now look like this:

```python
print("=== Simple Calculator ===")
first = input("Enter the first number: ")
second = input("Enter the second number: ")

first = float(first)
second = float(second)

print(f"{first} + {second} = {first + second}")
print(f"{first} - {second} = {first - second}")
print(f"{first} * {second} = {first * second}")
print(f"{first} / {second} = {first / second}")
```

Run it:

```
=== Simple Calculator ===
Enter the first number: 10
Enter the second number: 3
10.0 + 3.0 = 13.0
10.0 - 3.0 = 7.0
10.0 * 3.0 = 30.0
10.0 / 3.0 = 3.3333333333333335
```

It works. Notice that the numbers show as `10.0` and `3.0` because we used `float()`. That is fine. In the next chapter, you will learn about `if` statements, and then you can upgrade this calculator to only do the operation the user picked.

> This is how real programs are built. You start with something simple that works, then you improve it. Programmers call this **iterating**. Nobody writes a perfect program on the first try.

---

## What You Learned

- A **variable** is a name that stores a value. You create one with `=` (assignment).
- **Integers** are whole numbers. **Floats** are decimal numbers.
- Python supports arithmetic: `+`, `-`, `*`, `/`, `//`, `%`, `**`.
- **Strings** are text in quotation marks. You join them with `+` (concatenation).
- **f-strings** let you insert variables into text: `f"Hello, {name}"`.
- **Booleans** are either `True` or `False`.
- `input()` gets text from the user. It always returns a string.
- `int()`, `float()`, and `str()` convert between types.
- **Pseudocode** helps you plan before you code.

## Quick Quiz

1. What is the result of `10 // 3` in Python?
2. What does `input()` always return — a number or a string?
3. What happens if you try to run `int("banana")`?
4. Write an f-string that prints "I have 5 cats" using a variable called `count` that holds the value `5`.

## Experiment

- In `calculator.py`, change `float()` to `int()`. What happens when you enter decimal numbers like `3.5`?
- Make a program that asks for your name and then prints it in all uppercase using `.upper()`.
- Try using `**` to calculate large powers. What is `2 ** 10`? What about `2 ** 100`? Python can handle very large numbers.

## Chapter Challenge

Create a program called `about_me.py` that asks the user for their name, age, and favorite color. Then it prints a short summary using f-strings. The output should look something like this:

```
What is your name? Jordan
How old are you? 12
What is your favorite color? blue

=== About Jordan ===
Name: Jordan
Age: 12
Next birthday age: 13
Favorite color: BLUE
Name length: 6 letters
```

**Hint 1:** Use `input()` three times to get the three pieces of information.

**Hint 2:** You need `int()` to convert the age so you can add 1 to it.

**Hint 3:** Use `.upper()` on the color and `len()` on the name.
