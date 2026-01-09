# Buran for Lisp Programmers

## Introduction

If you know Lisp, you understand that simple foundations yield expressive power. S-expressions, seven primitive operators, and everything else built from there. Code is data. Recursion is natural. Parentheses are beautiful.

Buran shares this philosophy of radical simplicity, but builds on a different foundation: patterns instead of S-expressions, transformation arrows instead of evaluation, and structural matching instead of conditional branching.

This document will help you transition from Lisp to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Philosophical Kinship

Both languages believe in one powerful concept applied uniformly:

| Lisp                          | Buran                                 |
| ----------------------------- | ------------------------------------- |
| Everything is an S-expression | Everything is a pattern               |
| Code is data (homoiconicity)  | Functions are pattern transformations |
| Seven primitives + macros     | Patterns + arrows                     |
| `(operator args...)`          | `[constructor: args...]`              |
| Lists as fundamental          | Patterns as fundamental               |

The shape is different, but the spirit is the same: minimal core, maximum expressiveness.

---

## S-Expressions → Patterns

**Lisp:**

```lisp
; Atom
42
"hello"
symbol

; List
(1 2 3)

; Nested
(point 3 4)
(rgb 255 0 0)

; Code (same syntax as data!)
(+ 1 2)
(defun factorial (n) ...)
```

**Buran:**

```
# Value patterns
[42]
["hello"]
[symbol]    # or just symbol when unambiguous

# List pattern
[list: 1, 2, 3]

# Constructor patterns
[point: 3, 4]
[rgb: 255, 0, 0]

# Function definition (different syntax from data)
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

Buran separates data patterns from function definitions, unlike Lisp's homoiconic syntax.

---

## Core Comparison

**Lisp:**

```lisp
(defun factorial (n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

Buran replaces `if`/`cond` with pattern matching. Each clause is a pattern-to-result transformation.

---

## Basic Syntax Correspondences

### Atoms and Values

**Lisp:**

```lisp
42          ; number
"hello"     ; string
'foo        ; symbol
#\a         ; character
t           ; true
nil         ; false/empty
```

**Buran:**

```
[42]        # number
["hello"]   # string
[foo]       # symbol (constructor with no args)
["a"]       # character (single-char string)
[✔]         # true
[✘]         # false
[]          # empty/nil
```

### Defining Functions

**Lisp:**

```lisp
(defun add (a b)
  (+ a b))

(defun greet (name)
  (concatenate 'string "Hello, " name "!"))
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

### Function Calls

**Lisp:**

```lisp
(add 2 3)
(greet "World")
```

**Buran:**

```
add([2], [3])
greet("World")
```

Prefix notation becomes function-call notation. Literal arguments need brackets.

---

## Lists: car/cdr/cons → Pattern Destructuring

**Lisp:**

```lisp
(car '(1 2 3))       ; 1
(cdr '(1 2 3))       ; (2 3)
(cons 0 '(1 2 3))    ; (0 1 2 3)
(null '())           ; t
(null '(1))          ; nil

(defun length (lst)
  (if (null lst)
      0
      (+ 1 (length (cdr lst)))))
```

**Buran:**

```
# Destructuring replaces car/cdr
[list: first, ⟨rest⟩] ↦ lst    # first = car, rest = cdr

# Construction replaces cons
[list: 0, ⟨lst⟩] ↦ prepended

# Pattern matching replaces null check
length {
    [list: ] ↦ [0]
    [list: _, ⟨rest⟩] ↦ [1 + length(rest)]
}
```

The `⟨...⟩` mirror brackets capture or spread remaining elements.

---

## Conditionals: if/cond → Pattern Matching

**Lisp:**

```lisp
(defun classify (n)
  (cond ((> n 0) "positive")
        ((< n 0) "negative")
        (t "zero")))

; Or with if
(defun abs (n)
  (if (< n 0)
      (- n)
      n))
```

**Buran:**

```
classify {
    [𝑛 | 𝑛 > 0] ↦ ["positive"]
    [𝑛 | 𝑛 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

abs {
    [𝑛 | 𝑛 < 0] ↦ [−𝑛]
    [𝑛] ↦ 𝑛
}
```

Guards (`|`) replace predicates in `cond`. Pattern order determines precedence.

---

## Lambda → Pattern Blocks

**Lisp:**

```lisp
(lambda (x) (* x 2))
(lambda (a b) (+ a b))

; Using with mapcar
(mapcar (lambda (x) (* x 2)) '(1 2 3))

; Multiple conditions
(lambda (x)
  (cond ((> x 0) 'positive)
        ((< x 0) 'negative)
        (t 'zero)))
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 }
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 }

# Using with map
map({ 𝑥 ↦ 𝑥 × 2 }, [list: 1, 2, 3])

# Multiple clauses (built-in)
{
    [𝑥 | 𝑥 > 0] ↦ [positive],
    [𝑥 | 𝑥 < 0] ↦ [negative],
    _ ↦ [zero]
}
```

Pattern blocks can have multiple clauses — like a lambda with built-in `cond`.

---

## Higher-Order Functions

**Lisp:**

```lisp
(mapcar #'1+ '(1 2 3))           ; (2 3 4)
(remove-if-not #'evenp '(1 2 3 4)) ; (2 4)
(reduce #'+ '(1 2 3 4) :initial-value 0) ; 10

; Function composition
(defun compose (f g)
  (lambda (x) (funcall f (funcall g x))))
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 + 1 }, [list: 1, 2, 3])     # [list: 2, 3, 4]
filter({ 𝑥 | 𝑥 mod 2 = 0 }, [list: 1, 2, 3, 4])  # [list: 2, 4]
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], [list: 1, 2, 3, 4])  # [10]

# Composition
compose {
    f, g ↦ { 𝑥 ↦ f(g(𝑥)) }
}
```

---

## let/let* → Sequential Binding

**Lisp:**

```lisp
(let ((x 1)
      (y 2))
  (+ x y))

(let* ((x 1)
       (y (+ x 1)))
  (* x y))
```

**Buran:**

```
# Sequential binding with arrows
[1] ↦ x ↦
[2] ↦ y ↦
[x + y]

# Each binding can use previous
[1] ↦ x ↦
[x + 1] ↦ y ↦
[x × y]
```

---

## Quoting → Not Needed

**Lisp:**

```lisp
'(1 2 3)           ; List, not function call
'(a b c)           ; List of symbols
`(1 2 ,x)          ; Quasiquote with unquote
```

**Buran:**

```
# All data is explicitly constructed
[list: 1, 2, 3]
[list: [a], [b], [c]]

# No quoting needed — patterns are always data
# Function calls require explicit ()
```

Buran's syntax distinguishes data patterns from function calls, so quoting is unnecessary.

---

## Macros → No Direct Equivalent

Lisp's macros transform code at compile time:

**Lisp:**

```lisp
(defmacro when (test &body body)
  `(if ,test (progn ,@body)))

(defmacro with-open-file ...)
```

Buran doesn't have macros. The uniform pattern syntax reduces the need for them:

**Buran:**

```
# "when" is just pattern matching
when {
    [✔], action ↦ action()
    [✘], _ ↦ []
}

# File handling is built-in
[File: "data.txt"] ↦ content ↦
process(content) ↦ result ↦
result ↦ [File: "output.txt"]
```

Pattern blocks provide some of the expressiveness that macros offer in Lisp.

---

## Recursion

Both languages embrace recursion as the primary iteration mechanism:

**Lisp:**

```lisp
(defun factorial (n)
  (if (= n 0)
      1
      (* n (factorial (- n 1)))))

(defun map-tree (f tree)
  (if (atom tree)
      (funcall f tree)
      (cons (map-tree f (car tree))
            (map-tree f (cdr tree)))))
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}

map-tree {
    f, [leaf: value] ↦ [leaf: f(value)]
    f, [node: left, right] ↦ [node: map-tree(f, left), map-tree(f, right)]
}
```

Pattern matching makes recursive structure explicit.

---

## Association Lists → Maps

**Lisp:**

```lisp
(defvar *person* '((name . "Alice") (age . 30)))
(assoc 'name *person*)  ; (NAME . "Alice")
(cdr (assoc 'name *person*))  ; "Alice"

(acons 'city "Boston" *person*)
```

**Buran:**

```
[map: "name", "Alice", "age", 30] ↦ person

# Access through pattern matching or lookup function
lookup("name", person) ↦ name

# Add entry (creates new map)
[map: ⟨person⟩, "city", "Boston"] ↦ person2
```

---

## Property Lists → Identity

**Lisp:**

```lisp
(setf (get 'foo 'color) 'red)
(setf (get 'foo 'size) 'large)
(get 'foo 'color)  ; RED
(symbol-plist 'foo)  ; (SIZE LARGE COLOR RED)
```

**Buran:**

```
# Identity provides metadata for patterns
⟨[foo]⟩ ↤ [identity:
    "color": "red",
    "size": "large"
]

# Query identity
⟨[foo]⟩ ↦ metadata
```

---

## Multiple Values → Patterns

**Lisp:**

```lisp
(defun divide-with-remainder (a b)
  (values (floor a b) (mod a b)))

(multiple-value-bind (quotient remainder)
    (divide-with-remainder 10 3)
  (format t "~a r ~a" quotient remainder))
```

**Buran:**

```
divide-with-remainder {
    𝑎, 𝑏 ↦ [[𝑎 ÷ 𝑏], [𝑎 mod 𝑏]]    # Return a list
}

# Destructure the result
divide-with-remainder([10], [3]) ↦ [quotient, remainder] ↦
[string: quotient, " r ", remainder] ↦ [stdout]
```

Return multiple values as a pattern; destructure to extract.

---

## Error Handling

**Lisp:**

```lisp
(handler-case
    (/ 10 0)
  (division-by-zero (c)
    (format t "Error: ~a" c)))

(defun safe-divide (a b)
  (if (zerop b)
      (error "Division by zero")
      (/ a b)))
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [0]) ↦ {
    [error: msg] ↦ [string: "Error: ", msg] ↦ [stdout]
    result ↦ use(result)
}
```

---

## Mathematical Notation

Lisp uses prefix notation. Buran uses infix mathematical notation:

| Lisp         | Buran         | Meaning               |
| ------------ | ------------- | --------------------- |
| `(* a b)`    | `[𝑎 × 𝑏]`     | Multiplication        |
| `(/ a b)`    | `[𝑎 ÷ 𝑏]`     | Division              |
| `(expt x 2)` | `[𝑥²]`        | Square                |
| `(sqrt x)`   | `[√𝑥]`        | Square root           |
| `pi`         | `𝛑`           | Pi                    |
| `(<= a b)`   | `[𝑎 ≤ 𝑏]`     | Less than or equal    |
| `(>= a b)`   | `[𝑎 ≥ 𝑏]`     | Greater than or equal |
| `(/= a b)`   | `[𝑎 ≠ 𝑏]`     | Not equal             |
| `(and a b)`  | `[𝑎 ∧ 𝑏]`     | Logical AND           |
| `(or a b)`   | `[𝑎 ∨ 𝑏]`     | Logical OR            |
| `(not a)`    | `[¬𝑎]`        | Logical NOT           |
| `t`          | `[✔]`         | True                  |
| `nil`        | `[]` or `[✘]` | False/empty           |

---

## Comparison Table

| Concept       | Lisp                      | Buran                      |
| ------------- | ------------------------- | -------------------------- |
| Core unit     | S-expression              | Pattern                    |
| Function def  | `(defun f (x) ...)`       | `f { pattern ↦ result }`   |
| Function call | `(f x y)`                 | `f(x, y)`                  |
| Lambda        | `(lambda (x) ...)`        | `{ x ↦ ... }`              |
| List          | `'(1 2 3)`                | `[list: 1, 2, 3]`          |
| car           | `(car lst)`               | `[list: first, ⟨_⟩] ↦ lst` |
| cdr           | `(cdr lst)`               | `[list: _, ⟨rest⟩] ↦ lst`  |
| cons          | `(cons x lst)`            | `[list: x, ⟨lst⟩]`         |
| Conditional   | `(cond ...)` / `(if ...)` | Pattern clauses            |
| Quote         | `'expr`                   | Not needed                 |
| Symbol        | `'foo`                    | `[foo]`                    |
| nil           | `nil`                     | `[]`                       |
| t             | `t`                       | `[✔]`                      |
| Macro         | `(defmacro ...)`          | None (pattern blocks)      |
| let           | `(let ((x 1)) ...)`       | `[1] ↦ x ↦ ...`            |
| setf          | `(setf x 1)`              | Immutable rebinding        |

---

## Example: Symbolic Differentiation

**Lisp:**

```lisp
(defun deriv (expr var)
  (cond ((numberp expr) 0)
        ((eq expr var) 1)
        ((eq (car expr) '+)
         (list '+ (deriv (cadr expr) var)
                  (deriv (caddr expr) var)))
        ((eq (car expr) '*)
         (list '+ (list '* (cadr expr)
                          (deriv (caddr expr) var))
                  (list '* (deriv (cadr expr) var)
                          (caddr expr))))))
```

**Buran:**

```
deriv {
    ⟨𝑛 | "type": "number"⟩, _ ↦ [0]
    var, var ↦ [1]
    [add: 𝑎, 𝑏], var ↦ [add: deriv(𝑎, var), deriv(𝑏, var)]
    [mul: 𝑎, 𝑏], var ↦ [add:
        [mul: 𝑎, deriv(𝑏, var)],
        [mul: deriv(𝑎, var), 𝑏]
    ]
}

# Example: d/dx (x * x) = x + x = 2x
deriv([mul: [x], [x]], [x])
```

---

## Example: Eval/Apply

**Lisp:**

```lisp
(defun my-eval (expr env)
  (cond ((numberp expr) expr)
        ((symbolp expr) (lookup expr env))
        ((eq (car expr) 'quote) (cadr expr))
        ((eq (car expr) 'if)
         (if (my-eval (cadr expr) env)
             (my-eval (caddr expr) env)
             (my-eval (cadddr expr) env)))
        (t (my-apply (car expr)
                     (mapcar (lambda (e) (my-eval e env))
                             (cdr expr))))))
```

**Buran:**

```
eval {
    ⟨𝑛 | "type": "number"⟩, _ ↦ 𝑛
    [var: name], env ↦ lookup(name, env)
    [quote: expr], _ ↦ expr
    [if: test, then, else], env ↦
        eval(test, env) ↦ {
            [✔] ↦ eval(then, env)
            [✘] ↦ eval(else, env)
        }
    [call: f, ⟨args⟩], env ↦
        apply(f, map({ arg ↦ eval(arg, env) }, args))
}
```

---

## What's Different

Coming from Lisp, the main adjustments:

1. **Not homoiconic** — Code and data have different syntax
2. **No macros** — Pattern blocks provide some abstraction
3. **Infix math** — `[𝑎 + 𝑏]` not `(+ a b)`
4. **Bracketed values** — `[42]` not bare `42`
5. **No quote** — Patterns are always data
6. **Pattern matching** — Replaces `cond`/`if`/`case`

## What's Familiar

1. **Minimal core** — One concept (patterns) for everything
2. **Recursion** — Primary iteration mechanism
3. **First-class functions** — Pattern blocks are lambdas
4. **Symbolic computation** — Patterns can represent any structure
5. **Dynamic typing** — Identity-based type checking
6. **Garbage collection** — Automatic memory management

---

## Philosophical Alignment

Lisp asks: *"What is the value of this expression?"*

Buran asks: *"What pattern does this pattern become?"*

Both languages value:

- Simple, uniform foundations
- Expressive power from minimal primitives
- Recursion as natural control flow
- Symbolic computation

Lisp achieves this through S-expressions and evaluation. Buran achieves it through patterns and transformation.

If you appreciate Lisp's elegance and power, Buran offers a different take on the same philosophy: a language where patterns replace S-expressions, and transformation replaces evaluation.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
