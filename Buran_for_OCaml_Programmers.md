# Buran for OCaml Programmers

## Introduction

If you know OCaml, you're already thinking the right way. Pattern matching, algebraic data types, immutability, type inference — these are your daily tools. Buran takes these ideas and distills them further: everything is a pattern, and all computation is pattern transformation.

This document will help you transition from OCaml to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Core Similarity

OCaml and Buran share the same philosophical foundation. The transition is more about syntax than concepts:

**OCaml:**

```ocaml
let rec factorial = function
  | 0 -> 1
  | n -> n * factorial (n - 1)
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

The structure is nearly identical: pattern clauses mapping inputs to outputs. Buran just uses different delimiters and arrows.

---

## Key Differences at a Glance

| Aspect          | OCaml                         | Buran                    |
| --------------- | ----------------------------- | ------------------------ |
| Function syntax | `let f x = ...` or `function` | `f { pattern ↦ result }` |
| Pattern arrow   | `->`                          | `↦`                      |
| Values          | Bare literals: `42`           | Bracketed: `[42]`        |
| Type system     | Static, inferred, ML-style    | Identity-based metadata  |
| Modules         | First-class modules, functors | Not yet specified        |
| Currying        | Automatic                     | Not supported            |
| Mutability      | `ref`, mutable fields         | None                     |

---

## Basic Syntax Correspondences

### Let Bindings

**OCaml:**

```ocaml
let x = 42
let name = "hello"
let items = [1; 2; 3]
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
```

Key difference: All values are in square brackets. The arrow indicates data flow.

### Functions

**OCaml:**

```ocaml
let add a b = a + b

let greet name =
  "Hello, " ^ name ^ "!"
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

Functions are collections of pattern clauses. No `let`, no `=` for definition.

### Function Application

**OCaml:**

```ocaml
let sum = add 2 3
let msg = greet "World"
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Note: Buran uses parentheses and commas for arguments. Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Pattern Matching: Familiar Territory

Pattern matching works almost identically:

**OCaml:**

```ocaml
let classify = function
  | n when n > 0 -> "positive"
  | n when n < 0 -> "negative"
  | 0 -> "zero"
```

**Buran:**

```
classify {
    [𝑛 | 𝑛 > 0] ↦ ["positive"]
    [𝑛 | 𝑛 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

Guards use `|` instead of `when`, but the concept is identical.

### Destructuring

**OCaml:**

```ocaml
let first (a, _) = a

let head = function
  | x :: _ -> Some x
  | [] -> None

let rec length = function
  | [] -> 0
  | _ :: rest -> 1 + length rest
```

**Buran:**

```
first {
    [𝑎, _] ↦ 𝑎
}

head {
    [list: 𝑥, ⟨_⟩] ↦ 𝑥
    [list: ] ↦ []
}

length {
    [list: ] ↦ [0]
    [list: _, ⟨rest⟩] ↦ [1 + length(rest)]
}
```

The `⟨...⟩` mirror brackets capture remaining elements — like OCaml's `::` for list tails.

---

## Algebraic Data Types → Constructors

OCaml variants map directly to Buran constructors:

**OCaml:**

```ocaml
type shape =
  | Circle of float
  | Rectangle of float * float
  | Triangle of float * float * float

let area = function
  | Circle r -> Float.pi *. r *. r
  | Rectangle (w, h) -> w *. h
  | Triangle (a, b, c) ->
      let s = (a +. b +. c) /. 2.0 in
      sqrt (s *. (s -. a) *. (s -. b) *. (s -. c))
```

**Buran:**

```
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
    [triangle: 𝑎, 𝑏, 𝑐] ↦
        [(𝑎 + 𝑏 + 𝑐) ÷ 2] ↦ 𝑠 ↦
        [√(𝑠 × (𝑠 − 𝑎) × (𝑠 − 𝑏) × (𝑠 − 𝑐))]
}
```

No type declaration needed. Constructors are just pattern names with lowercase identifiers.

### Records → Named Constructors

**OCaml:**

```ocaml
type color = { r: int; g: int; b: int }

let red = { r = 255; g = 0; b = 0 }
let brightness { r; g; b } = (r + g + b) / 3
```

**Buran:**

```
[rgb: 255, 0, 0] ↦ red

brightness {
    [rgb: 𝑟, 𝑔, 𝑏] ↦ [(𝑟 + 𝑔 + 𝑏) ÷ 3]
}
```

Buran uses positional arguments in constructors. Field names are implicit in the constructor definition.

### Defining Types with Identity

**OCaml:**

```ocaml
type color = { r: int; g: int; b: int }
(* Type is structural *)
```

**Buran:**

```
⟨[rgb: _, _, _]⟩ ↤ [identity:
    "type": "Color",
    "description": "RGB color with three components"
]
```

Types are patterns with identity metadata attached via mirror brackets.

---

## Option and Result → Empty and Error Patterns

**OCaml:**

```ocaml
type 'a option = None | Some of 'a
type ('a, 'e) result = Ok of 'a | Error of 'e

let divide a b =
  if b = 0.0 then Error "division by zero"
  else Ok (a /. b)

let find pred = function
  | [] -> None
  | x :: rest ->
      if pred x then Some x
      else find pred rest
```

**Buran:**

```
# [] is None, [error: ...] is Error, anything else is Some/Ok

divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

find {
    _, [list: ] ↦ []
    pred, [list: 𝑥, ⟨rest⟩] | pred(𝑥) ↦ 𝑥
    pred, [list: _, ⟨rest⟩] ↦ find(pred, rest)
}
```

- `[]` (empty pattern) — like `None`
- `[error: message]` — like `Error message`
- Any other pattern — like `Some value` or `Ok value`

### Pattern Matching on Results

**OCaml:**

```ocaml
match result with
| Ok value -> process value
| Error msg -> handle_error msg
```

**Buran:**

```
result ↦ {
    [error: msg] ↦ handle-error(msg)
    value ↦ process(value)
}
```

---

## No Currying

This is a significant departure from OCaml:

**OCaml:**

```ocaml
let add a b = a + b
let add5 = add 5        (* Partial application *)
let result = add5 3     (* 8 *)

List.map (add 5) [1; 2; 3]  (* [6; 7; 8] *)
```

**Buran:**

```
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

# No partial application — use pattern blocks instead
{ 𝑥 ↦ add([5], 𝑥) } ↦ add5
add5([3]) ↦ result

map({ 𝑥 ↦ [𝑥 + 5] }, [list: 1, 2, 3])
```

Buran functions must be called with all arguments at once. Use pattern blocks for partial application patterns.

---

## Higher-Order Functions

**OCaml:**

```ocaml
let doubled = List.map (fun x -> x * 2) numbers
let evens = List.filter (fun x -> x mod 2 = 0) numbers
let sum = List.fold_left (+) 0 numbers
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, numbers) ↦ doubled
filter({ 𝑥 | 𝑥 mod 2 = 0 }, numbers) ↦ evens
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], numbers) ↦ sum
```

Pattern blocks `{ ... }` replace anonymous functions. The filter pattern block uses a guard without a result — it's a predicate.

### Anonymous Functions

**OCaml:**

```ocaml
fun x -> x + 1
fun (a, b) -> a + b
function
  | 0 -> "zero"
  | n when n > 0 -> "positive"
  | _ -> "negative"
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 + 1 }
{ [𝑎, 𝑏] ↦ 𝑎 + 𝑏 }
{
    [0] ↦ ["zero"],
    [𝑛 | 𝑛 > 0] ↦ ["positive"],
    _ ↦ ["negative"]
}
```

Pattern blocks can have multiple clauses, like OCaml's `function`.

---

## Lists

**OCaml:**

```ocaml
let items = [1; 2; 3]
let first :: rest = items   (* Destructuring *)
let empty = []
let prepended = 0 :: items
let concat = items @ [4; 5]
```

**Buran:**

```
[list: 1, 2, 3] ↦ items
[list: first, ⟨rest⟩] ↦ items    # Destructuring
[list: ] ↦ empty
[list: 0, ⟨items⟩] ↦ prepended
[list: ⟨items⟩, 4, 5] ↦ concat
```

The `⟨...⟩` captures remaining elements for destructuring or spreads elements for construction.

### List Functions

**OCaml:**

```ocaml
let rec map f = function
  | [] -> []
  | x :: rest -> f x :: map f rest

let rec filter pred = function
  | [] -> []
  | x :: rest when pred x -> x :: filter pred rest
  | _ :: rest -> filter pred rest

let rec fold_left f acc = function
  | [] -> acc
  | x :: rest -> fold_left f (f acc x) rest
```

**Buran:**

```
map {
    _, [list: ] ↦ [list: ]
    f, [list: 𝑥, ⟨rest⟩] ↦ [list: f(𝑥), ⟨map(f, rest)⟩]
}

filter {
    _, [list: ] ↦ [list: ]
    pred, [list: 𝑥, ⟨rest⟩] | pred(𝑥) ↦ [list: 𝑥, ⟨filter(pred, rest)⟩]
    pred, [list: _, ⟨rest⟩] ↦ filter(pred, rest)
}

fold {
    _, acc, [list: ] ↦ acc
    f, acc, [list: 𝑥, ⟨rest⟩] ↦ fold(f, f(acc, 𝑥), rest)
}
```

---

## Type Inference → Identity-Based Types

OCaml infers types at compile time. Buran uses identity metadata:

**OCaml:**

```ocaml
(* Type inferred: int -> int -> int *)
let add a b = a + b

(* Explicit annotation *)
let add (a: int) (b: int) : int = a + b

(* Polymorphic *)
let first (a, _) = a  (* 'a * 'b -> 'a *)
```

**Buran:**

```
# No type annotations — patterns are polymorphic by default
add {
    𝑎, 𝑏 ↦ [𝑎 + 𝑏]
}

first {
    [𝑎, _] ↦ 𝑎
}

# Type constraints through identity guards
add-integers {
    ⟨𝑎 | "type": "integer"⟩, ⟨𝑏 | "type": "integer"⟩ ↦ [𝑎 + 𝑏]
}
```

Type checking happens through identity guards when needed.

### Defining Custom Types

**OCaml:**

```ocaml
type point = { x: float; y: float }
type 'a tree =
  | Leaf of 'a
  | Node of 'a tree * 'a tree
```

**Buran:**

```
⟨[point: _, _]⟩ ↤ [identity:
    "type": "Point",
    "fields": ["x", "y"]
]

⟨[leaf: _]⟩ ↤ [identity: "type": "Tree", "variant": "leaf"]
⟨[node: _, _]⟩ ↤ [identity: "type": "Tree", "variant": "node"]
```

---

## Modules → Not Yet Specified

OCaml's module system (modules, signatures, functors) is one of its most powerful features:

**OCaml:**

```ocaml
module type STACK = sig
  type 'a t
  val empty : 'a t
  val push : 'a -> 'a t -> 'a t
  val pop : 'a t -> ('a * 'a t) option
end

module ListStack : STACK = struct
  type 'a t = 'a list
  let empty = []
  let push x s = x :: s
  let pop = function
    | [] -> None
    | x :: rest -> Some (x, rest)
end
```

Buran does not yet have a specified module system. For now, organize code in separate files:

```
# stack.buran
empty-stack ↦ [list: ]

push {
    𝑥, stack ↦ [list: 𝑥, ⟨stack⟩]
}

pop {
    [list: ] ↦ []
    [list: 𝑥, ⟨rest⟩] ↦ [[𝑥, rest]]
}
```

---

## No Mutation

OCaml allows controlled mutation. Buran is purely immutable:

**OCaml:**

```ocaml
let counter = ref 0
let increment () = counter := !counter + 1

type mutable_point = { mutable x: float; mutable y: float }
let p = { x = 0.0; y = 0.0 }
let () = p.x <- 1.0
```

**Buran:**

```
# No refs, no mutable fields
# State changes create new values

increment {
    counter ↦ [counter + 1]
}

# Transform to new point
move-x {
    [point: _, 𝑦], 𝑑𝑥 ↦ [point: 𝑑𝑥, 𝑦]
}
```

State must be threaded through functions or handled through other patterns.

---

## Let Expressions

**OCaml:**

```ocaml
let area r =
  let pi = 3.14159 in
  let r_squared = r *. r in
  pi *. r_squared
```

**Buran:**

```
area {
    𝑟 ↦
        [𝛑] ↦ pi ↦
        [𝑟 × 𝑟] ↦ r-squared ↦
        [pi × r-squared]
}

# Or more directly
area {
    𝑟 ↦ [𝛑 × 𝑟²]
}
```

Sequential bindings use chained arrows. Or just use mathematical notation directly.

---

## File I/O

**OCaml:**

```ocaml
(* Read *)
let content = In_channel.read_all "data.txt"

(* Write *)
Out_channel.write_all "output.txt" ~data:content

(* JSON with yojson *)
let json = Yojson.Safe.from_file "config.json"
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
content ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ config
```

Built-in format support: `:: json`, `:: yaml`, `:: csv`, `:: binary`.

---

## Mathematical Notation

OCaml uses ASCII operators. Buran uses proper mathematical symbols:

| OCaml       | Buran         | Meaning               |
| ----------- | ------------- | --------------------- |
| `*` or `*.` | `×`           | Multiplication        |
| `/` or `/.` | `÷`           | Division              |
| `**`        | `²`, `³`, `ⁿ` | Exponentiation        |
| `sqrt x`    | `√𝑥`          | Square root           |
| `Float.pi`  | `𝛑`           | Pi                    |
| `<=`        | `≤`           | Less than or equal    |
| `>=`        | `≥`           | Greater than or equal |
| `<>`        | `≠`           | Not equal             |
| `&&`        | `∧`           | Logical AND           |
| `\|\|`      | `∨`           | Logical OR            |
| `not`       | `¬`           | Logical NOT           |
| `true`      | `[✔]`         | Boolean true          |
| `false`     | `[✘]`         | Boolean false         |
| `None`      | `[]`          | Empty/none            |

---

## Comparison Table

| Concept         | OCaml                     | Buran                      |
| --------------- | ------------------------- | -------------------------- |
| Function        | `let f x = ...`           | `f { pattern ↦ result }`   |
| Pattern match   | `match x with \| ...`     | Implicit in functions      |
| Arrow           | `->`                      | `↦`                        |
| Guard           | `when cond`               | `\| cond`                  |
| Variant         | `type t = A \| B of int`  | `[a]`, `[b: value]`        |
| Record          | `{ field: value }`        | `[constructor: values]`    |
| Tuple           | `(a, b)`                  | `[a, b]`                   |
| List            | `[1; 2; 3]`               | `[list: 1, 2, 3]`          |
| Cons            | `x :: rest`               | `[list: x, ⟨rest⟩]`        |
| Lambda          | `fun x -> ...`            | `{ x ↦ ... }`              |
| Option          | `Some x`, `None`          | `x`, `[]`                  |
| Result          | `Ok x`, `Error e`         | `x`, `[error: e]`          |
| Let             | `let x = e in ...`        | `e ↦ x ↦ ...`              |
| Type annotation | `(x : int)`               | `⟨x \| "type": "integer"⟩` |
| Module          | `module M = struct...end` | File-based (TBD)           |

---

## Example: Binary Tree

**OCaml:**

```ocaml
type 'a tree =
  | Leaf of 'a
  | Node of 'a tree * 'a tree

let rec sum = function
  | Leaf n -> n
  | Node (left, right) -> sum left + sum right

let rec map f = function
  | Leaf x -> Leaf (f x)
  | Node (l, r) -> Node (map f l, map f r)

let rec fold f acc = function
  | Leaf x -> f acc x
  | Node (l, r) -> fold f (fold f acc l) r
```

**Buran:**

```
sum {
    [leaf: 𝑛] ↦ 𝑛
    [node: left, right] ↦ [sum(left) + sum(right)]
}

tree-map {
    f, [leaf: 𝑥] ↦ [leaf: f(𝑥)]
    f, [node: l, r] ↦ [node: tree-map(f, l), tree-map(f, r)]
}

tree-fold {
    f, acc, [leaf: 𝑥] ↦ f(acc, 𝑥)
    f, acc, [node: l, r] ↦ tree-fold(f, tree-fold(f, acc, l), r)
}
```

---

## Example: Expression Evaluator

**OCaml:**

```ocaml
type expr =
  | Num of int
  | Add of expr * expr
  | Mul of expr * expr
  | Var of string

type env = (string * int) list

let rec eval env = function
  | Num n -> n
  | Add (a, b) -> eval env a + eval env b
  | Mul (a, b) -> eval env a * eval env b
  | Var name ->
      match List.assoc_opt name env with
      | Some v -> v
      | None -> failwith ("Unbound: " ^ name)
```

**Buran:**

```
eval {
    _, [num: 𝑛] ↦ 𝑛
    env, [add: 𝑎, 𝑏] ↦ [eval(env, 𝑎) + eval(env, 𝑏)]
    env, [mul: 𝑎, 𝑏] ↦ [eval(env, 𝑎) × eval(env, 𝑏)]
    env, [var: name] ↦ lookup(env, name)
}

lookup {
    [map: ], name ↦ [error: [string: "Unbound: ", name]]
    [map: name, value, ⟨_⟩], name ↦ value
    [map: _, _, ⟨rest⟩], name ↦ lookup([map: ⟨rest⟩], name)
}
```

---

## Example: Parser Combinators

**OCaml:**

```ocaml
type 'a parser = string -> ('a * string) option

let pure x : 'a parser = fun s -> Some (x, s)

let (>>=) (p: 'a parser) (f: 'a -> 'b parser) : 'b parser =
  fun s ->
    match p s with
    | None -> None
    | Some (a, rest) -> f a rest

let char c : char parser = function
  | "" -> None
  | s when s.[0] = c -> Some (c, String.sub s 1 (String.length s - 1))
  | _ -> None
```

**Buran:**

```
# Parser returns [result, remaining] or []

pure {
    𝑥, input ↦ [[𝑥, input]]
}

bind {
    parser, f, input ↦
        parser(input) ↦ {
            [] ↦ []
            [[𝑎, rest]] ↦ f(𝑎, rest)
        }
}

char-parser {
    𝑐, [""] ↦ []
    𝑐, [string: 𝑐, ⟨rest⟩] ↦ [[𝑐, rest]]
    _, _ ↦ []
}
```

---

## What's Different (Summary)

Coming from OCaml, the main adjustments are:

1. **No currying** — Functions take all arguments at once
2. **Bracketed values** — Everything is `[value]`, not bare literals
3. **Identity-based types** — No compile-time type checking (yet)
4. **No modules** — File-based organization for now
5. **Different arrow** — `↦` instead of `->`
6. **Mathematical notation** — `×`, `÷`, `√`, `𝛑` instead of ASCII

## What's Familiar

1. **Pattern matching** — Core of both languages
2. **Algebraic data types** — Constructors work similarly
3. **Immutability** — Default in both
4. **Higher-order functions** — map, filter, fold
5. **Option/Result** — Similar patterns for absence/errors
6. **Recursion** — Primary iteration mechanism

---

## Philosophical Alignment

OCaml asks: *"What is the type of this expression?"*

Buran asks: *"What pattern does this pattern become?"*

Both languages value:

- Expressive pattern matching
- Algebraic data types
- Functional programming
- Type safety (achieved differently)

The transition from OCaml to Buran should feel natural. The syntax differs, but the thinking is the same: define data as types/patterns, operate on it through pattern matching, compose small functions into larger programs.

If you appreciate OCaml's expressiveness but want even more uniform syntax and mathematical notation, Buran offers a compelling evolution of the same ideas.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
