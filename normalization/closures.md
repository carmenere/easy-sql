# Table of contents
- [Table of contents](#table-of-contents)
- [Closures](#closures)
  - [Set of functional dependencies](#set-of-functional-dependencies)
  - [Attribute closure](#attribute-closure)
    - [Example 1](#example-1)
    - [Example 2](#example-2)
  - [FD closure](#fd-closure)
    - [Find candidate key by clojure](#find-candidate-key-by-clojure)
      - [Example](#example)
- [Closure example 1](#closure-example-1)
  - [All attribute closures](#all-attribute-closures)
  - [FD closure](#fd-closure-1)
    - [Trivial](#trivial)
    - [Infered from given FD set](#infered-from-given-fd-set)
    - [Violates 3NF](#violates-3nf)
- [Closure example 2](#closure-example-2)
  - [All attribute closures](#all-attribute-closures-1)
  - [FD closure](#fd-closure-2)
    - [Trivial](#trivial-1)
    - [Infered from given FD set](#infered-from-given-fd-set-1)
    - [Violates BCNF](#violates-bcnf)

<br>

# Closures
## Set of functional dependencies
**Set of FDs** `F` of a relation is the **set** of **all** FDs presented in this relation.<br>
For instance, there is given relation `R(A, B, C, D, E, F)` and its **FD set** `F = {AB->C, BC->AD, D->E, CF->B}`. 

<br>

## Attribute closure
The **closure** of **attribute set** `X` (or just **attribute closure**), denoted as `X+`, with respect to **FD set** `F` is a set of **all** attributes which can be functionally determined from *attribute set* `X`.<br>

Assume that there are some relation `R`, some `F` and the **set of attributes** `X = {A}`.<br>
The general algorithm to find **attribute closure** `X+` of `X`:
1. Add `A` and `B` to the result set `X+` (every set of attributes `X` is **trivially** dependent on itself).
2. Apply **Armstrong's inference rules** to determine new attributes that are functionally dependent on the attributes already contained in `X+`.
   - if such attributes exists add them to the result set `X+`
3. Repeat step **2** until no more attributes can be added to the result set `X+`.

<br>

Consider ralation `R = (A, B, C, D)` and `F = {{A,B} -> {D}, {B} -> {C}}`.<br>
Then, consider trivial FD: `{A,B} -> {A,B}`, so `{A,B}` can identify all attributes in `R` using attributes `{A,B}`.<br>
`{A,B}` is also called **candidate** key and by using **candidate** key we can identify complete tuple `(a, b, c, d)`.<br>
`{A,B}` is also called closure and denoted `{A,B}+`: `{A,B}+ = {A, B, C, D}`.<br>
Attribus `A` and `B` are **prime attributes**.<br>
Attribus `C` and `D` are **non-prime attributes**.<br>

<br>

### Example 1
Assume that there is the relation `R(A, B, C, D, E)` and `F = {AB -> CD, D -> E, A -> C, B -> D}`.<br>

We can determine `A+` as:    
1. Add `A` to the result set `A+ = {A}`.
2. Get all the attribute which are derived from `A` and add them to the result set `A+`:
   - `A -> C` (from given **set of FDs**);
   - add `C` to the `A+`;
   - now `A+` contains `{A,C}`;
3. No other attribute can be derived from `A` and `C` hence.

**Result**: `A+ = {A,C}`.

<br>

### Example 2
Assume that there is the relation `R(A, B, C, D, E)` and `F = {AB -> CD, D -> E, A -> C, B -> D}`.<br>

We can determine `B+` as:    
1. Add `B` to the result set `B+ = {B}`.
2. Get all the attribute which are derived from `B`:
   - `B -> D` (from given **set of FDs**);
   - add `D` to the `B+`;
   - now `B+` contains `{B,D}`;
3. Get all the attribute which are either derived by `D` or any combination of `BD` (all subsets of `B+` excluding `{B} and {}`)
   - `D -> E`
   - add `E` to the `B+`;
   - now `B+` contains `{B,D,E}`;
4. No other attribute can be derived from `B`, `D`, `E` or from any subset of `{B, D, E}` hence.

**Result**: `B+ = {B,D,E}`.

<br>

## FD closure
The **closure** of **set of FDs** (or just **FD closure**) `F`, denoted as `F+`, is the **set** of **all regular FDs** that can be derived from `F`.<br>
There are **inference rules** (aka **Armstrong's inference rules**) to infer **all regular FDs** from `F`.<br>
**Armstrong's inference rules** are **sufficient** to infer `F+` from given `F`.<br>
Some inferred FDs are trivial: each **dependent** is a **subset** of the **determinant**, e.g., `AB → A`, `AB → B`.

<br>

### Find candidate key by clojure
Let `F` is a set of FDs, and `R` a relation.<br>

A **candidate key** is a set `C` of attributes in `R` such that
- `C+` includes all the attributes in `R`;
- there is no **proper subset** `S` of `C` such that `S+` includes **all** the attributes in `R`.

<br>

A **proper subset** `S` is a **subset** of `C` such that `S != C`.

<br>

#### Example
Assume that there is relation `R(A, B, C , D)` and `F = {A → B, B → C }`:
- `A` is **not** a candidate key `C`, because `A+ = {A, B, C }` which does **not** include `D`.
- `ABD+ = {A, B, C , D}`, but `ABD` is **not** a candidate key `C` because there is a **proper subset** `AD` of `ABD` such that `AD+` includes all the attributes: `AD+ = {A, B, C , D}`.
- `AD` is a **candidate key** `C`.

<br>

# Closure example 1
Assume that there is relation `R(A, B, C)` and `F = {{A} → {B}, {B} → {C}}`.

<br>

## All attribute closures
- `{}+ = {}`
- `{A}+ = {A,B,C}` **Candidate** key
- `{B}+ = {B,C}`
- `{C}+ = {C}`
- `{A,B}+ = {A,B,C}` **Super** key
- `{A,C}+ = {a,c,b}` **Super** key
- `{B,C}+ = {B,C}`
- `{A,B,C}+ = {A,B,C}` **Super** key

<br>

## FD closure
`F+` is **set** of following FDs (some of them are **trivial**)

### Trivial
- `{A} → {A}`
- `{B} → {B}`
- `{C} → {C}`
- `{A,B} → {A,B}`
- `{A,B} → {A}`
- `{A,B} → {B}`
- `{A,C} → {A,C}`
- `{A,C} → {A}`
- `{A,C} → {C}`
- `{B,C} → {B,C}`
- `{B,C} → {B}`
- `{B,C} → {C}`
- `{A,B,C} → {A,B,C}`
- `{A,B,C} → {A,B}`
- `{A,B,C} → {A,C}`
- `{A,B,C} → {A}`
- `{A,B,C} → {B,C}`
- `{A,B,C} → {B}`
- `{A,B,C} → {C}`

<br>

### Infered from given FD set
- `{A} → {A,B,C}`
- `{A} → {A,B}`
- `{A} → {A,C}`
- `{A} → {B,C}`
- `{A} → {B}`
- `{A} → {C}`
- `{A,B} → {A,B,C}`
- `{A,B} → {A,C}`
- `{A,B} → {B,C}`
- `{A,B} → {C}`
- `{A,C} → {A,B,C}`
- `{A,C} → {A,B}`
- `{A,C} → {B,C}`
- `{A,C} → {B}`

### Violates 3NF
- `{B} → {B,C}`
- `{B} → {C}`

<br>

# Closure example 2
Assume that there is relation `R(A, B, C, D)` and `F = {{A,B,C}->{D}, {D}->{A}`.

<br>

## All attribute closures
- `{}+ = {}`
- `{a}+ = {a}`
- `{b}+ = {b}`
- `{c}+ = {c}`
- `{d}+ = {d,a}`
- `{a,b}+ = {a,b}`
- `{a,c}+ = {a,c}`
- `{a,d}+ = {a,d}`
- `{b,c}+ = {b,c}`
- `{b,d}+ = {b,d,a}`
- `{c,d}+ = {c,d,a}`
- `{a,b,c}+ = {a,b,c,d}` **Candidate** key
- `{a,b,d}+ = {a,b,d}`
- `{a,c,d}+ = {a,c,d}`
- `{b,c,d}+ = {b,c,d,a}` **Candidate** key
- `{a,b,c,d}+ = {a,b,c,d}` **Super** key

<br>

## FD closure
`F+` is **set** of following FDs (some of them are **trivial**)

<br>

### Trivial
- `{a} → {a}`
- `{b} → {b}`
- `{c} → {c}`
- `{d} → {d}`
- `{a,b} → {a,b}`
- `{a,b} → {a}`
- `{a,b} → {b}`
- `{a,c} → {a,c}`
- `{a,c} → {a}`
- `{a,c} → {c}`
- `{a,d} → {a,d}`
- `{a,d} → {a}`
- `{a,d} → {d}`
- `{b,c} → {b,c}`
- `{b,c} → {b}`
- `{b,c} → {c}`
- `{b,d} → {b,d}`
- `{b,d} → {b}`
- `{b,d} → {d}`
- `{c,d} → {c,d}`
- `{c,d} → {c}`
- `{c,d} → {d}`
- `{a,b,c} → {a,b,c}`
- `{a,b,c} → {a,b}`
- `{a,b,c} → {a,c}`
- `{a,b,c} → {a}`
- `{a,b,c} → {b,c}`
- `{a,b,c} → {b}`
- `{a,b,c} → {c}`
- `{a,b,d} → {a,b,d}`
- `{a,b,d} → {a,b}`
- `{a,b,d} → {a,d}`
- `{a,b,d} → {a}`
- `{a,b,d} → {b,d}`
- `{a,b,d} → {b}`
- `{a,b,d} → {d}`
- `{a,c,d} → {a,c,d}`
- `{a,c,d} → {a,c}`
- `{a,c,d} → {a,d}`
- `{a,c,d} → {a}`
- `{a,c,d} → {c,d}`
- `{a,c,d} → {c}`
- `{a,c,d} → {d}`
- `{b,c,d} → {b,c,d}`
- `{b,c,d} → {b,c}`
- `{b,c,d} → {b,d}`
- `{b,c,d} → {b}`
- `{b,c,d} → {c,d}`
- `{b,c,d} → {c}`
- `{b,c,d} → {d}`
- `{a,b,c,d} → {a,b,c,d}`
- `{a,b,c,d} → {a,b,c}`
- `{a,b,c,d} → {a,b,d}`
- `{a,b,c,d} → {a,b}`
- `{a,b,c,d} → {a,c,d}`
- `{a,b,c,d} → {a,c}`
- `{a,b,c,d} → {a,d}`
- `{a,b,c,d} → {a}`
- `{a,b,c,d} → {b,c,d}`
- `{a,b,c,d} → {b,c}`
- `{a,b,c,d} → {b,d}`
- `{a,b,c,d} → {b}`
- `{a,b,c,d} → {c,d}`
- `{a,b,c,d} → {c}`
- `{a,b,c,d} → {d}`

<br>

### Infered from given FD set
- `{a,b,c} → {a,b,c,d}`
- `{a,b,c} → {a,b,d}`
- `{a,b,c} → {a,c,d}`
- `{a,b,c} → {a,d}`
- `{a,b,c} → {b,c,d}`
- `{a,b,c} → {b,d}`
- `{a,b,c} → {c,d}`
- `{a,b,c} → {d}`
- `{b,c,d} → {a,b,c,d}`
- `{b,c,d} → {a,b,c}`
- `{b,c,d} → {a,b,d}`
- `{b,c,d} → {a,b}`
- `{b,c,d} → {a,c,d}`
- `{b,c,d} → {a,c}`
- `{b,c,d} → {a,d}`
- `{b,c,d} → {a}`

### Violates BCNF
- `{d} → {a,d}`
- `{d} → {a}`
- `{b,d} → {a,b,d}`
- `{b,d} → {a,b}`
- `{b,d} → {a,d}`
- `{b,d} → {a}`
- `{c,d} → {a,c,d}`
- `{c,d} → {a,c}`
- `{c,d} → {a,d}`
- `{c,d} → {a}`
