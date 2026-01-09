# Buran for Perl Programmers

## Introduction

If you know Perl, you understand the power of pattern matching — regular expressions are in your blood. Buran shares your love of patterns, but takes the idea in a different direction: instead of matching text with regexes, everything is a structured pattern, and all computation is pattern transformation.

You also know that There's More Than One Way To Do It. Buran takes the opposite philosophy: there's one way to do everything, and that way is pattern transformation.

This document will help you transition from Perl to Buran by showing correspondences between familiar constructs and new syntax.

---

## Two Kinds of Pattern Matching

Perl's patterns match text:

```perl
if ($str =~ /^hello\s+(\w+)/) {
    print "Greeting: $1\n";
}
```

Buran's patterns match structure:

```
greet {
    [greeting: "hello", name] ↦ [string: "Greeting: ", name]
}
```

Both languages are obsessed with patterns — just different kinds. Perl excels at text patterns. Buran excels at data structure patterns.

---

## The Paradigm Shift

Perl is imperative and multi-paradigm. Buran is purely declarative:

**Perl:**

```perl
sub factorial {
    my ($n) = @_;
    return 1 if $n == 0;
    return $n * factorial($n - 1);
}
```

**Buran:**

```
factorial {
    [0] ↦ [1]
    [𝑛] ↦ [𝑛 × factorial(𝑛 − 1)]
}
```

No `sub`, no `my`, no `return`, no `if`. You declare what patterns transform into, and computation emerges.

---

## No Sigils

Perl uses sigils to indicate variable types. Buran has none:

**Perl:**

```perl
my $scalar = 42;
my @array = (1, 2, 3);
my %hash = (a => 1, b => 2);
my $ref = \@array;
```

**Buran:**

```
[42] ↦ scalar
[list: 1, 2, 3] ↦ array
[map: "a", 1, "b", 2] ↦ hash
# No references needed
```

All values are patterns in square brackets. The pattern structure tells you what it is.

---

## Basic Syntax Correspondences

### Variables

**Perl:**

```perl
my $x = 42;
my $name = "hello";
my @items = (1, 2, 3);
my %person = (name => "Alice", age => 30);
```

**Buran:**

```
[42] ↦ 𝑥
["hello"] ↦ name
[list: 1, 2, 3] ↦ items
[map: "name", "Alice", "age", 30] ↦ person
```

Key differences:

- All values are in square brackets
- Arrow `↦` indicates data flow (like assignment)
- No sigils — structure determines type

### Subroutines

**Perl:**

```perl
sub add {
    my ($a, $b) = @_;
    return $a + $b;
}

sub greet {
    my ($name) = @_;
    return "Hello, $name!";
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

Functions are pattern-matching blocks. No `sub`, no `my`, no `@_`, no `return`.

### Function Calls

**Perl:**

```perl
my $sum = add(2, 3);
my $msg = greet("World");
```

**Buran:**

```
add([2], [3]) ↦ sum
greet("World") ↦ msg
```

Literal arguments need brackets: `add([2], [3])`. Variables don't: `add(𝑥, 𝑦)`.

---

## Arrays → Lists

**Perl:**

```perl
my @arr = (1, 2, 3, 4, 5);
my $first = $arr[0];
my @rest = @arr[1..$#arr];
push @arr, 6;
my $len = scalar @arr;
```

**Buran:**

```
[list: 1, 2, 3, 4, 5] ↦ arr

# Destructuring
[list: first, ⟨rest⟩] ↦ arr

# Create new list with additional element (immutable)
[list: ⟨arr⟩, 6] ↦ arr2

length(arr) ↦ len
```

The `⟨...⟩` mirror brackets capture remaining elements or spread elements into a new list.

### List Operations

**Perl:**

```perl
my @doubled = map { $_ * 2 } @numbers;
my @evens = grep { $_ % 2 == 0 } @numbers;
my $sum = 0; $sum += $_ for @numbers;
```

**Buran:**

```
map({ 𝑥 ↦ 𝑥 × 2 }, numbers) ↦ doubled
filter({ 𝑥 | 𝑥 mod 2 = 0 }, numbers) ↦ evens
fold({ acc, 𝑥 ↦ acc + 𝑥 }, [0], numbers) ↦ sum
```

Pattern blocks `{ ... }` replace Perl's blocks. `grep` becomes `filter`.

---

## Hashes → Maps

**Perl:**

```perl
my %hash = (
    name => "Alice",
    age  => 30,
);
my $name = $hash{name};
$hash{city} = "Boston";
my @keys = keys %hash;
exists $hash{name};
```

**Buran:**

```
[map: "name", "Alice", "age", 30] ↦ hash

# Access through pattern matching
# No direct key access syntax yet

# Create new map with additional entry (immutable)
[map: ⟨hash⟩, "city", "Boston"] ↦ hash2

# Membership through pattern matching
```

Buran maps are key-value pair sequences, not hash tables.

---

## No Context Sensitivity

Perl famously has scalar and list context. Buran has none:

**Perl:**

```perl
my @arr = (1, 2, 3);
my $count = @arr;        # Scalar context: 3
my @copy = @arr;         # List context: (1, 2, 3)
my $str = localtime;     # Scalar context: string
my @time = localtime;    # List context: array
```

**Buran:**

```
# Everything is a pattern — no context switching
[list: 1, 2, 3] ↦ arr
length(arr) ↦ count      # Explicit function call
arr ↦ copy               # Always the same value
```

What you see is what you get. No magic context-dependent behavior.

---

## Regular Expressions → Structural Patterns

Perl's regex matches text. Buran's patterns match structure:

**Perl:**

```perl
# Text pattern
if ($str =~ /^(\w+)\s+(\d+)$/) {
    my ($word, $num) = ($1, $2);
    process($word, $num);
}

# Substitution
$str =~ s/old/new/g;
```

**Buran:**

```
# Structural pattern
process-input {
    [input: word, num] ↦ process(word, num)
}

# Transformation (creates new value)
transform {
    [old] ↦ [new]
}
```

For text manipulation, Buran would need a string processing library (not yet specified). Its patterns are for data structures, not text.

### When You Need Text Patterns

Buran is not designed for text munging the way Perl is. If your work is primarily text processing, Perl remains an excellent choice. Buran excels at structured data transformation.

---

## Anonymous Subroutines → Pattern Blocks

**Perl:**

```perl
my $double = sub { $_[0] * 2 };
my $add = sub { $_[0] + $_[1] };
my $is_even = sub { $_[0] % 2 == 0 };

map { $_ * 2 } @arr;
grep { $_ > 0 } @arr;
sort { $a <=> $b } @arr;
```

**Buran:**

```
{ 𝑥 ↦ 𝑥 × 2 } ↦ double
{ 𝑎, 𝑏 ↦ 𝑎 + 𝑏 } ↦ add
{ 𝑥 | 𝑥 mod 2 = 0 } ↦ is-even

map({ 𝑥 ↦ 𝑥 × 2 }, arr)
filter({ 𝑥 | 𝑥 > 0 }, arr)
sort({ 𝑎, 𝑏 ↦ compare(𝑎, 𝑏) }, arr)
```

Pattern blocks can have multiple clauses:

```
{
    [0] ↦ ["zero"],
    [𝑛 | 𝑛 > 0] ↦ ["positive"],
    _ ↦ ["negative"]
}
```

---

## References → Not Needed

Perl uses references for complex data structures. Buran doesn't need them:

**Perl:**

```perl
my $arr_ref = [1, 2, 3];
my $hash_ref = { a => 1, b => 2 };
my @nested = ([1, 2], [3, 4]);

# Dereferencing
my @arr = @$arr_ref;
my %hash = %$hash_ref;
print $nested[0][1];  # 2
```

**Buran:**

```
[list: 1, 2, 3] ↦ arr
[map: "a", 1, "b", 2] ↦ hash
[list: [list: 1, 2], [list: 3, 4]] ↦ nested

# No dereferencing — patterns nest naturally
# Access through pattern matching
```

Everything is a value. No references, no dereferencing, no autovivification.

---

## Control Structures

**Perl:**

```perl
# If-elsif-else
if ($x > 0) {
    print "positive";
} elsif ($x < 0) {
    print "negative";
} else {
    print "zero";
}

# Unless
print "empty" unless @arr;

# For loop
for my $i (0..9) {
    process($i);
}

# Foreach
for my $item (@items) {
    process($item);
}

# While
while ($condition) {
    # ...
}

# Postfix
print $_ for @items;
```

**Buran:**

```
# Pattern matching replaces all conditionals
classify {
    [𝑥 | 𝑥 > 0] ↦ ["positive"]
    [𝑥 | 𝑥 < 0] ↦ ["negative"]
    [0] ↦ ["zero"]
}

# Empty check through pattern matching
check-empty {
    [list: ] ↦ ["empty"]
    _ ↦ []
}

# Recursion replaces loops
process-range {
    [0], _ ↦ []
    [𝑛], f ↦ f([𝑛 − 1]) ↦ _ ↦ process-range([𝑛 − 1], f)
}

# Map replaces foreach
map({ item ↦ process(item) }, items)
```

No `if`, no `unless`, no `for`, no `while`, no postfix conditionals. Pattern matching and recursion handle everything.

---

## Error Handling

**Perl:**

```perl
eval {
    risky_operation();
};
if ($@) {
    warn "Error: $@";
}

# Or with Try::Tiny
use Try::Tiny;
try {
    risky_operation();
} catch {
    warn "Error: $_";
};

# Die
die "Something went wrong" if $error;
```

**Buran:**

```
# Errors are just patterns
risky-operation() ↦ {
    [error: msg] ↦ handle-error(msg)
    result ↦ process(result)
}

# Return an error
check {
    condition | ¬valid(condition) ↦ [error: something went wrong]
    condition ↦ process(condition)
}
```

No `eval`/`$@`, no `die`/`warn`. Errors are patterns to match.

---

## File I/O

**Perl:**

```perl
# Read entire file
open my $fh, '<', 'data.txt' or die $!;
my $content = do { local $/; <$fh> };
close $fh;

# Or with File::Slurp
use File::Slurp;
my $content = read_file('data.txt');

# Write
write_file('output.txt', $data);

# JSON
use JSON;
my $data = decode_json($json_str);
my $json = encode_json($data);
```

**Buran:**

```
# Read
[File: "data.txt"] ↦ content

# Write
data ↦ [File: "output.txt"]

# JSON (built-in)
[File: "config.json" :: json] ↦ data
data ↦ [File: "output.json" :: json]
```

Built-in format support: `:: json`, `:: yaml`, `:: csv`, `:: binary`. No modules to load.

---

## String Operations

**Perl:**

```perl
my $str = "Hello, " . $name . "!";
my $upper = uc($str);
my $lower = lc($str);
my @parts = split /,/, $str;
my $joined = join("-", @parts);
my $len = length($str);
my $sub = substr($str, 0, 5);
```

**Buran:**

```
[string: "Hello, ", name, "!"] ↦ str
uppercase(str) ↦ upper
lowercase(str) ↦ lower
split([","], str) ↦ parts
join(["-"], parts) ↦ joined
length(str) ↦ len
# Substring through pattern matching or library function
```

Concatenation uses the `string:` constructor.

---

## One-Liners → Not Really

Perl is famous for one-liners:

```bash
perl -ne 'print if /pattern/' file.txt
perl -pe 's/old/new/g' file.txt
perl -lane 'print $F[0]' file.txt
```

Buran is not designed for one-liners. It's designed for readable, maintainable programs that AI systems can reliably generate.

If you need one-liners for text processing, keep using Perl.

---

## TIMTOWTDI → One Way

Perl's motto: "There Is More Than One Way To Do It."

**Perl** (many ways to iterate):

```perl
for my $i (0..$#arr) { ... }
foreach my $item (@arr) { ... }
for (@arr) { ... }
map { ... } @arr;
$_ and process($_) for @arr;
while (my $item = shift @arr) { ... }
```

**Buran** (one way):

```
map({ item ↦ process(item) }, arr)
```

Buran deliberately has one way to do things. This uniformity makes code:

- Easier to read
- Easier for AI to generate
- Easier to maintain

---

## Mathematical Notation

Perl uses ASCII operators. Buran uses proper mathematical symbols:

| Perl             | Buran         | Meaning               |
| ---------------- | ------------- | --------------------- |
| `*`              | `×`           | Multiplication        |
| `/`              | `÷`           | Division              |
| `**`             | `²`, `³`, `ⁿ` | Exponentiation        |
| `sqrt($x)`       | `√𝑥`          | Square root           |
| `3.14159...`     | `𝛑`           | Pi                    |
| `<=`             | `≤`           | Less than or equal    |
| `>=`             | `≥`           | Greater than or equal |
| `!=`             | `≠`           | Not equal             |
| `&&`             | `∧`           | Logical AND           |
| `\|\|`           | `∨`           | Logical OR            |
| `!`              | `¬`           | Logical NOT           |
| `1` (true)       | `[✔]`         | Boolean true          |
| `0`/`""` (false) | `[✘]`         | Boolean false         |
| `undef`          | `[]`          | Empty/undefined       |

---

## Special Variables → None

Perl has many special variables. Buran has none:

**Perl:**

```perl
$_          # Default variable
@_          # Subroutine arguments
$1, $2...   # Regex captures
$@          # Error from eval
$/          # Input record separator
$\          # Output record separator
$;          # Subscript separator
$"          # List separator
# ... dozens more
```

**Buran:**

```
# No special variables
# Everything is explicit through pattern matching
```

---

## Comparison Table

| Concept       | Perl                | Buran                    |
| ------------- | ------------------- | ------------------------ |
| Function      | `sub f { }`         | `f { pattern ↦ result }` |
| Call          | `f($x, $y)`         | `f(𝑥, 𝑦)`                |
| Scalar        | `$x`                | `𝑥` (no sigil)           |
| Array         | `@arr`              | `[list: ...]`            |
| Hash          | `%hash`             | `[map: k1, v1, ...]`     |
| Reference     | `\@arr`, `$ref->[]` | Not needed               |
| Anonymous sub | `sub { }`           | `{ pattern ↦ result }`   |
| Map           | `map { } @arr`      | `map({ }, arr)`          |
| Grep          | `grep { } @arr`     | `filter({ }, arr)`       |
| Regex match   | `=~`                | Structural patterns      |
| String concat | `.`                 | `[string: a, b]`         |
| Conditional   | `if/elsif/else`     | Pattern matching         |
| Loop          | `for`, `while`      | Recursion                |
| Error         | `die`, `eval/$@`    | `[error: ...]` patterns  |
| Undef         | `undef`             | `[]`                     |
| True/False    | `1`/`0`             | `[✔]`/`[✘]`              |
| Comment       | `#`                 | `#`                      |

---

## Example: Data Processing

**Perl:**

```perl
use JSON;

# Read JSON, filter, transform, write
my $json = read_file('input.json');
my $data = decode_json($json);

my @valid = grep { $_->{status} eq 'active' } @$data;
my @transformed = map {
    {
        name => uc($_->{name}),
        id   => $_->{id},
    }
} @valid;

write_file('output.json', encode_json(\@transformed));
```

**Buran:**

```
[File: "input.json" :: json] ↦ data ↦

filter({ record | status(record) = "active" }, data) ↦ valid ↦

map({
    record ↦ [record:
        uppercase(name(record)),
        id(record)
    ]
}, valid) ↦ transformed ↦

transformed ↦ [File: "output.json" :: json]
```

---

## Example: FizzBuzz

**Perl:**

```perl
for my $n (1..100) {
    if ($n % 15 == 0) {
        print "FizzBuzz\n";
    } elsif ($n % 3 == 0) {
        print "Fizz\n";
    } elsif ($n % 5 == 0) {
        print "Buzz\n";
    } else {
        print "$n\n";
    }
}
```

**Buran:**

```
fizzbuzz {
    [𝑛 | 𝑛 mod 15 = 0] ↦ ["FizzBuzz"]
    [𝑛 | 𝑛 mod 3 = 0] ↦ ["Fizz"]
    [𝑛 | 𝑛 mod 5 = 0] ↦ ["Buzz"]
    [𝑛] ↦ [string: 𝑛]
}

# Apply to range
map({ 𝑛 ↦ fizzbuzz([𝑛]) }, range([1], [100]))
```

---

## Example: Dispatch Table

**Perl:**

```perl
my %dispatch = (
    add => sub { $_[0] + $_[1] },
    sub => sub { $_[0] - $_[1] },
    mul => sub { $_[0] * $_[1] },
    div => sub { $_[0] / $_[1] },
);

my $result = $dispatch{$op}->($a, $b);
```

**Buran:**

```
dispatch {
    ["add"], 𝑎, 𝑏 ↦ [𝑎 + 𝑏]
    ["sub"], 𝑎, 𝑏 ↦ [𝑎 − 𝑏]
    ["mul"], 𝑎, 𝑏 ↦ [𝑎 × 𝑏]
    ["div"], 𝑎, 𝑏 ↦ [𝑎 ÷ 𝑏]
}

dispatch(op, 𝑎, 𝑏) ↦ result
```

Pattern matching *is* the dispatch table.

---

## What Buran Trades Away

Coming from Perl, you should know what Buran doesn't provide:

1. **Text processing power** — No regexes, no `s///`, no one-liners
2. **TIMTOWTDI** — One way to do things, not many
3. **Sigil-based context** — No automatic scalar/list behavior
4. **Special variables** — No `$_`, `@_`, `$1`, etc.
5. **CPAN** — No massive module ecosystem (yet)
6. **Quick scripts** — Not designed for command-line one-liners

## What Buran Offers Instead

1. **Structural patterns** — Pattern matching on data structures
2. **Uniform syntax** — Everything is pattern transformation
3. **Immutability** — No accidental mutations
4. **Mathematical notation** — Write `𝑥²` not `$x ** 2`
5. **AI-friendly** — Consistent structure for reliable code generation
6. **Unicode identifiers** — Name functions in any language

---

## When to Use Which

**Use Perl when:**

- Text processing is the primary task
- You need regex power
- Quick one-liners are valuable
- CPAN has the module you need
- Practicality matters more than purity

**Use Buran when:**

- Structured data transformation is the task
- Correctness matters more than brevity
- AI systems are generating code
- Mathematical notation aids readability
- You want uniform, predictable syntax

---

## Philosophical Difference

Perl asks: *"How can I get this done quickly and flexibly?"*

Buran asks: *"What pattern does this pattern become?"*

Perl is pragmatic and flexible — do what works, use whatever syntax fits. Buran is principled and uniform — one concept, applied consistently.

Both approaches have value. Perl's flexibility makes it powerful for practical tasks. Buran's uniformity makes it reliable for formal transformation.

If you love Perl's expressiveness but want something more structured for a different class of problems, Buran offers a complementary paradigm — not a replacement, but an alternative tool for different situations.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
