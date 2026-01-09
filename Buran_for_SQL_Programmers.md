# Buran for SQL Programmers

## Introduction

If you work with SQL, you think declaratively. You don't write loops to find matching rows — you describe what you want, and the database figures out how to get it. `SELECT name FROM users WHERE age > 21` says *what*, not *how*.

Buran shares this declarative philosophy. Instead of describing step-by-step procedures, you describe patterns and transformations. But where SQL operates on tables of rows, Buran operates on patterns of any shape. Where SQL has `WHERE` clauses, Buran has pattern guards. Where SQL has `NULL`, Buran has explicit emptiness.

This guide shows how SQL concepts map to Buran.

---

## The Declarative Connection

Both SQL and Buran are declarative:

| SQL                         | Buran                          |
| --------------------------- | ------------------------------ |
| Describe what data you want | Describe what patterns match   |
| Engine optimizes execution  | Runtime handles transformation |
| Set-based operations        | Pattern-based operations       |
| No explicit loops           | No explicit loops              |

You already think the right way. The syntax is different, but the mindset transfers.

---

## Tables vs Lists of Patterns

### SQL Table

```sql
CREATE TABLE users (
    id INTEGER,
    name TEXT,
    age INTEGER,
    department TEXT
);

-- Data
-- | id | name    | age | department |
-- |----|---------|-----|------------|
-- | 1  | Alice   | 30  | Engineering|
-- | 2  | Bob     | 25  | Marketing  |
-- | 3  | Charlie | 35  | Engineering|
```

### Buran: List of Patterns

```
[list:
    [user: 1, "Alice", 30, "Engineering"],
    [user: 2, "Bob", 25, "Marketing"],
    [user: 3, "Charlie", 35, "Engineering"]
] ↦ users
```

A SQL table is a set of rows. In Buran, that's a list of constructor patterns. Each "row" is a pattern like `[user: id, name, age, department]`.

---

## SELECT and WHERE

### SQL

```sql
SELECT name, age FROM users WHERE age > 25;
```

### Buran

```
filter({ [user: _, name, age, _] | age > 25 }, users) ↦ filtered ↦
map({ [user: _, name, age, _] ↦ [person: name, age] }, filtered)
```

Or combined:

```
users ↦ {
    [list: ] ↦ [list: ]
    [list: [user: _, name, age, _], ⟨rest⟩] | age > 25 ↦
        [list: [person: name, age], ⟨select-over-25(rest)⟩]
    [list: _, ⟨rest⟩] ↦ select-over-25(rest)
}
```

**Key insight:** SQL's `WHERE` becomes a guard (`|`), and `SELECT` columns become a new pattern structure.

### Mapping SQL to Buran

| SQL                 | Buran                       |
| ------------------- | --------------------------- |
| `SELECT col1, col2` | Pattern with desired fields |
| `FROM table`        | Input list                  |
| `WHERE condition`   | Guard `\| condition`        |
| `*` (all columns)   | Full pattern match          |

---

## Filtering (WHERE Clause)

### SQL

```sql
-- Simple condition
SELECT * FROM users WHERE department = 'Engineering';

-- Multiple conditions
SELECT * FROM users WHERE age > 25 AND department = 'Engineering';

-- OR condition
SELECT * FROM users WHERE department = 'Engineering' OR department = 'Marketing';

-- IN clause
SELECT * FROM users WHERE department IN ('Engineering', 'Marketing');
```

### Buran

```
# Simple condition
filter({ [user: _, _, _, dept] | dept = "Engineering" }, users)

# Multiple conditions (AND)
filter({ [user: _, _, age, dept] | age > 25 ∧ dept = "Engineering" }, users)

# OR condition
filter({ [user: _, _, _, dept] | dept = "Engineering" ∨ dept = "Marketing" }, users)

# IN clause (set membership)
filter({ [user: _, _, _, dept] | dept ∈ [set: "Engineering", "Marketing"] }, users)
```

Buran uses proper logical symbols:

- `∧` for AND (not `AND`)
- `∨` for OR (not `OR`)
- `∈` for IN (set membership)

---

## Projection (SELECT Columns)

### SQL

```sql
SELECT name FROM users;
SELECT name, department FROM users;
SELECT UPPER(name) as name FROM users;
```

### Buran

```
# Single field
map({ [user: _, name, _, _] ↦ name }, users)

# Multiple fields
map({ [user: _, name, _, dept] ↦ [person: name, dept] }, users)

# Transformation
map({ [user: _, name, _, _] ↦ uppercase(name) }, users)
```

Projection is `map` with a pattern that extracts what you want.

---

## Aggregation (COUNT, SUM, AVG, etc.)

### SQL

```sql
SELECT COUNT(*) FROM users;
SELECT SUM(age) FROM users;
SELECT AVG(age) FROM users;
SELECT MIN(age), MAX(age) FROM users;
SELECT department, COUNT(*) FROM users GROUP BY department;
```

### Buran

```
# COUNT
length(users)

# SUM
fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], users)

# AVG
fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], users) ↦ total ↦
[total ÷ length(users)]

# MIN, MAX
fold({ acc, [user: _, _, age, _] ↦ min(acc, age) }, [∞], users)
fold({ acc, [user: _, _, age, _] ↦ max(acc, age) }, [−∞], users)

# GROUP BY (more complex)
group-by-department {
    users ↦
        unique-departments(users) ↦ depts ↦
        map({ dept ↦
            [group: dept, length(filter({ [user: _, _, _, d] | d = dept }, users))]
        }, depts)
}
```

`fold` is your aggregate function — it combines all rows into a single result.

---

## ORDER BY (Sorting)

### SQL

```sql
SELECT * FROM users ORDER BY age;
SELECT * FROM users ORDER BY age DESC;
SELECT * FROM users ORDER BY department, age;
```

### Buran

```
# Sort by age (ascending)
sort({ [user: _, _, age1, _], [user: _, _, age2, _] ↦ age1 < age2 }, users)

# Sort by age (descending)
sort({ [user: _, _, age1, _], [user: _, _, age2, _] ↦ age1 > age2 }, users)

# Multi-column sort
sort({
    [user: _, _, age1, dept1], [user: _, _, age2, dept2] ↦
        { [✔] ↦ age1 < age2, [✘] ↦ dept1 < dept2 } ↤ [dept1 = dept2]
}, users)
```

---

## NULL vs Empty Pattern

SQL's `NULL` is notoriously tricky — it's not equal to anything, including itself.

### SQL NULL Problems

```sql
SELECT * FROM users WHERE manager_id = NULL;     -- Never matches!
SELECT * FROM users WHERE manager_id IS NULL;    -- Correct
SELECT NULL = NULL;                               -- Returns NULL, not TRUE
```

### Buran: Explicit Emptiness

```
# Empty pattern
[]

# Check for empty
{
    [] ↦ ["no value"]
    value ↦ ["has value: ", value]
}

# Empty equals empty
[[] = []] ↦ [✔]    # True! No NULL weirdness
```

In Buran, `[]` is a real pattern you can match and compare. No three-valued logic, no `IS NULL` vs `= NULL` confusion.

### Optional Fields

**SQL:**

```sql
SELECT name, COALESCE(nickname, name) as display_name FROM users;
```

**Buran:**

```
map({
    [user: name, [], _] ↦ [display: name, name]        # No nickname
    [user: name, nickname, _] ↦ [display: name, nickname]
}, users)
```

Pattern matching handles presence/absence naturally.

---

## JOINs

### SQL

```sql
-- Inner join
SELECT u.name, d.department_name
FROM users u
JOIN departments d ON u.department_id = d.id;

-- Left join
SELECT u.name, d.department_name
FROM users u
LEFT JOIN departments d ON u.department_id = d.id;
```

### Buran

Buran doesn't have built-in JOIN syntax, but you can express joins through pattern matching:

```
# Inner join
inner-join {
    users, departments ↦
        flatten(map({ [user: _, name, _, dept_id] ↦
            map({ [dept: id, dept_name] | id = dept_id ↦
                [joined: name, dept_name]
            }, departments)
        }, users))
}

# Left join (keep unmatched)
left-join {
    users, departments ↦
        map({ [user: _, name, _, dept_id] ↦
            find-department(dept_id, departments) ↦ {
                [] ↦ [joined: name, []]
                [dept: _, dept_name] ↦ [joined: name, dept_name]
            }
        }, users)
}

find-department {
    id, [list: ] ↦ []
    id, [list: [dept: d_id, name], ⟨rest⟩] | id = d_id ↦ [dept: d_id, name]
    id, [list: _, ⟨rest⟩] ↦ find-department(id, rest)
}
```

This is more verbose than SQL's JOIN syntax — SQL is optimized for relational operations. Buran is general-purpose.

---

## Set Operations

SQL has set operations. So does Buran, with mathematical notation:

### SQL

```sql
SELECT name FROM engineers
UNION
SELECT name FROM managers;

SELECT name FROM engineers
INTERSECT
SELECT name FROM managers;

SELECT name FROM engineers
EXCEPT
SELECT name FROM managers;
```

### Buran

```
# Union
[engineers ∪ managers]

# Intersection
[engineers ∩ managers]

# Difference (EXCEPT)
[engineers ∖ managers]

# For lists, convert to sets first
[set: ⟨engineer-names⟩] ↦ eng-set ↦
[set: ⟨manager-names⟩] ↦ mgr-set ↦
[eng-set ∪ mgr-set]
```

Buran uses standard mathematical symbols: `∪` (union), `∩` (intersection), `∖` (difference).

---

## Subqueries

### SQL

```sql
-- Subquery in WHERE
SELECT * FROM users
WHERE department_id IN (SELECT id FROM departments WHERE budget > 1000000);

-- Subquery in FROM
SELECT avg_age FROM (SELECT AVG(age) as avg_age FROM users);

-- Correlated subquery
SELECT * FROM users u
WHERE age > (SELECT AVG(age) FROM users WHERE department = u.department);
```

### Buran

```
# Subquery in filter
high-budget-depts ↤
    filter({ [dept: _, _, budget] | budget > 1000000 }, departments) ↦ hb ↦
    map({ [dept: id, _, _] ↦ id }, hb) ↦ [set: ⟨ids⟩]

filter({ [user: _, _, _, dept_id] | dept_id ∈ high-budget-depts }, users)

# Nested computation
fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], users) ↦ total ↦
[total ÷ length(users)] ↦ avg-age

# Correlated - compute per group
users-above-dept-avg {
    users ↦
        filter({ [user: _, _, age, dept] |
            age > department-avg(dept, users)
        }, users)
}

department-avg {
    dept, users ↦
        filter({ [user: _, _, _, d] | d = dept }, users) ↦ dept-users ↦
        fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], dept-users) ↦ total ↦
        [total ÷ length(dept-users)]
}
```

Buran handles subqueries through function composition and nested transformations.

---

## INSERT, UPDATE, DELETE

SQL modifies data in place. Buran creates new data.

### SQL (Mutation)

```sql
INSERT INTO users VALUES (4, 'Diana', 28, 'Sales');
UPDATE users SET age = 31 WHERE name = 'Alice';
DELETE FROM users WHERE department = 'Marketing';
```

### Buran (Transformation)

```
# "INSERT" - create new list with addition
[list: ⟨users⟩, [user: 4, "Diana", 28, "Sales"]] ↦ updated-users

# "UPDATE" - transform matching rows
map({
    [user: id, "Alice", _, dept] ↦ [user: id, "Alice", 31, dept]
    row ↦ row
}, users) ↦ updated-users

# "DELETE" - filter out matching rows
filter({ [user: _, _, _, dept] | dept ≠ "Marketing" }, users) ↦ remaining-users
```

Key difference: Buran doesn't modify the original — it produces a new collection. This is safer but requires thinking differently about state.

---

## DISTINCT

### SQL

```sql
SELECT DISTINCT department FROM users;
```

### Buran

```
# Extract departments, convert to set (automatically unique)
map({ [user: _, _, _, dept] ↦ dept }, users) ↦ all-depts ↦
[set: ⟨all-depts⟩] ↦ unique-depts
```

Sets in Buran are automatically unique — that's what sets are.

---

## CASE Expressions

### SQL

```sql
SELECT name,
    CASE
        WHEN age < 25 THEN 'Junior'
        WHEN age < 35 THEN 'Mid'
        ELSE 'Senior'
    END as level
FROM users;
```

### Buran

```
map({
    [user: _, name, age, _] ↦
        [result: name, classify-level(age)]
}, users)

classify-level {
    [age | age < 25] ↦ ["Junior"]
    [age | age < 35] ↦ ["Mid"]
    _ ↦ ["Senior"]
}
```

SQL's `CASE` becomes pattern matching with guards.

---

## Window Functions

SQL window functions are powerful for analytics:

### SQL

```sql
SELECT name, age,
    ROW_NUMBER() OVER (ORDER BY age) as row_num,
    RANK() OVER (ORDER BY age) as rank,
    SUM(age) OVER (PARTITION BY department) as dept_total
FROM users;
```

### Buran

Window functions require carrying context through the computation:

```
# Row numbers (after sorting)
sort({ [user: _, _, a1, _], [user: _, _, a2, _] ↦ a1 < a2 }, users) ↦ sorted ↦
add-row-numbers(sorted, [1])

add-row-numbers {
    [list: ], _ ↦ [list: ]
    [list: row, ⟨rest⟩], 𝑛 ↦
        [list: [numbered: row, 𝑛], ⟨add-row-numbers(rest, [𝑛 + 1])⟩]
}

# Department totals (partition)
add-dept-totals {
    users ↦
        map({ [user: id, name, age, dept] ↦
            [with-total: id, name, age, dept, dept-sum(dept, users)]
        }, users)
}

dept-sum {
    dept, users ↦
        filter({ [user: _, _, _, d] | d = dept }, users) ↦ dept-users ↦
        fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], dept-users)
}
```

This is more verbose than SQL's window function syntax — SQL is purpose-built for these operations.

---

## Transactions

SQL transactions ensure atomicity:

```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

Buran is purely functional — there's no mutable state to protect with transactions. Instead, you compute the entire new state:

```
transfer {
    from-id, to-id, amount, accounts ↦
        map({
            [account: id, balance] | id = from-id ↦ [account: id, balance − amount]
            [account: id, balance] | id = to-id ↦ [account: id, balance + amount]
            account ↦ account
        }, accounts)
}
```

The transformation is atomic by nature — you get the old state or the new state, never something in between.

---

## Reading from Actual Databases

Buran can read structured data from files:

```
# Read JSON data (like a document database)
[File: "users.json" :: json] ↦ users

# Read CSV (tabular, like SQL export)
[File: "users.csv" :: csv] ↦ user-rows
```

Direct database connections would be implementation-specific.

---

## Comparison Summary

| SQL Concept    | Buran Equivalent               |
| -------------- | ------------------------------ |
| Table          | List of patterns               |
| Row            | Constructor pattern            |
| SELECT columns | map with projection pattern    |
| WHERE          | filter with guard              |
| AND / OR       | ∧ / ∨                          |
| IN             | ∈ (set membership)             |
| NULL           | [] (empty pattern)             |
| IS NULL        | Pattern match on []            |
| COALESCE       | Pattern matching               |
| COUNT          | length                         |
| SUM / AVG      | fold                           |
| MIN / MAX      | fold with min/max              |
| GROUP BY       | Custom grouping function       |
| ORDER BY       | sort with comparator           |
| JOIN           | Nested map/filter              |
| UNION          | ∪                              |
| INTERSECT      | ∩                              |
| EXCEPT         | ∖                              |
| DISTINCT       | Convert to set                 |
| CASE           | Pattern matching               |
| INSERT         | Append to list                 |
| UPDATE         | map with conditional transform |
| DELETE         | filter                         |

---

## What SQL Does Better

Be honest: SQL excels at relational queries:

1. **JOIN syntax** — SQL's JOIN is more concise
2. **Query optimization** — Databases optimize execution plans
3. **Indexing** — Databases use indexes for fast lookups
4. **Window functions** — Built-in, optimized syntax
5. **Aggregation** — GROUP BY is very concise
6. **Large datasets** — Databases handle data larger than memory

Buran is a general-purpose language. SQL is specialized for relational data. Use each where it shines.

---

## What Buran Offers

1. **Pattern flexibility** — Not limited to tabular rows
2. **No NULL confusion** — `[]` is explicit and well-behaved
3. **Mathematical notation** — `∪`, `∩`, `∈` are natural
4. **Composability** — Functions compose freely
5. **Type safety** — Patterns encode structure
6. **General computation** — Not limited to queries

---

## Example: Complete Query Translation

### SQL

```sql
SELECT department,
       COUNT(*) as emp_count,
       AVG(age) as avg_age
FROM users
WHERE age >= 25
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY avg_age DESC;
```

### Buran

```
# Start with users
users ↦

# WHERE age >= 25
filter({ [user: _, _, age, _] | age ≥ 25 }, users) ↦ filtered ↦

# GROUP BY department
unique-depts(filtered) ↦ departments ↦
map({ dept ↦
    filter({ [user: _, _, _, d] | d = dept }, filtered) ↦ group ↦
    [dept-stats:
        dept,
        length(group),
        fold({ acc, [user: _, _, age, _] ↦ acc + age }, [0], group) ÷ length(group)
    ]
}, departments) ↦ grouped ↦

# HAVING COUNT(*) >= 2
filter({ [dept-stats: _, count, _] | count ≥ 2 }, grouped) ↦ having-filtered ↦

# ORDER BY avg_age DESC
sort({ [dept-stats: _, _, avg1], [dept-stats: _, _, avg2] ↦ avg1 > avg2 }, having-filtered)

# Helper
unique-depts {
    users ↦
        map({ [user: _, _, _, dept] ↦ dept }, users) ↦ all ↦
        [set: ⟨all⟩]
}
```

More verbose, but explicit about every step.

---

## Summary

SQL and Buran share declarative DNA — you describe *what*, not *how*. SQL is specialized for relational queries and does that brilliantly. Buran is general-purpose, handling any pattern structure with mathematical notation.

If you're processing tabular data at scale: use SQL.
If you need flexible pattern transformation: consider Buran.
If you're moving data between systems: learn both.

Your declarative thinking transfers perfectly. The syntax is just notation.

---

*Buran is in development. Specification and reference implementation expected early 2026.*

© 2026 Danslav Slavenskoj
