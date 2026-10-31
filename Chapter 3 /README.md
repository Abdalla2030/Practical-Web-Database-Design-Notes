# Chapter 3: Creating and Using Relational Databases with SQL

## Reference

- **Book:** Practical Web Database Design
- **Authors:** Chris Auld, Allan Kent, Rudy Limeback, Nigel Stanger, and Thearon Willis
- **Chapter:** 3
- **Topic:** Creating and Using Relational Databases with SQL

## 1. SQL Overview

SQL is a special-purpose, declarative language for defining, manipulating, and querying relational data.

You describe **what result you want**. The DBMS determines how to retrieve it, usually through an execution plan selected by its optimizer.

| Operation | Command |
|---|---|
| Create a table | `CREATE TABLE` |
| Change a table structure | `ALTER TABLE` |
| Remove a table | `DROP TABLE` |
| Add rows | `INSERT` |
| Modify rows | `UPDATE` |
| Remove rows | `DELETE` |
| Retrieve data | `SELECT` |

SQL products have different dialects. Data types, date handling, generated identifiers, and some syntax vary between DBMSs.

SQL also differs from the theoretical relational model: query results can contain duplicate rows unless they are explicitly removed.

## 2. Creating Tables

A table definition specifies its columns, data types, and constraints.

```sql
CREATE TABLE CUSTOMER (
    CustomerNo VARCHAR(9) NOT NULL,
    First VARCHAR(40),
    Last VARCHAR(40) NOT NULL,
    CreditLimit DECIMAL(12, 2) DEFAULT 0 NOT NULL,
    CONSTRAINT PK_CUSTOMER PRIMARY KEY (CustomerNo),
    CONSTRAINT CK_CUSTOMER_CREDIT CHECK (CreditLimit >= 0)
);
```

### Common Data Types

| Type | Purpose |
|---|---|
| `CHAR(n)` | Fixed-length character data |
| `VARCHAR(n)` | Variable-length character data |
| `INTEGER` | Whole numbers |
| `DECIMAL(p, s)` | Exact numeric values |
| `DATE` | Calendar dates |
| `TIMESTAMP` | Date and time, depending on the DBMS |

For `DECIMAL(p, s)`:

- `p` is the total number of digits.
- `s` is the number of digits after the decimal point.

Use an appropriate exact numeric type for monetary amounts.

An identifier containing digits is not necessarily numeric data. A customer number such as `1113-7741` is naturally represented as text.

### Constraints

| Constraint | Purpose |
|---|---|
| `NOT NULL` | Requires a value |
| `PRIMARY KEY` | Uniquely identifies each row and rejects `NULL` |
| `UNIQUE` | Prevents duplicate key values; treatment of `NULL` varies |
| `FOREIGN KEY` | Requires a reference to an existing key, unless the reference is permitted to be `NULL` |
| `CHECK` | Rejects values when its condition evaluates to false |
| `DEFAULT` | Supplies a value when an insertion omits the column |

A `DEFAULT` does not restrict which values are allowed.

A `CHECK` alone does not necessarily reject `NULL`: an unknown check result generally passes. Add `NOT NULL` when a value is mandatory.

### Foreign Keys

```sql
CREATE TABLE SALE (
    SaleNo INTEGER NOT NULL,
    CustomerNo VARCHAR(9),
    SaleDate DATE NOT NULL,
    Amount DECIMAL(12, 2) NOT NULL,
    CONSTRAINT PK_SALE PRIMARY KEY (SaleNo),
    CONSTRAINT FK_SALE_CUSTOMER
        FOREIGN KEY (CustomerNo)
        REFERENCES CUSTOMER (CustomerNo)
);
```

Here:

- Each sale has a unique sale number.
- A non-null customer number must reference an existing customer.
- A null customer number permits a sale without an identified customer.
- The example interprets such sales as cash sales.

Foreign keys enforce relationships; they do not automatically join tables in a query.

## 3. Changing and Removing Tables

### ALTER TABLE

Changes an existing table definition.

```sql
ALTER TABLE EMPLOYEE
ADD ManagerID INTEGER;
```

A foreign key can represent a relationship back to the same table:

```sql
ALTER TABLE EMPLOYEE
ADD CONSTRAINT FK_EMPLOYEE_MANAGER
FOREIGN KEY (ManagerID)
REFERENCES EMPLOYEE (EmployeeID);
```

Exact alteration syntax varies between DBMSs.

### DROP TABLE

```sql
DROP TABLE SALE;
```

Removes the table definition and its data. Dependencies can prevent the operation.

### DELETE versus DROP

| Command | Effect |
|---|---|
| `DELETE FROM SALE` | Removes rows; keeps the table |
| `DROP TABLE SALE` | Removes the table itself |

## 4. Inserting Data

Specify the destination columns explicitly:

```sql
INSERT INTO CUSTOMER (
    CustomerNo, First, Last, CreditLimit
)
VALUES (
    '1113-7741', 'Wayne', 'Jones', 0
);
```

Values correspond to the listed columns by position.

- Text literals use single quotes.
- Numeric literals are normally unquoted.
- Use `NULL` without quotes for missing information.
- An omitted column receives its default, or `NULL` if permitted.
- Constraints still apply.

To include an apostrophe in a string, double it:

```sql
'O''Brien'
```

## 5. Updating Data

```sql
UPDATE CUSTOMER
SET CreditLimit = 5000
WHERE CustomerNo = '1113-7741';
```

An update can use an expression based on the existing value:

```sql
UPDATE PRODUCT
SET Price = Price * 1.10;
```

**Without `WHERE`, every row is eligible for the update.**

Before modifying data, inspect the intended rows:

```sql
SELECT *
FROM CUSTOMER
WHERE CustomerNo = '1113-7741';
```

## 6. Deleting Data

```sql
DELETE FROM SALE
WHERE SaleNo = 12345;
```

**Without `WHERE`, all rows are eligible for deletion.**

Deleting referenced data may be rejected by a foreign key or trigger a configured referential action.

Deletion does not automatically imply that related records should also be deleted.

## 7. Basic SELECT Queries

```sql
SELECT CustomerNo, First, Last
FROM CUSTOMER;
```

- `SELECT` identifies the output columns or expressions.
- `FROM` identifies the source tables.
- `WHERE` filters rows.
- `ORDER BY` sorts the result.

### Selecting All Columns

```sql
SELECT *
FROM CUSTOMER;
```

Useful for inspection, but explicit column lists make application queries clearer and avoid retrieving unnecessary data.

### Duplicate Rows

```sql
SELECT Salesrep
FROM SALE;
```

A salesperson can appear repeatedly because multiple sales can have the same salesperson.

```sql
SELECT DISTINCT Salesrep
FROM SALE;
```

`DISTINCT` removes duplicates based on the **entire selected row**.

```sql
SELECT DISTINCT Salesrep, SaleDate
FROM SALE;
```

This returns distinct salesperson–date combinations.

## 8. Filtering Rows

### Comparison Operators

| Operator | Meaning |
|---|---|
| `=` | Equal |
| `<>` | Not equal |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |

```sql
SELECT Description, Price
FROM PRODUCT
WHERE Price > 10;
```

### Boolean Conditions

```sql
WHERE CreditLimit = 0
  AND Last = 'Jones'
```

- `AND`: both conditions must be true.
- `OR`: at least one condition must be true.
- `NOT`: negates a condition, subject to SQL's handling of unknown values.

Use parentheses to express the intended grouping:

```sql
WHERE (CreditLimit = 0 OR CustomerNo IS NULL)
  AND SaleDate >= '2002-08-01'
```

`AND` has higher precedence than `OR`. Missing parentheses can change the result.

### BETWEEN

```sql
WHERE Price BETWEEN 10 AND 20
```

Includes both endpoints.

For date-time ranges, an exclusive upper boundary avoids excluding times on the final day:

```sql
WHERE SaleDate >= '2002-08-01'
  AND SaleDate < '2002-09-01'
```

### IN

```sql
WHERE Last IN ('Jones', 'Adams', 'Johnston')
```

Tests membership in a list.

### LIKE

```sql
WHERE Last LIKE 'Jo%'
```

| Wildcard | Meaning |
|---|---|
| `%` | Zero or more characters |
| `_` | Exactly one character |

Case sensitivity depends on the DBMS and collation.

## 9. NULL and Three-Valued Logic

`NULL` represents missing, unknown, or inapplicable information.

It is not the same as:

- Zero.
- An empty string.
- A space.

Ordinary comparisons involving `NULL` produce an unknown result:

```sql
CustomerNo = NULL
```

Use:

```sql
CustomerNo IS NULL
```

or:

```sql
CustomerNo IS NOT NULL
```

`WHERE` keeps only rows for which its condition is **true**. False and unknown results are excluded.

Consequently:

```sql
WHERE CreditLimit <> 0
```

does not include rows whose credit limit is `NULL`.

## 10. Sorting Results

```sql
SELECT First, Last
FROM CUSTOMER
ORDER BY Last, First;
```

Sorting is applied by the listed columns in sequence.

```sql
ORDER BY Amount DESC, SaleNo ASC
```

- `ASC`: ascending; usually the default.
- `DESC`: descending.

Without `ORDER BY`, result order is not guaranteed.

`GROUP BY`, `DISTINCT`, and set operations do not guarantee output order.

## 11. Joining Tables

A join combines rows according to a relationship or condition.

### INNER JOIN

```sql
SELECT S.SaleNo, S.Amount, C.First, C.Last
FROM SALE S
JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo;
```

Returns only matching rows.

A sale with a null customer number has no matching customer and is excluded.

### LEFT JOIN

```sql
SELECT S.SaleNo, S.Amount, C.First, C.Last
FROM SALE S
LEFT JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo;
```

Preserves every row from `SALE`.

When no customer matches, the customer columns in the result contain `NULL`.

These result values can be null even when the original customer columns have `NOT NULL` constraints.

### Join Types

| Join | Rows retained |
|---|---|
| `INNER JOIN` | Matching rows |
| `LEFT JOIN` | Matching rows plus unmatched left rows |
| `RIGHT JOIN` | Matching rows plus unmatched right rows |
| `FULL OUTER JOIN` | Matching rows plus unmatched rows from both sides |
| `CROSS JOIN` | Every combination of left and right rows |

Support for specific join types varies between products.

### Join Cardinality

If a customer has three sales, joining that customer to sales produces three matching rows.

Repeated customer details are not automatically an error. Verify what one result row is supposed to represent.

An incorrect or missing join condition can multiply rows and inflate totals.

### Filtering an Outer Join

```sql
FROM SALE S
LEFT JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
WHERE C.CreditLimit = 0
```

The `WHERE` condition excludes unmatched sales because their customer credit limit is null.

Preserving rows during the join does not prevent a later filter from removing them.

## 12. Aggregate Functions

Aggregate functions summarize multiple rows.

| Function | Result |
|---|---|
| `COUNT(*)` | Number of rows |
| `COUNT(column)` | Number of non-null values |
| `SUM(column)` | Total of non-null values |
| `AVG(column)` | Average of non-null values |
| `MIN(column)` | Smallest non-null value |
| `MAX(column)` | Largest non-null value |

```sql
SELECT COUNT(*) AS NumSales,
       SUM(Amount) AS TotalSales,
       AVG(Amount) AS AverageSale
FROM SALE;
```

### NULL and Aggregation

For values `10`, `20`, and `NULL`:

| Expression | Result |
|---|---:|
| `COUNT(*)` | 3 |
| `COUNT(Value)` | 2 |
| `SUM(Value)` | 30 |
| `AVG(Value)` | 15 |

The average divides by the number of non-null values.

An aggregate query without `GROUP BY` returns one result row even when no input rows match:

- `COUNT` returns zero.
- `SUM`, `AVG`, `MIN`, and `MAX` return `NULL`.

### Counting After an Outer Join

When listing all customers, including those without sales:

```sql
SELECT C.CustomerNo,
       COUNT(S.SaleNo) AS NumSales
FROM CUSTOMER C
LEFT JOIN SALE S
  ON C.CustomerNo = S.CustomerNo
GROUP BY C.CustomerNo;
```

Use `COUNT(S.SaleNo)` to count actual sales.

`COUNT(*)` would also count the preserved customer row when no sale matches.

## 13. GROUP BY

`GROUP BY` forms groups so aggregates can be calculated for each group.

```sql
SELECT CustomerNo,
       COUNT(*) AS NumSales,
       SUM(Amount) AS TotalSales
FROM SALE
GROUP BY CustomerNo;
```

This returns one row per customer-number group.

Rows with a null grouping value form a single group.

### Selected Columns

For portable grouped queries, each selected expression should be:

- A grouping expression.
- An aggregate expression.
- An expression derived from those values.

```sql
SELECT C.CustomerNo, C.First, C.Last,
       COUNT(*) AS NumSales
FROM SALE S
JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
GROUP BY C.CustomerNo, C.First, C.Last;
```

Some DBMSs allow additional columns when they recognize a functional dependency.

### Before and After Grouping

Before:

| CustomerNo | Amount |
|---|---:|
| A | 100 |
| A | 150 |
| B | 200 |

After grouping by customer:

| CustomerNo | TotalSales |
|---|---:|
| A | 250 |
| B | 200 |

Grouping does not update the source rows or guarantee sorting.

## 14. HAVING

`WHERE` filters rows before grouping.

`HAVING` filters groups after aggregation.

```sql
SELECT CustomerNo,
       SUM(Amount) AS TotalSales
FROM SALE
WHERE Amount > 0
GROUP BY CustomerNo
HAVING SUM(Amount) > 2000;
```

Steps:

1. Keep individual sales with positive amounts.
2. Group the remaining sales by customer.
3. Calculate each group's total.
4. Keep groups whose total exceeds 2000.

This is invalid at the same query level:

```sql
WHERE SUM(Amount) > 2000
```

The group total is not available at the row-filtering stage.

| Clause | Filters |
|---|---|
| `WHERE` | Individual rows |
| `HAVING` | Groups |

## 15. Comments, Expressions, and Constants

### Comments

```sql
-- Retrieve products below their reorder level
SELECT Description
FROM PRODUCT
WHERE QtyInStock < ReorderLevel;
```

`--` starts a comment that continues to the end of the line.

### Calculated Columns

```sql
SELECT Description,
       QtyInStock - ReorderLevel AS QtyRemaining
FROM PRODUCT;
```

| Result | Meaning |
|---|---|
| Positive | Stock is above the reorder level |
| Zero | Stock equals the reorder level |
| Negative | Stock is below the reorder level |

The calculation appears in the query result. It does not add a stored column or change the table.

### Constants

```sql
SELECT S.Salesrep,
       'has sold to' AS Relationship,
       C.First,
       C.Last
FROM SALE S
JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo;
```

The text constant appears in every result row as a separate column.

It does not automatically concatenate the columns into a sentence.

## 16. Table and Column Aliases

### Table Aliases

```sql
FROM SALE S
JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
```

- `S` refers to the sale table reference.
- `C` refers to the customer table reference.

Aliases shorten references and distinguish different roles.

They do not rename stored tables or copy their data.

Some products allow `SALE AS S`; support for `AS` in table aliases varies.

### Column Aliases

```sql
SELECT AVG(Price) AS AveragePrice,
       AVG(QtyInStock) AS AverageQty
FROM PRODUCT;
```

Column aliases name output columns.

They do not rename the original stored columns.

## 17. Self-Joins

A self-join references the same table in different roles.

Example: list employees and their managers.

```sql
SELECT EMP.Name AS EmployeeName,
       MGR.Name AS ManagerName
FROM EMPLOYEE EMP
JOIN EMPLOYEE MGR
  ON EMP.ManagerID = MGR.EmployeeID;
```

| Alias | Role |
|---|---|
| `EMP` | Employee being listed |
| `MGR` | Employee acting as that person's manager |

The condition matches an employee's manager identifier to the manager's employee identifier.

A self-join is not a separate join operator. It can use an inner or outer join.

Use a left join if employees without a matching manager must remain:

```sql
FROM EMPLOYEE EMP
LEFT JOIN EMPLOYEE MGR
  ON EMP.ManagerID = MGR.EmployeeID
```

If the data represents the top employee as their own manager, that employee appears with their own name as manager.

## 18. Set Operations

Set operations combine rows from separate query results.

| Operator | Result |
|---|---|
| `UNION` | Rows in either result, with duplicate complete rows removed |
| `UNION ALL` | All rows from both results, including duplicates |
| `INTERSECT` | Rows present in both results |
| `EXCEPT` | Rows in the first result but absent from the second |

For:

- First result: `1, 2, 3`
- Second result: `2, 3, 4`

| Operation | Result |
|---|---|
| `UNION` | 1, 2, 3, 4 |
| `INTERSECT` | 2, 3 |
| First `EXCEPT` second | 1 |
| Second `EXCEPT` first | 4 |

These lists illustrate membership, not guaranteed output order.

### Compatibility Rules

Both queries must return:

1. The same number of columns.
2. Compatible data types in corresponding positions.

Columns are matched by position, not by name.

Output column names generally come from the first query.

Support and naming of set operators vary between DBMSs.

### JOIN versus Set Operations

| Feature | Purpose |
|---|---|
| `JOIN` | Combine related rows, often adding columns |
| Set operation | Combine compatible result rows |

### Example: Customer Sales and Cash Sales

```sql
SELECT C.CustomerNo,
       C.First AS FirstName,
       C.Last AS LastName,
       COUNT(*) AS NumSales
FROM SALE S
JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
GROUP BY C.CustomerNo, C.First, C.Last

UNION

SELECT ' ', ' ', 'Cash sale', COUNT(*)
FROM SALE
WHERE CustomerNo IS NULL

ORDER BY LastName, FirstName;
```

The first query returns sales counts for identified customers.

The second returns one row containing the total number of cash sales.

If there are five cash-sale records:

| CustomerNo | FirstName | LastName | NumSales |
|---|---|---|---:|
| *(space)* | *(space)* | Cash sale | 5 |

This is **five sales**, not five identified customers.

Important details:

- The cash branch returns a count of zero even if there are no cash sales.
- Customers without sales are absent from the inner-join branch.
- The constants must be compatible with the first query's column types.
- Spaces are display values, not equivalent to missing information.
- `ORDER BY` sorts the combined result.
- Repeating the same aliases in the second query is unnecessary.
- `UNION ALL` is also suitable when the branches cannot produce duplicate complete rows.

## 19. Building SELECT Queries Incrementally

Turn the business question into smaller decisions.

| Step | Question |
|---|---|
| `FROM` / `JOIN` | Which tables and relationships are needed? |
| `WHERE` | Which individual rows qualify? |
| `GROUP BY` | What should one result group represent? |
| `HAVING` | Which groups qualify? |
| `SELECT` | Which columns and calculations should appear? |
| `ORDER BY` | How should the result be sorted? |

This is a development approach, not a required physical execution order.

### Logical Processing Model

A useful simplified model is:

1. `FROM` and joins.
2. `WHERE`.
3. `GROUP BY` and aggregation.
4. `HAVING`.
5. `SELECT`.
6. `DISTINCT`, when specified.
7. `ORDER BY`.

The optimizer can choose a different physical plan while preserving the query's meaning.

### Testing Strategy

- Define what one output row should represent.
- Predict the result using a small known dataset.
- Test joins before adding filters.
- Test each condition separately.
- Combine conditions gradually.
- Check row counts before calculating totals.
- Add aggregation after validating the selected rows.
- Include boundary cases and rows that should be excluded.
- Compare calculated totals with manually verified values.

## 20. Detailed Example: Daily Non-Credit Sales

### Business Question

What is the total value of sales to non-credit customers, for each day during August 2002?

### Business Meaning

`CreditLimit` is the maximum credit allowed to a customer.

It is not:

- Their previous total purchases.
- Their total payments.
- Their remaining balance.

For this example:

| Condition | Interpretation |
|---|---|
| Customer credit limit is zero | Registered non-credit customer |
| Sale customer number is null | Cash sale without an identified customer |

Treating both categories as non-credit sales is an explicit business rule.

`SALE.Amount` stores a sale's value. Summing it calculates sales value, not necessarily money already collected.

### Step 1: Preserve Cash Sales

An inner join removes sales without a matching customer.

```sql
FROM SALE S
LEFT JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
```

With the book's data, the inner join returns six sales, while the left join preserves all seven.

### Step 2: Select Non-Credit Sales

```sql
WHERE (C.CreditLimit = 0 OR S.CustomerNo IS NULL)
```

The book tests:

```sql
WHERE (C.CreditLimit = 0 OR C.CreditLimit IS NULL)
```

The null credit-limit values arise from unmatched outer-join rows.

When the customer credit limit is mandatory and non-null foreign keys are valid, both conditions identify the intended categories.

Checking `S.CustomerNo IS NULL` expresses the cash-sale rule directly.

### Step 3: Restrict the Date Range

```sql
AND S.SaleDate >= '2002-08-01'
AND S.SaleDate < '2002-09-01'
```

The exclusive upper boundary includes all of August even when sale dates contain time values.

### Step 4: Calculate Daily Totals

Assuming `SaleDate` contains dates without times:

```sql
SELECT S.SaleDate,
       SUM(S.Amount) AS TotalSales
FROM SALE S
LEFT JOIN CUSTOMER C
  ON S.CustomerNo = C.CustomerNo
WHERE (C.CreditLimit = 0 OR S.CustomerNo IS NULL)
  AND S.SaleDate >= '2002-08-01'
  AND S.SaleDate < '2002-09-01'
GROUP BY S.SaleDate
ORDER BY S.SaleDate;
```

Expected result:

| SaleDate | TotalSales |
|---|---:|
| 2002-08-12 | 167.65 |
| 2002-08-13 | 5120.25 |
| 2002-08-14 | 2327.34 |

Each row contains the total qualifying sales value for one day.

If `SaleDate` includes times, group by a date-only expression appropriate for the DBMS.

Dates without qualifying sales do not appear automatically.

## 21. Stored Procedures

A stored procedure is a named routine stored in the database.

It can encapsulate SQL and accept parameters, such as a start date and an end date.

Instead of hard-coding August 2002, an application can supply the required period.

Potential benefits include:

- Reusing database operations.
- Encapsulating business logic.
- Providing controlled database access.
- Replacing repeated SQL text with procedure calls.

Stored procedures are not automatically secure.

Security still depends on parameter handling, permissions, safe dynamic SQL, and protected connections.

Applications can also execute parameterized queries directly.

## 22. Performance and Correctness

A query can return the expected values and still be unnecessarily expensive.

Check whether you are:

- Retrieving more rows or columns than needed.
- Joining large tables without appropriate conditions.
- Multiplying rows before aggregation.
- Removing duplicates unnecessarily.
- Depending on an order that was never specified.

Summaries can be precomputed to improve repeated reads, but this adds storage and requires a strategy for keeping them current.

Correctness comes first: verify what the query returns before optimizing how it runs.

## 23. Common Mistakes

| Mistake | Correction |
|---|---|
| Using `= NULL` | Use `IS NULL` |
| Treating a space as equivalent to `NULL` | Preserve the distinction |
| Omitting `WHERE` from an update or delete unintentionally | Verify the target rows first |
| Using an inner join when unmatched rows are required | Choose an appropriate outer join |
| Filtering away unmatched outer-join rows accidentally | Review the `WHERE` condition |
| Summing values after a join multiplies rows | Validate join cardinality first |
| Using `COUNT(*)` to count optional joined records | Count a non-null key from the optional side |
| Filtering an aggregate with `WHERE` | Use `HAVING` at that query level |
| Assuming grouping sorts results | Add `ORDER BY` |
| Assuming aliases rename or copy stored tables | Treat them as query references |
| Matching set-operation columns by name | Match positions and compatible types |
| Assuming `UNION` preserves duplicates | Use `UNION ALL` when required |
| Interpreting sales value as collected payments | Follow the data model and business rules |

## 24. Review Checklist

- [ ] Explain why SQL is declarative.
- [ ] Distinguish table-definition commands from row-manipulation commands.
- [ ] Choose suitable data types.
- [ ] Explain primary, foreign, unique, check, and not-null constraints.
- [ ] Distinguish defaults from constraints.
- [ ] Create, alter, and drop tables.
- [ ] Insert, update, and delete rows safely.
- [ ] Select specific columns and remove duplicates when required.
- [ ] Build Boolean conditions with explicit grouping.
- [ ] Use `BETWEEN`, `IN`, and `LIKE`.
- [ ] Explain `NULL` and use `IS NULL` correctly.
- [ ] Sort results explicitly.
- [ ] Distinguish inner, outer, cross, and self-joins.
- [ ] Predict row multiplication when joining tables.
- [ ] Distinguish `COUNT(*)` from `COUNT(column)`.
- [ ] Explain how aggregates handle null values.
- [ ] Group rows and filter groups using `HAVING`.
- [ ] Use comments, expressions, constants, and aliases.
- [ ] Distinguish `UNION`, `UNION ALL`, `INTERSECT`, and `EXCEPT`.
- [ ] Verify compatibility between set-operation results.
- [ ] Explain why cash sales require preservation in the chapter example.
- [ ] Distinguish credit limits, sales values, and actual payments.
- [ ] Build and test a query incrementally.
- [ ] Explain the purpose and limitations of stored procedures.
- [ ] Consider both query correctness and performance.
