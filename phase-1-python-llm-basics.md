# Phase 1 — Final Complete Study Guide
## AI-Augmented SDET Roadmap | Month 1–2

> **IMPORTANT: This is the FINAL, COMPLETE Phase 1 guide.**
> It covers everything you need to know before moving to Phase 2.
>
> **How to use this:**
> - In office (Copilot): Search each topic heading. Read theory only.
> - At home (laptop): Do practical exercises. Run code. Build the exit project.
> - Do NOT skip pitfalls — they save debugging hours.
> - Track progress using checklists at the end.

---

# INTRODUCTION — What Phase 1 Is About

Phase 1 has one goal: **Learn Python and how to call LLMs programmatically.**

By end of Phase 1, you should be able to:
1. Write Python scripts
2. Call Groq and Ollama APIs
3. Understand why LLM outputs are non-deterministic
4. Build a basic evaluator that tests LLM responses

You are NOT building a polished project yet. You are learning foundations.

---

# PART A — PYTHON BASICS FOR SDET (You Know Java, This Will Be Fast)

> Python and Java are 70% similar. The 30% difference will trip you up. We focus on that 30%.

---

## A1. Python vs Java — Key Differences

**Search in Copilot:** "Python vs Java key differences for Java developers"

### What to know:

Python and Java run differently. Understand these differences first:

| Feature | Java | Python | Impact |
|---|---|---|---|
| **Syntax** | Strict: `String name = "John";` | Flexible: `name = "John"` | Python feels loose at first |
| **Types** | Static: declare upfront | Dynamic: inferred at runtime | Fewer errors caught before running |
| **Indentation** | `{}` braces | 4-space indentation | Indentation IS code block delimiter |
| **Execution** | Compile then run | Interpret directly | Faster to iterate, slower to run |
| **Keywords** | More keywords | Fewer keywords | Python is simpler |

### Quick comparison table:

```
JAVA                                    PYTHON
-------------------------------------------
public class Main {                     # No class needed for scripts
    public static void main() {         
        String name = "John";           name = "John"
        int age = 25;                   age = 25
        System.out.println(name);       print(name)
        
        if (age > 18) {                 if age > 18:
            System.out.println("Adult")     print("Adult")
        }                               
    }                                   
}                                       
```

### ⚠️ Pitfall — Indentation is MANDATORY:
```python
# WRONG — will crash
if x > 5:
print("hello")  # ❌ IndentationError

# CORRECT — 4 spaces matter
if x > 5:
    print("hello")  # ✓ 4 spaces
```

### ⚠️ Pitfall — No semicolons:
```python
# WRONG
x = 5;  # ❌ Valid Python but confusing, don't use

# CORRECT
x = 5  # ✓ No semicolon
```

### ⚠️ Pitfall — No type declarations:
```python
# WRONG (Java style)
String name = "John"  # ❌ NameError: String is not defined

# CORRECT (Python style)
name = "John"  # ✓ Python infers it's a string
```

---

## A2. Python Installation (Complete Setup)

**Search in Copilot:** "how to install Python on Windows step by step 2024"
**Search in Copilot:** "Python PATH environment variable Windows setup"

### Step-by-step (CRITICAL — don't skip):

1. **Download:**
   - Go to python.org
   - Click Downloads → Windows
   - Get version 3.11 or 3.12 (latest stable)

2. **Install:**
   - Run installer
   - IMPORTANT: Check box "Add Python to PATH" (appears on first screen)
   - Click "Install Now"

3. **Verify:**
   - Open Command Prompt (Win+R, type `cmd`)
   - Type: `python --version`
   - Should show: `Python 3.11.x` or `Python 3.12.x`

### ⚠️ Pitfall — "python is not recognized":
This means Python is NOT in PATH.

**Fix:**
```
Windows key → type "environment variables" → 
Edit Environment Variables → 
Environment Variables button → 
PATH → Edit → 
New → C:\Users\YourName\AppData\Local\Programs\Python\Python311
(adjust version number)
```

Then restart Command Prompt and retry `python --version`.

### ⚠️ Pitfall — Multiple Python versions:
If you have Python 2.7 and 3.11 both installed:
- `python` might refer to Python 2.7
- Use `python3` instead: `python3 --version`

---

## A3. pip — Python's Package Manager

**Search in Copilot:** "what is pip Python package manager"
**Search in Copilot:** "pip install commands examples"

### What is pip:
pip = "pip installs packages" — Python's equivalent to Maven (Java).

### Basic commands:

```bash
# Install a package
pip install package-name

# See all installed packages
pip list

# Uninstall a package
pip uninstall package-name

# Install specific version
pip install package-name==1.2.3

# Upgrade a package
pip install --upgrade package-name
```

### Example (for Phase 1):
```bash
pip install groq              # Install Groq API client
pip install python-dotenv     # Install dotenv for .env files
pip install ollama            # Install Ollama client
```

### ⚠️ Pitfall — pip vs pip3:
On some systems:
- `pip` = Python 2 (old)
- `pip3` = Python 3 (current)

If `pip install` fails, try `pip3 install` instead.

### ⚠️ Pitfall — Installing without venv:
If you install packages globally, they pollute your system.
Always use virtual environments (see A4).

---

## A4. Virtual Environments (venv) — CRITICAL

**Search in Copilot:** "Python virtual environment venv what is it why use"
**Search in Copilot:** "how to create and activate venv Windows"

### What is venv:
A venv is an isolated Python environment for your project. Each project gets its own packages.

### Why it matters:
```
❌ NO VENV:
Project A needs numpy 1.20
Project B needs numpy 1.24
→ Conflict! Can only install one globally

✓ WITH VENV:
Project A: venv_a → numpy 1.20
Project B: venv_b → numpy 1.24
→ No conflict!
```

### How to use venv:

```bash
# Create venv in current folder
python -m venv venv

# Activate on Windows
venv\Scripts\activate

# Activate on Mac/Linux
source venv/bin/activate

# You'll see (venv) in terminal when active
(venv) C:\Users\Rahul\project>

# Deactivate when done
deactivate
```

### Standard workflow:
```bash
# 1. Create project folder
mkdir my_project
cd my_project

# 2. Create venv
python -m venv venv

# 3. Activate venv
venv\Scripts\activate

# 4. Install packages
pip install groq python-dotenv

# 5. Work on project...

# 6. Deactivate when done
deactivate
```

### ⚠️ Pitfall — Forgetting to activate:
```bash
# WRONG — installing globally
pip install groq

# CORRECT — installing in venv
venv\Scripts\activate
pip install groq
```

### ⚠️ Pitfall — Pushing venv to GitHub:
Never upload `venv/` folder.

Add to `.gitignore`:
```
venv/
__pycache__/
*.pyc
.env
```

---

## A5. Python Data Types — Foundation

**Search in Copilot:** "Python data types int str float bool list dict"
**Search in Copilot:** "Python variables assignment examples"

### Basic types you'll use:

```python
# String (text)
name = "Rahul"
city = 'Noida'  # both " and ' work

# Integer (whole numbers)
age = 25
score = 100

# Float (decimals)
temperature = 37.5
average = 9.8

# Boolean (True/False)
is_active = True
is_admin = False

# None (null equivalent)
value = None
```

### String operations (you'll use constantly):

```python
text = "Hello World"

# Length
len(text)  # 11

# Uppercase/lowercase
text.upper()   # "HELLO WORLD"
text.lower()   # "hello world"

# Replace
text.replace("Hello", "Hi")  # "Hi World"

# Split (split by space)
text.split(" ")  # ["Hello", "World"]

# Check if substring exists
"Hello" in text  # True
"Goodbye" in text  # False

# Slice (get part of string)
text[0:5]  # "Hello"
text[6:]   # "World"
text[-5:]  # "World" (last 5 chars)
```

### f-strings (IMPORTANT for LLM work):

```python
name = "Rahul"
age = 25

# Old way (avoid)
message = "Name: " + name + ", Age: " + str(age)

# New way (f-strings) — use this
message = f"Name: {name}, Age: {age}"
print(message)  # Name: Rahul, Age: 25

# Expressions in f-strings
score = 9.5
message = f"Score: {score * 2}"  # Score: 19.0
```

### ⚠️ Pitfall — String vs number:
```python
# WRONG
age = "25"  # String, not number
age + 5  # ❌ TypeError: can't add string and int

# CORRECT
age = 25  # Number
age + 5  # ✓ 30

# Converting
age_string = "25"
age_number = int(age_string)  # 25
```

---

## A6. Python Lists (Like Java ArrayList)

**Search in Copilot:** "Python list tutorial append remove index"

### What you need to know:

```python
# Create list
fruits = ["apple", "banana", "mango"]

# Access by index (0-based like Java)
fruits[0]    # "apple"
fruits[1]    # "banana"
fruits[-1]   # "mango" (last item, negative index)

# Add to list
fruits.append("orange")  # Add to end

# Remove from list
fruits.remove("banana")  # Remove by value

# Get length
len(fruits)  # 3

# Loop through list
for fruit in fruits:
    print(fruit)

# List comprehension (Python style, cool trick)
numbers = [1, 2, 3, 4, 5]
doubled = [x * 2 for x in numbers]  # [2, 4, 6, 8, 10]
```

### ⚠️ Pitfall — Index out of bounds:
```python
fruits = ["apple", "banana"]
fruits[5]  # ❌ IndexError: list index out of range

# Safe way
if len(fruits) > 5:
    print(fruits[5])
```

---

## A7. Python Dictionaries (Like Java HashMap)

**Search in Copilot:** "Python dictionary tutorial key value pairs"

### What you need to know:

```python
# Create dictionary
person = {
    "name": "Rahul",
    "age": 25,
    "city": "Noida"
}

# Access value by key
person["name"]  # "Rahul"
person["age"]   # 25

# Safe access (recommended)
person.get("name")      # "Rahul"
person.get("salary")    # None (key doesn't exist, no error)

# Add new key-value
person["email"] = "rahul@example.com"

# Delete key
del person["city"]

# Loop through dictionary
for key, value in person.items():
    print(f"{key}: {value}")
# Output:
# name: Rahul
# age: 25
# email: rahul@example.com

# Get all keys
person.keys()  # ["name", "age", "email"]

# Get all values
person.values()  # ["Rahul", 25, "rahul@example.com"]
```

### ⚠️ Pitfall — KeyError:
```python
person = {"name": "Rahul"}

person["age"]  # ❌ KeyError: 'age'

# Always use .get() for safety
person.get("age")  # None (safe)
person.get("age", 25)  # 25 (default value if key missing)
```

---

## A8. Python Functions (Methods)

**Search in Copilot:** "Python function definition def keyword examples"

### Basic function:

```python
def greet(name):
    return f"Hello {name}"

result = greet("Rahul")
print(result)  # Hello Rahul
```

### Function with default parameters:

```python
def greet(name, title="Mr"):
    return f"Hello {title} {name}"

greet("Rahul")            # Hello Mr Rahul
greet("Rahul", "Dr")      # Hello Dr Rahul
```

### Function with multiple return values:

```python
def get_info():
    name = "Rahul"
    age = 25
    return name, age  # Return tuple

name, age = get_info()
print(name, age)  # Rahul 25
```

### ⚠️ Pitfall — Return statement:
```python
# WRONG — function does nothing
def add(a, b):
    a + b  # Calculation happens but not returned

result = add(5, 3)
print(result)  # None

# CORRECT
def add(a, b):
    return a + b

result = add(5, 3)
print(result)  # 8
```

---

## A9. Python if/elif/else and Loops

**Search in Copilot:** "Python if elif else syntax examples"
**Search in Copilot:** "Python for loop while loop range"

### if/elif/else:

```python
score = 8

if score >= 9:
    print("Excellent")
elif score >= 7:
    print("Good")
elif score >= 5:
    print("Average")
else:
    print("Poor")

# Output: Good
```

### for loop:

```python
# Loop 5 times (0 to 4)
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# Loop 1 to 5 (inclusive)
for i in range(1, 6):
    print(i)  # 1, 2, 3, 4, 5

# Loop through list
fruits = ["apple", "banana", "mango"]
for fruit in fruits:
    print(fruit)

# Loop with index
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
    # 0: apple
    # 1: banana
    # 2: mango
```

### while loop:

```python
count = 0
while count < 5:
    print(count)
    count += 1
# Output: 0, 1, 2, 3, 4
```

### ⚠️ Pitfall — Infinite loop:
```python
# WRONG — infinite loop
while True:
    print("Hello")  # This will never stop!

# CORRECT — add break condition
while True:
    print("Hello")
    break  # Exit loop
```

---

## A10. Python File Handling

**Search in Copilot:** "Python read write files open with statement"

### Writing to file:

```python
# Write string to file
with open("output.txt", "w") as f:
    f.write("Hello World")
# File is automatically closed after "with" block

# Append to file (don't overwrite)
with open("output.txt", "a") as f:
    f.write("\nNew line")
```

### Reading from file:

```python
# Read entire file
with open("output.txt", "r") as f:
    content = f.read()
    print(content)

# Read line by line
with open("output.txt", "r") as f:
    for line in f:
        print(line.strip())  # .strip() removes newline
```

### ⚠️ Pitfall — File not found:
```python
# WRONG
with open("missing.txt", "r") as f:
    content = f.read()  # ❌ FileNotFoundError

# CORRECT
import os
if os.path.exists("output.txt"):
    with open("output.txt", "r") as f:
        content = f.read()
```

---

## A11. Python JSON Handling (CRITICAL for LLM Work)

**Search in Copilot:** "Python json module loads dumps parse JSON"

### What to know:
LLMs return JSON. You must know how to parse and create JSON.

```python
import json

# JSON string to Python dictionary
json_string = '{"name": "Rahul", "age": 25}'
data = json.loads(json_string)  # String → Dictionary

print(data["name"])  # Rahul
print(data["age"])   # 25

# Python dictionary to JSON string
person = {"name": "Rahul", "age": 25}
json_string = json.dumps(person)  # Dictionary → String
print(json_string)  # {"name": "Rahul", "age": 25}

# Pretty JSON (with indentation)
json_pretty = json.dumps(person, indent=2)
print(json_pretty)
# {
#   "name": "Rahul",
#   "age": 25
# }
```

### Save and load JSON files:

```python
# Save to file
data = {"name": "Rahul", "scores": [9, 8, 7]}
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Load from file
with open("data.json", "r") as f:
    loaded_data = json.load(f)

print(loaded_data["name"])  # Rahul
```

### ⚠️ Pitfall — JSON parsing errors:
```python
# WRONG — invalid JSON
json.loads('{"name": "Rahul",}')  # ❌ JSONDecodeError (trailing comma)

# CORRECT
json.loads('{"name": "Rahul"}')  # ✓

# Safe way
import json
try:
    data = json.loads(some_string)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
```

---

## A12. Python Error Handling (try/except)

**Search in Copilot:** "Python try except finally error handling"

### Basic structure:

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")

# Output: Error: division by zero
```

### Multiple exceptions:

```python
try:
    age = int("twenty-five")  # Will fail
except ValueError as e:
    print(f"ValueError: {e}")
except TypeError as e:
    print(f"TypeError: {e}")
except Exception as e:  # Catch all exceptions
    print(f"Unknown error: {e}")
```

### Finally (always runs):

```python
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("File not found")
finally:
    if 'file' in locals():
        file.close()  # Always close file
    print("Done")
```

### ⚠️ Pitfall — Bare except:
```python
# WRONG — catches everything including system interrupts
try:
    something()
except:  # ❌ Too broad
    pass

# CORRECT — catch specific exception
try:
    something()
except ValueError as e:  # ✓ Specific
    print(f"Error: {e}")
```

### For LLM API work (very important):

```python
from groq import Groq
import time

client = Groq(api_key="your_key")

try:
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[{"role": "user", "content": "Hello"}]
    )
    print(response.choices[0].message.content)
except Exception as e:
    print(f"API error: {e}")
    time.sleep(5)  # Wait before retrying
```

---

## A13. Python Imports and Modules

**Search in Copilot:** "Python import statement modules packages"

### Different import styles:

```python
# Import entire module
import os
result = os.path.exists("file.txt")

# Import specific function
from dotenv import load_dotenv
load_dotenv()

# Import class
from groq import Groq
client = Groq()

# Import with alias
import json as j
data = j.loads('{"name": "Rahul"}')

# Import everything from module
from os import *  # ❌ Avoid this, unclear what's imported
```

### Common modules you'll use:

```python
import os              # File system operations
import json            # JSON parsing
import time            # Time operations
from dotenv import load_dotenv  # Load .env files
from groq import Groq          # Groq API
import requests        # HTTP requests (if needed)
```

---

## A14. Environment Variables and .env Files

**Search in Copilot:** "Python dotenv .env file environment variables"
**Search in Copilot:** "why not hardcode API keys"

### The problem:

```python
# ❌ WRONG — API key exposed in code!
client = Groq(api_key="gsk_your_actual_key_here")

# If you push to GitHub, bots scrape keys within minutes
```

### The solution — use .env file:

**Step 1: Create .env file in project root:**
```
GROQ_API_KEY=gsk_your_actual_key_here
OLLAMA_HOST=http://localhost:11434
```

**Step 2: Add to .gitignore:**
```
.env
```

**Step 3: Load in Python:**
```python
from dotenv import load_dotenv
import os

load_dotenv()  # Must call this first!

api_key = os.getenv("GROQ_API_KEY")
ollama_host = os.getenv("OLLAMA_HOST")

print(api_key)  # gsk_your_actual_key_here
```

### ⚠️ Pitfall — load_dotenv timing:
```python
# WRONG — calling getenv before load_dotenv
import os
api_key = os.getenv("GROQ_API_KEY")  # None
from dotenv import load_dotenv
load_dotenv()

# CORRECT — load_dotenv first
from dotenv import load_dotenv
import os
load_dotenv()
api_key = os.getenv("GROQ_API_KEY")  # gsk_...
```

### ⚠️ Pitfall — .env in GitHub:
```bash
# Add to .gitignore IMMEDIATELY
echo ".env" >> .gitignore
git rm --cached .env  # Remove if already committed
git commit -m "Remove .env from tracking"
```

---

# PART B — LLM API FUNDAMENTALS

> Now you know Python. Time to understand LLMs conceptually.

---

## B1. What is an LLM (Large Language Model)

**Search in Copilot:** "what is large language model simple explanation"
**Search in Copilot:** "how does ChatGPT LLM work architecture"

### Conceptual understanding:

An LLM is a **statistical model trained on massive text data** that predicts the next most likely word given previous words.

**It does NOT:**
- Think or reason
- Understand meaning deeply
- Have consciousness or memory
- Always tell the truth

**It DOES:**
- Predict patterns from training data
- Generate human-like text
- Sometimes hallucinate (make things up)
- Depend heavily on input (prompt)

### Why this matters for you (SDET):

```
Traditional system: Input → Output (deterministic)
Example: Function isPrime(7) always returns true

LLM: Input → Model → Output (probabilistic)
Example: Question "Is AI useful?" → Response (varies each time)

Your job: Test the probabilistic one
```

### Simple example:

```
Training data: "The capital of India is New Delhi"
Model learns: capital + India → New Delhi
You ask: "What is the capital of India?"
Model predicts: "New Delhi" (or similar phrasing)
```

---

## B2. What is a Token (CRITICAL CONCEPT)

**Search in Copilot:** "what is token LLM API tokenization"
**Search in Copilot:** "how to count tokens in LLM API call"

### Simple definition:
A **token** is a unit of text — roughly a word, but not exactly.

### Token math:
```
Roughly:
- 1 token ≈ 4 characters
- 1 token ≈ 0.75 words

Examples:
"Hello" = 1 token
"software testing" = 2-3 tokens
"What is AI?" = 4-5 tokens
```

### Why tokens matter:

```
Tokens = Cost
If you use Groq API (paid tier), you pay per token.

Tokens = Speed
More tokens = slower response

Tokens = Limits
LLM context window is measured in tokens
llama-3.1-8b: 128,000 token context window
If your input exceeds this, model can't process it
```

### max_tokens parameter:

```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{"role": "user", "content": "What is Python?"}],
    max_tokens=50  # Maximum output tokens
)
```

This means: "Limit your response to 50 tokens (roughly 40 words)"

### ⚠️ Pitfall — max_tokens too low:
```python
# WRONG
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{"role": "user", "content": "Explain machine learning"}],
    max_tokens=10  # Too low!
)
# Response gets cut off mid-sentence

# CORRECT
max_tokens=500  # Better for detailed answers
```

---

## B3. What is Temperature (Controls Randomness)

**Search in Copilot:** "LLM temperature parameter explained 0 vs 1"
**Search in Copilot:** "temperature randomness creativity LLM response"

### Simple definition:
**Temperature** controls how "random" or "creative" the model is.

### Temperature scale:

```
temperature=0.0
↓
Most predictable. Same input → almost same output
Use for: Testing, evaluation, facts
↓
temperature=0.5
↓
Balanced. Some variation
Use for: General purpose
↓
temperature=1.0 (default)
↓
Most random. Same input → very different outputs
Use for: Creative writing, brainstorming
```

### Example with different temperatures:

```
Question: "What is AI?"

temperature=0:
Response 1: "AI is artificial intelligence..."
Response 2: "AI is artificial intelligence..."
(almost identical)

temperature=0.7:
Response 1: "AI is artificial intelligence..."
Response 2: "AI stands for artificial intelligence..."
(slightly different)

temperature=1.0:
Response 1: "AI is the field of artificial intelligence..."
Response 2: "Artificial intelligence, or AI, is..."
(very different)
```

### For your testing work:

```python
# For reproducible tests (use low temperature)
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{"role": "user", "content": "What is testing?"}],
    temperature=0,  # Minimal variation
    max_tokens=200
)

# For testing variation (use higher temperature)
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{"role": "user", "content": "What is testing?"}],
    temperature=0.7,  # More variation
    max_tokens=200
)
```

### ⚠️ Pitfall — temperature=0 is NOT fully deterministic:
Even at temperature=0, you might get slightly different outputs due to:
- Server-side floating-point variations
- Model precision differences

**Never assume absolute determinism at any temperature.**

---

## B4. System Prompt vs User Prompt

**Search in Copilot:** "LLM system prompt vs user prompt difference"
**Search in Copilot:** "how to write effective system prompts"

### Simple explanation:

**System prompt** = Instructions for HOW to behave
**User prompt** = The actual question/request

### Example:

```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant. Answer in 2 sentences maximum."
            # ↑ System prompt — HOW to respond
        },
        {
            "role": "user",
            "content": "What is Python?"
            # ↑ User prompt — WHAT to answer
        }
    ]
)
```

### System prompt importance:

```
Same question, different system prompts:

System: "You are a Python expert."
Q: "What is Python?"
A: "Python is a high-level, dynamically-typed programming language..."

System: "You are a child."
Q: "What is Python?"
A: "Python is a big snake that lives in jungles!"

System: "Respond in exactly 5 words."
Q: "What is Python?"
A: "Python is a programming language."
```

### For testing:

```python
# Test how system prompt affects response
responses = []

for system_msg in [
    "Be concise. Answer in 1 sentence.",
    "Be detailed. Provide 3 paragraphs.",
    "Be technical. Use programming terms."
]:
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[
            {"role": "system", "content": system_msg},
            {"role": "user", "content": "What is testing?"}
        ]
    )
    responses.append(response.choices[0].message.content)

# Compare responses — they should be different
```

---

## B5. Non-Determinism — Why It Matters for Testers

**Search in Copilot:** "non-deterministic systems testing challenges"
**Search in Copilot:** "how do you test non-deterministic output"

### What is non-determinism:

**Deterministic:** Same input always gives same output
```
Function: add(5, 3)
Output: 8 (always)
```

**Non-deterministic:** Same input can give different outputs
```
LLM: "What is the capital of India?"
Output 1: "The capital of India is New Delhi."
Output 2: "New Delhi is the capital of India."
Output 3: "India's capital city is New Delhi."
(all correct, but different)
```

### The tester's problem:

```python
# Traditional testing (doesn't work for LLMs)
assert response == "The capital of India is New Delhi."
# If LLM says "New Delhi is the capital", test FAILS
# But answer is CORRECT!
```

### Three strategies:

**1. Exact match** (only works for structured output):
```python
response = '{"capital": "New Delhi"}'
data = json.loads(response)
assert data["capital"] == "New Delhi"  # ✓ Works for JSON
```

**2. Rule-based validation** (keyword matching):
```python
response = "New Delhi is the capital of India."
keywords = ["New Delhi", "capital", "India"]
assert all(kw in response for kw in keywords)
```

**3. LLM-as-judge** (use another LLM to evaluate):
```python
question = "What is the capital of India?"
answer = "New Delhi is the capital of India."

eval_prompt = f"""
Is this answer correct?
Q: {question}
A: {answer}
Respond: yes or no
"""

eval_response = client.chat.completions.create(...)
is_correct = "yes" in eval_response.lower()
```

### This is your biggest differentiator in interviews:

Interviewers will ask: *"How would you test an LLM response?"*

Your answer should be:
> "I use a hybrid approach. First, rule-based validation for speed (keyword matching, format checks). For edge cases, I use LLM-as-judge (another LLM call to evaluate). This balances cost and accuracy."

---

## B6. Context Window (Token Limit)

**Search in Copilot:** "LLM context window token limit explained"

### What it is:

**Context window** = Total tokens the model can see in one interaction.

```
Context window includes:
- System prompt
- All previous messages in conversation
- Current user question
- Generated response
```

### Example context windows:

```
llama-3.1-8b: 128,000 tokens (~80,000 words)
GPT-4: 128,000 tokens

Older models:
GPT-3.5-turbo (old): 4,096 tokens (~2,700 words)
BERT: 512 tokens (~340 words)
```

### What happens if you exceed:

```python
# If your total tokens exceed window...
prompt = "Tell me the history of India in 50,000 words"
# llama-3.1-8b has 128,000 window
# This request exceeds it

# Model will either:
1. Refuse the request
2. Truncate (lose early information)
3. Output incomplete response
```

### For Phase 1 (not a concern):
You're working with small prompts that fit easily in context window.

In Phase 3 (RAG), this matters because you'll insert retrieved documents:
```
System prompt: 200 tokens
Retrieved doc: 5,000 tokens
User question: 100 tokens
Available for response: 122,700 tokens
```

---

## B7. Prompt Engineering Basics

**Search in Copilot:** "prompt engineering basics zero shot few shot"
**Search in Copilot:** "how to write better prompts for LLMs"

### Four key techniques:

**1. Zero-shot prompting (ask directly):**
```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{
        "role": "user",
        "content": "Is the capital of India New Delhi? Answer yes or no."
    }]
)
# No examples, just ask
```

**2. Few-shot prompting (give examples):**
```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{
        "role": "user",
        "content": """
Classify these cities:
Paris - Country: France
Tokyo - Country: Japan
New Delhi - Country: ?
"""
    }]
)
# Gives examples, then asks
```

**3. Chain-of-thought (ask to reason):**
```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{
        "role": "user",
        "content": "Is Python suitable for web development? Think step by step."
    }]
)
# Forces model to explain reasoning
```

**4. Structured output (ask for specific format):**
```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[{
        "role": "user",
        "content": """
Respond ONLY in JSON format:
{
  "is_correct": true/false,
  "confidence": 0-1,
  "reason": "one sentence"
}

Is New Delhi the capital of India?
"""
    }]
)
```

### ⚠️ Pitfall — Prompt is fragile:
```
Prompt 1: "Is this correct?"
Prompt 2: "Evaluate if this is correct."
Prompt 3: "Judge whether this statement is true."

Same question, 3 different prompts → potentially different answers!

Always document your exact prompt.
```

---

## B8. Structured Output (JSON)

**Search in Copilot:** "LLM structured JSON output how to request"

### Why structured output:

```python
# ❌ Free text (hard to parse)
response = "Yes, New Delhi is the capital of India."
# How do you extract the decision programmatically?

# ✓ Structured JSON (easy to parse)
response = '{"is_capital": true, "city": "New Delhi", "country": "India"}'
data = json.loads(response)
print(data["is_capital"])  # true
```

### How to request JSON:

```python
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[
        {
            "role": "system",
            "content": "You are an evaluator. Respond ONLY in valid JSON. No extra text."
        },
        {
            "role": "user",
            "content": """
Evaluate this answer:
Question: What is the capital of India?
Answer: New Delhi

Respond in this JSON format (ONLY JSON, nothing else):
{
  "is_correct": true or false,
  "score": 0 to 10,
  "confidence": 0 to 1,
  "reason": "one sentence explanation"
}
"""
        }
    ],
    temperature=0,
    max_tokens=200
)

# Parse JSON response
raw = response.choices[0].message.content
result = json.loads(raw)
print(f"Correct: {result['is_correct']}")
print(f"Score: {result['score']}/10")
```

### ⚠️ Pitfall — JSON parsing failures:
```python
# Sometimes LLM adds extra text
raw = 'Here is the result: {"is_correct": true}'
json.loads(raw)  # ❌ JSONDecodeError

# Safe way: extract JSON with regex
import re
match = re.search(r'\{.*\}', raw, re.DOTALL)
if match:
    result = json.loads(match.group())  # ✓
```

---

# PART C — GROQ API

> Time to make actual API calls.

---

## C1. What is Groq (Why Use It)

**Search in Copilot:** "what is Groq AI company LPU inference"

### What Groq offers:

**Groq** is a company that:
- Builds custom hardware (LPU = Language Processing Unit)
- Provides ultra-fast LLM inference
- Offers FREE tier access to models like Llama

### Why use Groq for Phase 1:

```
✓ Free tier
✓ Fast responses
✓ Open-source models (Llama 3.1)
✓ OpenAI-compatible API (easy to switch)
✓ Good for learning
```

### Free tier limits (April 2026):

```
Roughly:
- 14,400 requests/day (llama-3.1-8b-instant)
- 30 requests/minute

Sufficient for all Phase 1 learning
```

---

## C2. Groq API Key Setup

**Search in Copilot:** "how to get Groq API key free"

### Step-by-step:

1. Go to **console.groq.com**
2. Click "Sign In"
3. Create account (email/password or GitHub)
4. Go to **API Keys** section
5. Click **Create API Key**
6. Copy key immediately (you can't see it again)
7. Save in `.env` file:
```
GROQ_API_KEY=gsk_your_key_here
```

### Verify it works:

```python
from groq import Groq
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv("GROQ_API_KEY")

client = Groq(api_key=api_key)
print("✓ Groq API key loaded successfully")
```

### ⚠️ Pitfall — Key format:
```
Groq keys START with: gsk_

If your key doesn't start with gsk_, you copied wrong.
Delete and create new key.
```

### ⚠️ Pitfall — Key exposed:
```
If you accidentally commit .env to GitHub:

1. Remove from git: git rm --cached .env
2. Create new API key (old one is compromised)
3. Add to .gitignore for future
```

---

## C3. Available Groq Models

**Search in Copilot:** "Groq available models list 2024"

### Models you can use (April 2026):

| Model | Speed | Quality | RAM | Tokens/Context |
|---|---|---|---|---|
| llama-3.1-8b-instant | ⭐⭐⭐⭐⭐ Fastest | Good | 8GB | 128K |
| llama-3.1-70b-versatile | ⭐⭐⭐⭐ Fast | Better | 40GB | 128K |
| mixtral-8x7b-32768 | ⭐⭐⭐⭐ Fast | Good | 16GB | 32K |
| gemma2-9b-it | ⭐⭐⭐⭐⭐ Fastest | Good | 9GB | 8K |

### For Phase 1:
Use `llama-3.1-8b-instant` — best balance of speed and quality.

### Check current models:
```python
from groq import Groq

client = Groq()
models = client.models.list()
for model in models.data:
    print(model.id)
```

---

## C4. Making Your First API Call

**Search in Copilot:** "Groq Python API first call example"

### Complete code:

```python
from groq import Groq
import os
from dotenv import load_dotenv

# Load API key
load_dotenv()
api_key = os.getenv("GROQ_API_KEY")

# Create client
client = Groq(api_key=api_key)

# Make API call
response = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "What is software testing?"
        }
    ],
    temperature=0.3,
    max_tokens=200
)

# Print response
print(response.choices[0].message.content)

# Print metadata
print(f"\nTokens used: {response.usage.total_tokens}")
print(f"Model: {response.model}")
```

### Understanding the response object:

```python
response.choices[0].message.content
# The actual text response

response.usage.prompt_tokens
# Tokens in your question

response.usage.completion_tokens
# Tokens in the response

response.usage.total_tokens
# Total tokens (cost = based on this)
```

---

## C5. Handling API Errors

**Search in Copilot:** "Groq API error handling rate limit timeout"

### Common errors:

```python
from groq import Groq
import time

client = Groq(api_key="...")

try:
    response = client.chat.completions.create(...)
    
except Exception as e:
    error_message = str(e)
    
    if "429" in error_message:
        # Rate limit exceeded
        print("Hit rate limit, waiting 10 seconds...")
        time.sleep(10)
    
    elif "401" in error_message:
        # Invalid API key
        print("API key invalid, check .env file")
    
    elif "timeout" in error_message.lower():
        # Request timeout
        print("Request timed out, retrying...")
        time.sleep(5)
    
    else:
        # Other error
        print(f"Error: {e}")
```

### ⚠️ Pitfall — Rate limiting:
```python
# WRONG — too fast
for i in range(100):
    response = client.chat.completions.create(...)
    # ❌ Will hit rate limit after ~15 requests/min

# CORRECT — with delay
import time
for i in range(100):
    response = client.chat.completions.create(...)
    time.sleep(1)  # ✓ Wait 1 second between calls
```

---

# PART D — OLLAMA (LOCAL MODELS)

> Run LLMs on your laptop, no API needed.

---

## D1. What is Ollama

**Search in Copilot:** "what is Ollama local LLM run offline"

### What Ollama does:

```
Groq: Cloud-based inference (fast, needs internet)
Ollama: Local inference (slower, runs on your laptop)

Ollama pros:
✓ No internet needed
✓ No API key required
✓ No rate limits
✓ No API costs
✓ Private (data stays on your laptop)

Ollama cons:
✗ Slower (your laptop vs specialized hardware)
✗ Uses RAM
✗ Requires model download (~4-8 GB)
```

### When to use:

```
Use Groq: For fast development, need quality responses
Use Ollama: For offline work, testing loops without rate limits
Use Both: Groq for production, Ollama for development
```

---

## D2. Ollama Installation

**Search in Copilot:** "how to install Ollama on Windows step by step"

### Step-by-step:

1. Go to **ollama.com**
2. Click **Download**
3. Select **Windows**
4. Run installer
5. Ollama will start as background service
6. Open Command Prompt and verify:
```bash
ollama --version
# Should show: ollama version X.X.X
```

### ⚠️ CRITICAL — Check your RAM FIRST:

```
Your laptop: 32 GB RAM (from earlier screenshots) ✓
You can run any Ollama model comfortably
```

If your laptop had 8 GB, you'd need smaller models (llama3.2:1b).

---

## D3. Download and Run Ollama Models

**Search in Copilot:** "Ollama pull model run command examples"

### Download a model:

```bash
# Download llama 8B (takes ~5-10 minutes, 4.7 GB)
ollama pull llama3.1:8b

# Download smaller model (faster download)
ollama pull llama3.2:1b

# View downloaded models
ollama list
```

### Run a model interactively:

```bash
ollama run llama3.1:8b

# You'll see a prompt, type questions
>>> What is Python?
Python is a programming language...
>>> /bye  (exit)
```

### Stop a model:

```bash
ollama stop llama3.1:8b

# This frees the RAM
```

---

## D4. Using Ollama in Python

**Search in Copilot:** "Ollama Python API OpenAI compatible"

### Method 1 — Direct Ollama client:

```python
from ollama import Client

client = Client(host='http://localhost:11434')

response = client.chat(
    model='llama3.1:8b',
    messages=[
        {'role': 'system', 'content': 'You are helpful.'},
        {'role': 'user', 'content': 'What is Python?'}
    ]
)

print(response['message']['content'])
```

### Method 2 — OpenAI-compatible API (easier):

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # Can be anything, Ollama doesn't validate
)

response = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[
        {"role": "user", "content": "What is Python?"}
    ]
)

print(response.choices[0].message.content)
```

### ⚠️ Pitfall — Ollama must be running:
```python
# If you get connection error...
client = OpenAI(base_url="http://localhost:11434/v1", ...)
response = client.chat.completions.create(...)
# ❌ ConnectionError: Can't connect to Ollama

# Fix:
# Open new terminal and run: ollama serve
# Or just run Ollama app from start menu
```

---

# PART E — RESPONSE VARIABILITY AND PHASE 1 EXIT PROJECT

---

## E1. Understanding Response Variability

**Search in Copilot:** "LLM output variability different responses same input"

### The core concept:

```python
# Run this 5 times, observe different outputs
for i in range(5):
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[{"role": "user", "content": "What is Python?"}],
        temperature=0.7
    )
    print(f"Run {i+1}: {response.choices[0].message.content[:80]}")
    time.sleep(1)

# Output:
# Run 1: Python is a programming language created in 1989...
# Run 2: Python, created by Guido van Rossum, is a language...
# Run 3: A programming language called Python was developed...
# Run 4: Python is a language that emphasizes readability...
# Run 5: The programming language Python was introduced...
```

All different, all correct.

### Why this happens:

```
temperature=0.7
↓
Model generates probabilities for next token
↓
Sometimes picks highest probability
Sometimes picks slightly lower probability (randomness)
↓
Different token sequences = different responses
```

### Test this yourself:

```python
from groq import Groq
import time

client = Groq(api_key="your_key")

print("Testing temperature=0 (should be consistent):")
for i in range(3):
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[{"role": "user", "content": "Is Python a programming language? Answer yes or no."}],
        temperature=0  # No randomness
    )
    print(f"  {i+1}: {response.choices[0].message.content}")

print("\nTesting temperature=0.7 (should vary):")
for i in range(3):
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[{"role": "user", "content": "Is Python a programming language? Answer yes or no."}],
        temperature=0.7  # Some randomness
    )
    print(f"  {i+1}: {response.choices[0].message.content}")
```

---

## E2. Phase 1 Exit Project — Basic LLM Evaluator

**This is your deliverable for Phase 1.**

### Complete working code:

```python
"""
Phase 1 Exit Project — Basic LLM Evaluator
Tests LLM responses and scores them
"""

from groq import Groq
import os
import json
import time
from dotenv import load_dotenv

# Load API key
load_dotenv()
api_key = os.getenv("GROQ_API_KEY")

if not api_key:
    print("ERROR: GROQ_API_KEY not found in .env")
    exit(1)

# Initialize Groq client
client = Groq(api_key=api_key)

def get_llm_answer(question: str) -> str:
    """Get answer from LLM for a question"""
    response = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=[
            {"role": "system", "content": "You are a helpful assistant. Answer concisely."},
            {"role": "user", "content": question}
        ],
        temperature=0.3,
        max_tokens=200
    )
    return response.choices[0].message.content


def evaluate_answer(question: str, answer: str) -> dict:
    """
    Use LLM-as-judge to evaluate if answer is correct
    Returns dict with score and reasoning
    """
    eval_prompt = f"""
You are a strict evaluator. Score this answer.

Question: {question}
Answer: {answer}

Respond ONLY in this JSON format (no extra text):
{{
  "is_correct": true or false,
  "score": 0 to 10,
  "confidence": 0 to 1,
  "reason": "one sentence explanation"
}}
"""
    
    try:
        response = client.chat.completions.create(
            model="llama-3.1-8b-instant",
            messages=[
                {"role": "system", "content": "You are a JSON-only evaluator. Respond ONLY with valid JSON."},
                {"role": "user", "content": eval_prompt}
            ],
            temperature=0,
            max_tokens=200
        )
        
        # Extract JSON from response
        raw_response = response.choices[0].message.content
        
        # Handle cases where LLM adds extra text
        import re
        json_match = re.search(r'\{.*\}', raw_response, re.DOTALL)
        
        if json_match:
            json_str = json_match.group()
            result = json.loads(json_str)
        else:
            result = {
                "is_correct": False,
                "score": 0,
                "confidence": 0,
                "reason": "Could not parse evaluator response"
            }
        
        return result
    
    except Exception as e:
        return {
            "is_correct": False,
            "score": 0,
            "confidence": 0,
            "reason": f"Evaluation error: {str(e)}"
        }


def test_questions(qa_list: list):
    """
    Test multiple Q&A pairs
    
    qa_list: List of dicts with "question" and "expected_answer"
    Example:
    [
        {"question": "What is the capital of India?", "expected_answer": "New Delhi"},
        {"question": "What is Python?", "expected_answer": "programming language"}
    ]
    """
    results = []
    
    print("\n" + "="*70)
    print("PHASE 1 EXIT PROJECT — LLM EVALUATOR TEST")
    print("="*70)
    
    for i, item in enumerate(qa_list, 1):
        question = item["question"]
        expected = item.get("expected_answer", "")
        
        print(f"\n[{i}/{len(qa_list)}] Question: {question}")
        
        # Get LLM answer
        print("  Getting LLM answer...", end="", flush=True)
        llm_answer = get_llm_answer(question)
        print(" ✓")
        print(f"  LLM Answer: {llm_answer[:100]}...")
        
        # Evaluate answer
        print("  Evaluating...", end="", flush=True)
        evaluation = evaluate_answer(question, llm_answer)
        print(" ✓")
        
        # Print results
        print(f"  Correctness: {evaluation['is_correct']}")
        print(f"  Score: {evaluation['score']}/10")
        print(f"  Confidence: {evaluation['confidence']:.1%}")
        print(f"  Reason: {evaluation['reason']}")
        
        # Store result
        results.append({
            "question": question,
            "expected": expected,
            "llm_answer": llm_answer,
            "evaluation": evaluation
        })
        
        # Be nice to API, don't hammer it
        time.sleep(1)
    
    # Summary
    print("\n" + "="*70)
    print("SUMMARY")
    print("="*70)
    
    total = len(results)
    correct = sum(1 for r in results if r["evaluation"]["is_correct"])
    avg_score = sum(r["evaluation"]["score"] for r in results) / total if total > 0 else 0
    
    print(f"Total tested: {total}")
    print(f"Correct: {correct}/{total} ({correct/total*100:.0f}%)")
    print(f"Average score: {avg_score:.1f}/10")
    
    # Save results
    with open("phase1_results.json", "w") as f:
        json.dump(results, f, indent=2)
    print(f"\nResults saved to phase1_results.json")
    
    return results


if __name__ == "__main__":
    # Test questions
    test_qa = [
        {"question": "What is the capital of India?", "expected_answer": "New Delhi"},
        {"question": "What is Python?", "expected_answer": "programming language"},
        {"question": "What year was Python created?", "expected_answer": "1989"},
        {"question": "Who created Python?", "expected_answer": "Guido van Rossum"},
        {"question": "What is software testing?", "expected_answer": "verifying software works correctly"}
    ]
    
    # Run evaluation
    results = test_questions(test_qa)
```

### How to run:

```bash
# Make sure you're in project folder with .env file
python evaluator.py

# Expected output:
# [1/5] Question: What is the capital of India?
#   Getting LLM answer... ✓
#   LLM Answer: New Delhi is the capital of India...
#   Evaluating... ✓
#   Correctness: True
#   Score: 10/10
#   Confidence: 0.95
#   Reason: The answer correctly identifies...
# ...
# SUMMARY
# Total tested: 5
# Correct: 5/5 (100%)
# Average score: 9.2/10
# Results saved to phase1_results.json
```

---

# PHASE 1 — COMPLETE SETUP CHECKLIST

Use this to verify you've completed everything:

## Installation & Setup
- [ ] Python 3.11+ installed (`python --version`)
- [ ] pip working (`pip --version`)
- [ ] Virtual environment created (`venv/` folder exists)
- [ ] venv activated (`(venv)` shown in terminal)
- [ ] Groq API key obtained (from console.groq.com)
- [ ] `.env` file created with `GROQ_API_KEY=gsk_...`
- [ ] `.gitignore` includes `.env`
- [ ] Required packages installed (`pip install groq python-dotenv`)

## Python Fundamentals
- [ ] Can write and run simple Python scripts
- [ ] Understand variables, types, strings
- [ ] Can use lists and dictionaries
- [ ] Can write functions
- [ ] Can use if/else and loops
- [ ] Can handle JSON (parse and create)
- [ ] Can use try/except for errors
- [ ] Understand imports and modules

## LLM Concepts
- [ ] Understand what an LLM is (probabilistic, not deterministic)
- [ ] Know what a token is (4 chars ≈ 1 token)
- [ ] Understand temperature (0 = consistent, 1 = random)
- [ ] Know difference between system and user prompt
- [ ] Understand non-determinism (why LLM testing is hard)
- [ ] Know context window limits
- [ ] Understand prompt engineering basics

## API Integration
- [ ] Can call Groq API successfully
- [ ] Can pass system and user prompts
- [ ] Can adjust temperature and max_tokens
- [ ] Can handle API errors gracefully
- [ ] Can parse JSON responses

## Ollama (Optional but recommended)
- [ ] Ollama installed (`ollama --version`)
- [ ] Model downloaded (`ollama list` shows llama3.1:8b)
- [ ] Can call Ollama from Python

## Exit Project
- [ ] Created `evaluator.py` script
- [ ] Script runs without errors
- [ ] Script successfully evaluates LLM responses
- [ ] Results saved to `phase1_results.json`
- [ ] Can explain every line of code

## Understanding Check
Answer these without looking at notes:
- [ ] Why is LLM testing different from traditional testing?
- [ ] What does temperature control?
- [ ] What is the difference between system and user prompt?
- [ ] Why do you need virtual environments?
- [ ] How do you securely handle API keys?

---

# PHASE 1 EXIT CRITERIA

You are ready for Phase 2 when:

1. ✓ **You can call an LLM**
   - Groq API works
   - You can adjust parameters (temperature, max_tokens)
   - You can parse responses

2. ✓ **You understand non-determinism**
   - You've observed that same input gives different outputs
   - You know this is due to temperature/randomness
   - You understand why traditional testing fails

3. ✓ **You have a working evaluator**
   - The `evaluator.py` script runs
   - It gets LLM answers
   - It evaluates those answers using LLM-as-judge
   - It produces a report

4. ✓ **You can explain your code**
   - Point to your evaluator script
   - Explain what each section does
   - Explain why you made those design choices

---

# QUICK REFERENCE — Common Commands

## Python & pip
```bash
python --version              # Check Python version
python script.py              # Run Python script
pip install package          # Install package
pip list                      # See installed packages
```

## Virtual Environment
```bash
python -m venv venv          # Create venv
venv\Scripts\activate         # Activate (Windows)
source venv/bin/activate      # Activate (Mac/Linux)
deactivate                    # Exit venv
```

## Groq API
```bash
# Set up
export GROQ_API_KEY="gsk_..."  # Mac/Linux
set GROQ_API_KEY=gsk_...       # Windows

# Test
python -c "from groq import Groq; print('✓')"
```

## Ollama
```bash
ollama pull llama3.1:8b       # Download model
ollama list                   # Show models
ollama run llama3.1:8b        # Run interactively
ollama stop llama3.1:8b       # Unload model
ollama serve                  # Start service
```

---

# TROUBLESHOOTING

### "Python not found"
```
→ Python not in PATH
→ Fix: Add Python folder to Windows environment variables
```

### "Module not found: groq"
```
→ Forgot to activate venv before pip install
→ Fix: Activate venv, reinstall: pip install groq
```

### "GROQ_API_KEY returns None"
```
→ .env file not loading
→ Fix: Make sure load_dotenv() called BEFORE os.getenv()
```

### "Connection refused (Ollama)"
```
→ Ollama not running
→ Fix: Run `ollama serve` in separate terminal
```

### "JSONDecodeError when parsing LLM response"
```
→ LLM added text around JSON
→ Fix: Use regex to extract JSON: re.search(r'\{.*\}', ...)
```

---

*Final version: April 2026 | Phase 1 Complete | Ready for Phase 2*
