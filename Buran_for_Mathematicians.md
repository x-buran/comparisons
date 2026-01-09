# Buran for Mathematicians

## Introduction

If you work with mathematics — whether you're a pure mathematician, applied mathematician, physicist, or researcher in a mathematical field — you've likely been frustrated when trying to express your ideas on a computer. Traditional programming languages force you to translate elegant mathematical notation into obscure symbols. Variables become `x1`, `x2` instead of 𝑥₁, 𝑥₂. Multiplication becomes `*`. Your beautiful equations become unreadable code.

Buran was designed differently. It uses mathematical notation directly, follows ISO 80000-2 typographic standards, and treats computation the way mathematicians think about it: as transformation of structured expressions according to rules.

This document shows how mathematical concepts you already know map directly to Buran.

---

## Your Notation Works Here

### Standard Mathematical Symbols

Buran uses the symbols you write on paper:

| What you write | Buran     | Not this        |
| -------------- | --------- | --------------- |
| 𝑥 × 𝑦          | `[𝑥 × 𝑦]` | `x * y`         |
| 𝑎 ÷ 𝑏          | `[𝑎 ÷ 𝑏]` | `a / b`         |
| √𝑥             | `[√𝑥]`    | `sqrt(x)`       |
| 𝑥²             | `[𝑥²]`    | `x^2` or `x**2` |
| 𝑥⁻¹            | `[𝑥⁻¹]`   | `1/x`           |
| ∛27            | `[∛27]`   | `cbrt(27)`      |
| ⌊𝑥⌋            | `[⌊𝑥⌋]`   | `floor(x)`      |
| ⌈𝑥⌉            | `[⌈𝑥⌉]`   | `ceil(x)`       |
| \|𝑥\|          | `[\|𝑥\|]` | `abs(x)`        |
| 𝑛!             | `[𝑛!]`    | `factorial(n)`  |

### Constants Are Written Correctly

Mathematical constants use proper typography (bold upright per ISO 80000-2):

```
𝛑     # Pi — not PI or Math.PI
𝐞     # Euler's number — not E or Math.E
𝐢     # Imaginary unit — not i or 1j
𝛗     # Golden ratio — not phi or (1+sqrt(5))/2
∞     # Infinity — not inf or INFINITY
```

### Variables Look Like Variables

Buran distinguishes variables from constants through typography, just as you do on paper:

```
𝑥, 𝑦, 𝑧        # Italic — scalar variables
𝛑, 𝐞, 𝐢        # Bold upright — constants
𝒗, 𝒂, 𝑭        # Bold italic — vectors
𝑨, 𝑩, 𝑴        # Bold italic capitals — matrices
```

This means you can write `sin` as a function and `𝑠𝑖𝑛` as a variable storing a sine value, with no conflict.

---

## Functions Are Pattern Definitions

In mathematics, you define functions by cases:

$$
f(x) = \begin{cases}
1 & \text{if } x = 0 \\
x \cdot f(x-1) & \text{otherwise}
\end{cases}
$$

In Buran, this is written directly:

```
𝑓 {
    [0] ↦ [1]
    [𝑥] ↦ [𝑥 × 𝑓(𝑥 − 1)]
}
```

The arrow `↦` (maps to) shows transformation — exactly as you'd write "↦" in a function definition. Each line is a case. The first matching case applies.

### More Examples

**Absolute value:**

$$|x| = \begin{cases} x & \text{if } x \geq 0 \\ -x & \text{if } x < 0 \end{cases}$$

```
abs {
    [𝑥 | 𝑥 ≥ 0] ↦ 𝑥
    [𝑥] ↦ [−𝑥]
}
```

The `|` introduces a condition (a guard), just as "if" does in case notation.

**Sign function:**

$$\text{sgn}(x) = \begin{cases} 1 & \text{if } x > 0 \\ 0 & \text{if } x = 0 \\ -1 & \text{if } x < 0 \end{cases}$$

```
sgn {
    [𝑥 | 𝑥 > 0] ↦ [1]
    [0] ↦ [0]
    [𝑥] ↦ [−1]
}
```

**Fibonacci:**

$$F_n = \begin{cases} 0 & \text{if } n = 0 \\ 1 & \text{if } n = 1 \\ F_{n-1} + F_{n-2} & \text{otherwise} \end{cases}$$

```
fib {
    [0] ↦ [0]
    [1] ↦ [1]
    [𝑛] ↦ [fib(𝑛 − 1) + fib(𝑛 − 2)]
}
```

---

## Pattern Matching Is Case Analysis

You use case analysis constantly in mathematics:

- "Consider two cases: 𝑛 even or 𝑛 odd"
- "If 𝑥 ∈ 𝐴, then... If 𝑥 ∉ 𝐴, then..."
- "For the base case 𝑛 = 0... For the inductive case..."

This is exactly what Buran's pattern matching does. Each clause handles one case.

### Structural Patterns

When working with structured objects (pairs, tuples, sequences), you can decompose them directly:

**First element of a pair:**

$$\pi_1(a, b) = a$$

```
first {
    [pair: 𝑎, 𝑏] ↦ 𝑎
}
```

**Sum of a sequence:**

$$\sum_{i} a_i = \begin{cases} 0 & \text{if sequence is empty} \\ a_1 + \sum_{i>1} a_i & \text{otherwise} \end{cases}$$

```
sum {
    [list: ] ↦ [0]
    [list: 𝑎, ⟨rest⟩] ↦ [𝑎 + sum(rest)]
}
```

Here `⟨rest⟩` captures the remaining elements — like "the tail of the sequence."

---

## Types Are Sets

Mathematicians think in terms of sets. A function 𝑓: ℝ → ℝ takes real numbers to real numbers. In Buran, types work similarly.

### Built-in Types

Buran recognizes standard mathematical sets:

```
ℕ    # Natural numbers {0, 1, 2, 3, ...}
ℤ    # Integers {..., −2, −1, 0, 1, 2, ...}
ℚ    # Rational numbers
ℝ    # Real numbers
ℂ    # Complex numbers
```

### Type Checking

You can constrain patterns to certain types:

```
square-root {
    ⟨𝑥 | "type": "real"⟩ | 𝑥 ≥ 0 ↦ [√𝑥]
    ⟨𝑥 | "type": "real"⟩ | 𝑥 < 0 ↦ [error: domain error]
}
```

Or work with complex numbers when needed:

```
[√(−1) :: complex] ↦ [𝐢]
```

### Defining New Types

Create types as you would define sets with structure:

```
# Define points in ℝ²
⟨[point: 𝑥, 𝑦]⟩ ↤ [identity: "type": "Point2D"]

# Define complex numbers as pairs
⟨[complex: 𝑎, 𝑏]⟩ ↤ [identity: "type": "Complex"]
```

---

## Sets and Set Operations

Buran supports standard set notation:

```
# Set construction
[set: 1, 2, 3, 4, 5]

# Membership
[3 ∈ [set: 1, 2, 3]]     # True
[7 ∉ [set: 1, 2, 3]]     # True

# Operations
[𝐴 ∪ 𝐵]    # Union
[𝐴 ∩ 𝐵]    # Intersection
[𝐴 ∖ 𝐵]    # Set difference
[𝐴 ⊂ 𝐵]    # Proper subset
[𝐴 ⊆ 𝐵]    # Subset or equal
[∅]        # Empty set
```

---

## Logic and Propositions

Boolean operations use standard logical symbols:

```
[𝑝 ∧ 𝑞]    # Conjunction (AND)
[𝑝 ∨ 𝑞]    # Disjunction (OR)
[¬𝑝]       # Negation (NOT)
[𝑝 → 𝑞]    # Implication
[𝑝 ↔ 𝑞]    # Biconditional (if and only if)
[𝑝 ⊕ 𝑞]    # Exclusive or (XOR)
```

Truth values are written clearly:

```
[✔]    # True
[✘]    # False
```

---

## Rational Numbers Are Exact

Unlike most computer systems that approximate everything with floating-point, Buran preserves exact rational arithmetic:

```
[1⁄3 + 1⁄3 + 1⁄3] ↦ [1]      # Exactly 1, not 0.9999...
[½ × ⅔] ↦ [⅓]                # Exact
[22⁄7]                        # Stored as 22/7, not 3.14285...
```

Buran uses Unicode fraction characters when available:

```
[½]    # One half
[⅓]    # One third
[¾]    # Three quarters
[⅝]    # Five eighths
```

For fractions without special characters, it uses the fraction slash:

```
[7⁄11]    # Seven elevenths
```

To convert to decimal approximation when needed:

```
[real: 1⁄3] ↦ [0.333...]
```

---

## Comparisons and Relations

All standard comparison operators:

```
[𝑥 = 𝑦]     # Equality
[𝑥 ≠ 𝑦]     # Inequality
[𝑥 < 𝑦]     # Less than
[𝑥 > 𝑦]     # Greater than
[𝑥 ≤ 𝑦]     # Less than or equal
[𝑥 ≥ 𝑦]     # Greater than or equal
[𝑥 ≈ 𝑦]     # Approximately equal
[𝑥 ≪ 𝑦]     # Much less than
[𝑥 ≫ 𝑦]     # Much greater than
[𝑥 ∝ 𝑦]     # Proportional to
```

Divisibility:

```
[6 ∣ 18]    # 6 divides 18 (true)
[5 ∤ 18]    # 5 does not divide 18 (true)
```

---

## Sums and Products

Standard notation for sums and products:

```
[∑(𝑖=1 to 𝑛) 𝑖] ↦ [𝑛(𝑛+1)⁄2]     # Sum formula
[∏(𝑖=1 to 𝑛) 𝑖] ↦ [𝑛!]           # Product is factorial
```

Sequences with ellipsis:

```
[1, 2, 3, …, 10]    # Sequence from 1 to 10
```

---

## Mathematical Domains

Different branches of mathematics require different interpretations. Buran uses domain annotations to specify context:

### Basic Arithmetic (Default)

No annotation needed:

```
[2 + 3] ↦ [5]
[√16] ↦ [4]
[3!] ↦ [6]
```

### Complex Numbers

```
[𝐢²] ↦ [−1]
[𝐞^(𝐢𝛑) + 1] ↦ [0]              # Euler's identity
[√(−4) :: complex] ↦ [2𝐢]
```

### Matrix Algebra

```
[𝑨𝑩 :: matrix]                   # Matrix multiplication
[det(𝑨) :: matrix]               # Determinant
[𝑨⁻¹ :: matrix]                  # Matrix inverse
[𝑨ᵀ :: matrix]                   # Transpose
```

### Symbolic Computation

```
[d⁄d𝑥(𝑥²) :: symbolic] ↦ [2𝑥]
[∫ 𝑥² d𝑥 :: symbolic] ↦ [𝑥³⁄3 + 𝐶]
```

### Modular Arithmetic

```
[17 mod 5 :: modular] ↦ [2]
[3⁻¹ mod 7 :: modular] ↦ [5]     # Modular inverse
```

---

## Trigonometric Functions

Standard trigonometric functions work naturally:

```
[sin(𝛑⁄4)] ↦ [√2⁄2]
[cos(0)] ↦ [1]
[tan(𝛑⁄4)] ↦ [1]
```

Degrees are supported:

```
[sin(45°)] ↦ [√2⁄2]
[90°] ↦ [𝛑⁄2]              # Converts to radians
[30° 15′ 45″]              # Degrees, minutes, seconds
```

---

## Examples from Pure Mathematics

### GCD by Euclidean Algorithm

$$\gcd(a, b) = \begin{cases} a & \text{if } b = 0 \\ \gcd(b, a \mod b) & \text{otherwise} \end{cases}$$

```
gcd {
    𝑎, [0] ↦ 𝑎
    𝑎, 𝑏 ↦ gcd(𝑏, [𝑎 mod 𝑏])
}
```

### Prime Checking

```
is-prime {
    [𝑛 | 𝑛 < 2] ↦ [✘]
    [2] ↦ [✔]
    [𝑛 | 𝑛 mod 2 = 0] ↦ [✘]
    [𝑛] ↦ check-divisors(𝑛, [3])
}

check-divisors {
    𝑛, [𝑑 | 𝑑² > 𝑛] ↦ [✔]
    𝑛, [𝑑 | 𝑛 mod 𝑑 = 0] ↦ [✘]
    𝑛, 𝑑 ↦ check-divisors(𝑛, [𝑑 + 2])
}
```

### Binomial Coefficient

$$\binom{n}{k} = \begin{cases} 1 & \text{if } k = 0 \text{ or } k = n \\ \binom{n-1}{k-1} + \binom{n-1}{k} & \text{otherwise} \end{cases}$$

```
binomial {
    𝑛, [0] ↦ [1]
    𝑛, 𝑘 | 𝑛 = 𝑘 ↦ [1]
    𝑛, 𝑘 ↦ [binomial(𝑛 − 1, 𝑘 − 1) + binomial(𝑛 − 1, 𝑘)]
}
```

### Collatz Sequence

$$
a_{n+1} = \begin{cases}
a_n / 2 & \text{if } a_n \text{ is even} \\
3a_n + 1 & \text{if } a_n \text{ is odd}
\end{cases}
$$

```
collatz-step {
    [𝑛 | 𝑛 mod 2 = 0] ↦ [𝑛 ÷ 2]
    [𝑛] ↦ [3 × 𝑛 + 1]
}

collatz-sequence {
    [1] ↦ [list: 1]
    [𝑛] ↦ [list: 𝑛, ⟨collatz-sequence(collatz-step([𝑛]))⟩]
}
```

---

## Working with Sequences

### Map (Apply Function to Each Element)

Given 𝑓 and sequence (𝑎₁, 𝑎₂, ..., 𝑎ₙ), produce (𝑓(𝑎₁), 𝑓(𝑎₂), ..., 𝑓(𝑎ₙ)):

```
# Square each element
map({ 𝑥 ↦ 𝑥² }, [list: 1, 2, 3, 4]) ↦ [list: 1, 4, 9, 16]
```

### Filter (Select Elements Satisfying Predicate)

```
# Keep only positive
filter({ 𝑥 | 𝑥 > 0 }, [list: −2, 3, −1, 5]) ↦ [list: 3, 5]
```

### Fold (Reduce Sequence to Single Value)

```
# Sum via fold
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], [list: 1, 2, 3, 4]) ↦ [10]

# Product via fold
fold({ acc, 𝑥 ↦ acc × 𝑥 }, [1], [list: 1, 2, 3, 4]) ↦ [24]
```

---

## Error Handling (No Exceptions)

Mathematics has undefined operations: division by zero, square root of negative reals, logarithm of non-positive numbers. Buran handles these through error patterns:

```
safe-divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

safe-sqrt {
    [𝑥 | 𝑥 < 0] ↦ [error: negative argument]
    [𝑥] ↦ [√𝑥]
}
```

You can match error patterns to handle them:

```
result ↦ {
    [error: msg] ↦ handle-error(msg)
    value ↦ use-value(value)
}
```

---

## No Undefined, No Null

Programming languages have "null", "undefined", "None" — values that aren't values. This causes endless confusion.

Buran has one pattern for "nothing": `[]`

```
find-first-even {
    [list: ] ↦ []                        # Nothing found
    [list: 𝑥, ⟨rest⟩ | 𝑥 mod 2 = 0] ↦ 𝑥  # Found it
    [list: _, ⟨rest⟩] ↦ find-first-even(rest)
}
```

The empty pattern `[]` is explicit — you can match it, check for it, and handle it clearly.

---

## Reading and Writing

### Reading from Files

```
[File: "data/values.txt"] ↦ content
```

### Structured Data

```
[File: "data/matrix.json" :: json] ↦ matrix-data
```

### Writing Results

```
result ↦ [File: "output/answer.txt"]
```

---

## What Buran Offers Mathematicians

### Direct Notation

Write 𝛑, √, ×, ≤, ∈, ∑ — not `pi`, `sqrt`, `*`, `<=`, `in`, `sum`.

### Exact Arithmetic

Rationals stay rational. ⅓ + ⅓ + ⅓ = 1 exactly.

### Case-Based Definition

Define functions by cases as you do in mathematical writing.

### Type Discipline

Types are sets. Functions have domains and codomains.

### Multiple Domains

Matrix algebra, complex analysis, symbolic computation — each has appropriate notation and semantics.

### No Imperative Baggage

No loops, no mutation, no assignments. Just transformation of expressions.

---

## What Buran Does Not Offer (Yet)

**Proof assistants**: Buran is for computation, not formal proof. For proof, consider Lean, Coq, or Isabelle.

**Computer algebra systems**: Full symbolic computation (factoring, solving equations) requires domain-specific libraries that are still in development.

**Numerical methods**: High-performance numerical computing requires optimization that Buran's implementation is still developing.

---

## Complete Example: Polynomial Evaluation

Evaluate polynomial 𝑎ₙ𝑥ⁿ + 𝑎ₙ₋₁𝑥ⁿ⁻¹ + ... + 𝑎₁𝑥 + 𝑎₀ at point 𝑥 using Horner's method:

$$P(x) = a_0 + x(a_1 + x(a_2 + x(...)))$$

```
# Coefficients given as [a₀, a₁, a₂, ..., aₙ]
horner {
    [list: ], 𝑥 ↦ [0]
    [list: 𝑎], 𝑥 ↦ 𝑎
    coeffs, 𝑥 ↦
        reverse(coeffs) ↦ reversed ↦
        fold({ acc, 𝑎 ↦ 𝑎 + 𝑥 × acc }, [0], reversed)
}

# Evaluate 3x² + 2x + 1 at x = 5
# Coefficients: [1, 2, 3] (constant term first)
horner([list: 1, 2, 3], [5]) ↦ [86]    # 3(25) + 2(5) + 1 = 86
```

---

## Summary

Buran lets you write mathematics as mathematics. The syntax is minimal, the notation is standard, and computation happens through pattern transformation — the same case analysis you use in proofs and definitions.

No learning a programming language. Just expressing mathematical ideas in a form a computer can evaluate.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
