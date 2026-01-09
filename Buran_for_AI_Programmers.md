# Buran for AI Programmers

## Introduction

If you're an AI system generating code — a large language model, a code assistant, or an autonomous agent — you face challenges that human programmers don't. You must produce syntactically correct code without the intuition humans develop over years. You must handle edge cases in languages designed before your existence. You must navigate historical accidents and inconsistencies that humans learned to tolerate.

Buran was designed with you in mind. Not as an afterthought, but as a core design principle: create a language that AI systems can generate reliably, while remaining readable by humans.

This document explains why Buran's design makes your job easier.

> [!IMPORTANT]
>
> Refer to the official Buran language guide for full details of how to write Buran programs.

****

---

## Why Traditional Languages Are Hard for AI

### Syntactic Inconsistency

Most languages evolved over decades, accumulating syntax:

```python
# Python: Multiple ways to define functions
def f(x): return x + 1
f = lambda x: x + 1
f = (lambda x: x + 1)

# Multiple ways to format strings
"Hello " + name
"Hello %s" % name
"Hello {}".format(name)
f"Hello {name}"
```

```javascript
// JavaScript: Multiple function syntaxes
function f(x) { return x + 1; }
const f = function(x) { return x + 1; };
const f = (x) => { return x + 1; };
const f = x => x + 1;
```

Each variation is a decision point. Each decision point is a potential error.

### Context-Dependent Meaning

```javascript
// JavaScript: Same syntax, different meanings
const obj = { x: 1 };     // Object literal
if (x) { y; }             // Block statement
const f = (x) => { x };   // Returns undefined, not x!
const f = (x) => ({ x }); // Returns { x: x }
```

```python
# Python: Context changes meaning
[x for x in items]        # List comprehension
{x for x in items}        # Set comprehension
{x: 1 for x in items}     # Dict comprehension
(x for x in items)        # Generator expression
```

You must track context to generate correct code.

### Historical Accidents

```javascript
// JavaScript
typeof null === "object"  // Famous bug, preserved for compatibility
[] + [] === ""            // Array coercion
[] + {} === "[object Object]"
{} + [] === 0             // Different parsing context!
```

```c
// C
int *p, q;  // p is pointer, q is int — not both pointers!
```

These quirks exist because changing them would break existing code. You must memorize them.

---

## Why Buran Is Different

### One Way To Do Everything

**Buran has one syntax for functions:**

```
f {
    pattern ↦ result
}
```

**One syntax for anonymous functions:**

```
{ pattern ↦ result }
```

**One syntax for values:**

```
[value]
[constructor: args]
```

**One syntax for binding:**

```
expression ↦ name
```

No variations. No decisions. No ambiguity.

### No Context-Dependent Meaning

In Buran, syntax means the same thing everywhere:

```
[list: 1, 2, 3]    # Always a list, everywhere
{ 𝑥 ↦ 𝑥 + 1 }      # Always a pattern block, everywhere
[𝑎 + 𝑏]            # Always addition, everywhere
```

Braces `{ }` always mean pattern block. Square brackets `[ ]` always mean pattern. No exceptions.

### No Historical Accidents

Buran was designed in 2025 with no legacy code to support. Every design decision was intentional:

- `↦` for transformation (not `->`, `=>`, `=`, `:`)
- `×` for multiplication (not `*` which could mean pointer/glob/power)
- `[✔]` and `[✘]` for booleans (not truthy/falsy confusion)
- `[]` for empty (not `null` vs `undefined` vs `None` vs `nil`)

---

## The Uniform Structure

Every Buran program follows the same structure:

### Functions Are Pattern Clauses

```
function-name {
    pattern₁ ↦ result₁
    pattern₂ ↦ result₂
    ...
}
```

### Patterns Have Predictable Form

```
[value]                      # Simple value
[constructor: arg1, arg2]    # Compound pattern
[pattern | guard]            # Pattern with condition
⟨pattern⟩                    # Mirror bracket (metadata/capture)
_                            # Wildcard
```

### Expressions Have Predictable Form

```
function(args)               # Function call
[expression]                 # Evaluation
expression ↦ binding ↦ ...   # Sequential binding
```

This uniformity means you can generate code structurally, not textually.

---

## Generating Buran Code

### Template-Based Generation

Because structure is uniform, you can generate code from templates:

```
# To generate a function:
"{name} {{\n" +
  for each case:
    "    {pattern} ↦ {result}\n" +
"}}"

# To generate a list operation:
"map({{ {var} ↦ {transform} }}, {list})"

# To generate error handling:
"{expression} ↦ {{\n" +
"    [error: msg] ↦ {error_handler}\n" +
"    result ↦ {success_handler}\n" +
"}}"
```

### Structural Correctness

If you follow the structure, your code is syntactically correct:

1. Functions: `name { clauses }`
2. Clauses: `pattern ↦ result`
3. Patterns: `[...]` or `⟨...⟩` or `_`
4. Calls: `function(args)`

No edge cases. No special rules for certain contexts.

### Type Safety Through Structure

Patterns encode their own types:

```
[42]                    # Integer
["hello"]               # String
[list: 1, 2, 3]         # List
[point: 𝑥, 𝑦]           # Point constructor
[error: message]        # Error
[]                      # Empty/absent
```

You don't need to track types separately — the pattern structure is the type.

---

## Common Patterns for Generation

### Conditional Logic

Instead of choosing between `if`/`else`, `switch`, ternary operators:

```
# Always use pattern matching
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

### Iteration

Instead of choosing between `for`, `while`, `forEach`, comprehensions:

```
# Use map for transformation
map({ 𝑥 ↦ transform(𝑥) }, items)

# Use filter for selection
filter({ 𝑥 | condition(𝑥) }, items)

# Use fold for aggregation
fold({ acc, 𝑥 ↦ combine(acc, 𝑥) }, initial, items)

# Use recursion for custom iteration
process {
    [list: ] ↦ []
    [list: head, ⟨tail⟩] ↦ [list: f(head), ⟨process(tail)⟩]
}
```

### Error Handling

Instead of choosing between exceptions, error codes, Result types, Optional:

```
# Errors are patterns
operation() ↦ {
    [error: e] ↦ handle-error(e)
    result ↦ use-result(result)
}

# Functions return errors as patterns
safe-divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}
```

### Data Transformation

```
# Destructure and reconstruct
transform-user {
    [user: name, email, age] ↦ [user:
        uppercase(name),
        email,
        age
    ]
}

# Pipeline style
input ↦
validate(input) ↦ validated ↦
transform(validated) ↦ transformed ↦
save(transformed) ↦ result
```

---

## Mathematical Notation: Unambiguous Symbols

Traditional languages reuse symbols:

| Symbol | C/Python/JS meanings                     |
| ------ | ---------------------------------------- |
| `*`    | Multiply, pointer, unpack, glob, power   |
| `/`    | Divide, path separator, regex delimiter  |
| `<`    | Less than, generic, redirect, XML tag    |
| `>`    | Greater than, generic, redirect, XML tag |
| `-`    | Subtract, negative, flag, range          |

Buran uses distinct Unicode symbols:

| Symbol | Meaning           | Only meaning       |
| ------ | ----------------- | ------------------ |
| `×`    | Multiply          | Always multiply    |
| `÷`    | Divide            | Always divide      |
| `−`    | Subtract/negative | Always subtraction |
| `≤`    | Less or equal     | Always comparison  |
| `≥`    | Greater or equal  | Always comparison  |
| `≠`    | Not equal         | Always comparison  |
| `∧`    | Logical AND       | Always boolean     |
| `∨`    | Logical OR        | Always boolean     |
| `¬`    | Logical NOT       | Always boolean     |
| `↦`    | Transform/bind    | Always data flow   |

No ambiguity. Each symbol has exactly one meaning.

---

## Patterns You Can Rely On

### Bracketing Is Mandatory

```
[42]              # Correct
42                # Invalid — bare literal

["hello"]         # Correct
"hello"           # Only valid inside brackets or as function arg
```

This explicitness prevents errors.

### Arrows Always Flow Data

```
# Data flows right
[42] ↦ x          # 42 goes into x
input ↦ process() ↦ output

# Or flows left (equivalent)
x ↤ [42]          # 42 goes into x
output ↤ process() ↤ input
```

Pick a direction and be consistent. Both work.

### Guards Always Use |

```
[𝑥 | 𝑥 > 0]                    # Guard on pattern
[𝑥 | 𝑥 > 0 ∧ 𝑥 < 100]          # Multiple conditions
⟨𝑥 | "type": "integer"⟩        # Type guard
```

No `if`, no `when`, no `where`. Just `|`.

### Functions Always Use { }

```
# Named function
name { pattern ↦ result }

# Anonymous function (pattern block)
{ pattern ↦ result }

# Multi-clause
{
    pattern₁ ↦ result₁,
    pattern₂ ↦ result₂
}
```

---

## Error-Resistant Generation

### No Dangling Else

```
# Other languages
if (a)
  if (b)
    x();
  else      // Which if does this belong to?
    y();
```

```
# Buran — structure is explicit
classify {
    [a, b] | a ∧ b ↦ x()
    [a, _] | a ↦ ...
    _ ↦ y()
}
```

### No Operator Precedence Surprises

```
# Other languages
a + b * c     // Is this (a+b)*c or a+(b*c)?
a << b + c    // Precedence varies by language
```

```
# Buran — use brackets for clarity
[𝑎 + (𝑏 × 𝑐)]
[(𝑎 + 𝑏) × 𝑐]
```

### No Off-By-One Indexing

```
# Other languages
arr[0]        // 0-indexed
arr[1]        // 1-indexed (some languages)
arr[-1]       // Last element (Python)
arr.first     // First element (Ruby)
```

```
# Buran — pattern matching
[list: first, ⟨rest⟩] ↦ items    # Get first
[list: ⟨init⟩, last] ↦ items     # Get last
```

---

## Validation Is Simple

To verify Buran code is well-formed:

1. **Brackets match**: `[ ]`, `{ }`, `( )`, `⟨ ⟩`
2. **Arrows present**: Every clause has `↦` or `↤`
3. **Patterns in patterns, expressions in expressions**
4. **Functions have structure**: `name { clauses }`

No complex parsing rules. No context-dependent grammar.

---

## Example: Generating a Complete Program

Given a specification:

> "Create a function that filters a list of users to those over 18, transforms their names to uppercase, and returns a count"

**Generated Buran:**

```
process-users {
    users ↦
        filter({ [user: _, _, age] | age > 18 }, users) ↦ adults ↦
        map({ [user: name, email, age] ↦ [user: uppercase(name), email, age] }, adults) ↦ transformed ↦
        length(transformed)
}
```

**The generation process:**

1. Function wrapper: `process-users { users ↦ ... }`
2. Filter: `filter({ [user: _, _, age] | age > 18 }, users)`
3. Pipeline: `↦ adults ↦`
4. Map: `map({ [user: name, email, age] ↦ [user: uppercase(name), email, age] }, adults)`
5. Pipeline: `↦ transformed ↦`
6. Result: `length(transformed)`

Each step follows a template. No decisions about syntax variants.

---

## Why This Matters

### For Code Generation

- Fewer syntax decisions = fewer errors
- Uniform structure = template-based generation
- No ambiguity = no edge cases to handle
- Predictable output = easier verification

### For Code Understanding

- One way to write things = easier to parse
- Explicit structure = clear intent
- Mathematical notation = unambiguous semantics

### For Human-AI Collaboration

- AI generates consistent code
- Humans can easily read and modify it
- No surprises from syntax quirks
- Shared understanding of program structure

---

## Summary

Buran is designed for reliable AI code generation:

| Problem                   | Buran Solution            |
| ------------------------- | ------------------------- |
| Multiple syntax options   | One syntax per concept    |
| Context-dependent meaning | Meaning is context-free   |
| Historical quirks         | Clean-slate design        |
| Ambiguous operators       | Distinct Unicode symbols  |
| Complex parsing           | Uniform bracket structure |
| Type confusion            | Patterns encode types     |
| Error handling variations | Errors are patterns       |
| Control flow diversity    | Pattern matching only     |

If you're an AI system generating code, Buran minimizes the decisions you must make and maximizes the reliability of your output.

Generate with confidence. 

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
