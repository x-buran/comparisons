# Buran for Raku Programmers

## Introduction

If you know Raku, you're already ahead of the curve. You understand grammars and pattern matching. You embrace Unicode operators and identifiers. You use rational numbers and sets as first-class types. You appreciate multi-dispatch.

Buran shares many of your values, distilled to a single principle: everything is a pattern, and all computation is pattern transformation.

This document will help you transition from Raku to Buran by showing correspondences between familiar constructs and new syntax.

---

## Philosophical Kinship

Raku and Buran share uncommon ground:

| Feature             | Raku                   | Buran                  |
| ------------------- | ---------------------- | ---------------------- |
| Unicode identifiers | ✓ `sub 階乗($n) { }`   | ✓ `階乗 { [𝑛] ↦ ... }` |
| Unicode operators   | ✓ `×`, `÷`, `∈`        | ✓ `×`, `÷`, `∈`        |
| Rational numbers    | ✓ `1/3 + 1/3` → `2/3`  | ✓ `[⅓ + ⅓]` → `[⅔]`    |
| Sets                | ✓ `set(1, 2, 3)`       | ✓ `[set: 1, 2, 3]`     |
| Pattern matching    | ✓ Grammars, signatures | ✓ Core paradigm        |
| Multi-dispatch      | ✓ `multi sub`          | ✓ Pattern clauses      |

The main difference: Raku adds these as features; Buran makes pattern transformation the *only* feature.

---

## Multi-Dispatch → Pattern Clauses

Raku's multi-dispatch maps directly to Buran's function clauses:

**Raku:**

```raku
multi sub factorial(0) { 1 }
multi sub factorial(Int $n where * > 0) {
    $n × factorial($n - 1)
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛 | 𝑛 > 0] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

Same concept, different syntax. Raku uses `multi sub` declarations; Buran uses pattern clauses in braces.

---

## Basic Syntax Correspondences

### Variables and Binding

**Raku:**

```raku
my $x = 42;
my $name = "hello";
my @items = 1, 2, 3;
my %hash = a => 1, b => 2;
my \bound = 42;  # Sigilless
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
[map: "a", 1, "b", 2] ↦ hash
```

Key differences:

- All values in square brackets
- No sigils ever — all bindings are sigilless
- Arrow `↦` indicates data flow

### Functions/Subroutines

**Raku:**

```raku
sub add($a, $b) { $a + $b }

sub greet($name) {
    "Hello, $name!"
}
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

No `sub`, no `{ }` for function body — functions *are* pattern matching blocks.

### Function Calls

**Raku:**

```raku
my $sum = add(2, 3);
my $msg = greet("World");

# Method call syntax
@items.map(* × 2);
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg

# No method syntax — use function calls
map({ 𝑥 ↦ 𝑥 × 2 }, items)
```

Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Grammars → Structural Patterns

Raku's grammars match text. Buran's patterns match structure:

**Raku:**

```raku
grammar Calculator {
    rule TOP { <expr> }
    rule expr { <term> [ <op> <term> ]* }
    token term { \d+ }
    token op { '+' | '-' | '*' | '/' }
}

my $match = Calculator.parse("2 + 3 * 4");
```

**Buran:**

```
# Structural pattern for AST
eval {
    [num: 𝑛] ↦ 𝑛
    [add: 𝑎, 𝑏] ↦ [eval(𝑎) + eval(𝑏)]
    [mul: 𝑎, 𝑏] ↦ [eval(𝑎) × eval(𝑏)]
}

# Apply to parsed structure
eval([add: [num: 2], [mul: [num: 3], [num: 4]]]) ↦ result  # [14]
```

Raku excels at parsing text into structure. Buran excels at transforming structure. They're complementary.

---

## Signatures with Constraints → Guards

**Raku:**

```raku
sub process(Int $n where * > 0) { ... }
sub classify(Numeric $x where { $_ > 0 }) { "positive" }
sub classify(Numeric $x where { $_ < 0 }) { "negative" }
sub classify(0) { "zero" }
```

**Buran:**

```
process {
    [𝑛 | 𝑛 > 0] ↦ ...
}

classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

Guards use `|` instead of `where`, but the concept is the same.

---

## Junctions → No Equivalent

Raku's junctions are unique:

**Raku:**

```raku
my $x = 1 | 2 | 3;          # any junction
say "yes" if $input == any(1, 2, 3);
say "all positive" if all(@nums) > 0;
```

Buran doesn't have junctions. Use explicit conditions:

**Buran:**

```
# Check membership
check {
    input | input ∈ [set: 1, 2, 3] ↦ ["yes"]
}

# Check all positive
all-positive {
    list ↦ fold({ acc, 𝑥 ↦ [acc ∧ (𝑥 > 0)] }, [✔], list)
}
```

---

## Whatever Star → Pattern Blocks

**Raku:**

```raku
@nums.map(* × 2);           # Whatever star
@nums.map({ $_ × 2 });      # Block
@nums.map(-> $x { $x × 2 }); # Pointy block
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, nums)    # One syntax for all
```

No Whatever star, no `$_`, no pointy blocks. Just pattern blocks.

---

## Rational Numbers

Both languages have built-in rationals:

**Raku:**

```raku
my $half = 1/2;
my $third = 1/3;
say $half + $third;  # 5/6
say (1/3).nude;      # (1 3) — numerator, denominator
```

**Buran:**

```
[½] ↦ half
[⅓] ↦ third
[half + third] ↦ result    # [⅚]

# Unicode fraction literals
[½], [⅓], [¼], [⅔], [¾], [⅕], [⅖], [⅗], [⅘], [⅙], [⅚]
```

Buran uses Unicode fraction characters directly.

---

## Sets, Bags, Mixes

**Raku:**

```raku
my $set = set(1, 2, 3);
my $bag = bag(1, 1, 2, 3, 3, 3);
say 2 ∈ $set;           # True
say $set ∪ set(4, 5);   # set(1, 2, 3, 4, 5)
say $set ∩ set(2, 3, 4); # set(2, 3)
```

**Buran:**

```
[set: 1, 2, 3] ↦ s
# Bags not yet specified

[2 ∈ s] ↦ has-two           # [✔]
[s ∪ [set: 4, 5]] ↦ union   # [set: 1, 2, 3, 4, 5]
[s ∩ [set: 2, 3, 4]] ↦ intersection  # [set: 2, 3]
```

Same operators! Both use `∈`, `∪`, `∩`, `⊂`, `⊆`.

---

## Unicode Operators

Raku pioneered Unicode operators in mainstream languages. Buran follows:

| Operation     | Raku            | Buran |
| ------------- | --------------- | ----- |
| Multiply      | `×` or `*`      | `×`   |
| Divide        | `÷` or `/`      | `÷`   |
| Not equal     | `≠` or `!=`     | `≠`   |
| Less/equal    | `≤` or `<=`     | `≤`   |
| Greater/equal | `≥` or `>=`     | `≥`   |
| Element of    | `∈` or `(elem)` | `∈`   |
| Union         | `∪` or `(|)`    | `∪`   |
| Intersection  | `∩` or `(&)`    | `∩`   |
| Subset        | `⊂` or `(<)`    | `⊂`   |
| Pi            | `π`             | `𝛑`   |
| And           | `&&`            | `∧`   |
| Or            | `\|\|`          | `∨`   |
| Not           | `!`             | `¬`   |

Buran uses *only* the Unicode forms — no ASCII fallbacks.

---

## Unicode Identifiers

Both languages embrace Unicode names:

**Raku:**

```raku
sub 階乗(Int $n) {
    $n == 0 ?? 1 !! $n × 階乗($n - 1)
}

my $φ = (1 + sqrt(5)) / 2;  # Golden ratio
my $Δx = $x₂ - $x₁;
```

**Buran:**

```
階乗 {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × 階乗(𝑛 − 1)]
}

𝛗 ↦ golden-ratio    # Built-in constant
[𝑥₂ − 𝑥₁] ↦ Δx
```

Both use mathematical italic for variables, subscripts, Greek letters.

---

## Hyper Operators → Map

**Raku:**

```raku
my @doubled = @nums »×» 2;
my @sums = @a »+« @b;
my @negated = -« @nums;
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, nums) ↦ doubled
zip-with({ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 }, a, b) ↦ sums
map({ 𝑥 ↦ −𝑥 }, nums) ↦ negated
```

No special operators — use explicit higher-order functions.

---

## Lazy Lists and Sequences

**Raku:**

```raku
my @infinite = 1, 2, 4 ... *;        # Geometric
my @fibs = 0, 1, *+* ... *;          # Fibonacci
my @first-ten = @infinite[^10];
```

**Buran:**

```
# Laziness not yet specified
# Define sequences through functions

fib {
    [0] ↦ [0]
    [1] ↦ [1]
    [𝑛] ↦ [fib(𝑛 − 1) + fib(𝑛 − 2)]
}

# Generate list explicitly
map({ 𝑛 ↦ fib(𝑛) }, range([0], [9])) ↦ first-ten-fibs
```

---

## Gradual Typing → Identity-Based Types

**Raku:**

```raku
sub add(Int $a, Int $b --> Int) {
    $a + $b
}

# Gradual — types optional
sub process($data) { ... }
```

**Buran:**

```
# Types through identity guards
add {
    ⟨𝑎 | "type": "integer"⟩, ⟨𝑏 | "type": "integer"⟩ ↦ [𝑎 + 𝑏]
}

# Or simply untyped
process {
    data ↦ ...
}
```

Both languages allow optional typing. Raku uses annotations; Buran uses identity guards.

### Defining Custom Types

**Raku:**

```raku
class Point {
    has Numeric $.x;
    has Numeric $.y;
}

my $p = Point.new(x => 3, y => 4);
```

**Buran:**

```
⟨[point: _, _]⟩ ↤ [identity:
    "type": "Point",
    "fields": ["x", "y"]
]

[point: 3, 4] ↦ p
```

---

## Roles → Pattern Matching

**Raku:**

```raku
role Drawable {
    method draw() { ... }
}

class Circle does Drawable {
    has $.radius;
    method draw() { say "Drawing circle" }
}

class Square does Drawable {
    has $.side;
    method draw() { say "Drawing square" }
}
```

**Buran:**

```
# No roles — pattern matching handles polymorphism
draw {
    [circle: 𝑟] ↦ ["Drawing circle"] ↦ [stdout]
    [square: 𝑠] ↦ ["Drawing square"] ↦ [stdout]
}
```

Pattern matching on structure replaces role-based dispatch.

---

## Blocks and Closures

**Raku:**

```raku
my &double = -> $x { $x × 2 };
my &add = { $^a + $^b };       # Placeholder variables
my &greet = { "Hello, $_!" }; # Topic variable
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 } ↦ double
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ add
{ name ↦ [string: "Hello, ", name, "!"] } ↦ greet
```

No placeholder variables (`$^a`), no topic (`$_`). Explicit naming only.

---

## Given/When → Pattern Matching

**Raku:**

```raku
given $value {
    when 0 { say "zero" }
    when * > 0 { say "positive" }
    when * < 0 { say "negative" }
}
```

**Buran:**

```
value ↦ {
    [0] ↦ ["zero"]
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
} ↦ [stdout]
```

No `given`/`when` — inline pattern blocks handle matching.

---

## Error Handling

**Raku:**

```raku
sub divide($a, $b) {
    die "Division by zero" if $b == 0;
    $a / $b
}

try {
    my $result = divide(10, 0);
    CATCH {
        default { say "Error: $_" }
    }
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [0]) ↦ {
    [error: msg] ↦ [string: "Error: ", msg] ↦ [stdout]
    result ↦ process(result)
}
```

No exceptions — errors are patterns to match.

---

## File I/O

**Raku:**

```raku
my $content = "data.txt".IO.slurp;
"output.txt".IO.spurt($data);

use JSON::Fast;
my $data = from-json($json-str);
my $json = to-json($data);
```

**Buran:**

```
[File: "data.txt"] ↦ content
data ↦ [File: "output.txt"]

# JSON built-in
[File: "config.json" :: json] ↦ data
data ↦ [File: "output.json" :: json]
```

---

## Promises and Supplies → Not Yet Specified

**Raku:**

```raku
my $promise = start { long-computation() };
my $result = await $promise;

my $supply = Supply.interval(1);
$supply.tap: { say "tick" };
```

Buran's concurrency model is not yet specified.

---

## Comparison Table

| Concept         | Raku                 | Buran                      |
| --------------- | -------------------- | -------------------------- |
| Function        | `sub f($x) { }`      | `f { pattern ↦ result }`   |
| Multi           | `multi sub f(0) { }` | Clauses in single function |
| Guard           | `where * > 0`        | `\| 𝑥 > 0`                 |
| Block           | `{ }`, `-> { }`      | `{ pattern ↦ result }`     |
| Whatever        | `*`                  | Not available              |
| Topic           | `$_`                 | Not available              |
| Junction        | `1 \| 2 \| 3`        | Not available              |
| List            | `(1, 2, 3)`          | `[list: 1, 2, 3]`          |
| Set             | `set(1, 2, 3)`       | `[set: 1, 2, 3]`           |
| Hash            | `%( a => 1 )`        | `[map: "a", 1]`            |
| Rational        | `1/3`                | `[⅓]` or `[1⁄3]`           |
| True/False      | `True`, `False`      | `[✔]`, `[✘]`               |
| Nil             | `Nil`                | `[]`                       |
| Error           | `die`, `CATCH`       | `[error: ...]` patterns    |
| Type constraint | `Int $x`             | `⟨𝑥 \| "type": "integer"⟩` |
| Grammar         | `grammar { }`        | Structural patterns        |

---

## Example: Tree Operations

**Raku:**

```raku
role Tree { }
class Leaf does Tree { has $.value }
class Node does Tree { has Tree $.left; has Tree $.right }

multi sub sum(Leaf $l) { $l.value }
multi sub sum(Node $n) { sum($n.left) + sum($n.right) }

multi sub map-tree(&f, Leaf $l) { Leaf.new(value => f($l.value)) }
multi sub map-tree(&f, Node $n) {
    Node.new(left => map-tree(&f, $n.left),
             right => map-tree(&f, $n.right))
}
```

**Buran:**

```
sum {
    [leaf: 𝑣] ↦ 𝑣
    [node: left, right] ↦ [sum(left) + sum(right)]
}

tree-map {
    f, [leaf: 𝑣] ↦ [leaf: f(𝑣)]
    f, [node: l, r] ↦ [node: tree-map(f, l), tree-map(f, r)]
}
```

---

## Example: Expression Evaluator

**Raku:**

```raku
role Expr { }
class Num does Expr { has $.value }
class Add does Expr { has Expr $.left; has Expr $.right }
class Mul does Expr { has Expr $.left; has Expr $.right }

multi sub eval(Num $n) { $n.value }
multi sub eval(Add $a) { eval($a.left) + eval($a.right) }
multi sub eval(Mul $m) { eval($m.left) × eval($m.right) }
```

**Buran:**

```
eval {
    [num: 𝑛] ↦ 𝑛
    [add: 𝑎, 𝑏] ↦ [eval(𝑎) + eval(𝑏)]
    [mul: 𝑎, 𝑏] ↦ [eval(𝑎) × eval(𝑏)]
}
```

---

## Example: Parser Actions

**Raku:**

```raku
grammar Arithmetic {
    rule TOP { <expr> }
    rule expr { <term> [ '+' <term> ]* }
    rule term { <factor> [ '*' <factor> ]* }
    token factor { \d+ }
}

class ArithmeticActions {
    method TOP($/) { make $<expr>.made }
    method expr($/) {
        make [+] $<term>».made
    }
    method term($/) {
        make [×] $<factor>».made
    }
    method factor($/) { make +$/ }
}
```

**Buran:**

```
# Assumes parsing produces AST, then evaluate
eval {
    [num: 𝑛] ↦ 𝑛
    [add: terms] ↦ fold({ acc, t ↦ acc + eval(t) }, [0], terms)
    [mul: factors] ↦ fold({ acc, f ↦ acc × eval(f) }, [1], factors)
}
```

Buran focuses on transforming the parsed structure, not the parsing itself.

---

## What's Missing from Buran

Coming from Raku, you'll notice:

1. **No grammars** — Buran isn't designed for parsing text
2. **No junctions** — Use explicit set operations
3. **No Whatever star** — Use explicit pattern blocks
4. **No hyper operators** — Use map/zip-with
5. **No lazy sequences** — Not yet specified
6. **No concurrency** — Not yet specified
7. **No slangs** — No language extensibility

## What's Familiar

1. **Multi-dispatch** — Pattern clauses work the same way
2. **Unicode everything** — Operators, identifiers, fractions
3. **Rationals** — Built-in exact arithmetic
4. **Sets** — Same operators: `∈`, `∪`, `∩`
5. **Gradual typing** — Optional type constraints
6. **Functional style** — map, filter, fold

---

## Philosophical Alignment

Raku asks: *"How can we make programming more expressive and human-friendly?"*

Buran asks: *"What if pattern transformation was the only concept?"*

Both languages value:

- Unicode as first-class
- Mathematical notation
- Expressive, readable code
- Pattern matching as core feature

Raku achieves expressiveness through feature richness — many ways to do things beautifully. Buran achieves expressiveness through feature minimalism — one way to do everything.

If you appreciate Raku's embrace of Unicode and patterns but want something more uniform, Buran offers a distilled essence of similar ideas.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
