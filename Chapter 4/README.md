# Chapter 4: Fundamentals of Database Design

## Book Information

- **Book:** Practical Web Database Design
- **Authors:** Chris Auld, Allan Kent, Rudy Limeback, Nigel Stanger, Thearon Willis
- **Chapter:** 4
- **Chapter Title:** Fundamentals of Database Design

These notes cover the chapter material studied, including clarifications of inaccurate or outdated statements. Supplementary examples are identified where relevant.

---

## 1. Data Modeling

**Data modeling** creates an abstract representation of the real world, retaining information relevant to the system and excluding unnecessary details.

A model identifies:

- The things the system stores information about.
- Their properties.
- Their relationships.
- The rules governing valid data.

The model can then be translated into tables, columns, keys, and constraints.

### Why Model Before Implementing?

Modeling helps us:

- Clarify requirements.
- Share a common understanding with developers and customers.
- Discover missing cases and incorrect assumptions.
- Change the design before application code and existing data depend on it.
- Document and understand existing databases.

A small mistake is generally easier to correct during modeling than after deployment.

Good design supports growth and change, but does not guarantee unlimited scalability or eliminate future redesign.

### Modeling Is Iterative

The general process is:

1. Gather and analyze requirements.
2. Create a model.
3. Review it with users or customers.
4. Refine the requirements and model.
5. Repeat until sufficiently validated.
6. Implement the database.

Customers may discover additional requirements when reviewing the model. Developers may also discover questions they did not initially know to ask.

### Platform Independence

A conceptual or platform-independent logical model describes business data without committing to a specific DBMS.

The physical design depends on the selected platform.

Requirements such as storage capacity and target query response times help inform platform selection. Migrating a prototype to production may still require changes to SQL, types, indexes, and other platform-specific features.

---

## 2. Entities, Attributes, and Relationships

The **Entity Relationship Model**, or **ER Model**, has three main elements:

| Element | Meaning | Example |
|---|---|---|
| Entity | A thing or concept represented by the model | Book, Publisher |
| Attribute | A property of an entity | ISBN, Title |
| Relationship | An association between entities | A publisher publishes books |

In a relational implementation:

- An entity type generally maps to a table.
- An entity instance maps to a row.
- An attribute maps to a column.

### Example: Books and Publishers

| Table | Attribute | Key |
|---|---|---|
| Books | ISBN | PK |
| Books | PublisherID | FK |
| Books | Title | — |
| Books | Author | — |
| Publishers | PublisherID | PK |
| Publishers | Name | — |
| Publishers | Address | — |

`Books.PublisherID` references `Publishers.PublisherID`.

The example's author field is a simplification; it does not establish a general design for books with multiple authors.

### Reading Entity Boxes

The book's diagrams show:

1. Key information.
2. Attribute name.
3. Data type.

Key markers:

- **PK:** Primary Key.
- **FK:** Foreign Key.
- **FK1, FK2:** labels distinguishing foreign key constraints, not relationship cardinality.

In the book's convention:

- Bold attributes are required.
- Plain attributes allow NULL.

This formatting is not universal.

A modeling tool may hide attributes and show only entity names to simplify large diagrams.

---

## 3. Cardinality and Optionality

A relationship must be understood in both directions.

- **Maximum cardinality:** one or many.
- **Optionality:** whether zero participation is allowed.

Some references use **cardinality** for the entire minimum-to-maximum range.

### Crow's Foot and Numeric Notation

The text symbols below are approximate representations.

| Crow's Foot approximation | Numeric notation | Meaning |
|---|---|---|
| `o|` | `0..1` | Zero or one |
| `||` | `1..1` | Exactly one |
| `o<` | `0..*` | Zero or many |
| `|<` | `1..*` | One or many |

Crow's Foot is common in ER diagrams. Numeric ranges are also common in UML, where they are called multiplicities.

The symbols beside an entity describe how many instances of that entity may relate to one instance at the opposite end.

### Books and Publishers

- One publisher may have zero or many books.
- Each book must have exactly one publisher.

“One-to-Many” describes the maximum relationship; it does not automatically require every parent to have children.

---

## 4. Common Relationship Structures

| Structure | Example | Implementation |
|---|---|---|
| One-to-Many | Customer and Orders | FK on the Many side |
| Many-to-Many | Orders and Products | Junction table |
| One-to-Many Recursive | Categories and subcategories | Self-referencing FK |
| Many-to-Many Recursive | Product bundles | Junction table referencing the same table twice |
| One-to-One | Department and its manager | FK with uniqueness |

Relationships follow business rules, not just the data currently stored.

---

## 5. One-to-Many

One parent row can relate to many child rows. Each child relates to at most one parent through that relationship.

Whether the parent is mandatory depends on optionality.

### Foreign Key Placement

The FK belongs in the **child table**, the Many side.

| Relationship | Foreign Key |
|---|---|
| Categories → Products | `Products.CategoryID` |
| Customers → Orders | `Orders.CustomerID` |

### Example

Supplementary data:

| ProductID | ProductName | CategoryID |
|---|---|---|
| 101 | Running Shoes | 1 |
| 102 | Summer Tracksuit | 2 |
| 103 | Winter Tracksuit | 2 |

Category `2` contains multiple products.

A Foreign Key may repeat. A Primary Key may not.

If the business allows one product to belong to multiple categories, that relationship becomes Many-to-Many.

---

## 6. Many-to-Many

Both sides can relate to multiple instances on the other side.

Examples:

- A student takes many classes; a class contains many students.
- An order contains many products; a product appears in many orders.

A normal relational implementation uses an **associative entity**, **junction table**, or **joining table**.

This produces two One-to-Many relationships.

Do not use repeating columns such as `ProductID1`, `ProductID2`, and `ProductID3`; they impose an arbitrary limit.

### Orders and Products

`OrderItems` connects Orders and Products.

Supplementary data:

| OrderID | ProductID | Quantity | PurchasePrice |
|---|---|---|---|
| 1 | 4 | 2 | 100 |
| 2 | 4 | 1 | 100 |
| 2 | 7 | 3 | 50 |

- Order `2` contains products `4` and `7`.
- Product `4` appears in orders `1` and `2`.

`Quantity` and `PurchasePrice` describe the association and belong in `OrderItems`.

The purchase price preserves the historical price independently of the product's current price.

### Composite Primary Key

The book uses:

```text
PRIMARY KEY (OrderID, ProductID)
```

Individual values may repeat, but the complete pair must be unique.

This design allows each product only once per order, with its number of units stored in `Quantity`.

### Junction Tables Without Extra Attributes

`UserGroup` may contain only:

```text
UserGUID
GroupID
```

The pair can form its PK, preventing duplicate membership.

To identify Many-to-Many requirements, ask both questions:

- Can a user belong to many groups?
- Can a group contain many users?

---

## 7. One-to-Many Recursive

A recursive, or self-referencing, relationship connects instances of the same entity type.

It does not mean a row must reference itself.

### Category Hierarchy

Supplementary data:

| CategoryID | CategoryName | ParentCategory |
|---|---|---|
| 1 | Computers | NULL |
| 2 | Databases | 1 |
| 3 | Database Design | 2 |
| 4 | Programming | 1 |

```text
Categories.ParentCategory references Categories.CategoryID
```

- A category has zero or one parent.
- A category can have zero or many children.
- An intermediate node can be both parent and child.
- The design does not need a separate column or table for each level.

A root has no parent, represented here by NULL.

Multiple roots form a **forest**.

### Employee Hierarchy

The same pattern applies to:

```text
Employees.Manager references Employees.EmployeeID
```

The CEO has no manager; other employees reference their manager.

### Preventing Cycles

A normal FK confirms that the parent exists. It does not guarantee a valid tree.

Reject assignments where the new parent is:

- The row itself.
- A descendant of that row.

Otherwise, a cycle may form.

Validation may use application logic or database mechanisms such as triggers. A trigger runs database code in response to events such as INSERT, UPDATE, or DELETE.

### NULL Clarification

The root representation in this design requires a nullable parent FK.

Not every recursive relationship universally requires NULL. A non-NULL marker such as `0` violates RI only when no matching referenced row exists.

---

## 8. Programming Recursive Relationships

### Traversal Directions

| Direction | Method | Example use |
|---|---|---|
| Top-down | Start at roots, visit descendants | Category menu |
| Bottom-up | Start at a node, follow parents | Breadcrumb |

### Corrected Pseudo-code

The book's initial query selects all categories despite describing top-level records. That would revisit descendants. Start with roots instead.

```text
roots = DBQuery(
    "SELECT * FROM Categories WHERE ParentCategory IS NULL"
)

traverseTree(roots, 0)

Function traverseTree(pRecordset, pDepth)
    For each currentRow in pRecordset
        Print currentRow.CategoryName with indentation pDepth

        children = DBQuery(
            "SELECT * FROM Categories WHERE ParentCategory = ?",
            currentRow.CategoryID
        )

        traverseTree(children, pDepth + 1)
    End For
End Function
```

`?` represents a bound parameter.

The recursion stops when a node has no children: the empty recordset causes no further loop iterations.

This assumes the data contains no cycles.

`pDepth` determines indentation. Sibling order requires an explicit ordering rule, such as ORDER BY.

### One Branch

Starting with:

```sql
SELECT *
FROM Categories
WHERE CategoryID = 3;
```

traverses that node and its descendants, not its ancestors.

### Query Cost

With one initial query and one child query for every visited node:

```text
N visited nodes → N + 1 queries
```

The cost depends on node count, not only depth.

Alternatives include:

- Fetching flat rows once and building the tree in application memory.
- Caching a tree representation.

XML/XSLT are the book's suggested technologies, not requirements. Objects and collections can also represent a tree.

Cached data needs an update or invalidation strategy.

---

## 9. Many-to-Many Recursive

A table relates to itself through a junction table.

### Product Bundles

A bundle is a product made up of other products.

```text
ProductBundles.BundleID references Products.ProductID
ProductBundles.BundledProductID references Products.ProductID
```

| Column | Role |
|---|---|
| BundleID | Containing product |
| BundledProductID | Included product |

Supplementary data:

| BundleID | BundledProductID |
|---|---|
| 10 | 3 |
| 10 | 4 |
| 11 | 3 |
| 11 | 10 |

- Bundle `10` contains products `3` and `4`.
- Product `3` belongs to multiple bundles.
- Bundle `11` contains bundle `10`.

The PK can be:

```text
(BundleID, BundledProductID)
```

A more detailed Bill of Materials might also store `Quantity`.

Prevent direct and indirect cycles. FKs alone do not stop a bundle containing itself through other bundles.

---

## 10. One-to-One

Each row relates to at most one row on the other side. Optionality determines whether participation is required.

### Actual One-to-One

Business rules:

- Each department has one manager.
- Each manager manages one department.

With `Departments.DepartmentManager`:

| Constraint | Purpose |
|---|---|
| FK | Employee exists |
| UNIQUE | Employee cannot manage multiple departments |
| NOT NULL | Department must have a manager |

A Foreign Key alone does not enforce One-to-One.

Ordinary employees may manage zero departments.

### Which Side Holds the FK?

It can be placed on either side, but placement affects what ordinary constraints enforce.

Alternative:

```text
Employees.DeptManaged references Departments.DepartmentID
```

With UNIQUE, one department cannot be assigned to multiple employees. NULL allows employees who manage no department.

However, that alone does not guarantee every department has a manager.

Usually store a single representation of the relationship, rather than duplicate FKs on both sides.

### Super-type/Sub-type

Use this when entities share attributes but also have type-specific attributes.

| Table | Purpose |
|---|---|
| Employees | Shared attributes |
| ContractEmployees | HourlyRate |
| SalariedEmployees | Salary |

In each subtype:

```text
EmployeeID is both PK and FK to Employees
```

This permits at most one row per employee in each subtype table.

It does not prevent the same employee appearing in both subtype tables. Disjoint subtype rules require additional enforcement.

An ordinary row-level CHECK does not generally enforce exclusion across separate tables.

### Items and Files

The book stores shared tree information in `Items` and file-specific information in `Files`.

- NULL `Items.FileID` represents a folder.
- Non-NULL `Items.FileID` references file details.

The FK needs uniqueness to enforce One-to-One.

Placing the FK in the subtype is also a valid common design, even with only one subtype. The subtype count does not impose an absolute FK-placement rule.

---

## 11. Integrity Constraints

Integrity constraints enforce valid database states regardless of which application writes the data.

Application validation remains useful for feedback, but should not be the only protection for rules the database can enforce.

| Area | Concern |
|---|---|
| Entity Integrity | Valid row identifiers |
| Attribute / Domain Integrity | Valid attribute values |
| Relational Integrity | Valid relationships |

---

## 12. Entity Integrity and Keys

### Primary Key Rules

| Key type | Rule |
|---|---|
| Single-column | Unique and NOT NULL |
| Composite | Complete combination unique; every component NOT NULL |

A composite key is a tuple of values, not literal string concatenation.

Other columns may match values in other rows.

### Natural Keys

A Natural Key, called an Intelligent Key in the book, uses meaningful business data.

The combination:

```text
(Author, Title, Publisher)
```

may appear unique until another edition has the same values.

Uniqueness in sample data does not prove uniqueness under all valid business cases.

ISBN can identify an edition/product format, but:

- Some publications have no ISBN.
- Multiple physical copies share an ISBN.

First decide what one row represents.

### Surrogate Keys

A Surrogate Key is an artificially generated identifier without business meaning.

Common forms:

- Numeric sequence keys.
- GUIDs/UUIDs.

A surrogate PK does not prevent duplicate business records. Add suitable UNIQUE constraints when required.

### Numeric Keys

Examples include SQL Server IDENTITY, MySQL AUTO_INCREMENT, and database sequences.

Generated numbers are not guaranteed to be gapless and should not be treated as row counts.

### GUIDs / UUIDs

Useful when independent systems generate identifiers and later combine data.

Proper generation makes collisions extremely unlikely; retain database uniqueness enforcement.

Random UUIDs make guessing harder, but never replace record-level authorization.

A timestamp alone is not a reliable unique key.

### When a Surrogate Key Is Unnecessary

For `UserGroup`:

```text
PRIMARY KEY (UserGUID, GroupID)
```

already identifies membership and prevents duplicates.

Adding `UserGroupID` still requires uniqueness on the pair. A separate ID can be useful if other records must reference the membership itself.

### Natural Keys and Joins

Storing a natural value as the FK may avoid a join when that value is exactly what a query needs.

However, consider key size, stability, localization, and update costs. A country name is not automatically a better key than an ID or stable code.

Single-column keys are not universally faster than composite keys. Evaluate actual indexes, key widths, and workloads.

---

## 13. Attribute / Domain Integrity

An attribute's **Domain** is its set of permitted values.

It is defined through:

- Data type.
- Length, range, precision, and scale.
- Constraints.
- Optionality.
- Default values.

Requirements should clarify unknown values, maximum lengths, numeric accuracy, and permitted ranges.

### Integer

A signed 32-bit integer ranges from:

```text
-2,147,483,648 to 2,147,483,647
```

Ensure application types can represent the database range.

For Java:

- Signed 32-bit → int / Integer.
- Signed 64-bit → long / Long.
- Wrapper types can represent NULL.

### Character Types

| Type | Meaning |
|---|---|
| CHAR | CHAR(1) |
| CHAR(n) | Fixed-length text, normally space-padded |
| VARCHAR(n) | Variable-length text up to n |

Physical storage and update costs depend on the DBMS. VARCHAR growth does not universally require moving data to new disk space.

Use requirements to select lengths; the author's preferred sizes are conventions.

Phone numbers belong in text types because leading zeros and symbols such as `+`, `*`, and `#` matter.

### DECIMAL

```sql
DECIMAL(precision, scale)
```

- Precision: total digits.
- Scale: fractional digits.

`157.4568` requires seven digits and scale four:

```sql
DECIMAL(7,4)
```

The book's `DECIMAL(7,2)` example does not preserve that value's full fractional precision.

For the portfolio example:

```sql
PricePaid DECIMAL(9,4)
```

allows five integer digits and four fractional digits. Its positive maximum is `99999.9999`.

DECIMAL stores exact decimal values within its precision and scale. Calculations or conversions may still require rounding.

### Floating Point

FLOAT, REAL, and DOUBLE generally represent approximate values.

MONEY and CURRENCY are not universally floating-point types; their behavior is platform-specific.

### Dates and Times

Choose a type matching the requirement:

- Date only.
- Time only.
- Date and time.
- Time zone information, where needed.

Store DOB rather than continuously changing current age.

The book's text selects DATE for DOB, while its diagram shows smalldatetime; this is an inconsistency.

### Credit Risk

A Boolean-compatible type represents Yes/No.

If using an integer, restrict permitted values appropriately.

“No risk” and “not assessed” are different business states unless requirements explicitly equate them.

---

## 14. Optionality and NULL

Optionality determines whether NULL is allowed.

NULL may represent missing, unknown, or inapplicable information. It does not explain the reason by itself.

| PricePaid | Meaning |
|---|---|
| NULL | No price recorded |
| 0 | Recorded price is zero |

Do not replace missing values with zero or empty strings unless that meaning is correct.

Use:

```sql
WHERE PricePaid IS NULL
```

or:

```sql
WHERE PricePaid IS NOT NULL
```

Do not use `= NULL`.

Ordinary comparisons with NULL evaluate to UNKNOWN.

For example, `PricePaid <> 0` does not select NULL prices.

---

## 15. Default Values

A DEFAULT provides a value when the column is omitted from an INSERT or DEFAULT is explicitly requested.

It does not generally replace an explicitly supplied NULL.

```sql
CREATE TABLE Example (
    Status VARCHAR(20) NOT NULL DEFAULT 'Pending'
);
```

- DEFAULT supplies an initial value.
- NOT NULL prohibits NULL.
- CHECK restricts allowed values.

These are separate responsibilities.

SQL Server example:

```sql
CREATE TABLE Example (
    CreatedAt DATETIME DEFAULT (GETDATE())
);
```

This supplies insertion time, not necessarily the real event time.

A default does not automatically update the column during UPDATE.

Defaulting CreditRisk to false guarantees a value only with appropriate nullability; it does not prove an assessment occurred.

---

## 16. CHECK Constraints

CHECK constraints test inserted or updated values against a condition.

```sql
CHECK (PricePaid >= 0)
```

This allows zero and positive values.

### CHECK and NULL: Important Correction

In SQL Server, CHECK rejects FALSE but accepts TRUE or UNKNOWN.

| Value | Result of PricePaid >= 0 | Accepted if nullable? |
|---|---|---|
| 10 | TRUE | Yes |
| 0 | TRUE | Yes |
| -1 | FALSE | No |
| NULL | UNKNOWN | Yes |

Therefore:

```sql
CHECK (PricePaid >= 0 OR PricePaid IS NULL)
```

does not need the explicit NULL branch for this case.

To require a value and prohibit negative prices:

```sql
PricePaid DECIMAL(9,4) NOT NULL
    CHECK (PricePaid >= 0)
```

Unlike CHECK, WHERE selects only TRUE.

---

## 17. Referential Integrity

Referential Integrity ensures that foreign key references are valid.

For a simple nullable FK, the value is either NULL or matches an existing referenced key.

The target can be a suitable UNIQUE key, not only a PK.

### Junction Table Example

SQL Server example; parent tables must already exist with compatible keys:

```sql
CREATE TABLE PropertyFeature (
    feature_code CHAR(5) NOT NULL,
    property_id CHAR(8) NOT NULL,

    CONSTRAINT propertyfeature_pk
        PRIMARY KEY CLUSTERED (feature_code, property_id),

    CONSTRAINT propertyfeature_fk_featurecode
        FOREIGN KEY (feature_code)
        REFERENCES Feature (feature_code),

    CONSTRAINT propertyfeature_fk_propertyid
        FOREIGN KEY (property_id)
        REFERENCES Property (property_id)
);
```

The composite PK prevents duplicate associations. The FKs ensure both referenced rows exist.

CLUSTERED is an indexing/storage choice, not what enforces RI.

### Referential Anomalies

| Operation | Potential violation |
|---|---|
| INSERT | Child references a nonexistent parent |
| UPDATE | FK or referenced key changes, leaving an invalid reference |
| DELETE | Parent is removed while children still reference it |

An **orphaned row** references a parent that no longer exists.

The same anomaly names also appear in normalization discussions, where their meaning concerns redundancy-related problems.

### Practical Ordering

With immediate FK checks:

- Insert the parent before its children.
- Without delete actions, remove or reassign children before deleting the parent.

Updating unrelated parent attributes does not violate the FK.

### MySQL Correction

MySQL does enforce FKs with supporting engines such as InnoDB. Engines such as MyISAM do not enforce them. The book's blanket statement is inaccurate.

---

## 18. Referential Actions

Referential actions define what happens when a referenced parent key changes or a parent is deleted.

Support and exact semantics vary by DBMS.

| Action | Parent key updated | Parent deleted |
|---|---|---|
| RESTRICT / NO ACTION | Prevent invalid references | Prevent deletion while referenced |
| CASCADE | Update matching child FKs | Delete matching children |
| SET NULL | Set matching child FKs to NULL | Set matching child FKs to NULL |

RESTRICT and NO ACTION can differ in constraint-check timing.

### CASCADE

Appropriate when children should disappear with the parent.

Example: deleting an order may delete its order items.

It is dangerous when the child has an independent business lifecycle.

Deleting an employee should not automatically delete valuable projects merely because the employee managed them.

One application statement can initiate cascading work, but the database still performs the underlying row and index operations.

Manual deletion does not require a separate SELECT just to check whether children exist.

### SET NULL

Appropriate when the child remains but the association becomes absent.

Requirements:

- FK columns permit NULL.
- Business rules allow the missing association.
- Other constraints permit it.

A PK component cannot be set to NULL.

Choose actions based on business meaning, not SQL convenience.

---

## 19. Denormalization for Performance

Denormalization deliberately introduces redundancy or stored derived information to improve selected operations.

Example: storing `Publishers.BookCount` rather than counting Books on every request.

### Trade-offs

| Benefit | Cost |
|---|---|
| Faster targeted reads | Additional write work |
| Less repeated aggregation | More storage |
| Precomputed results | Consistency maintenance |

Adding, deleting, or reassigning a book must update the relevant count.

Concurrent updates and partial failures must not corrupt stored totals.

### Decision Process

1. Build a sound normalized design.
2. Test realistic workloads.
3. Identify actual bottlenecks.
4. Review queries, indexes, and execution plans.
5. Denormalize only where justified.
6. Define how derived values remain correct.

Read-heavy workloads may benefit, but not every web application is read-heavy.

### Background and Batch Summaries

Recommendations such as “customers also purchased” can be precomputed outside the purchase request.

- Background processing avoids expensive work in the immediate user request.
- Batch processing refreshes summaries periodically.

The trade-off is freshness: periodic results may not reflect the latest transaction.

---

## 20. CASE Tools

CASE means **Computer-Aided Software Engineering**.

Database-aware modeling tools may:

- Represent entities, attributes, relationships, and constraints.
- Validate structural rules.
- Generate platform-specific DDL.
- Extract models from existing databases.
- Compare model and database changes.

A drawing tool is not necessarily a database modeling tool.

The chapter's product list, pricing, links, and Visio 2002 demonstration are historical.

### Model Versus Diagram

A model may contain all database objects while each diagram displays only a subset.

Use separate diagrams for understandable areas of a large system.

### Forward Engineering

```text
Model → Database definition
```

The tool generates DDL or applies supported changes.

Validation does not prove business correctness, and generated changes still need review for data-loss risks.

### Reverse Engineering

```text
Existing database → Model / Diagram
```

Useful for documentation and legacy-system analysis.

The book demonstrates adding Publisher.PhoneNumber in the database and importing it into the model.

Tools cannot reliably recover undocumented business rules or relationships that are not declared.

### Round-trip Engineering

Supports comparing and managing changes between model and database.

Having forward and reverse features does not guarantee automatic, conflict-free synchronization.

Tools help, but good modeling can begin with paper or a whiteboard.

---

## 21. Sample Data and Validation

Validate the model continuously using realistic data.

Client-friendly formats help customers review whether the structure reflects their actual work.

### Three Layers

| Layer | Purpose |
|---|---|
| Client-supplied realistic data | Validate requirements and workflows |
| Valid and deliberately invalid data | Test implemented integrity rules |
| Realistic volume and distribution | Test performance and support tuning |

An independent contributor helps avoid examples that merely confirm the modeler's assumptions.

### Integrity Test Cases

Test:

- Duplicate PKs.
- NULL in required columns.
- Invalid FKs.
- Negative values prohibited by CHECK.
- Duplicate junction memberships.
- Valid boundary values.
- NULL in optional columns.

Verify valid data is accepted as well as invalid data rejected.

### Performance Test Data

Realism includes distribution, not just row count.

A database where every customer has one order does not test the same behavior as one with customers holding thousands of orders.

### Load Versus Stress Testing

| Test | Purpose |
|---|---|
| Load testing | Evaluate expected production workload |
| Stress testing | Exceed expectations and assess limits and recovery |

Failure may mean unacceptable latency, timeouts, or errors rather than a crash.

Use realistic mixes of reads, writes, and concurrent activity.

---

## 22. Important Corrections to Remember

- Good design is extensible, not infinitely scalable.
- Composite keys are not inherently inefficient.
- Surrogate keys do not replace business uniqueness constraints.
- GUIDs do not replace authorization.
- Recursive FKs do not prevent cycles.
- A root's nullable FK is a design choice for the demonstrated hierarchy, not a universal recursive rule.
- One-to-One requires uniqueness, not just a FK.
- Separate subtype PK/FKs do not enforce subtype exclusivity.
- DECIMAL(7,4), not DECIMAL(7,2), preserves 157.4568.
- MONEY/CURRENCY are not universally floating-point types.
- DEFAULT does not imply NOT NULL.
- CHECK can accept UNKNOWN; nullable values may pass.
- MySQL FK enforcement depends on the storage engine and configuration.
- Referential actions depend on business rules and platform support.
- Modeling-tool validation cannot validate all business requirements.
- Denormalization should follow evidence and include a consistency strategy.

---

## Review Checklist

- [ ] Explain why modeling is iterative.
- [ ] Distinguish entity type, instance, attribute, and relationship.
- [ ] Read minimum and maximum cardinality at both relationship ends.
- [ ] Place the FK correctly in One-to-Many.
- [ ] Resolve Many-to-Many with a junction table.
- [ ] Explain attributes belonging to an association.
- [ ] Explain recursive category and employee hierarchies.
- [ ] Identify direct and indirect cycles.
- [ ] Explain recursive traversal and its query cost.
- [ ] Model product bundles using two FKs to Products.
- [ ] Enforce One-to-One with FK and uniqueness.
- [ ] Explain how FK placement affects mandatory participation.
- [ ] Distinguish supertype and subtype attributes.
- [ ] State PK and composite PK integrity rules.
- [ ] Compare Natural and Surrogate Keys.
- [ ] Explain Numeric IDs versus UUIDs.
- [ ] Define an attribute's Domain.
- [ ] Choose appropriate text, integer, decimal, and temporal types.
- [ ] Calculate precision, scale, and integer capacity.
- [ ] Distinguish NULL, zero, and an empty string.
- [ ] Explain DEFAULT, NOT NULL, and CHECK separately.
- [ ] Explain CHECK versus WHERE handling of UNKNOWN.
- [ ] Identify referential insert, update, and delete violations.
- [ ] Choose appropriate referential actions.
- [ ] Explain denormalization benefits, costs, and freshness.
- [ ] Distinguish forward, reverse, and round-trip engineering.
- [ ] Describe the three layers of sample data.
- [ ] Distinguish load testing from stress testing.
