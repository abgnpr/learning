+++
title = 'Data Modeling: From Concept to Physical Design'
date = 2026-01-25T12:00:00+05:30
draft = false
tags = ['databases', 'data-modeling', 'design', 'erd']
summary = 'Understanding the data modeling process, conceptual/logical/physical models, entities and attributes, and crow-foot notation for ER diagrams.'
mermaid = true
+++

***

# Data Modeling Process

## What is Data Modeling?

**Data modeling** is the process of **analyzing data requirements** and **creating a structured design** that defines:

* what data is needed,
* how it relates,
* and how it will be stored and used.

### Why it matters

Data modeling helps you:

* reduce ambiguity in requirements,
* improve data quality and consistency,
* make databases easier to maintain,
* support reporting/analytics reliably.

## The Three Types of Data Models

Data modeling typically progresses through **three levels**:

1. **Conceptual Model** (high-level business view)
2. **Logical Model** (detailed structure, still tech-agnostic)
3. **Physical Model** (implementation in a specific database)

Think of it like:

> **Conceptual = "What?"**
> **Logical = "How is it organized?"**
> **Physical = "How is it implemented?"**

### 1) Conceptual Data Model (CDM)

#### Purpose

A **business-friendly** model that captures **core entities and relationships** without technical details.

#### Key Characteristics

* Focuses on **business concepts**
* Minimal attributes
* No database-specific details
* Used to align stakeholders and clarify scope

#### Typical Outputs

* High-level ER diagram
* Entities + relationships
* Business definitions (glossary)

#### Audience

* Business stakeholders
* Product owners
* Analysts

#### Example (E-commerce)

Entities:

* **Customer**
* **Order**
* **Product**

Relationships:

* Customer **places** Order
* Order **contains** Product

*No primary keys, no data types, no normalization yet.*

### 2) Logical Data Model (LDM)

#### Purpose

Defines the **detailed structure of data** and rules, but remains **database-agnostic**.

#### Key Characteristics

* Adds **attributes** to entities
* Defines **primary keys (PK)** and **foreign keys (FK)**
* Normalization is applied (often up to 3NF depending on needs)
* Captures business rules and constraints (e.g., cardinality)

#### Typical Outputs

* Logical ERD with:
  * attributes
  * PK/FK
  * cardinalities (1:1, 1:M, M:N)
* Data dictionary (definitions, constraints)

#### Audience

* Data analysts
* Data architects
* Developers (early design)

#### Example (E-commerce)

Customer

* CustomerID (PK)
* Name
* Email

Order

* OrderID (PK)
* OrderDate
* CustomerID (FK)

Product

* ProductID (PK)
* Name
* Price

OrderItem *(resolves M:N between Order and Product)*

* OrderID (FK)
* ProductID (FK)
* Quantity

Still *no* database indexes, partitioning, storage engine—those come next.

### 3) Physical Data Model (PDM)

#### Purpose

Specifies how the model will be **implemented in a specific DBMS** (SQL Server, PostgreSQL, Oracle, etc.).

#### Key Characteristics

* Includes **table names, columns, data types**
* Specifies **indexes**, constraints, default values
* Includes **performance considerations**
* DB-specific features:
  * partitioning
  * clustering
  * compression
  * storage parameters
  * schema/namespace

#### Typical Outputs

* SQL DDL scripts (`CREATE TABLE`, `CREATE INDEX`, etc.)
* Physical ERD
* Storage + performance plan

#### Audience

* DBAs
* Backend engineers
* Platform / DevOps (sometimes)

#### Example (Physical – SQL-ish)

```sql
CREATE TABLE Customer (
  CustomerID BIGINT PRIMARY KEY,
  Name       VARCHAR(100) NOT NULL,
  Email      VARCHAR(255) UNIQUE
);

CREATE TABLE [Order] (
  OrderID    BIGINT PRIMARY KEY,
  OrderDate  TIMESTAMP NOT NULL,
  CustomerID BIGINT NOT NULL,
  CONSTRAINT FK_Order_Customer
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
);

CREATE INDEX IX_Order_CustomerID ON CustomerID;
```

### How They Connect (CDM → LDM → PDM)

#### Mapping Summary

* **Conceptual**: entities + relationships (business terms)
* **Logical**: entities + attributes + keys + normalization
* **Physical**: tables + columns + types + indexes + constraints

#### Rule of thumb

As you move forward:

* **detail increases**
* **technical specificity increases**
* **audience shifts from business → engineering**

### Quick Comparison Cheat Sheet

#### Conceptual

* **Goal:** align business understanding
* **Includes:** entities, relationships
* **Excludes:** attributes, keys, data types

#### Logical

* **Goal:** precise structure + rules
* **Includes:** attributes, PK/FK, normalization
* **Excludes:** DB-specific performance/storage

#### Physical

* **Goal:** implementation-ready blueprint
* **Includes:** tables, columns, types, indexes, partitions
* **Includes:** DBMS-specific features

### Mini Self-Check Questions

Use these to test your understanding:

* **Conceptual:** Can a non-technical stakeholder understand it?
* **Logical:** Does it clearly define keys, constraints, and resolve M:N relationships?
* **Physical:** Can you generate DDL and consider performance (indexes, partitioning)?

### Determining The Goal Of The Database

* What the goal of this database, what is it trying to achieve.
* Determining the goal of the database helps you determine what needs to be stored.
* You should have some kind of scope or boundary for what you to or need to be stored.

### Consider The Curent System

* Identify the problems with the current system or database (data quality, missing data).

### Future Growth

* Databases should cater for future growth (data type and size should allow for it).
* Should last many years.
* Technology may change, data model should be same.

### Exceptions

* Finding exceptions to the rules during your design phase is important.
* Determine if there are any exceptions to your requirements.
* Watch out for the word "usually".
* Question any specific field length or type restriction.

## Entities and Attributes in Database Design

### Entities

An **entity** is a real-world object or concept that a database needs to store information about.
In practice, entities become **tables** in a database.

**Examples:** Customer, Order, Product, Employee

### Attributes

An **attribute** is a detail that describes an entity.
Attributes become **columns** in a table.

**Examples:**

* For Customer → CustomerID, Name, Email
* For Product → ProductID, Price, Category

### Best Practices (Crisp & Practical)

#### When defining **Entities**

* Identify only *business‑relevant* objects—avoid unnecessary tables.
* Name entities in **singular form** (Customer, not Customers).
* Ensure each entity has a **clear purpose** and fits one logical concept.
* Avoid mixing unrelated concepts in one entity (e.g., Customer + Address in same table).

#### When defining **Attributes**

* Always include a **primary key** (natural or surrogate) that uniquely identifies each record.
* Use **clear, meaningful names** (OrderDate, not Date1).
* Choose **correct data types** early (e.g., decimal for money, date for dates).
* Avoid storing **derived/calculated attributes** unless necessary for performance.
* Maintain **atomic attributes** (break full name into FirstName, LastName if needed).

#### General Modeling Best Practices

* Follow **normalization** (at least 1NF–3NF) to reduce redundancy.
* Keep entities loosely coupled with **well-defined relationships**.
* Add attributes only if they serve a real reporting or operational need.
* Document entities and attributes with short, clear definitions.

### Relationship Notation Cheat Sheet (Crow-Foot Notation)

Crow-foot notation (also called Information Engineering notation) is the standard way to show relationships in ERDs.

#### Symbol Components

Each relationship line has **two ends**, and each end shows:

1. **Cardinality** (how many) — closest to the entity box
2. **Modality/Optionality** (required or optional) — furthest from the entity box

#### Cardinality Symbols (Inner, closest to entity)

* **`|`** (single line) = **One** (exactly one)
* **`<`** (crow's foot) = **Many** (zero or more)

#### Modality Symbols (Outer, away from entity)

* **`|`** (single line) = **Mandatory** (must exist)
* **`o`** (circle) = **Optional** (may or may not exist)

#### Combined Notation Patterns

```text
Symbol   Meaning                    Read as
------   -------                    -------
||       One and only one           Mandatory one
o|       Zero or one                Optional one
|<       One or more                Mandatory many
o<       Zero or more               Optional many
```

#### Common Relationship Patterns

##### One-to-One (1:1)

```text
PERSON ||--|| PASSPORT
```

* One person has exactly one passport
* One passport belongs to exactly one person

##### One-to-Many (1:M)

```text
CUSTOMER ||--o{ ORDER
```

* One customer can have zero or more orders
* Each order must belong to exactly one customer

##### Many-to-Many (M:N) — Resolved with Junction Table

```text
STUDENT }o--o{ COURSE
```

**After normalization (recommended):**

```text
STUDENT ||--o{ ENROLLMENT : enrolls-in
COURSE  ||--o{ ENROLLMENT : has
```

##### Optional One-to-Many

```text
DEPARTMENT ||--o{ EMPLOYEE
```

* One department can have zero or more employees
* Each employee must belong to exactly one department

##### Mandatory One-to-Many

```text
ORDER ||--|{ ORDER_LINE
```

* One order must have one or more order lines
* Each order line belongs to exactly one order

#### Visual Reference Card

```text
┌──────────────┬────────────────────────────────────┐
│ Notation     │ Meaning                            │
├──────────────┼────────────────────────────────────┤
│ Entity ||    │ Mandatory relationship (must have) │
│ Entity o|    │ Optional relationship (may have)   │
│ Entity |<    │ Mandatory many (one or more)       │
│ Entity o<    │ Optional many (zero or more)       │
└──────────────┴────────────────────────────────────┘

Reading relationships left-to-right:
  A ||--o{ B  =  "One A has zero or more B"
               =  "Each B belongs to exactly one A"
```

#### Practical Examples

##### E-commerce System

```text
CUSTOMER ||--o{ ORDER       : places
ORDER    ||--|{ ORDER_LINE  : contains
PRODUCT  ||--o{ ORDER_LINE  : appears-in
CUSTOMER ||--o{ ADDRESS     : has
```

**Translation:**

* A customer can place zero or more orders
* An order must have at least one order line
* A product can appear in zero or more order lines
* A customer can have zero or more addresses

##### University System

```text
DEPARTMENT ||--o{ COURSE     : offers
INSTRUCTOR ||--o{ COURSE     : teaches
STUDENT    ||--o{ ENROLLMENT : registers
COURSE     ||--o{ ENROLLMENT : has
```

**Translation:**

* A department can offer zero or more courses
* An instructor can teach zero or more courses
* A student can register for zero or more enrollments
* A course can have zero or more enrollments

#### Tips for Reading Crow-Foot Diagrams

1. **Start from the entity you're asking about**
2. **Read the symbols closest to that entity first** (cardinality)
3. **Then check if it's mandatory or optional** (modality)
4. **Flip and read from the other direction** for the complete picture

**Example:**

```text
AUTHOR ||--o{ BOOK
```

Reading from AUTHOR:
"One author has zero or more books" (an author may not have written any books yet)

Reading from BOOK:
"Each book belongs to exactly one author" (every book must have an author)

***

## Next: Normalization

Once you have a data model, you'll want to normalize it to reduce redundancy and prevent anomalies. See **[Database Normalization: 1NF to 5NF]({{< ref "normalization" >}})** for a complete guide.
