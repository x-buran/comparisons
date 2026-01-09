# Buran for Java Programmers

## Introduction

If you know Java, you understand strong typing, object-oriented design, and enterprise-grade reliability. You've built systems with classes, interfaces, and inheritance hierarchies. Buran takes a radically different approach: no classes, no objects, no inheritance. Everything is a pattern, and all computation is pattern transformation.

This might feel like a large leap, but if you've used Java's newer features — records, sealed classes, switch expressions with pattern matching — you've already taken steps in this direction.

This document will help you transition from Java to Buran by showing correspondences between familiar constructs and new syntax.

---

## The Paradigm Shift

Java is object-oriented: you model the world as objects with state and behavior.

Buran is pattern-oriented: you model the world as patterns that transform into other patterns.

**Java:**

```java
public class Factorial {
    public static int compute(int n) {
        if (n == 0) {
            return 1;
        }
        return n * compute(n - 1);
    }
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

No class, no `public static`, no `if`, no `return`. Just patterns transforming into patterns.

---

## No Classes, No Objects

This is the fundamental shift. Java organizes code into classes:

**Java:**

```java
public class Point {
    private final double x;
    private final double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public double distance() {
        return Math.sqrt(x * x + y * y);
    }
}

Point p = new Point(3.0, 4.0);
double d = p.distance();  // 5.0
```

**Buran:**

```
# Data is just a pattern
[point: 3.0, 4.0] ↦ p

# Behavior is a separate function
distance {
    [point: 𝑥, 𝑦] ↦ [√(𝑥² + 𝑦²)]
}

distance(p) ↦ d    # [5.0]
```

Data and behavior are separate. Patterns hold data; functions transform patterns.

### Records Are Closer

If you've used Java records (Java 14+), you're already thinking this way:

**Java:**

```java
public record Point(double x, double y) {
    public double distance() {
        return Math.sqrt(x * x + y * y);
    }
}
```

**Buran:**

```
# Define type through identity
⟨[point: _, _]⟩ ↤ [identity:
    "type": "Point",
    "fields": ["x", "y"]
]

# Method as standalone function
distance {
    [point: 𝑥, 𝑦] ↦ [√(𝑥² + 𝑦²)]
}
```

---

## Interfaces and Inheritance → Pattern Matching

Java uses interfaces and inheritance for polymorphism:

**Java:**

```java
public interface Shape {
    double area();
}

public class Circle implements Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}

public double totalArea(List<Shape> shapes) {
    return shapes.stream()
                 .mapToDouble(Shape::area)
                 .sum();
}
```

**Buran:**

```
# No interface declaration — pattern matching handles dispatch
area {
    [circle: 𝑟] ↦ [𝛑 × 𝑟²]
    [rectangle: 𝑤, ℎ] ↦ [𝑤 × ℎ]
    [triangle: 𝑎, 𝑏, 𝑐] ↦
        [(𝑎 + 𝑏 + 𝑐) ÷ 2] ↦ 𝑠 ↦
        [√(𝑠 × (𝑠 − 𝑎) × (𝑠 − 𝑏) × (𝑠 − 𝑐))]
}

total-area {
    shapes ↦ fold({ acc, shape ↦ acc + area(shape) }, [0], shapes)
}
```

No interfaces, no class hierarchy. The pattern structure *is* the type.

### Sealed Classes Are Closer

Java's sealed classes (Java 17+) move toward this:

**Java:**

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double a, double b, double c) implements Shape {}

public static double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> {
            double s = (t.a() + t.b() + t.c()) / 2;
            yield Math.sqrt(s * (s - t.a()) * (s - t.b()) * (s - t.c()));
        }
    };
}
```

This Java pattern matches Buran's approach exactly.

---

## Basic Syntax Correspondences

### Variables

**Java:**

```java
int x = 42;
String name = "hello";
List<Integer> items = List.of(1, 2, 3);
Map<String, Integer> scores = Map.of("a", 1, "b", 2);
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
[map: "a", 1, "b", 2] ↦ scores
```

All values are in square brackets. Arrow indicates binding.

### Methods → Functions

**Java:**

```java
public static int add(int a, int b) {
    return a + b;
}

public static String greet(String name) {
    return "Hello, " + name + "!";
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

No access modifiers, no return types, no `return` statement.

### Method Calls

**Java:**

```java
int sum = add(2, 3);
String msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Null → Empty Pattern

Java's billion-dollar mistake:

**Java:**

```java
String name = null;
if (name != null) {
    System.out.println(name.length());
}

// Or with Optional
Optional<String> maybeName = Optional.ofNullable(name);
maybeName.ifPresent(n -> System.out.println(n.length()));
```

**Buran:**

```
[] ↦ name    # Empty pattern = no value

name ↦ {
    [] ↦ []    # Nothing to do
    n ↦ length(n) ↦ [stdout]
}
```

No `null`, no `NullPointerException`. The empty pattern `[]` explicitly represents absence.

### Optional → Direct Pattern Matching

**Java:**

```java
public Optional<User> findUser(String id) {
    // ...
    return Optional.ofNullable(result);
}

findUser("123")
    .map(User::getName)
    .orElse("Unknown");
```

**Buran:**

```
find-user {
    id ↦ ...    # Returns user or []
}

find-user("123") ↦ {
    [] ↦ ["Unknown"]
    user ↦ name(user)
}
```

No `Optional` wrapper needed — pattern matching handles presence/absence directly.

---

## Exceptions → Error Patterns

**Java:**

```java
public double divide(double a, double b) throws ArithmeticException {
    if (b == 0) {
        throw new ArithmeticException("Division by zero");
    }
    return a / b;
}

try {
    double result = divide(10, 0);
    process(result);
} catch (ArithmeticException e) {
    System.err.println("Error: " + e.getMessage());
}
```

**Buran:**

```
divide {
    𝑎, [0] ↦ [error: division by zero]
    𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

divide([10], [0]) ↦ {
    [error: msg] ↦ [string: "Error: ", msg] ↦ [stderr]
    result ↦ process(result)
}
```

No `throw`, no `try`/`catch`, no exception hierarchy. Errors are patterns.

### Typed Error Handling

**Java:**

```java
try {
    // ...
} catch (IOException e) {
    handleIO(e);
} catch (ParseException e) {
    handleParse(e);
} catch (Exception e) {
    handleGeneral(e);
}
```

**Buran:**

```
result ↦ {
    ⟨e | "type": "error", "category": "io"⟩ ↦ handle-io(e)
    ⟨e | "type": "error", "category": "parse"⟩ ↦ handle-parse(e)
    ⟨e | "type": "error"⟩ ↦ handle-general(e)
    value ↦ process(value)
}
```

Mirror brackets access error metadata for categorized handling.

---

## Generics → Implicit Polymorphism

**Java:**

```java
public static <T> T first(List<T> list) {
    if (list.isEmpty()) {
        return null;
    }
    return list.get(0);
}

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T item : list) {
        result.add(f.apply(item));
    }
    return result;
}
```

**Buran:**

```
first {
    [list: ] ↦ []
    [list: head, ⟨_⟩] ↦ head
}

map {
    f, [list: ] ↦ [list: ]
    f, [list: head, ⟨tail⟩] ↦ [list: f(head), ⟨map(f, tail)⟩]
}
```

No `<T>`, no `<T, R>`. Patterns are naturally polymorphic — they match any structure.

### Type Constraints

**Java:**

```java
public static <T extends Number> double sum(List<T> numbers) {
    return numbers.stream()
                  .mapToDouble(Number::doubleValue)
                  .sum();
}
```

**Buran:**

```
sum {
    [list: ] ↦ [0]
    [list: ⟨head | "type": "number"⟩, ⟨tail⟩] ↦ [head + sum(tail)]
}
```

Identity guards constrain types when needed.

---

## Collections

**Java:**

```java
// List
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
list.add(4);
int first = list.get(0);

// Map
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
int value = map.get("a");

// Set
Set<Integer> set = new HashSet<>(Set.of(1, 2, 3));
set.add(4);
boolean has = set.contains(2);
```

**Buran:**

```
# List (immutable)
[list: 1, 2, 3] ↦ list
[list: ⟨list⟩, 4] ↦ list2    # New list
[list: first, ⟨_⟩] ↦ list    # Destructuring

# Map
[map: "a", 1] ↦ m
[map: ⟨m⟩, "b", 2] ↦ m2      # New map

# Set
[set: 1, 2, 3] ↦ s
[set: ⟨s⟩, 4] ↦ s2           # New set
[2 ∈ s] ↦ has                 # Membership
```

All collections are immutable. Operations create new collections.

---

## Streams → map/filter/fold

**Java:**

```java
List<Integer> doubled = numbers.stream()
    .map(x -> x * 2)
    .collect(Collectors.toList());

List<Integer> evens = numbers.stream()
    .filter(x -> x % 2 == 0)
    .collect(Collectors.toList());

int sum = numbers.stream()
    .reduce(0, Integer::sum);

// Chained
int result = numbers.stream()
    .filter(x -> x > 0)
    .map(x -> x * 2)
    .reduce(0, Integer::sum);
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, numbers) ↦ doubled

filter({ 𝑥 | 𝑥 mod 2 = 0 }, numbers) ↦ evens

fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], numbers) ↦ sum

# Chained
numbers ↦
filter({ 𝑥 | 𝑥 > 0 }, numbers) ↦ positives ↦
map({ 𝑥 ↦ 𝑥 × 2 }, positives) ↦ doubled ↦
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], doubled) ↦ result
```

---

## Lambdas → Pattern Blocks

**Java:**

```java
Function<Integer, Integer> double = x -> x * 2;
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
Predicate<Integer> isPositive = x -> x > 0;

// With multiple expressions
Function<Integer, String> classify = x -> {
    if (x > 0) return "positive";
    if (x < 0) return "negative";
    return "zero";
};
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 } ↦ double
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ add
{ 𝑥 | 𝑥 > 0 } ↦ is-positive

# Multiple clauses
{
    [𝑥 | 𝑥 > 0] ↦ ["positive"],
    [𝑥 | 𝑥 < 0] ↦ ["negative"],
    [0] ↦ ["zero"]
} ↦ classify
```

Pattern blocks can have multiple clauses — like a lambda with built-in switch.

---

## Control Flow

**Java:**

```java
// If-else
if (x > 0) {
    result = "positive";
} else if (x < 0) {
    result = "negative";
} else {
    result = "zero";
}

// Switch expression (Java 14+)
String result = switch (x) {
    case Integer i when i > 0 -> "positive";
    case Integer i when i < 0 -> "negative";
    default -> "zero";
};

// For loop
for (int i = 0; i < n; i++) {
    process(i);
}

// For-each
for (var item : items) {
    process(item);
}

// While
while (condition) {
    // ...
}
```

**Buran:**

```
# Pattern matching replaces if/switch
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

# Recursion replaces loops
process-range {
    [0] ↦ []
    [𝑛] ↦ process([𝑛 − 1]) ↦ _ ↦ process-range([𝑛 − 1])
}

# Map replaces for-each
map({ item ↦ process(item) }, items)

# Tail recursion replaces while
process-while {
    state | ¬condition(state) ↦ state
    state ↦ process-while(next-state(state))
}
```

No `if`, no `for`, no `while`. Pattern matching and recursion only.

---

## File I/O

**Java:**

```java
// Read
String content = Files.readString(Path.of("data.txt"));

// Write
Files.writeString(Path.of("output.txt"), data);

// JSON (with Jackson or Gson)
ObjectMapper mapper = new ObjectMapper();
MyClass obj = mapper.readValue(new File("config.json"), MyClass.class);
String json = mapper.writeValueAsString(obj);
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
data ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ obj
obj ↦ [File: "output.json" :: json]
```

Built-in format support: `:: json`, `:: yaml`, `:: csv`, `:: binary`.

---

## Mathematical Notation

Java uses method calls and ASCII. Buran uses mathematical symbols:

| Java             | Buran   | Meaning               |
| ---------------- | ------- | --------------------- |
| `a * b`          | `𝑎 × 𝑏` | Multiplication        |
| `a / b`          | `𝑎 ÷ 𝑏` | Division              |
| `Math.pow(x, 2)` | `𝑥²`    | Square                |
| `Math.sqrt(x)`   | `√𝑥`    | Square root           |
| `Math.PI`        | `𝛑`     | Pi                    |
| `<=`             | `≤`     | Less than or equal    |
| `>=`             | `≥`     | Greater than or equal |
| `!=`             | `≠`     | Not equal             |
| `&&`             | `∧`     | Logical AND           |
| `\|\|`           | `∨`     | Logical OR            |
| `!`              | `¬`     | Logical NOT           |
| `true`           | `[✔]`   | Boolean true          |
| `false`          | `[✘]`   | Boolean false         |
| `null`           | `[]`    | Empty/null            |

---

## Comparison Table

| Concept     | Java                    | Buran                     |
| ----------- | ----------------------- | ------------------------- |
| Class       | `public class C { }`    | Patterns + identity       |
| Constructor | `new Point(x, y)`       | `[point: x, y]`           |
| Method      | `public int f(int x)`   | `f { pattern ↦ result }`  |
| Interface   | `interface I { }`       | Pattern matching          |
| Inheritance | `extends`, `implements` | Pattern matching dispatch |
| Generic     | `<T>`                   | Implicit polymorphism     |
| Lambda      | `x -> x + 1`            | `{ 𝑥 ↦ 𝑥 + 1 }`           |
| List        | `List.of(1, 2, 3)`      | `[list: 1, 2, 3]`         |
| Map         | `Map.of("a", 1)`        | `[map: "a", 1]`           |
| Set         | `Set.of(1, 2, 3)`       | `[set: 1, 2, 3]`          |
| Optional    | `Optional<T>`           | Value or `[]`             |
| Exception   | `throw`, `try/catch`    | `[error: ...]` patterns   |
| null        | `null`                  | `[]`                      |
| true/false  | `true`, `false`         | `[✔]`, `[✘]`              |
| Switch      | `switch (x) { }`        | Pattern matching          |
| For loop    | `for (...)`             | Recursion or map          |
| While       | `while (...)`           | Tail recursion            |

---

## Example: Visitor Pattern → Pattern Matching

**Java:**

```java
// Classic visitor pattern
public interface ExprVisitor<T> {
    T visitNum(NumExpr expr);
    T visitAdd(AddExpr expr);
    T visitMul(MulExpr expr);
}

public interface Expr {
    <T> T accept(ExprVisitor<T> visitor);
}

public class NumExpr implements Expr {
    private final int value;
    public NumExpr(int value) { this.value = value; }
    public int getValue() { return value; }
    public <T> T accept(ExprVisitor<T> visitor) {
        return visitor.visitNum(this);
    }
}

// ... AddExpr, MulExpr similarly ...

public class EvalVisitor implements ExprVisitor<Integer> {
    public Integer visitNum(NumExpr expr) {
        return expr.getValue();
    }
    public Integer visitAdd(AddExpr expr) {
        return expr.getLeft().accept(this) + expr.getRight().accept(this);
    }
    public Integer visitMul(MulExpr expr) {
        return expr.getLeft().accept(this) * expr.getRight().accept(this);
    }
}
```

**Buran:**

```
eval {
    [num: 𝑛] ↦ 𝑛
    [add: left, right] ↦ [eval(left) + eval(right)]
    [mul: left, right] ↦ [eval(left) × eval(right)]
}
```

The entire visitor pattern collapses to a single function with pattern clauses.

---

## Example: Builder Pattern → Just Patterns

**Java:**

```java
public class User {
    private final String name;
    private final int age;
    private final String email;

    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
    }

    public static class Builder {
        private String name;
        private int age;
        private String email;

        public Builder name(String name) {
            this.name = name;
            return this;
        }
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        public User build() {
            return new User(this);
        }
    }
}

User user = new User.Builder()
    .name("Alice")
    .age(30)
    .email("alice@example.com")
    .build();
```

**Buran:**

```
[user: "Alice", 30, "alice@example.com"]
```

No builder pattern needed. Patterns are already declarative.

---

## Example: Strategy Pattern → Pattern Blocks

**Java:**

```java
public interface SortStrategy {
    void sort(int[] array);
}

public class QuickSort implements SortStrategy {
    public void sort(int[] array) { /* ... */ }
}

public class MergeSort implements SortStrategy {
    public void sort(int[] array) { /* ... */ }
}

public class Sorter {
    private SortStrategy strategy;

    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }

    public void sort(int[] array) {
        strategy.sort(array);
    }
}
```

**Buran:**

```
# Just pass different functions
sort-with {
    strategy, data ↦ strategy(data)
}

sort-with(quicksort, data)
sort-with(mergesort, data)
```

Functions are first-class. No interface/implementation ceremony.

---

## What Buran Trades Away

Coming from Java, you should know what Buran doesn't provide:

1. **Static type checking** — No compile-time type errors
2. **IDE support** — No autocomplete, refactoring tools (yet)
3. **Enterprise ecosystem** — No Spring, Hibernate, etc.
4. **Familiar OOP patterns** — No classes, inheritance, encapsulation
5. **Mutable state** — Everything is immutable
6. **Explicit control flow** — No loops, just recursion

## What Buran Offers Instead

1. **Radical simplicity** — One concept: pattern transformation
2. **No null pointer exceptions** — Empty pattern is explicit
3. **No boilerplate** — No getters, setters, builders
4. **Mathematical notation** — Write `√𝑥` not `Math.sqrt(x)`
5. **Concise code** — Visitor pattern in 4 lines
6. **AI-friendly syntax** — Consistent structure for code generation

---

## Philosophical Difference

Java asks: *"What objects exist, and how do they interact?"*

Buran asks: *"What pattern does this pattern become?"*

Java models the world as a network of objects sending messages. Buran models the world as patterns flowing through transformations.

If you've enjoyed Java's newer features — records, sealed classes, pattern matching in switch — you've been moving toward Buran's philosophy. Buran is what you get when pattern matching isn't a feature, but the entire language.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
