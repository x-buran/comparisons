# Buran for HyperTalk Programmers

## Introduction

If you programmed in HyperTalk — the language of HyperCard — you experienced something special: a programming language designed for humans first. HyperTalk read like English. You could `put 5 into x` and `repeat with i = 1 to 10`. Non-programmers could write useful scripts because the language met them where they were.

Buran shares HyperTalk's philosophy of accessibility, but takes a different path. Where HyperTalk used English-like syntax, Buran uses mathematical notation that humans already know. Where HyperTalk had message passing and handlers, Buran has pattern matching and transformation. Both languages reject unnecessary complexity. Both believe programming should be readable.

This guide shows how your HyperTalk intuitions translate to Buran.

---

## The Philosophy Connection

HyperTalk and Buran share core beliefs:

| Principle          | HyperTalk                  | Buran                        |
| ------------------ | -------------------------- | ---------------------------- |
| Readability        | English-like syntax        | Mathematical notation        |
| Accessibility      | Non-programmers welcome    | Non-programmers welcome      |
| Simplicity         | One obvious way            | One way per concept          |
| No cryptic symbols | `put x into y` not `y = x` | `×` not `*`, `÷` not `/`     |
| Clear structure    | Handlers and messages      | Patterns and transformations |

Both languages trust users to understand clear notation — HyperTalk trusted English, Buran trusts mathematics.

---

## Variables and Assignment

### HyperTalk

```hypertalk
put 5 into x
put "hello" into greeting
add 3 to counter
put x * 2 into doubled
```

### Buran

```
[5] ↦ x
["hello"] ↦ greeting
[counter + 3] ↦ counter    # But note: Buran prefers immutability
[x × 2] ↦ doubled
```

The arrow `↦` shows data flow, similar to HyperTalk's `into`. But there's a key difference: Buran values are immutable. Instead of modifying variables, you create new bindings.

**HyperTalk style (mutation):**

```hypertalk
put 0 into total
repeat with i = 1 to 10
  add i to total
end repeat
```

**Buran style (transformation):**

```
fold({ acc, 𝑖 ↦ acc + 𝑖 }, [0], [list: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) ↦ total
```

---

## Handlers Become Functions

### HyperTalk Handlers

```hypertalk
on greet theName
  put "Hello, " & theName & "!" into greeting
  return greeting
end greet
```

### Buran Functions

```
greet {
    name ↦ [string: "Hello, ", name, "!"]
}
```

Both define named operations that take parameters and return results. Buran functions are more concise because patterns describe both input and output.

### Multiple Cases

**HyperTalk:**

```hypertalk
function factorial n
  if n = 0 then
    return 1
  else
    return n * factorial(n - 1)
  end if
end factorial
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

Buran's pattern matching replaces if-then-else with case-by-case definitions. Each line handles one case.

---

## Control Structures

### Conditionals

**HyperTalk:**

```hypertalk
if x > 0 then
  put "positive"
else if x < 0 then
  put "negative"
else
  put "zero"
end if
```

**Buran:**

```
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}
```

The `|` in Buran is like HyperTalk's `if` — it adds a condition. But instead of nested if-then-else, you list cases.

### Repeat Loops

**HyperTalk:**

```hypertalk
repeat with i = 1 to 5
  put i * i into line i of field "squares"
end repeat
```

**Buran uses map instead of repeat:**

```
map({ 𝑖 ↦ 𝑖 × 𝑖 }, [list: 1, 2, 3, 4, 5]) ↦ squares
```

**HyperTalk repeat while:**

```hypertalk
put 1 into n
repeat while n < 100
  put n * 2 into n
end repeat
```

**Buran uses recursion:**

```
double-until-100 {
    [𝑛 | 𝑛 ≥ 100] ↦ 𝑛
    [𝑛] ↦ double-until-100([𝑛 × 2])
}
```

This is a significant difference. HyperTalk's loops modify state; Buran's recursion transforms values.

---

## Chunking Expressions vs Pattern Matching

HyperTalk's chunking was revolutionary — you could say `word 3 of line 2 of field "data"` and it just worked.

### HyperTalk Chunking

```hypertalk
put word 1 of "Hello World" into firstWord
put char 1 to 5 of myString into prefix
put line 3 of field "notes" into thirdLine
put the number of words in myText into wordCount
```

### Buran Pattern Matching

Buran achieves similar goals through pattern matching on structure:

```
# Get first element of a list
first {
    [list: head, ⟨rest⟩] ↦ head
}

# Get third element
third {
    [list: _, _, x, ⟨rest⟩] ↦ x
}

# Split list into first and rest
[list: first, ⟨rest⟩] ↦ items
```

For strings, Buran treats them as character sequences:

```
# First character
first-char {
    [string: c, ⟨rest⟩] ↦ c
}
```

The `⟨ ⟩` mirror brackets capture "the rest" — like HyperTalk's ability to work with parts of containers.

---

## Message Passing vs Pattern Transformation

HyperTalk's message passing was elegant: send a message, and handlers respond.

### HyperTalk Messages

```hypertalk
on mouseUp
  send "updateDisplay" to card "Dashboard"
end mouseUp

on updateDisplay
  put the date into field "lastUpdated"
end updateDisplay
```

### Buran Transformation

Buran doesn't have message passing. Instead, computation is pattern transformation:

```
# Data flows through transformations
input ↦
validate(input) ↦ validated ↦
transform(validated) ↦ result
```

Where HyperTalk sends messages between objects, Buran passes data through functions.

**Conceptual mapping:**

- HyperTalk message → Buran function call
- HyperTalk handler → Buran function
- HyperTalk object → Buran pattern (data structure)

---

## String Operations

### HyperTalk

```hypertalk
put "Hello" & " " & "World" into greeting
put the length of greeting into len
put char 1 to 5 of greeting into prefix
put offset("World", greeting) into pos
```

### Buran

```
[string: "Hello", " ", "World"] ↦ greeting
length(greeting) ↦ len
# String slicing would use pattern matching or library functions
```

HyperTalk's string handling was exceptionally natural. Buran's is more explicit but follows the same principle: strings are made of parts you can work with.

---

## Containers and Fields

HyperTalk had cards, fields, buttons — containers with persistent state.

### HyperTalk

```hypertalk
put "Hello" into field "greeting"
put field "input" into userInput
put line 2 of field "data" into secondLine
```

### Buran

Buran uses file connections for persistent storage:

```
# Read from file
[File: "data/greeting.txt"] ↦ greeting

# Write to file
["Hello"] ↦ [File: "data/greeting.txt"]

# Structured data
[File: "data/config.json" :: json] ↦ config
```

There's no card/field metaphor — Buran is a general-purpose language, not tied to a specific interface paradigm.

---

## Lists and Collections

### HyperTalk

```hypertalk
put "apple,banana,cherry" into fruits
put item 2 of fruits into secondFruit  -- "banana"
put the number of items in fruits into count
```

### Buran

```
[list: "apple", "banana", "cherry"] ↦ fruits

# Get second item via pattern matching
get-second {
    [list: _, second, ⟨rest⟩] ↦ second
}

length(fruits) ↦ count
```

HyperTalk used comma-delimited strings as lists. Buran has proper list patterns.

---

## Booleans and Logic

### HyperTalk

```hypertalk
put true into isReady
if isReady and count > 0 then
  put "go"
end if

put not isEmpty into hasContent
```

### Buran

```
[✔] ↦ is-ready
is-ready ∧ count > 0 ↦ {
    [✔] ↦ ["go"]
    [✘] ↦ []
}

[¬ is-empty] ↦ has-content
```

Buran uses visual symbols:

- `[✔]` for true (not `true`)
- `[✘]` for false (not `false`)
- `∧` for AND (not `and`)
- `∨` for OR (not `or`)
- `¬` for NOT (not `not`)

---

## Error Handling

### HyperTalk

```hypertalk
on errorDialog errorMessage
  answer "Error: " & errorMessage
end errorDialog
```

### Buran

```
safe-divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

# Handle errors through pattern matching
result ↦ {
    [error: msg] ↦ handle-error(msg)
    value ↦ use-value(value)
}
```

Buran makes errors explicit patterns you can match, rather than special error handlers.

---

## Mathematical Operations

This is where Buran shines compared to HyperTalk.

### HyperTalk

```hypertalk
put 3 * 4 into product
put 10 / 3 into quotient
put sqrt(16) into root
put 2 ^ 8 into power
```

### Buran

```
[3 × 4] ↦ product       # × not *
[10 ÷ 3] ↦ quotient     # ÷ not /
[√16] ↦ root            # √ not sqrt()
[2⁸] ↦ power            # superscript not ^
```

Buran uses standard mathematical notation — the symbols you learned in school.

Additional notation:

```
[𝛑]           # Pi
[𝐞]           # Euler's number
[⌊3.7⌋]       # Floor → 3
[⌈3.2⌉]       # Ceiling → 4
[5!]          # Factorial → 120
[|−5|]        # Absolute value → 5
```

---

## The "on mouseUp" Equivalent

HyperTalk was event-driven: things happened in response to user actions.

```hypertalk
on mouseUp
  put "Button clicked!" into field "status"
end mouseUp

on openCard
  put the date into field "today"
end openCard
```

Buran isn't inherently event-driven — it's a transformation language. Event handling would be part of an application framework built on Buran, not the language itself.

For scripts that respond to input:

```
# Read input, process, write output
[stdin] ↦ input ↦
process(input) ↦ output ↦
output ↦ [stdout]
```

---

## Translation Examples

### Finding Maximum

**HyperTalk:**

```hypertalk
function findMax theList
  put item 1 of theList into maxVal
  repeat with i = 2 to the number of items in theList
    if item i of theList > maxVal then
      put item i of theList into maxVal
    end if
  end repeat
  return maxVal
end findMax
```

**Buran:**

```
find-max {
    [list: 𝑥] ↦ 𝑥
    [list: 𝑥, ⟨rest⟩] ↦
        find-max(rest) ↦ max-of-rest ↦
        { [✔] ↦ 𝑥, [✘] ↦ max-of-rest } ↤ [𝑥 > max-of-rest]
}
```

Or simply:

```
max([list: 3, 1, 4, 1, 5, 9]) ↦ [9]   # Built-in
```

### Reversing a List

**HyperTalk:**

```hypertalk
function reverseList theList
  put empty into reversed
  repeat with i = the number of items in theList down to 1
    put item i of theList & comma after reversed
  end repeat
  delete last char of reversed
  return reversed
end reverseList
```

**Buran:**

```
reverse {
    [list: ] ↦ [list: ]
    [list: head, ⟨tail⟩] ↦ [list: ⟨reverse(tail)⟩, head]
}
```

### Counting Occurrences

**HyperTalk:**

```hypertalk
function countOccurrences searchStr, theText
  put 0 into count
  put 1 into startPos
  repeat
    put offset(searchStr, theText, startPos) into foundPos
    if foundPos = 0 then exit repeat
    add 1 to count
    put foundPos + length(searchStr) into startPos
  end repeat
  return count
end countOccurrences
```

**Buran:**

```
count-occurrences {
    needle, [list: ] ↦ [0]
    needle, [list: head, ⟨tail⟩] | head = needle ↦ [1 + count-occurrences(needle, tail)]
    needle, [list: _, ⟨tail⟩] ↦ count-occurrences(needle, tail)
}
```

---

## What Carries Over

From HyperTalk to Buran, these intuitions transfer:

| HyperTalk Concept | Buran Equivalent             |
| ----------------- | ---------------------------- |
| Readability first | Mathematical clarity         |
| `put X into Y`    | `X ↦ Y`                      |
| Handlers          | Functions                    |
| Messages          | Function calls               |
| Repeat loops      | map/filter/fold/recursion    |
| if-then-else      | Pattern matching with guards |
| Chunking          | Pattern destructuring        |
| Containers        | File connections             |

---

## What's Different

| HyperTalk                       | Buran                 |
| ------------------------------- | --------------------- |
| English-like syntax             | Mathematical notation |
| Mutable state                   | Immutable values      |
| Event-driven                    | Transformation-based  |
| Object-oriented (cards, fields) | Pattern-oriented      |
| Loops                           | Recursion             |
| String-based lists              | Typed collections     |
| `true`/`false`                  | `[✔]`/`[✘]`           |

---

## The Same Spirit

Both HyperTalk and Buran believe:

1. **Programming should be accessible** — HyperTalk to English speakers, Buran to anyone who learned basic math
2. **Notation matters** — readable code is maintainable code
3. **Simplicity over power** — do common things easily
4. **One way to do things** — consistency aids understanding

HyperTalk showed that programming doesn't have to be cryptic. Buran continues that tradition with a different vocabulary — the universal language of mathematics.

---

## Summary

If you loved HyperTalk's clarity, you'll appreciate Buran's directness. The syntax is different — mathematical rather than English — but the philosophy is kindred: make programming readable, make it accessible, make it make sense.

Where you once wrote `put 5 into x`, you'll write `[5] ↦ x`.
Where you wrote handlers, you'll write pattern functions.
Where you used repeat loops, you'll use map and recursion.

The tools change, but the goal remains: programming for humans.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
