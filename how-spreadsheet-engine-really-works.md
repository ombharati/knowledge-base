# Building a Spreadsheet Engine from Scratch: A First-Principles Guide

Welcome, curious explorer! 

Have you ever wondered how a spreadsheet works? When you type a number into a box, and another box instantly changes, how does the computer know what to do? How does it trace the paths between numbers, solve math equations, and keep everything running without getting confused?

In this guide, we are going to build a spreadsheet engine from the absolute ground up. We will assume you know **nothing** about spreadsheets, programming, or computer science. We will start from the very beginning—with how computers store single pieces of information—and build our way up to a fully functional, cycle-detecting, formula-parsing spreadsheet engine.

Let us begin our journey!

---

## Historical Context: The Software That Changed the World

Before we dive into the code, let's take a quick look at why spreadsheets are so important. They are not just handy utilities; they are the reason personal computers became popular in offices!

```text
VisiCalc (1979)         Lotus 1-2-3 (1983)         Excel (1985)          Google Sheets (2006)
┌──────────────┐        ┌──────────────┐         ┌──────────────┐        ┌──────────────┐
│ First dynamic│ ────►  │ Standardized │  ────►  │ GUI-driven,  │ ────►  │ Cloud-based, │
│ grid program │        │ speed/charts │         │ cached graphs│        │ collab engine│
└──────────────┘        └──────────────┘         └──────────────┘        └──────────────┘
```

* **VisiCalc (1979):** The first spreadsheet. Before VisiCalc, computers were seen as expensive toys or systems only for scientists. VisiCalc allowed accountants to do in 5 seconds what used to take 2 weeks on paper. It was the first "killer app"—people bought Apple II computers just to run it.
* **Lotus 1-2-3 (1983):** Added charting and database capabilities, making the IBM PC a massive success in businesses.
* **Microsoft Excel (1985):** Re-imagined spreadsheets for graphical screens (with mice, menus, and visual fonts) and introduced advanced dependency calculations.
* **Google Sheets (2006):** Moved the calculation engine to the web, allowing multiple people to edit the same sheet at the same time.

---

## Visual Legend

Throughout this textbook, we will use standard symbols in our diagrams to represent different parts of our computer system:

```text
□ Object             (A complex box containing data)
○ Value              (A raw piece of data, like a number or text)
→ Reference          (A link pointing from one variable to another)
⇒ Dependency         (A logical connection: if A changes, B must change)
↓ Flow               (The direction information moves through time)
★ Changed Cell       (A cell that has just received new input)
⚠ Error              (Something went wrong during evaluation)
```

---

## Chapter 1 — What Is Python?

Before we can build a spreadsheet engine, we need to understand the tool we are using to build it. We are using **Python**, a programming language. 

But what is a program? 

### The Kitchen Analogy
Think of a computer program as a **recipe**, and the computer as a **robot chef**. 
* The **instructions** in the recipe tell the chef what to do (e.g., "chop", "mix", "bake").
* The **data** is the set of ingredients (e.g., "apples", "sugar", "flour").

To cook anything, the chef needs a place to put ingredients. In a computer, this kitchen counter is the computer's **memory**. When we want to store an ingredient, we put it in a box on the counter and write a label on the box. 

In programming, this labeled box is called a **variable**.

```text
Memory (The Kitchen Counter)
┌──────────────────────────────────────┐
│                                      │
│   ┌─────────────┐      ┌─────────┐   │
│   │  ○ "Alice"  │      │  ○ 20   │   │
│   └─────────────┘      └─────────┘   │
│      ▲                    ▲          │
│      │                    │          │
│    name                  age         │
│  (Variable)            (Variable)    │
│                                      │
└──────────────────────────────────────┘
```

In the diagram above, `name` is a label pointing to a box containing the text `"Alice"`. `age` is a label pointing to a box containing the number `20`.

### Python Data Types
Not all ingredients are the same. You cannot chop water, and you cannot boil flour the same way you boil pasta. Computers need to know what *type* of data they are holding. Here are the four basic types we will use:

#### 1. Integers (Whole Numbers)
These are plain numbers without decimals.
```python
age = 20
```
*   **Comparison:** A counting box containing physical tokens.

#### 2. Strings (Text)
Text in Python is called a **string** because it is a string of characters joined together. We put strings in quotation marks so the computer knows they are text, not instructions.
```python
name = "Alice"
```
*   **Comparison:** A banner with letters written on it.

#### 3. Lists (Ordered Shelves)
A list is a single variable that holds multiple items in a specific order.
```python
shopping_list = ["apples", "bananas", "milk"]
```
*   **Comparison:** A shelf with numbered slots starting at `0`.
```text
Slot 0        Slot 1        Slot 2
┌───────────┐ ┌───────────┐ ┌───────────┐
│"apples"   │ │"bananas"  │ │ "milk"    │
└───────────┘ └───────────┘ └───────────┘
```

#### 4. Dictionaries (Labeled Cubby Holes)
A dictionary stores pairs of information. Each item has a unique **Key** (the label) and a **Value** (what is inside).
```python
user = {
    "name": "Alice",
    "age": 20
}
```
*   **Comparison:** A wall of post-office boxes. You look at the label on the outside (the key) to find the contents inside (the value).

```text
Key (Label)      Value (Content)
┌─────────────┐  ┌─────────────┐
│   "name"    │ ─►│  ○ "Alice"  │
├─────────────┤  ├─────────────┤
│   "age"     │ ─►│    ○ 20     │
└─────────────┘  └─────────────┘
```

---

## Why Do Variables and Types Exist?

### What Problem Do They Solve?
If a computer didn't have variables, every piece of data would float around in memory without a name. It would be like a kitchen counter piled high with unlabeled ingredients, where you have to count "the third grain of rice from the left" to find what you need. Variables give data human-readable names.

### What Breaks Without Them?
Without variable names, you would have to write programs using physical memory numbers (like `0x7fff5fbff618`). If your program loaded on a different computer, those numbers would change, and your program would crash or execute garbage instructions.

### Why Not Use Something Simpler?
Instead of different data types (Integers, Strings, Dictionaries), we could store everything as raw text. But if we store everything as text, how does the computer add two numbers? To add `10` and `20`, it would first have to read the characters, parse them into digits, and perform math, which is incredibly slow. Types tell the computer *how* to process the data immediately.

---

## Mini Project: Setup and Variable Tracking

Let's write your first mini-program. Copy this Python code and run it in a terminal:

```python
# Create variables
item_name = "Lemonade"
price = 5
quantity = 3

# Compute total cost
total = price * quantity

# Print the results
print("Item:", item_name)
print("Total Cost:", total)
```

### Mental Model Tracking
When you run this code, your computer's memory updates line-by-line:

```text
Line 1: item_name ──► ○ "Lemonade"
Line 2: price ────────► ○ 5
Line 3: quantity ─────► ○ 3
Line 5: total ────────► price (5) * quantity (3) ──► ○ 15
```

---

## Pause and Think

What happens if we run `item_name + price`? 

If `item_name` is the string `"Lemonade"` and `price` is the number `5`, what does it mean to add text to a number? Try to run it. You will see a `TypeError`. Why does Python refuse to do this?

---

## Chapter 2 — Thinking Like A Computer

A computer is incredibly fast, but it is also incredibly simple-minded. It does not have human intuition. It cannot guess what you want. It only does exactly what you tell it to do.

Every action a computer performs follows a strict pipeline:

```text
┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│    INPUT     │ ───► │    STORE     │ ───► │   PROCESS    │ ───► │    OUTPUT    │
│              │      │  (Variables/ │      │ (Math/Logic) │      │ (Screen/     │
│ (Keyboard/   │      │   Memory)    │      │              │      │  Files)      │
│  Mouse)      │      │              │      │              │      │              │
└──────────────┘      └──────────────┘      └──────────────┘      └──────────────┘
```

Let's look at an example. If you write a program to calculate sales tax:
1. **Input:** You type the item price (`100`) and the tax rate (`0.05`).
2. **Store:** The computer puts `100` into a variable named `price` and `0.05` into a variable named `tax_rate`.
3. **Process:** The computer multiplies them: `price * tax_rate` to get `5.0`.
4. **Output:** The computer prints `5.0` on your screen.

If you don't store the inputs, you can't process them. If you don't process them, your output is empty. A spreadsheet does this exact pipeline millions of times per second.

---

## Chapter 3 — Thinking Like A Builder

When professional software engineers set out to write a program, they don't start typing code randomly. They start with a mental toolkit designed to manage complexity. A spreadsheet engine is a complex system, and the only way to build it without getting lost is to think like a builder.

Here is the general blueprint of how program builders work:

```text
  Big Problem (The Spreadsheet Engine)
       │
       ▼ (Decomposition)
  Smaller, Isolated Problems (Storage, Parsing, Recalculation, Rendering)
       │
       ▼ (Data Representation)
  Represent Data (Variables, Cells, Dictionaries, Graphs)
       │
       ▼ (Data Transformation)
  Transform Data (Text ──► Tokens ──► Tree ──► Values)
       │
       ▼
  Produce Output (Grid printout to screen)
```

To guide our journey, we must master four core builder principles:

### 1. Decomposition
Decomposition means breaking a massive, intimidating problem down into tiny, friendly components. Instead of trying to build "a spreadsheet" all at once, we solve these small problems independently:
*   How do we remember values? (We build a **Storage** module).
*   How do we read a formula? (We build a **Parsing** module).
*   How do we chain updates? (We build a **Graph** module).
*   How do we show the results? (We build a **UI** module).

### 2. State
State is the computer's memory of the world at any given millisecond. For our spreadsheet, the state is the collection of cells and their current formulas and values. When you edit a cell, you mutate (change) the state of the engine.

### 3. Responsibilities
Every piece of code should do one job and do it cleanly.
*   The **Tokenizer** splits text into chunks. It does *not* calculate math.
*   The **Evaluator** does math. It does *not* print characters to your screen.
*   The **UI Renderer** prints characters. It does *not* know how to solve formulas.
By isolating responsibilities, if we find a bug in math evaluation, we know exactly which folder and file to look in.

### 4. Invariants
An invariant is a truth or a rule that **must never be broken** while the program is running. For example:
*   *Invariant 1:* A cell's `value` must always equal the math result of its `raw_input` formula.
*   *Invariant 2:* The dependency graph must never contain circular paths (cycles).
As builders, we write check-valves (like cycle detection tests) to protect these invariants. If an action would break an invariant, our program rejects it immediately.

---

## Chapter 4 — Why Spreadsheets Exist

Imagine you run a lemonade stand. You want to track your costs, sales, and profits. 

Without a spreadsheet, you might write your calculations on a piece of paper:
```text
Sugar: $5
Lemons: $10
Cups: $3
Total Cost = Sugar + Lemons + Cups = $18
```
If the price of lemons rises to `$12`, you have to erase `$10`, write `$12`, cross out `$18`, recalculate the sum in your head, and write `$20`. This is tedious and prone to mistakes.

A spreadsheet is a visual tool that automates this recalculation. 

### What the User Sees
The user sees a grid of rows (labeled with numbers) and columns (labeled with letters). Each box is called a **cell**, and has a unique address like `A1`, `B1`, or `C1`.

```text
    A      B      C
┌──────┬──────┬──────┐
│  10  │  20  │  30  │  1
└──────┴──────┴──────┘
```

### What the Computer Sees
The computer does not see a physical grid. The computer sees a structured set of instructions and values. 

For the computer, cell `A1` is just a coordinate pointing to the number `10`, `B1` points to `20`, and `C1` points to `30`.

---

## Chapter 5 — The Great Illusion

Here is the secret: **A spreadsheet is not actually a table.**

When you look at a spreadsheet, it looks like a flat table of numbers:
```text
┌────┬────┬────┐
│10  │20  │30  │
└────┴────┴────┘
```

But underneath the hood, the spreadsheet is a **reactive network of calculations**. 

If cell `C1` contains the formula `=A1+B1`, cell `C1` is not independent. It is bound to `A1` and `B1`. If `A1` changes, a signal flows through the network to update `C1`.

```text
User's view (A flat table):
┌───────────┬───────────┬───────────┐
│    10     │    20     │  =A1+B1   │
└───────────┴───────────┴───────────┘

Computer's view (A network of flows):
  ┌──────┐
  │  A1  │ ───┐
  │ (10) │    │
  └──────┘    │      ┌──────┐
              ├────► │  C1  │ (Evaluates to 30)
  ┌──────┐    │      └──────┘
  │  B1  │ ───┘
  │ (20) │
  └──────┘
```

This is called a **reactive system**. The value of `C1` reactively adapts to changes in `A1` and `B1`. If we want to build a spreadsheet engine, we need to build this network.

---

## Chapter 6 — Designing From Nothing

How do we start building this in Python? What is the smallest, simplest spreadsheet engine we can make?

Let us start with a single, empty dictionary.

```python
cells = {}
```

```text
cells (Empty Storage)
┌──┐
└──┘
```

Why is a dictionary perfect here? 
Because cell addresses are unique names (strings) like `"A1"`, `"B12"`, or `"Z100"`. A dictionary allows us to map these names directly to their contents.

Let's add some data:
```python
cells["A1"] = 10
cells["B1"] = 20
```

Now our memory looks like this:
```text
cells
 │
 ├── "A1" ──► ○ 10
 └── "B1" ──► ○ 20
```

---

## Complexity Sidebar: Searching Lists vs Dictionaries

How long does it take a computer to find a cell? Let's compare storing our spreadsheet cells in a list vs. a dictionary.

### Storing in a List (Linear Search)
If we store cells as list items: `[("A1", 10), ("B1", 20), ..., ("Z99", 50)]`

To find cell `"Z99"`, the computer must look through the list item-by-item:
```text
"A1"? No. ──► "B1"? No. ──► ... ──► "Z99"? Yes!
```
If our spreadsheet has $n$ cells, this search takes $n$ steps in the worst case. We call this linear time complexity, written as **O(n)**.

### Storing in a Dictionary (Hash Map)
A dictionary uses a mathematical shortcut. It runs the key `"Z99"` through a formula to get the exact location in memory:
```text
"Z99" ──► Hash Function ──► Memory slot 853 ──► Jump instantly!
```
This lookup takes exactly 1 step, whether we have 5 cells or 5,000,000 cells. We call this constant time complexity, written as **O(1)**.

---

## Mini Project: Build an Instant Address Book

Let's test this lookup speed. Run this Python code:

```python
# Create a tiny spreadsheet dictionary
cells = {
    "A1": 100,
    "A2": 200,
    "B1": 500
}

# Fetch cell data
search_address = "A2"
value = cells[search_address]

print("Cell", search_address, "holds:", value)
```

---

## Chapter 7 — Following Execution Through Time

To understand how a computer runs code, we must train ourselves to follow its execution line-by-line, watching how the variables and references inside memory change frame-by-frame.

Let's trace this simple script:
```python
cells = {}
cells["A1"] = 10
cells["B1"] = 20
cells["C1"] = cells["A1"] + cells["B1"]
```

Here is exactly what happens in memory at each step:

### Frame 1: Creation
```python
cells = {}
```
The computer allocates space on the kitchen counter for a new, empty dictionary labeled `cells`.
```text
Variables Frame:
┌───────────┐
│ cells ────┼──► ○ {} (Empty Dictionary)
└───────────┘
```

### Frame 2: First Entry
```python
cells["A1"] = 10
```
The computer hashes the key `"A1"`, creates a slot inside our dictionary, and binds it to the numeric value `10`.
```text
Variables Frame:
┌───────────┐
│ cells ────┼──► ┌────────────────┐
└───────────┘    │ "A1" ──► ○ 10  │
                 └────────────────┘
```

### Frame 3: Second Entry
```python
cells["B1"] = 20
```
The computer hashes the key `"B1"`, creates another slot inside the dictionary, and binds it to the value `20`.
```text
Variables Frame:
┌───────────┐
│ cells ────┼──► ┌────────────────┐
└───────────┘    │ "A1" ──► ○ 10  │
                 │ "B1" ──► ○ 20  │
                 └────────────────┘
```

### Frame 4: Calculation and Write
```python
cells["C1"] = cells["A1"] + cells["B1"]
```
This line is processed in four tiny sub-steps:
1.  **Lookup `"A1"`:** The computer checks the `cells` dictionary for key `"A1"`, following the reference to retrieve value `10`.
2.  **Lookup `"B1"`:** The computer checks the dictionary for `"B1"`, retrieving value `20`.
3.  **Process Addition:** The computer runs the CPU adder: `10 + 20 = 30`.
4.  **Write `"C1"`:** The computer creates a new slot in the dictionary for `"C1"` and stores the calculated value `30`.

```text
Variables Frame:
┌───────────┐
│ cells ────┼──► ┌────────────────┐
└───────────┘    │ "A1" ──► ○ 10  │
                 │ "B1" ──► ○ 20  │
                 │ "C1" ──► ○ 30  │
                 └────────────────┘
```

By tracing memory changes line-by-line, we demystify the machine. Programming is simply controlling how these memory states change over time.

---

## Chapter 8 — Reading Cells

Now that we can write cells into our dictionary, we need to read them. 

Imagine we want to build a simple grid layout:
```text
┌──────┬──────┐
│  A1  │  B1  │
├──────┼──────┤
│  10  │  20  │
└──────┴──────┘
```

To render this, we ask our dictionary for the values:
```python
val_a = cells["A1"] # Returns 10
val_b = cells["B1"] # Returns 20
```

### The Missing Cell Problem
What happens if the user asks for cell `"C1"`, which we haven't created yet?
If we run `cells["C1"]`, Python will panic and crash with a `KeyError` because `"C1"` does not exist in our dictionary.

To prevent crashes, we use a safer lookup method. If a cell does not exist, we will pretend it is empty and return `0`:

```python
# Look up "C1". If not found, return 0.
val_c = cells.get("C1", 0) 
```

```text
                      Lookup "C1"
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
      Exists in cells?          Does NOT exist in cells?
             │                           │
             ▼                           ▼
      Return stored value          Return default (0)
```

---

## Why Does This Exist? (The Default Value Pattern)

### What Problem Does It Solve?
In spreadsheets, users constantly reference empty cells. For example, you might write `=A1+B1` before you have entered any data in `B1`. If the engine crashed every time it encountered an uninitialized cell, users would be blocked from building templates.

### What Breaks Without It?
Without a default fallback, every cell lookup would need to be wrapped in an `if` check. The code would quickly become a messy forest of checks:
```python
if "A1" in cells:
    val_a = cells["A1"]
else:
    val_a = 0
```
This bloats code size and increases bugs.

---

## Chapter 9 — Why Values Alone Stop Working

So far, our spreadsheet only stores plain numbers:
```python
cells["A1"] = 10
cells["B1"] = 20
```

But what if the user wants to enter a formula like `=A1+B1`?
If we write:
```python
cells["C1"] = "=A1+B1"
```
We have a major problem.
* If we print `cells["C1"]`, the screen shows the string `"=A1+B1"`, not the number `30`.
* If we evaluate the formula and store `30` in `cells["C1"]`, we lose the formula! If `A1` later changes to `100`, we won't know how to recalculate `C1` because we erased `=A1+B1`.

### The System Evolution Roadmap

Let's look at how our spreadsheet engine design evolves as we require more features:

```text
Naive
┌──────────────────────────────────────┐
│  cells["A1"] = 10                    │ (Simple values in a dictionary)
└──────────────────┬───────────────────┘
                   │ User wants formulas!
                   ▼
Better
┌──────────────────────────────────────┐
│  cells["C1"] = "=A1+B1"              │ (Values alone fail; we lose the formula)
└──────────────────┬───────────────────┘
                   │ We need to keep both raw input and computed value!
                   ▼
Professional
┌──────────────────────────────────────┐
│  Cell Objects:                       │
│  - raw_input = "=A1+B1"              │ (We store state in structured objects)
│  - value = 30                        │
└──────────────────────────────────────┘
```

### The Cell Object in Memory

To solve this, we cannot just store raw numbers or strings. We need to store structured objects that contain *both* the raw input from the user and the calculated value.

We will create a **Cell** class. A class is a blueprint that tells the computer how to construct custom containers.

```python
class Cell:
    def __init__(self, address, raw_input):
        self.address = address      # e.g., "C1"
        self.raw_input = raw_input  # e.g., "=A1+B1"
        self.value = None           # Will hold the evaluated result (e.g., 30)
        self.cell_type = None       # e.g., "NUMBER" or "FORMULA"
        self.error = None           # Will hold error messages if something breaks
```

Let's visualize how the dictionary and objects are arranged in memory:

```text
cells (Dictionary)
 │
 ├── "A1" ──►  □ Cell Object
 │                ├── address = "A1"
 │                ├── raw_input = "10"
 │                ├── value = 10.0
 │                └── cell_type = "NUMBER"
 │
 └── "C1" ──►  □ Cell Object
                  ├── address = "C1"
                  ├── raw_input = "=A1+B1"
                  ├── value = 30.0
                  └── cell_type = "FORMULA"
```

---

## Why Do Cell Objects Exist?

### What Problem Do They Solve?
They group multiple related details about a single cell into one bundle. If we didn't use objects, we would have to maintain separate dictionaries for raw inputs, calculated values, type labels, and errors:
```python
raw_inputs = {"A1": "10", "C1": "=A1+B1"}
values = {"A1": 10, "C1": 30}
errors = {"C1": None}
```
Keeping these multiple dictionaries in sync when cells are deleted or renamed is an administrative nightmare.

### What Breaks Without Them?
If we tried to use simple tuples instead of objects:
`cells["C1"] = ("=A1+B1", 30, "FORMULA", None)`
We wouldn't be able to edit specific fields easily. In Python, tuples are immutable (unchangeable). To change just the value field, we would have to reconstruct the entire tuple, which is slow and makes code hard to read.

---

## Chapter 10 — User Input

When a user types something into a cell, we need to inspect it and determine what it is. This is called input classification.

There are three basic things a user can type:
1. **A Formula:** Always starts with an equals sign `=`. (e.g., `=A1+B1`)
2. **A Number:** Can be parsed into a decimal number. (e.g., `12.5`, `-3`)
3. **A Label (Text):** Anything else. (e.g., `"Total Sales"`)

Let's look at how we classify this in code:

```python
def classify_input(raw_input):
    cleaned = raw_input.strip()
    
    if cleaned.startswith("="):
        return "FORMULA"
        
    try:
        float(cleaned)
        return "NUMBER"
    except ValueError:
        return "TEXT"
```

Let's trace what happens when we process three different user inputs:

```text
Input: "10"         ──► strip() ──► Check "=" (No) ──► try float() (Yes) ──► Type: NUMBER
Input: " =A1+B1 "   ──► strip() ──► Check "=" (Yes) ─────────────────────► Type: FORMULA
Input: "Apples"     ──► strip() ──► Check "=" (No) ──► try float() (Fail) ──► Type: TEXT
```

---

## Mini Project: Build an Input Classifier

Let's write a simple script that processes inputs like a real cell generator. Run this code:

```python
class Cell:
    def __init__(self, address, raw_input):
        self.address = address
        self.raw_input = raw_input
        self.value = None
        self.cell_type = self.classify(raw_input)
        
    def classify(self, raw_input):
        cleaned = raw_input.strip()
        if cleaned.startswith("="):
            return "FORMULA"
        try:
            float(cleaned)
            return "NUMBER"
        except ValueError:
            return "TEXT"

# Create two test cells
cell_a = Cell("A1", " 45.2 ")
cell_b = Cell("B1", "=A1+100")

print(f"Cell {cell_a.address} is a {cell_a.cell_type}")
print(f"Cell {cell_b.address} is a {cell_b.cell_type}")
```

---

## Chapter 11 — How Humans Think vs How Computers Think

When a human looks at a formula like `=A1+B1`, they immediately understand its meaning. A computer, however, sees nothing but a raw list of text characters. It has no concept of "math" or "references" until we build structures to represent them.

Let's look at the difference in how we process this:

### The Human Process
1. Look at `=A1+B1`.
2. Instantly realize it means "add the value of B1 to the value of A1".
3. Look up B1 and A1 in the grid, calculate, and write down the result.

### The Computer Process
The computer has to move through multiple stages of representation, progressively translating text characters into meaning:

```text
  ○ "=A1+B1"               (Raw characters)
      │
      ▼
  [ "A1", "+", "B1" ]      (Tokens: grouped symbols)
      │
      ▼
      +
    /   \                  (Abstract Syntax Tree: structural relation)
  A1     B1
      │
      ▼
   ○ 30.0                  (Evaluated numerical value)
```

By breaking the problem down into these distinct stages, we can write small, focused modules that each do one simple job.

---

## Chapter 12 — Spreadsheets Are Tiny Programming Languages

You might think of a spreadsheet as a calculator, but it is actually a fully-fledged **programming language environment**. Every cell with a formula is a tiny script!

Let's compare the parts of a spreadsheet engine to the parts of a professional language interpreter (like Python itself):

| Spreadsheet Concept | Programming Language Equivalent | Role |
| :--- | :--- | :--- |
| **Formula Text** (e.g., `=A1+5`) | **Source Code File** (e.g., `main.py`) | The raw instructions typed by the human. |
| **Formula Parser** | **Compiler Frontend** | Translates raw text into structural trees. |
| **Syntax Tree (AST)** | **Abstract Syntax Tree** | Represents the order of operations. |
| **Cell Reference** (`A1`) | **Variable Lookup** (`x`) | Resolves a name to its current value. |
| **Dependency Recalculator** | **Execution Engine / Runtime** | Ensures lines run in the correct order. |

When you build a spreadsheet engine, you are building a custom language interpreter!

---

## Chapter 13 — Formulas

How does a computer understand math?

If we write `=A1+B1*2`, a human reading this knows that multiplication happens before addition. We calculate `B1 * 2` first, then add `A1`. This order of operations is called **precedence**.

But a computer reads text character-by-character, from left to right:
1. `A1`
2. `+`
3. `B1`
4. `*`
5. `2`

If the computer blindly calculated left-to-right, it would do `A1 + B1` first, then multiply the result by `2`. This would give the wrong answer!

```text
Expression: 10 + 20 * 2

Incorrect (Left to Right):
  10 + 20 = 30
  30 * 2  = 60 ❌

Correct (Precedence):
  20 * 2  = 40
  10 + 40 = 50  
```

To solve this, we must teach the computer how to break down the formula into pieces and organize those pieces into a structure that respects mathematical rules.

---

## Chapter 14 — Parsing

Parsing is the process of translating raw text into a structural model the computer can run. It happens in two phases: **Tokenization** and **Syntax Tree Construction**.

### Phase 1: Tokenization (Lexing)
Tokenization takes a continuous string of text and chops it up into individual meaningful chunks, called **tokens**.

Imagine the formula text: `"=A1+B1*2"`

We discard the leading `=` because it just tells us this is a formula. We are left with `"A1+B1*2"`. 

Our tokenizer reads this string character-by-character and groups them:
* Letters followed by numbers become **Cell Addresses** (`A1`, `B1`).
* Symbols like `+`, `-`, `*`, `/` become **Operators**.
* Continuous digits become **Numbers** (`2`).

```text
Raw String:   "A1 + B1 * 2"
                 │
                 ▼ (Tokenization)
Tokens:      [ Address("A1"), Operator("+"), Address("B1"), Operator("*"), Number("2") ]
```

### Phase 2: Building the Abstract Syntax Tree (AST)
Once we have the list of tokens, we build an **Abstract Syntax Tree (AST)**. 

*   **Comparison:** AST is like sentence grammar. You break a sentence into a subject, verb, and object to understand who did what.

A tree is a structure where parent boxes point to child boxes. In math expressions, the operator (like `+` or `*`) is the parent, and the numbers or cell addresses are the children.

```text
             +   (Parent Node)
           /   \
(Child)  A1     *  (Parent Node)
               / \
     (Child) B1   2 (Child)
```

Look at how the hierarchy of the tree naturally represents math precedence:
* The multiplication `*` is placed deeper in the tree than the addition `+`.
* Because it is deeper, it must be solved *before* we can solve the addition.

### Evaluating the AST
To get the final value of the formula, we perform a process called **tree evaluation** (specifically, post-order traversal):
1. Go to the root operator: `+`.
2. To solve `+`, we first need the value of the left child (`A1`) and the right child (`*`).
3. We resolve `A1` ──► returns `10`.
4. We look at the right child: `*`.
5. To solve `*`, we need its children: `B1` and `2`.
6. We resolve `B1` ──► returns `20`.
7. We resolve the number `2` ──► returns `2`.
8. Now we perform the multiplication: `20 * 2 = 40`.
9. The `*` node returns `40` to its parent `+`.
10. Finally, we perform the addition: `10 + 40 = 50`.

```text
   Step 1: Unsolved Tree           Step 2: Resolve Leaves         Step 3: Solve Multiply
            [+]                             [+]                            [+]
          /     \                         /     \                        /     \
       [A1]     [*]                     10      [*]                    10       40
               /   \                           /   \
            [B1]   [2]                       20     2

                                   Step 4: Solve Addition
                                           [50]
```

---

## Why Do Syntax Trees Exist?

### What Problem Do They Solve?
They eliminate ambiguity. If you write `10 + 20 * 2`, there is only one mathematical way to compute it. The tree visually codes this order of calculations so the evaluator doesn't have to guess or check precedence rules while doing math.

### What Breaks Without Them?
Without an AST, you would have to write complex, recursive text search functions to evaluate expressions directly. If you wanted to support parentheses like `(10 + 20) * 2`, your evaluator code would become incredibly nested and slow.

### Why Not Use Something Simpler?
Instead of a tree, you could write a regex-based parser. But regex cannot handle nested parentheses of arbitrary depth (like `(((10 + 20) * 2) - 5)`). It is mathematically impossible for regular expressions to parse nested structures. Trees solve this cleanly.

---

## Chapter 15 — Dependencies

What happens when cell values change?

If cell `C1` contains the formula `=A1+B1`, we say **C1 depends on A1 and B1**.
* If `A1` changes, `C1` must recalculate.
* If `B1` changes, `C1` must recalculate.

In computer science, we model these relationships using a **Graph**. A graph consists of **Nodes** (objects, which are cells) connected by **Edges** (arrows showing directions of influence).

*   **Comparison:** Think of a graph like a road map. Nodes are towns, and edges are one-way highways connecting them.

```text
Directed Dependency Graph
  ┌──────┐
  │  A1  │ ───┐
  └──────┘    │ (Edge)
              ▼
           ┌──────┐
           │  C1  │
           └──────┘
              ▲
  ┌──────┐    │ (Edge)
  │  B1  │ ───┘
  └──────┘
```

### Adjacency Lists
To store this graph in our computer, we use an **Adjacency List**. This is a dictionary where the key is a cell address, and the value is a list of all cells that depend on it.

If `C1 = A1 + B1` and `D1 = C1 * 2`, our graph looks like this:

```text
  ┌──────┐      ┌──────┐      ┌──────┐
  │  A1  │ ───► │  C1  │ ───► │  D1  │
  └──────┘      └──────┘      └──────┘
                   ▲
  ┌──────┐         │
  │  B1  │ ────────┘
  └──────┘
```

The Adjacency List representing this structure:
```python
dependents = {
    "A1": ["C1"],
    "B1": ["C1"],
    "C1": ["D1"],
    "D1": []
}
```

Whenever a cell changes, we inspect this dictionary to find out who needs to be updated next.

---

## Why Do Dependency Graphs Exist?

### What Problem Do They Solve?
They tell the engine exactly which cells are impacted when a value changes. If `A1` updates, we don't want to recompute every single cell in our 10,000-cell spreadsheet. We only want to recompute cells that read `A1`.

### What Breaks Without Them?
Without a dependency graph, when any cell changes, your only option is to recalculate the entire spreadsheet from scratch. In a massive spreadsheet, this causes a painful, noticeable freeze after every keypress.

---

## Chapter 16 — Recalculation

Let's say the user updates cell `A1` from `10` to `100`. 
How do we update the rest of the spreadsheet?

### The Naive Approach
A naive way would be to look at the list of dependents for `A1`, update them, and repeat.
If `A1` changes, we update `C1`. Since `C1` changed, we update `D1`.

But what if we have a network like this?
```text
      ┌──────┐
   ┌─►│  B1  │──┐
   │  └──────┘  │
┌──┴───┐     ┌──▼───┐
│  A1  │     │  C1  │
└──┬───┘     ▲──┬───┘
   │  ┌──────┐  │
   └─►│  D1  │──┘
      └──────┘
```
Here, `C1 = B1 + D1`. Both `B1` and `D1` depend on `A1`.
If `A1` changes:
1. We update `B1`.
2. Since `B1` changed, we update `C1` (using the old, outdated value of `D1`).
3. Then we update `D1`.
4. Since `D1` changed, we update `C1` *again*.

We calculated `C1` twice! In a large sheet with thousands of cells, this double-calculation creates massive slowdowns.

*   **Comparison:** Dependency propagation is like a chain of dominos. Tipping the first domino (`A1`) triggers a cascade that flows through all downstream cells in sequence.

### Topological Sorting
To recalculate efficiently, we must sort the cells in a specific order so that **no cell is evaluated until all of its dependencies are ready**. This order is called a **Topological Sort**.

*   **Comparison:** Topological sorting is like getting dressed in the morning. You must put on your socks before your shoes, and your shirt before your coat. You cannot do it in any other order without breaking things!

For our graph:
```text
A1 ──► B1 ──┐
  └──► D1 ──┴──► C1
```
A valid topological sort is: `[A1, B1, D1, C1]`.
If we calculate in this order, we update `A1` first, then `B1`, then `D1`, and finally `C1`. `C1` is calculated exactly once, using the correct, fresh values of both `B1` and `D1`.

#### Kahn's Algorithm
To find this topological order, we use Kahn's Algorithm. It works by looking at the **in-degree** of each node. The in-degree is the number of arrows pointing *into* a cell.

```text
Node In-Degrees:
  A1: 0 (No incoming arrows)
  B1: 1 (A1 points to B1)
  D1: 1 (A1 points to D1)
  C1: 2 (B1 and D1 point to C1)
```

```text
Kahn's Algorithm Execution Flow:

Step 1: Find all nodes with In-Degree = 0.
        ──► Queue: [A1]

Step 2: Pull A1 from queue. Add to sorted list.
        ──► Sorted List: [A1]
        Remove A1's outgoing edges. Update in-degrees:
        B1 in-degree: 1 ──► 0
        D1 in-degree: 1 ──► 0
        Queue now gets nodes that hit 0 in-degree: [B1, D1]

Step 3: Pull B1 from queue. Add to sorted list.
        ──► Sorted List: [A1, B1]
        Remove B1's outgoing edges. Update in-degrees:
        C1 in-degree: 2 ──► 1
        Queue: [D1]

Step 4: Pull D1 from queue. Add to sorted list.
        ──► Sorted List: [A1, B1, D1]
        Remove D1's outgoing edges. Update in-degrees:
        C1 in-degree: 1 ──► 0
        Queue gets C1: [C1]

Step 5: Pull C1 from queue. Add to sorted list.
        ──► Sorted List: [A1, B1, D1, C1]
        Queue is empty. Done!
```

---

## Why Does Topological Recalculation Exist?

### What Problem Does It Solve?
It ensures we calculate every cell exactly once, and always in the correct order. No cell gets computed with outdated data.

### What Breaks Without It?
Without topological sorting, your spreadsheet will suffer from **calculation glitches**. If a formula reads two cells that both depend on a common third cell, the formula will evaluate with one updated value and one old value, producing a temporarily incorrect number on the screen.

---

## Pause and Think

What happens if you have a chain of cells: `A1 -> B1 -> C1 -> D1 -> E1`? 

If `A1` is updated, what is the only valid topological order to recalculate this chain? What happens to the in-degree of `E1` at each step of Kahn's Algorithm?

---

## Chapter 17 — Circular Dependencies

What happens if you type:
* In cell `A1`: `=B1 + 1`
* In cell `B1`: `=A1 + 1`

```text
┌──────┐      ┌──────┐
│  A1  │ ───► │  B1  │
└──────┘      └──────┘
   ▲             │
   └─────────────┘
```

This is a **Circular Dependency** (a cycle). 
* To calculate `A1`, we need `B1`.
* To calculate `B1`, we need `A1`.

If we run Kahn's algorithm or try to recalculate, the computer will get stuck in an infinite loop, constantly updating `A1` and `B1` forever, eventually crashing the program.

*   **Comparison:** A circular reference is like putting two mirrors face-to-face. They create an infinite tunnel of reflections with no end.

### Cycle Detection (DFS Coloring)
To protect our engine, we must check for cycles *before* we evaluate any formulas. We do this using a Depth-First Search (DFS) with a **three-color marking system**.

We mark each cell node with one of three colors:
1. **White (Unvisited):** We haven't looked at this cell yet.
2. **Gray (Visiting):** We are currently exploring this cell's dependency path. If we hit a cell that is already Gray, we have found a cycle!
3. **Black (Visited):** We have fully explored this cell and all its dependencies, and confirmed there are no cycles.

Let's trace a cycle detection:

```text
Path: A1 ──► B1 ──► C1 ──► A1

Step 1: Start at A1. Mark A1 as GRAY.
        Visit A1's dependency: B1.

Step 2: Mark B1 as GRAY.
        Visit B1's dependency: C1.

Step 3: Mark C1 as GRAY.
        Visit C1's dependency: A1.

Step 4: A1 is already GRAY! 
        🚨 CYCLE DETECTED! 🚨
```

---

## Why Does Cycle Detection Exist?

### What Problem Does It Solve?
It acts as a shield for the computer's memory. It catches user mistakes (like circular logic) and stops them before they run, preventing the application from freezing or crashing due to an Out of Memory error.

### What Breaks Without It?
Without cycle detection, the moment a user accidentally types a formula pointing back to itself (even indirectly, like `A1 -> B1 -> C1 -> A1`), the engine will start recalculating in an infinite loop. The CPU will hit 100% usage, the UI will freeze, and the user will lose all unsaved work.

---

## Chapter 18 — Failure Chapters: What Can Go Wrong?

In a real spreadsheet, things go wrong all the time. Users write bad formulas, delete cells that others need, or type text where numbers belong. Our engine must handle these failures gracefully instead of crashing.

Let's look at the five main errors our engine must catch:

### 1. Missing Cells (`#REF!`)
Occurs when cell `C1` contains `=A1+B1` but `B1` has been deleted or does not exist.
```text
C1 (Formula) ──► Looks up B1 ──► Does not exist! ──► ⚠ "#REF! (Missing Cell)"
```

### 2. Division By Zero (`#DIV/0!`)
Occurs when you try to divide a number by zero.
```text
Formula: "=A1/0" ──► Math evaluation ──► ⚠ "#DIV/0! (Division by Zero)"
```

### 3. Circular References (`#REF!`)
Occurs when a formula loops back into itself (as we saw in Chapter 15).
```text
A1 ──► B1 ──► A1 ──► Cycle detected! ──► ⚠ "#REF! (Circular Reference)"
```

### 4. Invalid Functions (`#NAME?`)
Occurs when you type a function that does not exist in our registry.
```text
Formula: "=HELLO(A1)" ──► Function Registry lookup ──► ⚠ "#NAME? (Unknown Function)"
```

### 5. Type Mismatch (`#VALUE!`)
Occurs when you try to perform math on incompatible data types (like adding text to a number).
```text
Formula: "=A1 + 5" (A1 holds "apple") ──► Math evaluation ──► ⚠ "#VALUE! (Invalid Type)"
```

### Error Propagation
Errors are contagious! If `A1` has an error, any cell that reads `A1` also gets marked with the error:

```text
  ┌──────────────┐
  │      A1      │ ──► ⚠ "#DIV/0!" (Trigger Cell)
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │      B1      │ ──► ⚠ "#DIV/0!" (Propagated: reads A1)
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐
  │      C1      │ ──► ⚠ "#DIV/0!" (Propagated: reads B1)
  └──────────────┘
```

---

## Chapter 19 — Functions

Real spreadsheets don't just add and multiply. They use functions like `SUM()`, `AVG()`, `MIN()`, and `MAX()`.

To support this, our parsing engine needs a **Function Registry**. This is a dictionary that maps text names of functions to the Python code that executes them.

*   **Comparison:** The function registry is like a phone book. When the parser encounters the name `"SUM"`, it looks it up in the registry to find the number (the Python code) to call.

```text
Formula Input: "=SUM(A1:A3)"
                  │
                  ▼ (Registry lookup)
              "SUM" ──► Python sum() function
```

### The Registry Pattern
```python
FUNCTION_REGISTRY = {
    "SUM": lambda args: sum(args),
    "AVG": lambda args: sum(args) / len(args) if args else 0,
    "MIN": lambda args: min(args) if args else 0,
    "MAX": lambda args: max(args) if args else 0,
    "COUNT": lambda args: len(args)
}
```

### Handling Ranges (`A1:B3`)
When a user writes `SUM(A1:B2)`, they are referencing a range of cells. The parser must expand this range into a list of individual cell addresses:

```text
Range "A1:B2" ──► Expanded ──► [ "A1", "A2", "B1", "B2" ]
```

We do this by converting the column letters to numbers (A=1, B=2), looping through the rows and columns, and converting them back into cell addresses. The values of these cells are then fed into the registry function.

---

## Why Does the Function Registry Exist?

### What Problem Does It Solve?
It makes our spreadsheet engine extensible. If we want to add a new function (like `MEDIAN` or `CONCAT`), we don't have to change the parser or AST tree evaluator. We just write one line of code and register it in our registry dictionary.

---

## Chapter 20 — Saving Work

A spreadsheet engine is useless if you lose all your work when you close the program. We need a way to save our cells to the computer's hard drive and load them back later.

This process is called **Serialization** (writing to disk) and **Deserialization** (reading from disk).

There are three common formats we can choose:

| Format | Pros | Cons |
| :--- | :--- | :--- |
| **JSON** | Human-readable, structures are preserved, built-in library support. | Files can get large. |
| **CSV** | Universally supported by Excel/Google Sheets. | **Loses formulas.** Only saves the final values as text. |
| **Binary** | Extremely small file size, fast load times. | Not human-readable, hard to debug. |

For our engine, **JSON** is the ideal choice because it allows us to store the `raw_input` of each cell, which preserves our formulas.

```text
Save Flow:
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ Cell Objects │ ───► │ JSON String  │ ───► │  File on     │
│  (In Memory) │      │              │      │  Disk (.json)│
└──────────────┘      └──────────────┘      └──────────────┘

Load Flow:
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   File on    │ ───► │ JSON String  │ ───► │ Cell Objects │
│ Disk (.json) │      │              │      │  (In Memory) │
└──────────────┘      └──────────────┘      └──────────────┘
```

---

## Chapter 21 — Terminal Interface

So far, we have built the brain of the spreadsheet. Now we need to build the body—the interface that lets the user interact with the grid.

### Separation of Concerns
In software design, we separate the code into two independent layers:
1. **The Engine (Model):** Handles math, trees, and dependencies. It doesn't know what a screen is.
2. **The User Interface (View/Controller):** Handles keypresses, cursor position, and printing the grid.

```text
┌──────────────────────────────────────────────┐
│                  Terminal UI                 │
└──────────────────────┬───────────────────────┘
                       │ User actions / Input
                       ▼
┌──────────────────────────────────────────────┐
│              Spreadsheet Engine              │
└──────────────────────┬───────────────────────┘
                       │ Formulas/Dependencies
                       ▼
┌──────────────────────────────────────────────┐
│                    Parser                    │
└──────────────────────────────────────────────┘
```

### Viewports and Scrolling
In a terminal, we only have limited space (e.g., 80 characters wide by 24 rows tall). The entire spreadsheet might have columns A to Z and rows 1 to 100. 

To solve this, we define a **Viewport**. The viewport is a window showing a small slice of the spreadsheet. If the user moves the cursor past the edge of the screen, we shift the viewport window.

*   **Comparison:** Viewport is like a camera window sliding across a map. The map is huge, but you only see whatever is inside the lens.

```text
Real Spreadsheet Grid (A1 to Z100)
┌─────────────────────────────────────────┐
│ A1   B1   C1   D1   E1   F1 ...    Z1   │
│ A2   B2   C2   D2   E2   F2 ...    Z2   │
│ A3 ┌─────────────────┐ E3   F3 ...    Z3   │
│ A4 │  Viewport Area  │ E4   F4 ...    Z4   │
│ A5 │  (Visible rows) │ E5   F5 ...    Z5   │
│ A6 └─────────────────┘ E6   F6 ...    Z6   │
│ ...                                     │
│ A100                               Z100 │
└─────────────────────────────────────────┘
```

---

## Chapter 22 — Architecture: How Information Flows

Let's trace the complete lifecycle of a cell edit. When a user presses a key on their keyboard to change cell `B1` to `=A1+5`, how does that action propagate through all the layers and eventually update their monitor screen?

```text
[Keyboard Keypress: User types "=A1+5" in B1 and presses Enter]
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. Terminal UI Layer                                        │
│    - Captures input string.                                 │
│    - Calls `engine.set_cell_input("B1", "=A1+5")`.          │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Spreadsheet Engine Layer                                 │
│    - Classifies the raw input as "FORMULA".                 │
│    - Instantiates/updates the Cell Object for "B1".         │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Parser & Tokenizer Layer                                 │
│    - Lexer cuts "=A1+5" into tokens: [A1, +, 5].            │
│    - Parser creates AST Node: AddNode(CellNode("A1"), Num(5)).│
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Dependency Graph Layer                                   │
│    - Inspects AST and finds dependency link: A1 -> B1.      │
│    - Adds B1 to A1's list of dependents in the graph.       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Recalculation Layer (Topological Sort)                   │
│    - Runs Kahn's algorithm starting at trigger cell: B1.    │
│    - Resolves B1's AST: loads A1's value (e.g. 10) + 5 = 15.│
│    - Saves value 15.0 to cell B1's object.                  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Renderer (UI View)                                       │
│    - Formats the value 15.0 for display.                    │
│    - Redraws the characters on the terminal grid.           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
    [Monitor Screen: Box B1 displays "15.0" to the user]
```

---

## Chapter 23 — Folder Structure

When building a spreadsheet engine project, we don't want to cram everything into one giant, messy file. We want to organize our code into modular modules, each responsible for one specific task. 

Here is a professional project structure:

```text
spreadsheet_engine/
│
├── cell.py            # Contains the Cell class (address, raw_input, value)
├── spreadsheet.py     # Main coordinator (stores cell dict, handles updates)
├── parser.py          # Translates formula text into syntax trees
├── tokenizer.py       # Splits formula strings into mathematical tokens
├── ast_nodes.py       # Defines tree node classes (AddNode, CellRefNode, etc.)
├── evaluator.py       # Traverses AST and runs math calculations
├── graph.py           # Dependency graph structures and Topological Sort
├── functions.py       # The function registry (SUM, AVG, MIN, etc.)
├── errors.py          # Error handling codes and propagation logic
├── persistence.py     # Handles CSV and JSON saving/loading
│
├── ui/
│    ├── terminal.py   # Handles cursor keys and terminal grid layout
│    └── renderer.py   # Renders characters to the console viewport
│
└── main.py            # Entry point of the application
```

---

## Chapter 24 — Code Reading: Tracing an Evaluation

Let's read a snippet of core engine code. Your job as a programmer is to read this line-by-line and track what objects are updated.

```python
def update_cell_value(cells, address):
    # 1. Fetch cell object from database
    cell = cells[address]
    
    # 2. If it's a number or text, value is just the raw input
    if cell.cell_type != "FORMULA":
        cell.value = float(cell.raw_input) if cell.cell_type == "NUMBER" else cell.raw_input
        return
    
    # 3. Parse formula to get AST tree
    ast_tree = parse_formula(cell.raw_input)
    
    # 4. Evaluate the AST using cells database to look up variables
    try:
        cell.value = evaluate(ast_tree, cells)
        cell.error = None
    except ZeroDivisionError:
        cell.value = None
        cell.error = "#DIV/0!"
```

### Let's Trace line-by-line:
Imagine cell `B1` has `raw_input = "=10 / 0"`.
*   **Line 3:** Fetch cell object. `cell` points to the object in memory holding our formula.
*   **Line 6:** `cell.cell_type` is `"FORMULA"`, so we skip this block.
*   **Line 11:** Parse formula. `ast_tree` becomes a tree structure representing division: `DivNode(10, 0)`.
*   **Line 15:** Evaluate tree. During evaluation of `DivNode`, Python triggers a division by zero error.
*   **Line 17:** The `ZeroDivisionError` block catches the panic!
*   **Line 18:** `cell.value` is set to `None`.
*   **Line 19:** `cell.error` is set to the warning flag `"⚠ #DIV/0!"`. The program survives and keeps running!

---

## Chapter 25 — How Excel Probably Thinks

Microsoft Excel and Google Sheets are massive, highly optimized engines. While the core principles we discussed are the same, they use several advanced techniques to speed up calculations:

### 1. Lazy Evaluation
Our simple engine recalculates formulas immediately when inputs change. Excel sometimes uses **Lazy Evaluation**: it marks cells as dirty but does not recalculate them until you actually scroll to them, or until a script asks for their value.

### 2. Dependency Graph Caching
Excel does not reconstruct the dependency graph from scratch every time you edit a cell. It caches the paths and updates only the small sub-branches of the graph that were affected.

### 3. Parallel Recalculation
Modern computers have multiple processor cores. Excel distributes independent branches of the topological sort to different cores to calculate them simultaneously.

---

## Chapter 26 — Building a Real Spreadsheet Engine: Roadmap

If you want to turn this guide into a real portfolio project, here is the order you should implement features. Avoid starting with nice-to-have features first; get the core calculations running first!

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        Feature Implementation Map                      │
├───────────────────┬───────────────────┬────────────────────────────────┤
│ Phase 1: MUST     │ Phase 2: SHOULD   │ Phase 3: NICE TO HAVE          │
├───────────────────┼───────────────────┼────────────────────────────────┤
│ - Cell Dictionary │ - Kahn's sorting  │ - Viewport scrolling           │
│ - Number/Formula  │ - Three-color DFS │ - Function range expanders     │
│   classification  │   cycle detection │   (A1:B10)                     │
│ - Basic +, -, *   │ - Div/0 error     │ - JSON Save / Load             │
│   AST parsing     │   handling        │ - Grid interface               │
└───────────────────┴───────────────────┴────────────────────────────────┘
```

### 1. The "Must Have" Core
*   An empty Python dictionary that accepts string keys (`"A1"`) and Cell objects.
*   A parser that can evaluate basic operations (`+`, `-`, `*`, `/`) between cell names and numbers.
*   A classification loop to split user inputs.

### 2. The "Should Have" Core
*   A dependency graph builder that links cells.
*   A topological recalculator (Kahn's algorithm) that recalculates dependencies in order.
*   Cycle detection that prints `#REF!` if formulas reference themselves.

### 3. The "Nice to Have" Features
*   Range functions (like `SUM(A1:B3)`).
*   CSV/JSON serialization to save grids to disk.
*   Terminal keyboard controls to select cells and edit inputs visually.

---

## Conclusion

You have journeyed from understanding what a variable is, to visualizing memory, to parsing formula trees, sorting dependencies topologically, detecting cycles, and designing a multi-layered spreadsheet application.

What started as a simple flat table is now revealed to be a beautiful, complex network of mathematical flows. You now understand how a spreadsheet engine works from the inside out! 

Happy building!
