# Buran for Wolfram Language Programmers

## Introduction

If you know the Wolfram Language, you already think in patterns. Rules like `f[0] -> 1` and `x_ :> x^2` are your daily tools. You use `/.` and `//` to transform expressions. You appreciate that everything is an expression with a head.

Buran speaks your language — literally. Pattern transformation rules are the *only* thing in Buran. No imperative constructs, no special cases. Just patterns becoming patterns.

This document will help you transition from Wolfram Language to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Deep Similarity

Wolfram Language and Buran share the same foundational idea:

| Wolfram Language            | Buran                           |
| --------------------------- | ------------------------------- |
| Everything is an expression | Everything is a pattern         |
| Rules transform expressions | Functions transform patterns    |
| `f[x_] := x^2`              | `f { [𝑥] ↦ [𝑥²] }`              |
| Pattern matching is core    | Pattern matching is *only*      |
| `->` and `:>` for rules     | `↦` for transformation          |
| Heads identify structure    | Constructors identify structure |

Buran is what you get when pattern transformation becomes the *entire* language, not just a powerful feature.

---

## Rules → Function Clauses

**Wolfram Language:**

```mathematica
factorial[0] = 1;
factorial[n_] := n * factorial[n - 1]

(* Or with pure function and recursion *)
factorial = If[# == 0, 1, # * #0[# - 1]] &;
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

The arrow `↦` replaces both `->` (Rule) and `:>` (RuleDelayed). Functions are collections of pattern clauses.

---

## Basic Syntax Correspondences

### Expressions and Values

**Wolfram Language:**

```mathematica
42
"hello"
{1, 2, 3}
<|"a" -> 1, "b" -> 2|>
f[x, y]
```

**Buran:**

```
[42]
["hello"]
[list: 1, 2, 3]
[map: "a", 1, "b", 2]
[f: x, y]
```

All values are patterns in square brackets. Heads become constructor names.

### Function Definitions

**Wolfram Language:**

```mathematica
add[a_, b_] := a + b

greet[name_] := StringJoin["Hello, ", name, "!"]
```

**Buran:**

```
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

greet {
    name ↦ [string: "Hello, ", name, "!"]
}
```

No `:=` vs `=` distinction. All definitions use `↦`.

### Function Calls

**Wolfram Language:**

```mathematica
add[2, 3]
greet["World"]
```

**Buran:**

```
add([2], [3])
greet("World")
```

Parentheses for calls, not brackets. Literal arguments need pattern brackets.

---

## Pattern Matching: Familiar Territory

Wolfram Language's patterns translate almost directly:

**Wolfram Language:**

```mathematica
(* Named pattern *)
x_

(* Named with head constraint *)
x_Integer

(* Blank sequence *)
x__

(* Null sequence *)
x___

(* Condition *)
x_ /; x > 0

(* Alternatives *)
x : (_Integer | _Real)
```

**Buran:**

```
# Named pattern
𝑥

# Type constraint via identity
⟨𝑥 | "type": "integer"⟩

# Rest of list (like __)
⟨rest⟩

# Empty allowed (like ___)
# Handled through pattern clauses for empty case

# Guard condition
[𝑥 | 𝑥 > 0]

# Alternatives via multiple clauses
process {
    ⟨𝑥 | "type": "integer"⟩ ↦ handle-int(𝑥)
    ⟨𝑥 | "type": "real"⟩ ↦ handle-real(𝑥)
}
```

---

## Blank Patterns → Mirror Brackets

**Wolfram Language:**

```mathematica
(* Single element *)
{first_, rest__} = list

(* Destructure *)
f[{a_, b_, c_}] := a + b + c

(* Match any head *)
_[x_, y_]
```

**Buran:**

```
# Single element + rest
[list: first, ⟨rest⟩] ↦ list

# Destructure
f {
    [list: 𝑎, 𝑏, 𝑐] ↦ [𝑎 + 𝑏 + 𝑐]
}

# Any constructor with two elements
[_: 𝑥, 𝑦]
```

The `⟨...⟩` mirror brackets capture remaining elements or access metadata.

---

## Conditions (/;) → Guards (|)

**Wolfram Language:**

```mathematica
f[x_] /; x > 0 := "positive"
f[x_] /; x < 0 := "negative"
f[0] := "zero"

abs[x_] /; x >= 0 := x
abs[x_] := -x
```

**Buran:**

```
f {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

abs {
    [𝑥 | 𝑥 ≥ 0] ↦ 𝑥
    [𝑥] ↦ [−𝑥]
}
```

The vertical bar `|` replaces `/;` for conditions.

---

## Heads → Constructors

**Wolfram Language:**

```mathematica
(* Head identifies expression type *)
Head[{1, 2, 3}]  (* List *)
Head[f[x, y]]    (* f *)

(* Pattern matching on head *)
f[x_List] := Length[x]
f[x_Integer] := x^2
```

**Buran:**

```
# Constructor identifies pattern type
[list: 1, 2, 3]    # list constructor
[f: x, y]          # f constructor

# Pattern matching on constructor
f {
    [list: ⟨items⟩] ↦ length(items)
    ⟨𝑥 | "type": "integer"⟩ ↦ [𝑥²]
}
```

---

## Pure Functions (# &) → Pattern Blocks

**Wolfram Language:**

```mathematica
(* Pure function *)
#^2 &
#1 + #2 &
Function[x, x^2]
Function[{x, y}, x + y]

(* Using with Map *)
Map[#^2 &, {1, 2, 3}]
```

**Buran:**

```
# Pattern block
{ 𝑥 ↦ 𝑥² }
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 }

# Using with map
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3])
```

No `#` slots, no `&`. Named parameters in pattern blocks.

---

## ReplaceAll (/.) → Pattern Matching

**Wolfram Language:**

```mathematica
(* Replacement rules *)
expr /. x -> 1
expr /. {x -> 1, y -> 2}
expr //. x -> f[x]  (* Repeated *)

(* Pattern-based replacement *)
{1, 2, 3} /. {a_, b_, c_} :> a + b + c
```

**Buran:**

```
# Direct pattern matching in functions
transform {
    [expr: 𝑥, 𝑦] ↦ [expr: 1, 2]
}

# Inline matching
value ↦ {
    [a, b, c] ↦ [a + b + c]
    other ↦ other
}
```

Pattern matching is built into function definition, not a separate operation.

---

## Lists

**Wolfram Language:**

```mathematica
{1, 2, 3}
First[list]
Rest[list]
Prepend[list, 0]
Append[list, 4]
Join[list1, list2]
Length[list]
```

**Buran:**

```
[list: 1, 2, 3]

# First (via destructuring)
[list: first, ⟨_⟩] ↦ list

# Rest (via destructuring)
[list: _, ⟨rest⟩] ↦ list

# Prepend
[list: 0, ⟨list⟩]

# Append
[list: ⟨list⟩, 4]

# Join
[list: ⟨list1⟩, ⟨list2⟩]

# Length
length(list)
```

---

## Map, Select, Fold

**Wolfram Language:**

```mathematica
Map[f, list]
Map[#^2 &, {1, 2, 3}]

Select[list, # > 0 &]
Select[{-1, 2, -3, 4}, Positive]

Fold[f, init, list]
Fold[Plus, 0, {1, 2, 3, 4}]
```

**Buran:**

```
map(f, list)
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3])

filter({ 𝑥 | 𝑥 > 0 }, list)
filter({ 𝑥 | 𝑥 > 0 }, [list: -1, 2, -3, 4])

fold(f, init, list)
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], [list: 1, 2, 3, 4])
```

---

## Mathematical Notation

Both languages embrace mathematical symbols:

| Wolfram           | Buran         | Meaning          |
| ----------------- | ------------- | ---------------- |
| `*` or `×`        | `×`           | Multiplication   |
| `/` or `÷`        | `÷`           | Division         |
| `^`               | `²`, `³`, `ⁿ` | Exponentiation   |
| `Sqrt[x]` or `√x` | `√𝑥`          | Square root      |
| `Pi` or `π`       | `𝛑`           | Pi               |
| `<=` or `≤`       | `≤`           | Less or equal    |
| `>=` or `≥`       | `≥`           | Greater or equal |
| `!=` or `≠`       | `≠`           | Not equal        |
| `&&` or `∧`       | `∧`           | Logical AND      |
| `\|\|` or `∨`     | `∨`           | Logical OR       |
| `!` or `¬`        | `¬`           | Logical NOT      |
| `True`            | `[✔]`         | Boolean true     |
| `False`           | `[✘]`         | Boolean false    |
| `Null` or `None`  | `[]`          | Empty/none       |

Buran uses *only* the Unicode forms — no ASCII alternatives.

---

## Associations → Maps

**Wolfram Language:**

```mathematica
<|"name" -> "Alice", "age" -> 30|>
assoc["name"]
Append[assoc, "city" -> "Boston"]
KeyValueMap[f, assoc]
```

**Buran:**

```
[map: "name", "Alice", "age", 30]

# Access via pattern matching or lookup
lookup("name", m) ↦ name

# Add entry
[map: ⟨m⟩, "city", "Boston"]

# Key-value iteration through fold
```

---

## Module and Block → Sequential Binding

**Wolfram Language:**

```mathematica
Module[{x = 1, y = 2},
  x + y
]

Block[{x = 1},
  x^2 + x
]

With[{x = 1, y = 2},
  x + y
]
```

**Buran:**

```
# Sequential binding with arrows
[1] ↦ x ↦
[2] ↦ y ↦
[x + y]

# Same pattern
[1] ↦ x ↦
[x² + x]
```

No distinction between `Module`, `Block`, and `With`. Just sequential binding.

---

## Symbolic Computation

**Wolfram Language:**

```mathematica
(* Symbolic expressions *)
D[x^2, x]  (* Derivative *)
Integrate[x^2, x]  (* Integral *)
Simplify[expr]
Expand[(x + 1)^3]
```

**Buran:**

```
# Symbolic domains (planned)
[∂/∂𝑥 (𝑥²) :: symbolic] ↦ derivative
[∫ 𝑥² d𝑥 :: symbolic] ↦ integral

# Pattern-based symbolic manipulation
simplify {
    [add: 𝑥, 0] ↦ 𝑥
    [mul: 𝑥, 1] ↦ 𝑥
    [mul: _, 0] ↦ [0]
    ...
}
```

Symbolic computation in Buran uses domain annotations and pattern matching.

---

## Error Handling

**Wolfram Language:**

```mathematica
Check[expr, fallback]
Catch[... Throw[x] ...]
Quiet[expr]
```

**Buran:**

```
# Errors are patterns
expr ↦ {
    [error: msg] ↦ fallback
    result ↦ result
}

# Functions return errors as patterns
safe-operation {
    bad-input ↦ [error: invalid input]
    good-input ↦ compute(good-input)
}
```

---

## File I/O

**Wolfram Language:**

```mathematica
Import["data.txt"]
Export["output.txt", data]
Import["config.json", "JSON"]
Export["data.csv", table, "CSV"]
```

**Buran:**

```
[File: "data.txt"] ↦ content
data ↦ [File: "output.txt"]

# Format parsing built-in
[File: "config.json" :: json] ↦ config
table ↦ [File: "data.csv" :: csv]
```

---

## Comparison Table

| Concept        | Wolfram Language       | Buran                 |
| -------------- | ---------------------- | --------------------- |
| Function def   | `f[x_] := ...`         | `f { [𝑥] ↦ ... }`     |
| Rule           | `x -> y`               | `↦` (in context)      |
| Delayed rule   | `x :> y`               | `↦` (always)          |
| Pure function  | `# &` or `Function`    | `{ 𝑥 ↦ ... }`         |
| Condition      | `/;`                   | `\|`                  |
| Pattern        | `x_`                   | `𝑥`                   |
| Blank sequence | `x__`                  | `⟨x⟩`                 |
| List           | `{1, 2, 3}`            | `[list: 1, 2, 3]`     |
| Association    | `<\|...\|>`            | `[map: ...]`          |
| Head           | `Head[expr]`           | Constructor name      |
| True/False     | `True`, `False`        | `[✔]`, `[✘]`          |
| Null           | `Null`, `None`         | `[]`                  |
| Map            | `Map[f, list]`         | `map(f, list)`        |
| Select         | `Select[list, pred]`   | `filter(pred, list)`  |
| Fold           | `Fold[f, init, list]`  | `fold(f, init, list)` |
| ReplaceAll     | `expr /. rules`        | Pattern matching      |
| Module         | `Module[{x = 1}, ...]` | `[1] ↦ x ↦ ...`       |

---

## Example: Symbolic Differentiation

**Wolfram Language:**

```mathematica
myD[n_?NumberQ, _] := 0
myD[x_, x_] := 1
myD[a_ + b_, x_] := myD[a, x] + myD[b, x]
myD[a_ * b_, x_] := a * myD[b, x] + myD[a, x] * b
myD[a_^n_?NumberQ, x_] := n * a^(n-1) * myD[a, x]
```

**Buran:**

```
deriv {
    ⟨𝑛 | "type": "number"⟩, _ ↦ [0]
    𝑥, 𝑥 ↦ [1]
    [add: 𝑎, 𝑏], 𝑥 ↦ [deriv(𝑎, 𝑥) + deriv(𝑏, 𝑥)]
    [mul: 𝑎, 𝑏], 𝑥 ↦ [𝑎 × deriv(𝑏, 𝑥) + deriv(𝑎, 𝑥) × 𝑏]
    [pow: 𝑎, ⟨𝑛 | "type": "number"⟩], 𝑥 ↦ [𝑛 × [pow: 𝑎, 𝑛 − 1] × deriv(𝑎, 𝑥)]
}
```

---

## Example: Expression Simplifier

**Wolfram Language:**

```mathematica
simplify[a_ + 0] := a
simplify[0 + a_] := a
simplify[a_ * 1] := a
simplify[1 * a_] := a
simplify[a_ * 0] := 0
simplify[0 * a_] := 0
simplify[a_^0] := 1
simplify[a_^1] := a
simplify[x_] := x
```

**Buran:**

```
simplify {
    [add: 𝑎, 0] ↦ simplify(𝑎)
    [add: 0, 𝑎] ↦ simplify(𝑎)
    [mul: 𝑎, 1] ↦ simplify(𝑎)
    [mul: 1, 𝑎] ↦ simplify(𝑎)
    [mul: _, 0] ↦ [0]
    [mul: 0, _] ↦ [0]
    [pow: _, 0] ↦ [1]
    [pow: 𝑎, 1] ↦ simplify(𝑎)
    𝑥 ↦ 𝑥
}
```

---

## Example: Tree Traversal

**Wolfram Language:**

```mathematica
treeSum[leaf[n_]] := n
treeSum[node[left_, right_]] := treeSum[left] + treeSum[right]

treeMap[f_, leaf[n_]] := leaf[f[n]]
treeMap[f_, node[left_, right_]] :=
  node[treeMap[f, left], treeMap[f, right]]
```

**Buran:**

```
tree-sum {
    [leaf: 𝑛] ↦ 𝑛
    [node: left, right] ↦ [tree-sum(left) + tree-sum(right)]
}

tree-map {
    f, [leaf: 𝑛] ↦ [leaf: f(𝑛)]
    f, [node: left, right] ↦ [node: tree-map(f, left), tree-map(f, right)]
}
```

---

## What's Different

Coming from Wolfram Language:

1. **No built-in knowledge** — No curated data, no Wolfram|Alpha integration
2. **No notebooks** — Plain text files, not interactive documents
3. **No delayed evaluation** — All `↦` is like `:>`
4. **Explicit constructors** — `[list: ...]` not `{...}`
5. **No `/. ` operator** — Pattern matching in functions
6. **No slot syntax** — Named parameters only

## What's Familiar

1. **Pattern matching** — Core of both languages
2. **Rule-based thinking** — Transformation rules everywhere
3. **Mathematical notation** — Unicode operators
4. **Symbolic expressions** — Patterns can represent anything
5. **Functional style** — map, filter, fold
6. **Head/constructor** — Structure identification

---

## Philosophical Alignment

Wolfram Language asks: *"What symbolic transformations express this computation?"*

Buran asks: *"What pattern does this pattern become?"*

Both languages value:

- Pattern transformation as fundamental
- Mathematical notation
- Symbolic thinking
- Rules as first-class concepts

Wolfram Language provides this plus vast built-in knowledge and computational capabilities. Buran provides a distilled essence: just the pattern transformation, applied uniformly.

If you appreciate Wolfram Language's pattern matching but want something simpler and more uniform — without the computational knowledge base — Buran offers that focused experience.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
