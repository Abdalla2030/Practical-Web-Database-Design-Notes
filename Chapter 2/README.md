# Chapter 2 — Core Database Concepts

> **Book:** Practical Web Database Design  
> A focused review of the chapter’s main concepts.

## 1. Data Models

A **data model** is a framework for structuring and manipulating data.

The term can also mean the design of a specific database, such as an ER diagram for a library.

| Aspect | Purpose |
|---|---|
| **Structure** | Defines how data is organized |
| **Integrity** | Defines rules for valid and consistent data |
| **Operations** | Defines how to retrieve, insert, update, and delete data |

Good design makes databases easier to maintain, extend, and scale.

### Physical Data Independence

Applications work with logical structures—tables and columns—without needing to know how data is physically stored.

The DBMS handles the mapping to files, pages, and record locations.

Physical storage can change while the logical interface remains the same. Renaming a table or column, however, is a logical change and may require application changes.

## 2. The Relational Model

Developed by **Edgar F. Codd** and introduced in **1970**, the Relational Model is based on set theory and predicate logic.

A database management system based on it is called an **RDBMS**.

Examples: Oracle, SQL Server, DB2, MySQL, and PostgreSQL.

### Tables, Rows, and Columns

| Common term | Formal term | Represents |
|---|---|---|
| Table | Relation | A collection of related data |
| Row | Tuple | One instance, such as one employee |
| Column | Attribute | A property, such as name or salary |

```text
EMPLOYEE(EmployeeNo, Name, Position, Location, Salary)
```

This notation describes the table’s structure, not its stored data.

### Main Properties

- Tables and columns have names.
- Each column has an allowed set of values.
- Each cell contains a single value under the chapter’s 1NF treatment.
- Relations do not contain duplicate complete rows.
- Row and column order have no inherent meaning.
- SQL tables need suitable constraints to prevent duplicate rows.

To guarantee query-result order, use `ORDER BY`.

**Relation ≠ Relationship:** A relation is a table; a relationship is an association between data.

Tables, rows, and columns are **logical structures**. They do not necessarily map directly to physical files and records.

## 3. Domains and Data Types

A **data type** defines the kind of value. A **domain** defines which values are permitted.

| Column | Data type | Domain example |
|---|---|---|
| Age | Integer | 0–120 |
| Price | Decimal | Zero or greater |
| Status | Text | Pending, Approved, Rejected |

Common data categories:

- **Integer / Decimal:** Numbers.
- **CHAR / VARCHAR:** Text.
- **DATE / TIME / TIMESTAMP:** Dates and times.
- **Boolean:** True/False.
- **Interval:** Duration.
- **BLOB:** Large binary content, such as images or documents.
- **CLOB:** Large text content.

Type names and support vary by DBMS. Domain restrictions can be enforced using constraints.

## 4. Missing Data: NULL

**NULL represents missing information, not an ordinary value.**

| Reason | Example |
|---|---|
| **Unknown / Not supplied** | A customer did not provide their email |
| **Inapplicable** | An employee does not receive commission |

NULL alone does not distinguish between these reasons.

### NULL Is Not Zero or Empty Text

- `0` is a known number.
- `''` is conceptually an empty string.
- `NULL` indicates missing information.

DBMS behavior can differ; Oracle treats zero-length character strings as NULL.

Avoid arbitrary placeholders such as `-1` for unknown prices: calculations would treat them as real values.

### Calculations and Comparisons

| Expression | Result |
|---|---|
| `NULL + 2000` | NULL |
| `NULL > 2000` | UNKNOWN |
| `NULL = NULL` | UNKNOWN |

SQL uses **TRUE, FALSE, and UNKNOWN**. A `WHERE` clause keeps only rows whose condition is TRUE.

Use:

```sql
WHERE CreditLimit IS NULL
```

Not:

```sql
WHERE CreditLimit = NULL
```

### Aggregate Functions

For values `100, 200, NULL`:

| Expression | Result |
|---|---:|
| SUM(Amount) | 300 |
| AVG(Amount) | 150 |
| COUNT(Amount) | 2 |
| COUNT(*) | 3 |

`SUM` and `AVG` ignore NULL inputs. If there are no non-NULL inputs, both return NULL.

### NULL Versus Defaults

- **NOT NULL:** Requires a value.
- **Default:** Supplies a predefined value when a column is omitted from an insert or its default is requested.
- A default does not normally replace an explicitly supplied NULL.

Use a default only when it has a valid business meaning. An unknown price is not automatically zero.

## 5. Primary Keys

A **Primary Key (PK)** uniquely identifies each row.

It must be:

- **Unique.**
- **Non-NULL**, including every component of a composite key.
- Preferably **stable** over time.

A table has one declared PK, which may contain one or several columns. Many DBMSs allow tables without a declared PK, but defining one is generally good design.

### Single and Composite Keys

```text
PRODUCT
Primary Key: ProductNo
```

```text
ORDER_ITEM
Primary Key: (OrderNo, ProductNo)
```

For the composite key, neither column must be unique alone—the combination must be unique.

### Key Terminology

| Term | Meaning |
|---|---|
| **Candidate key** | A minimal set of columns that uniquely identifies a row |
| **Primary key** | The selected candidate key |
| **Alternate key** | A candidate key not selected as the PK |
| **Surrogate key** | An artificial identifier, often generated automatically |

Names and descriptions are usually poor identifiers because they can repeat or change.

A surrogate key can serve as the PK, but adding one does not automatically remove redundant business data.

## 6. Foreign Keys

A **Foreign Key (FK)** references a primary key or another eligible unique key.

```text
CUSTOMER(CustomerNo, Name, Address)
SALE(SaleNo, CustomerNo, ProductNo, Qty, Amount)
```

`SALE.CustomerNo` references `CUSTOMER.CustomerNo`.

- **Parent:** CUSTOMER.
- **Child:** SALE.

### Main Rules

- A table can have multiple FKs.
- An FK can contain multiple columns.
- FK values can repeat: one customer can make many sales.
- An FK can be NULL when allowed.
- A non-NULL single-column FK must match the referenced key when the constraint is enforced.
- A parent row can exist without child rows.

Repeating `CustomerNo` in SALE represents the relationship. Repeating the customer’s name and address for every sale is usually unnecessary.

**FK versus JOIN:**

- An FK protects the reference.
- A JOIN combines related data when querying.

## 7. Normalization

**Normalization reduces unnecessary redundancy by organizing data into related tables.**

### Why Normalize?

Storing customer details with every sale causes:

| Problem | Example |
|---|---|
| **Update anomaly** | An address must be changed in many rows |
| **Delete anomaly** | Deleting the last sale also deletes customer details |
| **Insert anomaly** | A customer cannot be recorded naturally before buying |
| **Wasted storage** | Customer details repeat for every sale |

Separate the facts:

```text
CUSTOMER(CustomerNo, Name, Address)
SALE(SaleNo, CustomerNo, ...)
```

### Functional Dependency

```text
ProductNo → Description, Price
```

Knowing the product number determines its description and price. This describes a data rule, not a calculation.

### First Normal Form — 1NF

**Each cell contains one value, not a list or repeating group.**

Instead of:

```text
OrderNo = 100, Products = "P1, P2"
```

Use:

| OrderNo | ProductNo |
|---|---|
| 100 | P1 |
| 100 | P2 |

1NF alone does not remove all redundancy.

### Second Normal Form — 2NF

**Be in 1NF, with no non-key attribute depending on only part of a candidate key.**

Example:

```text
ORDER_ITEM(OrderNo, ProductNo, ProductDescription, Qty)
Key: (OrderNo, ProductNo)
```

- `Qty` depends on the full key.
- `ProductDescription` depends only on `ProductNo`.

Split into:

```text
PRODUCT(ProductNo, ProductDescription)
ORDER_ITEM(OrderNo, ProductNo, Qty)
```

This removes the **partial dependency**.

### Third Normal Form — 3NF

**Be in 2NF and remove transitive dependencies between non-key attributes.**

Example:

```text
ProductNo → SupplierCode → SupplierName, SupplierAddress
```

Supplier details describe the supplier, not the product directly.

Split into:

```text
PRODUCT(ProductNo, Description, Price, SupplierCode)
SUPPLIER(SupplierCode, SupplierName, SupplierAddress)
```

`SupplierCode` remains in PRODUCT as an FK.

Memory aid:

> The key, the whole key, and nothing but the key.

### Fourth Normal Form — 4NF

**Separate independent collections of facts that otherwise create redundant combinations.**

Suppose customer addresses and phone numbers are independent:

| CustomerNo | Address | Phone |
|---|---|---|
| C1 | Cairo | 111 |
| C1 | Cairo | 222 |
| C1 | Giza | 111 |
| C1 | Giza | 222 |

Each cell contains one value, so the table satisfies the basic 1NF requirement. But every address is repeated with every phone.

Split into:

```text
CUSTOMER_ADDRESS(CustomerNo, Address)
CUSTOMER_PHONE(CustomerNo, Phone)
```

**1NF concerns values inside cells; 4NF concerns independent multivalued dependencies.**

### Higher Forms and Practical Design

5NF addresses further decomposition involving join dependencies. Detailed rules are beyond this chapter’s scope.

For practical design:

- 3NF or BCNF is a common target.
- Apply higher forms when the data requires them.
- More joins do not automatically mean worse performance.

### Denormalization

**Denormalization deliberately introduces redundancy**, usually for a specific read or reporting requirement.

| Operational database | Data warehouse |
|---|---|
| Daily business transactions | Reporting and analysis |
| Frequent inserts and updates | Large reads and aggregations |
| Often normalized | Often uses denormalized designs |

Duplicated data needs a consistency strategy.

Normalization is **bottom-up**: start with attributes and dependencies.

Conceptual modeling is **top-down**: start with business entities and relationships, often using ER diagrams.

## 8. Data Integrity

**Data integrity means data is valid and internally consistent.**

| Type | Rule |
|---|---|
| **Entity integrity** | PK values are unique and have no NULL components |
| **Referential integrity** | References point to existing keys, subject to permitted NULLs |
| **Domain integrity** | Values belong to the allowed domain |

Business-specific rules include:

- Employee names are required.
- Discounts cannot exceed 30%.
- Credit limits must stay within an agreed range.

Define and document these rules explicitly.

### Where to Enforce Rules

- **Browser:** Immediate user feedback.
- **Server:** Validate requests and business operations.
- **Database:** Protect data across applications and access paths.

Browser validation alone can be bypassed.

### Valid Does Not Mean Correct

If permitted credit limits are 0–5000, entering `4000` passes validation—even if the customer’s actual limit is `1000`.

Constraints check rules. Verification against reliable information checks factual accuracy.

## 9. Metadata and the Data Dictionary

**Metadata describes the database.**

| Business data | Metadata |
|---|---|
| An employee earns 22000 | Salary is a decimal column |
| A customer lives in Cairo | Address is a text column |
| An employee number is 15 | EmployeeNo is the PK |

The **Data Dictionary**, or **Catalog**, stores:

- Table and column definitions.
- Data types and sizes.
- Keys and integrity rules.
- Users and privileges.
- Index definitions.
- Storage information and statistics.

The DBMS uses it to understand structures, validate operations, control access, and plan queries.

## 10. Physical Storage and Access

### Pages and Caching

Database files are commonly divided into **pages / blocks**. Each page can contain several records.

To retrieve a record, the DBMS locates its page and loads it into memory if needed.

**Caching** keeps pages in RAM for reuse. A page already cached does not need another storage read.

Reducing unnecessary page reads often improves performance, although CPU, locking, and query design also matter.

### Three Access Methods

These are alternative approaches—not three mandatory steps. One query can use different methods for different tables.

A **search key** is the value used to locate data. It need not be a PK or unique.

| Method | How it works | Useful for |
|---|---|---|
| **Sequential** | Scan records/pages | Small tables or many required rows |
| **Indexed** | Navigate an index | Exact matches and ranges |
| **Direct / Hashing** | Calculate a bucket | Exact-match lookups |

### Sequential Access

Scan until the record is found or the data ends.

For one existing match equally likely to be anywhere:

| Case | Records examined |
|---|---:|
| Best | 1 |
| Average | About n / 2 |
| Worst | n |

Finding all matches may require a full scan.

One record examined does not mean one disk read: a page can hold many records.

### Indexed Access

An **index** stores search-key values with information for locating associated data.

B-tree-style indexes keep keys ordered across levels:

```text
Root → Internal nodes → Leaf → Associated data
```

They support efficient exact-match, range, and suitable prefix searches.

Important points:

- Index order does not necessarily match physical table order.
- Use `ORDER BY` to guarantee output order.
- **Unique indexes** restrict duplicates; **non-unique indexes** allow them.
- Indexes consume storage and require maintenance during writes.
- Full pages may split as the index grows.
- A scan may be cheaper for a tiny table or a query needing many rows.

### Direct Access Through Hashing

```text
Hash(SearchKey) → Bucket
```

The DBMS calculates a bucket, accesses it, and checks for the matching record.

Hashing does not preserve key order:

- Good fit: `EmployeeNo = 101`.
- Poor fit for ordered retrieval: `EmployeeNo BETWEEN 100 AND 200`.

A bucket can hold multiple entries and may occupy multiple pages.

- **Collision:** Different keys map to the same bucket.
- **Overflow:** The bucket needs more capacity.

Hash-organized storage places records according to hashing. A hash index may instead contain references to records stored elsewhere.

A single storage read is possible in ideal conditions, not guaranteed. Hashing is not always faster than a tree index.

## 11. Putting It Together

| Component | Role |
|---|---|
| **Data** | Business facts |
| **Metadata** | Describes the database |
| **Integrity rules** | Protect validity and consistency |
| **Indexes** | Support efficient access |
| **DBMS** | Manages these components and controls access |

A DBMS can manage multiple databases and also handles responsibilities such as security, transactions, and application interfaces.

## Final Checklist

After this chapter, you should be able to explain:

- Logical data models and physical data independence.
- Tables, rows, columns, domains, and data types.
- NULL versus zero, empty text, and defaults.
- Primary, composite, surrogate, and foreign keys.
- Redundancy and update/delete/insert anomalies.
- The purpose of 1NF, 2NF, 3NF, and 4NF.
- Data integrity versus factual correctness.
- Metadata and the data dictionary.
- Pages, caching, sequential access, indexes, and hashing.

**Next chapter:** Creating and Using Relational Databases with SQL.
