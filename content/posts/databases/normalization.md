+++
title = 'Database Normalization: 1NF to 5NF'
date = 2026-01-25T12:30:00+05:30
draft = false
tags = ['databases', 'normalization', 'design']
summary = 'Complete guide to database normalization covering 1NF through 5NF with practical examples, smell tests, and denormalization guidelines.'
mermaid = true
+++

***

## What is Normalization?

**Normalization** is the process of structuring relational data to **reduce redundancy** and **prevent update/insert/delete anomalies**, by organizing data into tables that reflect clear dependencies and relationships.

### Why it matters

* Prevents inconsistent duplicate values (same fact stored in multiple places).
* Makes updates safe (change once, not everywhere).
* Keeps relationships explicit and enforceable (keys + constraints).
* Improves long-term maintainability (especially in OLTP systems).

### How to tell if data is "normalized"

A schema is "normalized to **N-th Normal Form**" when it satisfies **all normal forms up to N**.

> Example: If a schema is in **3NF**, it is also in **1NF** and **2NF**.

A practical mindset:

* **1NF:** "Are values atomic and rows well-formed?"
* **2NF:** "Does every non-key attribute depend on the whole key?"
* **3NF:** "Do non-key attributes depend only on the key (not on other non-keys)?"
* **BCNF:** "Is every determinant a candidate key?"
* **4NF:** "Are independent multi-valued facts split apart?"
* **5NF:** "Are there join dependencies that force decomposition?"

### Common anomalies (what normalization prevents)

* **Update anomaly:** changing a repeated fact requires updating many rows → inconsistent states.
* **Insert anomaly:** cannot insert a fact because unrelated fact is missing.
* **Delete anomaly:** deleting a row accidentally deletes a needed fact.

### Quick glossary

* **Key / Candidate key:** minimal attribute set that uniquely identifies a row.
* **PK:** chosen candidate key.
* **Determinant:** an attribute/set that determines another attribute (X → Y).
* **Functional dependency (FD):** X → Y (X uniquely determines Y).
* **MVD (multi-valued dependency):** X ↠ Y (for one X, multiple Ys independent of other attributes).

### Quick Reference Table (All Normal Forms)

```text
┌───────┬─────────────────────────────┬────────────────────────────────┬─────────────────────────────────┐
│ Form  │ Violation                   │ Fix                            │ Key Question                    │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ 1NF   │ Multi-valued columns,       │ Split into child table,        │ Are all values atomic?          │
│       │ repeating groups            │ ensure PK exists               │                                 │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ 2NF   │ Partial dependency          │ Move attributes to table       │ Does every non-key depend on    │
│       │ (attr depends on part of    │ where that key-part is PK      │ the WHOLE key?                  │
│       │ composite PK)               │                                │                                 │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ 3NF   │ Transitive dependency       │ Extract into lookup table      │ Do non-keys depend only on      │
│       │ (non-key → non-key)         │ keyed by the determinant       │ keys (not other non-keys)?      │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ BCNF  │ Non-key determinant         │ Make determinant a key in      │ Is every determinant a          │
│       │ (X → Y where X not a key)   │ its own table                  │ candidate key?                  │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ 4NF   │ Independent MVDs            │ Split independent multi-valued │ Are multi-valued facts          │
│       │ (X ↠ Y, X ↠ Z independent)  │ facts into separate tables     │ independent of each other?      │
├───────┼─────────────────────────────┼────────────────────────────────┼─────────────────────────────────┤
│ 5NF   │ Join dependency             │ Decompose only if lossless     │ Does decomposition create       │
│       │ (ternary can't be split)    │ and business rules allow       │ spurious tuples on rejoin?      │
└───────┴─────────────────────────────┴────────────────────────────────┴─────────────────────────────────┘
```

**Hierarchy:** 1NF ⊂ 2NF ⊂ 3NF ⊂ BCNF ⊂ 4NF ⊂ 5NF

**Practical target:** Most OLTP systems aim for **3NF** (or BCNF if business rules demand it).

***

## 1NF — First Normal Form

### When is something in 1NF?

A table is in **1NF** if:

* Each column contains **atomic** (indivisible) values (no lists/arrays/CSV strings).
* Each row is uniquely identifiable (a **key** exists).
* No repeating groups like `Phone1, Phone2, Phone3`.

### When 1NF is used

* Always used as the **baseline** for relational design (OLTP and reporting staging).
* Used when you see:
  * comma-separated values in a column,
  * repeated "slot" columns (`Item1, Item2, Item3`),
  * "multi-value" cells that require parsing.

### 1NF checklist / recipe

* Find columns that hold multiple values (lists).
* Split repeating groups into a **child table**.
* Ensure each table has a **primary key**.

### Short, effective example (1NF)

**Not 1NF** (multi-valued column):

```text
Student(StudentId, Name, Phones)
1, "Asha", "9001,9002"
```

**1NF fix**:

```text
Student(StudentId, Name)
1, "Asha"

StudentPhone(StudentId, Phone)
1, "9001"
1, "9002"
```

#### Mermaid (after 1NF)

```mermaid
erDiagram
  STUDENT ||--o{ STUDENT_PHONE : has
  STUDENT {
    int StudentId PK
    string Name
  }
  STUDENT_PHONE {
    int StudentId FK
    string Phone
  }
```

***

## 2NF — Second Normal Form

### When is something in 2NF?

A table is in **2NF** if:

* It is already in **1NF**, and
* Every non-key attribute depends on the **entire** primary key (no **partial dependency**).

> Partial dependency only happens when the PK is **composite** (e.g., (OrderId, ProductId)).

### When 2NF is used

* Used when you have **composite primary keys** (common in junction tables, line items).
* Used when you notice:
  * attributes that "belong to" only one part of the composite key,
  * repeated descriptive data across many line rows (product name repeated per order line).

### 2NF checklist / recipe

* Identify the **primary key** (especially if composite).
* For each non-key attribute, ask:\
    "Does this depend on the whole key, or only part of it?"
* If it depends on part → move it to a new table where that part is the key.

### Short, effective example (2NF)

**Not 2NF**:

```text
OrderLine(OrderId, ProductId, ProductName, UnitPrice, Qty)
PK = (OrderId, ProductId)

ProductName and UnitPrice depend only on ProductId (part of the key).
```

**2NF fix**:

```text
Product(ProductId, ProductName, UnitPrice)

OrderLine(OrderId, ProductId, Qty)
PK = (OrderId, ProductId)
```

#### Mermaid (after 2NF)

```mermaid
erDiagram
  ORDER ||--o{ ORDER_LINE : contains
  PRODUCT ||--o{ ORDER_LINE : "is in"
  ORDER {
    int OrderId PK
    date OrderDate
  }
  PRODUCT {
    int ProductId PK
    string ProductName
    decimal UnitPrice
  }
  ORDER_LINE {
    int OrderId FK
    int ProductId FK
    int Qty
  }
```

### Determining Foreign Key Placement

After splitting a table during normalization, you need to establish the relationship between the new tables. Use this question:

> **"Does table1 have many table2s, or does table2 have many table1s?"**

The table on the **"many" side** gets the foreign key pointing to the "one" side.

#### Example: Category and Subject

After normalizing, you have two tables: `Category` and `Subject`. Ask:

* Does a subject have many categories? No.
* Does a category have many subjects? **Yes.**

Since a category has many subjects, the **Subject table** gets the `CategoryId` foreign key.

```mermaid
erDiagram
  CATEGORY ||--o{ SUBJECT : contains
  CATEGORY {
    int CategoryId PK
    string CategoryName
  }
  SUBJECT {
    int SubjectId PK
    string SubjectName
    int CategoryId FK
  }
```

#### General Rule

```text
┌─────────────────────────────────────────────────────────────┐
│ If A has many B → B gets the FK pointing to A               │
│ If B has many A → A gets the FK pointing to B               │
│ If both have many of each other → junction table needed     │
└─────────────────────────────────────────────────────────────┘
```

This same question applies whenever you decompose tables during any normalization step (1NF, 2NF, 3NF).

***

## 3NF — Third Normal Form

### When is something in 3NF?

A table is in **3NF** if:

* It is already in **2NF**, and
* No non-key attribute depends on another non-key attribute (**no transitive dependency**).

Practical test:

* If you can say **PK → A** and **A → B**, and **B** is not a key attribute → you likely violate 3NF.

### When 3NF is used

* Used as the **practical target for most OLTP schemas**.
* Used when you see "lookup data" embedded in transactional tables:
  * `DeptName` stored in `Employee`,
  * `CityName` stored with `ZipCode`,
  * `CustomerTierName` stored on every order.

### 3NF checklist / recipe

* List non-key attributes.
* For each non-key attribute **A**, ask:\
    "Does A determine some other non-key attribute B?"
* If yes → split: move B into a table keyed by A (or by a proper key).

### Short, effective example (3NF)

**Not 3NF**:

```text
Employee(EmployeeId, DeptId, DeptName)
PK = EmployeeId

DeptId -> DeptName (non-key determines non-key)
So EmployeeId -> DeptId -> DeptName (transitive dependency)
```

**3NF fix**:

```text
Employee(EmployeeId, DeptId)

Department(DeptId, DeptName)
```

#### Mermaid (after 3NF)

```mermaid
erDiagram
  DEPARTMENT ||--o{ EMPLOYEE : employs
  DEPARTMENT {
    int DeptId PK
    string DeptName
  }
  EMPLOYEE {
    int EmployeeId PK
    int DeptId FK
  }
```

### The Codd Mnemonic

A famous way to remember the first three normal forms:

> **"The key, the whole key, and nothing but the key, so help me Codd."**

* **The key** (1NF) — every table must have a primary key
* **The whole key** (2NF) — non-key attributes must depend on the *entire* key, not just part of it
* **Nothing but the key** (3NF) — non-key attributes must depend *only* on the key, not on other non-key attributes

### Smell Test for 3NF Violations

> **"If changing one non-key value forces you to change another non-key value in the same row, you likely have a 3NF violation."**

Example: If updating `DeptId` in an `Employee` row means you must also update `DeptName` to stay consistent → transitive dependency → violates 3NF.

***

## BCNF — Boyce–Codd Normal Form

### When is something in BCNF?

A table is in **BCNF** if:

* For every functional dependency **X → Y**, **X** is a **superkey** (i.e., X uniquely identifies a row).

Why BCNF exists:

* 3NF can still allow subtle redundancy when **a non-key determinant** exists.

### When BCNF is used

* Used when there are **non-obvious business rules** that create functional dependencies:
  * "Each employee belongs to exactly one union branch" (branch determines something else),
  * "Each instructor teaches exactly one course".
* Used when 3NF still leaves:
  * duplication that causes anomalies,
  * determinants that aren't modeled as keys.

### BCNF checklist / recipe

* Identify real functional dependencies (business rules), not just "what happens today".
* For each FD **X → Y**, check if **X** is a candidate key.
* If not, decompose so that determinants become keys in their own tables.

### Short, effective example (BCNF)

Business rule: *each instructor teaches exactly one course; a course may have many instructors.*

**Not BCNF**:

```text
Teaching(Instructor, Course, Room)
FD: Instructor -> Course
But (Instructor, Room) might be treated as the key while Instructor isn't modeled as a key,
creating redundancy and anomalies.
```

**BCNF fix (one valid approach)**:

```text
InstructorCourse(Instructor, Course)   // Instructor is key here
CourseRoom(Course, Room)              // if room depends on course (or course+slot)
```

> BCNF decompositions depend on actual rules; the key is spotting "determinant not a key".

***

## 4NF — Fourth Normal Form (Multi-valued dependencies)

### When is something in 4NF?

A table is in **4NF** if:

* It is in **BCNF**, and
* It has **no non-trivial multi-valued dependencies** except those where the determinant is a key.

When 4NF matters:

* When one entity has **two independent multi-valued facts**, and you store them together causing combinatorial duplication.

### When 4NF is used

* Used when you model:
  * "people have many skills and many languages" (independent lists),
  * "products have many tags and many suppliers" (independent lists),
  * any scenario where lists multiply into redundant combinations.
* Used when you see a table whose row count explodes due to **cross-product duplication**.

### 4NF checklist / recipe

* Look for tables where a key (X) has multiple values of Y and multiple values of Z **independently**.
* If Y and Z are independent, split into:
  * X–Y and X–Z tables.

### Short, effective example (4NF)

**Not 4NF** (independent multi-valued attributes):

```text
PersonSkillLanguage(PersonId, Skill, Language)

One person can have many skills and many languages, independently.
This creates redundant combinations (every skill paired with every language).
```

**4NF fix**:

```text
PersonSkill(PersonId, Skill)
PersonLanguage(PersonId, Language)
```

#### Mermaid (after 4NF)

```mermaid
erDiagram
  PERSON ||--o{ PERSON_SKILL : has
  PERSON ||--o{ PERSON_LANGUAGE : speaks
  PERSON {
    int PersonId PK
    string Name
  }
  PERSON_SKILL {
    int PersonId FK
    string Skill
  }
  PERSON_LANGUAGE {
    int PersonId FK
    string Language
  }
```

### Smell Test for 4NF Violations (Cartesian Explosion)

> **"If your row count equals the product of two independent list sizes, you likely have a 4NF violation."**

Example: A person has 3 skills and 4 languages.

* **4NF violation:** `PersonSkillLanguage` table has 3 × 4 = **12 rows** (every skill paired with every language)
* **4NF compliant:** `PersonSkill` has **3 rows** + `PersonLanguage` has **4 rows** = **7 rows total**

```text
┌────────────────────────────────────────────────────────────────┐
│ Quick check: rows = listA_size × listB_size?                   │
│ If yes → the lists are probably independent → split them       │
└────────────────────────────────────────────────────────────────┘
```

The Cartesian explosion wastes storage and creates update anomalies (adding a new skill requires adding rows for every language).

***

## 5NF — Fifth Normal Form (Join dependencies; rare)

### When is something in 5NF?

A table is in **5NF** if:

* It cannot be decomposed further without losing information, and
* Any join dependency is implied by candidate keys (decompositions are lossless and necessary).

When 5NF appears:

* In complex "three-way relationship" cases where storing all combinations creates redundancy, but decomposing must be done carefully to avoid generating invalid combinations upon re-join.

### When 5NF is used

* Used in rare cases with **ternary relationships** and strict business constraints:
  * Supplier–Part–Project style relationships where not all combinations are valid.
* Used when:
  * a 3-column relationship table contains redundancy, and
  * naive decomposition into binary tables creates **spurious (invalid) combinations** after re-joining.

### 5NF checklist / recipe (practical)

* If you see a table representing a **ternary relationship** (A–B–C), ask:
  * Are all combinations valid?
  * Or are there constraints that make some combinations invalid?
* Test decomposition:
  * If splitting into three binary relations and joining them back creates rows that never existed → 5NF concern.
* Apply 5NF only when:
  * the business constraints are well-defined,
  * you can prove joins don't invent invalid combinations.

### Short example (intuition)

```text
SupplierPartProject(Supplier, Part, Project)

If you decompose to:
SupplierPart(Supplier, Part)
SupplierProject(Supplier, Project)
PartProject(Part, Project)

Joining can generate Supplier-Part-Project combinations that were never valid in reality.
```

> 5NF is uncommon in everyday OLTP; it's mainly for specialized modeling.

***

## Running Example: Course Registration System (1NF → 3NF)

This example uses **one consistent scenario** to show how normalization progressively improves a schema.

### Starting Point: Unnormalized Data

```text
CourseRegistration(
  StudentId, StudentName, StudentEmail,
  Courses,                              -- "CS101,CS102,CS103"
  InstructorIds,                        -- "I01,I02"
  DeptId, DeptName, DeptBuilding
)
```

**Sample row:**

```text
(S001, "Asha", "asha@uni.edu", "CS101,CS102", "I01,I02", D01, "Computer Science", "Tech Hall")
```

**Problems:**

* `Courses` and `InstructorIds` are comma-separated lists
* `DeptName` and `DeptBuilding` repeat for every student in that dept
* Can't easily query "all students in CS101"

***

### Step 1: Apply 1NF (Atomic Values)

**Violation:** Multi-valued columns (`Courses`, `InstructorIds`)

**Fix:** Split lists into child tables

```text
Student(StudentId, StudentName, StudentEmail, DeptId, DeptName, DeptBuilding)

StudentCourse(StudentId, CourseId)

StudentInstructor(StudentId, InstructorId)
```

```mermaid
erDiagram
  STUDENT ||--o{ STUDENT_COURSE : registers
  STUDENT ||--o{ STUDENT_INSTRUCTOR : "assigned to"
  STUDENT {
    string StudentId PK
    string StudentName
    string StudentEmail
    string DeptId
    string DeptName
    string DeptBuilding
  }
  STUDENT_COURSE {
    string StudentId FK
    string CourseId
  }
  STUDENT_INSTRUCTOR {
    string StudentId FK
    string InstructorId
  }
```

**Result:** 1NF — all values atomic, PKs defined

**Remaining problems:** `DeptName`, `DeptBuilding` still duplicated per student

***

### Step 2: Apply 2NF (No Partial Dependencies)

**Check:** Does `Student` have a composite PK? No — PK is just `StudentId`.

**Result:** Already in 2NF (2NF only applies when composite keys exist)

> Note: If `StudentCourse` had `CourseName` stored in it, that would be a 2NF violation (CourseName depends only on CourseId, not the full key StudentId+CourseId).

***

### Step 3: Apply 3NF (No Transitive Dependencies)

**Violation:** In `Student` table:

```text
StudentId → DeptId → DeptName, DeptBuilding
```

`DeptName` and `DeptBuilding` depend on `DeptId`, not directly on `StudentId` (transitive dependency).

**Fix:** Extract `Department` as its own table

```text
Student(StudentId, StudentName, StudentEmail, DeptId)

Department(DeptId, DeptName, DeptBuilding)

StudentCourse(StudentId, CourseId)

StudentInstructor(StudentId, InstructorId)
```

```mermaid
erDiagram
  DEPARTMENT ||--o{ STUDENT : has
  STUDENT ||--o{ STUDENT_COURSE : registers
  STUDENT ||--o{ STUDENT_INSTRUCTOR : "assigned to"
  DEPARTMENT {
    string DeptId PK
    string DeptName
    string DeptBuilding
  }
  STUDENT {
    string StudentId PK
    string StudentName
    string StudentEmail
    string DeptId FK
  }
  STUDENT_COURSE {
    string StudentId FK
    string CourseId FK
  }
  STUDENT_INSTRUCTOR {
    string StudentId FK
    string InstructorId FK
  }
```

**Result:** 3NF — no transitive dependencies, each fact stored once

***

### Summary: Before and After

| Aspect              | Before (Unnormalized)            | After (3NF)                     |
|---------------------|----------------------------------|---------------------------------|
| **Redundancy**      | DeptName repeated per student    | DeptName stored once            |
| **Update anomaly**  | Change dept name in many rows    | Change in one place             |
| **Insert anomaly**  | Can't add dept without students  | Dept exists independently       |
| **Delete anomaly**  | Delete last student loses dept   | Dept preserved                  |
| **Query ease**      | Parse CSV to find courses        | Simple JOIN on StudentCourse    |

***

## Normalization "recipes" — step-by-step workflow (real-project friendly)

### Recipe A: Fast 1NF → 3NF normalization

#### Step 1: Make it 1NF

* Remove repeating groups / lists into child tables.
* Ensure each table has a primary key.

#### Step 2: Make it 2NF (only if composite keys exist)

* For composite keys, move attributes dependent on **part** of the key into separate tables.

#### Step 3: Make it 3NF

* Extract attributes that depend on other non-key attributes (transitive dependencies).

#### Step 4: Add constraints

* Add uniqueness, required-ness, relationships, and simple domain rules:
  * `UNIQUE`, `NOT NULL`, FKs, `CHECK`.

#### Step 5: Validate with anomaly tests

* Ask:
  * Can I update a fact in one place only?
  * Can I insert an entity without unrelated info?
  * Can I delete a record without losing unrelated facts?

***

## "How do I know which NF I'm at?" — quick tests

* **1NF test:** any column storing multiple values? any repeating columns?
* **2NF test:** any non-key attribute depends only on a subset of a composite PK?
* **3NF test:** any non-key attribute depends on another non-key attribute?
* **BCNF test:** any FD where determinant isn't a candidate key?
* **4NF test:** independent multi-valued lists causing cross-product duplication?
* **5NF test:** ternary relationship where decomposition creates invalid recombinations?

***

## Practical best practices (OLTP + Reporting)

### OLTP (transactional systems)

* Normalize to **3NF** (often enough for correctness).
* Apply **BCNF** when business rules create determinants that aren't keys.
* Add constraints early; they're part of the model.
* Denormalize only with measured performance reasons + a clear maintenance strategy.

### Reporting (analytics)

* Don't force OLTP normalization into reporting models.
* Use **star schema** (facts + dimensions), intentionally denormalized for query speed and usability.
* Keep OLTP as source of truth; build reporting projections with CDC/ETL/ELT.

## When to Denormalize (Decision Guide)

Denormalization means **intentionally breaking normal form rules** to optimize for specific use cases. It's a trade-off, not a failure.

### When Denormalization Makes Sense

| Scenario                            | Why Denormalize                            | Example                                    |
|-------------------------------------|--------------------------------------------|--------------------------------------------|
| **Read-heavy dashboards**           | Avoid expensive JOINs on every request     | Store `CustomerName` on `Order` table      |
| **Reporting/Analytics**             | Star schema is faster for aggregations     | Dimension tables with flattened attributes |
| **Caching derived values**          | Avoid repeated calculations                | Store `OrderTotal` instead of computing    |
| **Historical snapshots**            | Preserve point-in-time values              | Store `Price` at time of order (not FK)    |
| **Reducing latency-critical JOINs** | Sub-millisecond reads can't afford JOINs   | Embed lookup values in hot tables          |
| **Simplifying queries**             | Analysts can query without complex JOINs   | Pre-joined summary tables                  |

### When NOT to Denormalize

| Scenario                          | Why Stay Normalized                         |
|-----------------------------------|---------------------------------------------|
| **Write-heavy OLTP**              | Updates become complex (multiple places)    |
| **Data frequently changes**       | Keeping denormalized copies in sync is hard |
| **No measured performance issue** | Premature optimization adds complexity      |
| **Source of truth / master data** | Normalization ensures single source         |
| **Compliance/audit requirements** | Redundancy can cause conflicting records    |

### Denormalization Checklist (Before You Do It)

Ask yourself:

1. **Is there a measured problem?**
   * Don't denormalize based on assumptions — profile first
   * Identify the specific query/latency issue

2. **What is the source of truth?**
   * Which table holds the "real" value?
   * Document this clearly

3. **How will you keep data in sync?**
   * Synchronous (same transaction)?
   * Async trigger/event?
   * Batch refresh?

4. **What's the acceptable staleness?**
   * Real-time (must be same transaction)?
   * Seconds? Minutes? Hours?

5. **What happens when source changes?**
   * Do you need to update historical records?
   * Or preserve point-in-time values?

### Common Denormalization Patterns

#### 1. Cached Aggregates

```text
-- Normalized: calculate on every read
SELECT COUNT(*) FROM OrderLine WHERE OrderId = ?

-- Denormalized: store count on Order
Order(OrderId, ..., LineItemCount)
-- Update LineItemCount when lines change
```

**Sync strategy:** Trigger or application code on insert/delete

#### 2. Snapshot Values (Point-in-Time)

```text
-- Normalized: always get current price
OrderLine(OrderId, ProductId, Qty)
JOIN Product ON ... to get Price

-- Denormalized: capture price at order time
OrderLine(OrderId, ProductId, Qty, UnitPriceAtOrder)
```

**Rationale:** Price changes shouldn't affect past orders

#### 3. Embedded Lookup (Avoid JOINs)

```text
-- Normalized
Order(OrderId, CustomerId, ...)
JOIN Customer to get CustomerName

-- Denormalized (for display/reports)
Order(OrderId, CustomerId, CustomerName, ...)
```

**Sync strategy:** Update on customer name change (or accept staleness)

#### 4. Materialized Views / Summary Tables

```text
-- Base tables stay normalized
-- Create pre-joined summary for reporting
DailySalesSummary(Date, ProductId, ProductName, CategoryName, TotalQty, TotalRevenue)
```

**Sync strategy:** Nightly rebuild or incremental refresh

### Red Flags: Signs of Bad Denormalization

* Multiple "sources of truth" with no clear owner
* No documented sync strategy
* Inconsistent values found during audits
* Updates require touching many tables
* "We denormalized because JOINs are slow" (without measuring)

> **Golden rule:** Denormalize with intention, document the trade-off, and maintain sync discipline.

## Mini Self-Check Questions

* Can I update a business fact in exactly one place?
* Do any columns contain lists or repeated groups?
* In composite-key tables, are there attributes that belong to only one part of the key?
* Do I have attributes describing other attributes (e.g., DeptName inside Employee)?
* Are any independent multi-valued lists causing duplicate combinations?

***

## Related

* **[Data Modeling: From Concept to Physical Design]({{< ref "data-modeling" >}})** — Understanding CDM, LDM, PDM and ER diagrams
* **[Enterprise Database Design Patterns]({{< ref "enterprise-database-design-patterns" >}})** — Production-grade patterns for OLTP, dimensional modeling, transactions, and schema evolution
