# Phase 1 → Topic 1: Class and Object

Good plan. We'll do **one topic at a time**, in your exact 7-part structure, with an AI/ML + Backend interview slant. Today: Topic 1. By the end you should be able to answer *"What is a class and an object?"* in 30 seconds and then survive 10 minutes of follow-ups.

---

## 1. What is it? (Beginner-friendly)

| Concept | Plain meaning | Analogy |
|---|---|---|
| **Class** | A blueprint/template that defines *what data something holds* and *what it can do*. It's a definition, not a thing. | The architectural drawing of a house |
| **Object (instance)** | A concrete thing built from that blueprint, living in memory, with its **own** values. | The actual house at 12 MG Road |

One class → many objects. Each object has its own **state** (data), but they share the same **behaviour** (methods defined on the class).

**The Python-specific twist interviewers love:** in Python, *everything* is an object — including classes. A class is itself an object created at runtime (an instance of `type`). So classes can be stored in lists, passed to functions, and returned from functions. Java/C++ candidates often miss this.

---

## 2. Why do we need it?

Because the alternative — loose dicts + scattered functions — breaks down as soon as your system grows.

**Without a class (typical junior code in a RAG project):**

```python
doc = {"text": "RAG grounds LLM answers.", "source": "handbook.md"}

def word_count(doc):          # nothing enforces the dict's shape
    return len(doc["text"].split())

def preview(doc, n=30):
    return doc["text"][:n]

# Later, someone passes {"content": "..."} → KeyError at 2 AM in production
```

Problems: no guaranteed structure, no validation, data and the functions that operate on it live in different places, and you can't have "two of these with independent internal state" cleanly.

**With a class:**

```python
doc = Document("RAG grounds LLM answers.", "handbook.md")
doc.word_count()   # data + behaviour travel together
```

You get:
1. **Encapsulation of state + behaviour** in one unit (foundation for Topic 7).
2. **Many independent instances** — two rate limiters that don't interfere.
3. **A contract**: every `Document` is guaranteed to have `text` and `source`.
4. **Extensibility**: subclass/swap implementations later (Topics 8–10).
5. **Testability**: you can instantiate, inject, and mock objects.

> **Maturity signal for interviews:** classes are not always the answer. If something is *pure data with no behaviour*, a `dataclass`/`NamedTuple`/dict is often better. If it's *one function with no state*, just write a function. Say this out loud in an interview — it separates you from people who class-ify everything.

---

## 3. Syntax and code examples

### 3.1 The absolute minimum

```python
class Document:      # 'class' keyword, PascalCase name by convention (PEP 8)
    """A unit of text in a RAG pipeline."""
    pass             # empty body -> still a valid class

d = Document()       # calling the class creates an object (instantiation)
print(d)             # <__main__.Document object at 0x7f...>
```

### 3.2 A useful class

```python
class Document:
    """Blueprint: text + where it came from + what you can do with it."""

    def __init__(self, text: str, source: str):
        self.text = text          # instance variable (Topic 4)
        self.source = source

    def word_count(self) -> int:
        return len(self.text.split())

    def preview(self, n: int = 30) -> str:
        return self.text[:n] + ("..." if len(self.text) > n else "")


# --- creating objects ---
d1 = Document("Retrieval augmented generation grounds LLM answers.", "handbook.md")
d2 = Document("FastAPI uses Pydantic for validation.", "fastapi_docs.md")

print(d1.word_count())   # 6
print(d2.word_count())   # 5
print(d1.preview(20))    # Retrieval augmented ...
print(d2.source)         # fastapi_docs.md
```

### 3.3 Proving what an object actually is

```python
print(type(d1))                  # <class '__main__.Document'>
print(isinstance(d1, Document))  # True
print(d1 is d2)                  # False -> two distinct objects in memory
print(d1 == d2)                  # False -> default == compares identity!
print(id(d1) != id(d2))          # True  -> different memory addresses

print(d1.__dict__)   # {'text': '...', 'source': 'handbook.md'}  <- DATA lives here
print("word_count" in Document.__dict__)  # True <- METHODS live on the class
```

**Key takeaway:** *data lives on the instance, behaviour lives on the class.* This one sentence answers half a dozen interview questions.

### 3.4 Classes are objects too (the Python flex)

```python
print(type(Document))        # <class 'type'>  -> the class is an object of type 'type'

Alias = Document             # classes are first-class: assign them
d3 = Alias("hello world", "notes.txt")

def build(cls, text):        # pass a class as an argument (factory pattern)
    return cls(text, source="unknown")

print(build(Document, "hi").source)   # unknown

# You can even create a class dynamically at runtime:
Config = type("Config", (), {"debug": True})
print(Config().debug)        # True
```

---

## 4. Line-by-line explanation

Using the class from 3.2:

| Line | What's happening |
|---|---|
| `class Document:` | Python **executes the class body right now** (at definition/import time), collects the resulting namespace, and creates a class object named `Document`. It does *not* create any instance. |
| `"""A unit of text..."""` | Docstring, stored as `Document.__doc__`. |
| `def __init__(self, text, source):` | Defines the initializer **on the class**. Python calls it automatically right after a new object is created. `self` will be that new object. |
| `self.text = text` | Creates an attribute `text` **inside that instance's `__dict__`**. `d1.text` and `d2.text` are separate slots. |
| `self.source = source` | Same — per-object state. |
| `def word_count(self):` | A function stored on the class. When you call `d1.word_count()`, Python translates it to `Document.word_count(d1)` — that's how `self` gets filled in (Topic 3). |
| `return len(self.text.split())` | Reads *this* object's text, not any global. |
| `d1 = Document("...", "handbook.md")` | Three steps: (1) create a blank `Document` object, (2) call `__init__(new_obj, "...", "handbook.md")`, (3) bind the object to the name `d1`. |
| `d1.word_count()` | Attribute lookup: not in `d1.__dict__` → found on `Document` → bound to `d1` → called. |

> Note: `d1` is a **name/reference** pointing to the object, not the object itself. `d_copy = d1` copies the reference, not the object. Interviewers probe this constantly.

---

## 5. Real-world examples

### (a) AI/ML — the example that makes it click instantly

```python
import torch.nn as nn

layer1 = nn.Linear(768, 256)   # class -> object
layer2 = nn.Linear(768, 256)   # same class, DIFFERENT weights
```

`nn.Linear` is the blueprint. `layer1` and `layer2` are objects with identical structure but **independent parameters** — training one doesn't change the other. Your entire model is a tree of objects. Same story with `AutoTokenizer.from_pretrained(...)`, `ChatOpenAI(...)`, `FAISS` index objects.

### (b) Backend — independent state per object (FastAPI-style)

```python
import time
from collections import deque

class RateLimiter:
    """Sliding-window limiter. Each instance keeps its OWN call history."""

    def __init__(self, max_calls: int, window_seconds: float):
        self.max_calls = max_calls
        self.window = window_seconds
        self.calls = deque()          # per-object state

    def allow(self) -> bool:
        now = time.monotonic()
        while self.calls and now - self.calls[0] > self.window:
            self.calls.popleft()
        if len(self.calls) < self.max_calls:
            self.calls.append(now)
            return True
        return False


chat_limiter  = RateLimiter(max_calls=5,   window_seconds=60)   # /chat
embed_limiter = RateLimiter(max_calls=100, window_seconds=60)   # /embeddings

for _ in range(6):
    print(chat_limiter.allow())    # True True True True True False

print(embed_limiter.allow())       # True  <- unaffected by chat_limiter
```

**Why a class here?** Because you need *state that persists across calls* and *multiple independent copies*. A plain function can't do that without globals. This is a very common backend interview design question.

---

## 6. Common interview questions (with model answers)

**Q1. Difference between class and object?**
A class is a logical blueprint defining attributes and methods; it consumes no per-instance memory. An object is a runtime instance of that class with its own identity, state, and memory. One class can produce unlimited objects.

**Q2. Is everything in Python an object? Are classes objects?**
Yes. Integers, functions, modules, and classes are all objects. A class is an instance of its metaclass, which is `type` by default: `type(Document) is type` → `True`.

**Q3. What actually happens when Python executes a `class` statement?**
The body runs immediately in a new namespace; that namespace dict plus the name and bases is passed to the metaclass (`type`), which builds the class object and binds it to the class name. No instances are created.

**Q4. Where are attributes stored?**
Instance data in `obj.__dict__`; methods and class-level attributes in `Class.__dict__` (a `mappingproxy`). Lookup checks the instance first, then the class, then base classes via the MRO.

**Q5. `type()` vs `isinstance()`?**
`type(x)` returns the exact class; `isinstance(x, C)` also returns `True` for subclasses. Prefer `isinstance` for checks, `type` only when you need exact-type equality.

**Q6. `==` vs `is` for objects?**
`is` compares identity (same memory object). `==` calls `__eq__`, which by default falls back to identity — so two `Document("a","b")` objects are `==` `False` until you implement `__eq__` (Topic 15) or use a dataclass.

**Q7. What is `object`?**
The root base class of every Python class. `class A: pass` implicitly inherits from `object`, which supplies default `__init__`, `__str__`, `__eq__`, etc.

**Q8. What's the use of an empty class (`pass`)?**
Custom exception hierarchies, marker/sentinel types, lightweight namespaces, and stubs you fill in later.

**Q9. Can two objects of the same class have different attributes?**
Yes — Python is dynamic: `d1.extra = 5` works and affects only `d1`. It's usually bad practice; use `__slots__` or a dataclass to enforce a fixed shape (and reduce memory).

**Q10. When would you *not* use a class?**
Pure data → `dataclass`/`NamedTuple`/Pydantic model. Stateless single operation → function. Classes earn their keep when state and behaviour belong together.

### Output-based questions (very common in screening rounds)

```python
# Puzzle 1
class A: pass
a = A(); a.x = 10
b = A()
print(a.x, hasattr(b, "x"))
```
→ `10 False` — attributes attach to individual instances.

```python
# Puzzle 2
class Counter:
    count = 0
    def __init__(self):
        Counter.count += 1
a, b, c = Counter(), Counter(), Counter()
print(Counter.count, a.count)
```
→ `3 3` — `count` is a **class variable** shared by all; `a.count` falls back to the class. (Full treatment in Topic 5.)

```python
# Puzzle 3
class Dog:
    def __init__(self, name): self.name = name
d1, d2 = Dog("Rex"), Dog("Rex")
print(d1 == d2, d1 is d2)
```
→ `False False` — no custom `__eq__`, so equality is identity.

### Mistakes to avoid
- Writing `def word_count():` without `self` → `TypeError: takes 0 positional arguments but 1 was given`.
- Calling `Document.word_count()` instead of `d1.word_count()` → missing `self`.
- Thinking `d_copy = d1` creates a new object (it doesn't — use `copy.copy`/`deepcopy`).
- Naming classes `snake_case` (PEP 8 says `PascalCase`).

---

## 7. Practical coding exercises

Do these in a file or notebook — **don't just read them**. Attempt 1 and 2, paste your code here, and I'll review it like an interviewer before we move to Topic 2.

### Exercise 1 — Easy (backend flavour)
Create a `ChatMessage` class.
- Attributes: `role` (`"user"`/`"assistant"`/`"system"`), `content`.
- Methods: `char_count()`, `is_from_user()`.
- Build a list of 3 messages (a mini conversation) and print the total characters and how many came from the user.

Expected shape:
```python
m = ChatMessage("user", "What is RAG?")
m.char_count()     # 12
m.is_from_user()   # True
```

### Exercise 2 — Medium (directly RAG-relevant)
Create `Document` and `Chunk` classes.
- `Document(text, source)` with a method `to_chunks(size: int, overlap: int) -> list[Chunk]` that splits **by words** with a sliding window.
- `Chunk(text, index, source)`.
- Validation: raise `ValueError` if `size <= 0` or `overlap >= size`.
- Edge case: empty text → return `[]`.

Test with: `"a b c d e f g h"`, `size=3`, `overlap=1` → chunks `["a b c", "c d e", "e f g", "g h"]`.
Then answer in a comment: *why is `Chunk` its own class instead of just returning strings?*

### Exercise 3 — Interview-style (design reasoning)
Refactor this into a class and justify your design in 3 lines:

```python
hits, misses = 0, 0
cache = {}

def get_embedding(text):
    global hits, misses
    if text in cache:
        hits += 1
        return cache[text]
    misses += 1
    vec = [len(text)] * 8      # pretend model call
    cache[text] = vec
    return vec
```
Requirements: two independent caches must be able to coexist, and you must expose `hit_rate()`. Then state which parts are instance state and which behaviour belongs on the class.

---

## Your 30-second interview answer (memorise the shape, not the words)

> "A class is a blueprint that defines attributes and behaviour; an object is a runtime instance of it with its own identity and state. In Python, instance data lives in the object's `__dict__` while methods live on the class, and classes themselves are objects created by the `type` metaclass — so they can be passed around like any other value. Practically, I use classes when state and behaviour belong together, like a `RateLimiter` per endpoint or a `Retriever` holding an index; for pure data I'd reach for a dataclass or Pydantic model instead."

---

**Next up: Topic 2 — Constructor (`__init__`)**, where we'll cover why `__init__` is *not* the object creator (`__new__` is), default-argument traps, and constructors in inheritance chains.

Want to attempt Exercises 1 & 2 first, or shall I go straight into Topic 2?