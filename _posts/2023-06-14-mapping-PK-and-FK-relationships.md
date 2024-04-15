# Mapping PK and FK relationships

Is using the ‘fk_’ prefix in column names good or bad?

#### Category: SQL

---

Many people argue that the column names in the parent table and child tables should be identical if they are primary and foreign keys. They insist that putting a prefix such as pk_ or fk_ in column names be avoided. Others believe that it is ok to use such prefixes in the column names. Both options serve their purposes and I think the long lasted debates to argue their preference will never end. Fortunately, I’ve had chances to work on databases where two different conventions were used and I would like to share my PERSONAL thoughts on some of the pros and cons of the choices.

## Primary key column ‘ID’ in almost all major tables

The database that I was exposed to when I took baby steps in SQL was using the option of using fk_ in the column names of the child tables. It was for the in-house manufacturing ERP software that was developed by a small IT team of a few masterminds. As seen in the mockup below, the name of the primary key column in the vendor table is just “ID” and the name of the foreign key columns for the vendor ID in the purchase invoice column is “fk_vendorID”.

```SQL
SELECT * 
FROM purchase_inv p
LEFT JOIN vendor v
  ON p.fk_vendorID = v.ID;
```

### Cons – the challenges that I experienced

- At first glance, I was confused when I saw columns named just “ID” in so many major tables. “ID” column in the raw material table, “ID” column in the bill of lading table, etc. Having “ID” columns literally everywhere, the table to which the “ID” column belongs always needed to be specified otherwise people had no idea which ID I was talking about. When the relationship between the foreign keys and the primary key was not explicitly identified by sharing the unified column name, the users really had to become subject matter experts (SMEs) to tell the relationship.
- As a newbie in the IT team with a finance background, it was not always clear which columns I should use to join two tables because their names were not identical. It took a relatively larger learning effort for mapping the relationship between tables because of the ambiguity. I was just lucky that the developers there were patient enough to answer my inquiries. So, I think this option is not for the larger companies with many users of the database with relatively high employee turnover.

### Pros – so is that really an issue?

- When tables are joined, the names of the tables are specified anyway, so repeating the name of the column seems redundant. For example, ‘ON sales_inv.customerID = customer.customerID’ could have been ‘ON sales_inv.fk_customerID = customer.ID’ according to this argument.
- The name of the column is descriptive enough to tell whether it is a primary key or a foreign key.
- The referential integrity can be maintained through the ‘Foreign key constraint’ when triggers are used in most relational database management systems (RDBMS). It ensures only the values in the primary key columns are allowed to be entered in the foreign key columns. So, there is a list of the constraints that the users can refer to.
- I honestly don’t really have things to complain about this option because employees will eventually need to become SMEs. Although there are some disadvantages such as the slower learning time that I mentioned, it doesn’t make sense that databases should be designed for someone who occasionally consumes them or who merely spends time or effort to understand them.
- As mentioned, the database was designed by a few dedicated people and they already have a very clear blueprint of all the relationships. Likewise, if it is expected that there would be only a few developers in a closed group, I would say why not use any convention they prefer?

## PK and FK columns with the identical name

This is the way how the parent and child tables are related in the database that I am currently working on. The pros and cons for this are pretty much the opposite of what they were for the other option. So, I will simply list them in point form and jump to describing what I did for database mapping.

```SQL
SELECT *
FROM CI_BILL
INNER JOIN CI_BSEG
  ON CI_BILL.BILL_ID = CI_BSEG.BILL_ID;
```

### Cons

- Some may argue that they cannot instantly tell if the column is a primary or foreign key by looking at the name of the column. The example above itself without any context doesn’t tell if the column BILL_ID is PK/FK in which table.

### Pros

- Explicit relationship between tables when you see the columns with the same name
- Lessor learning effort
- Do we really need to know if the column is PK or FK? Isn’t that mainly what the SQL engine would need, not us? Plus, Can’t we make the best guess by referring to the table name the column belong to? For example, it is more likely that the BILL_ID is PK of the BILL table rather than the Bill segment (BSEG) table.

## What effort have I made for mapping?

Besides the regular effort that I put to have a better understanding of the subject matters such as business processes and mapping of frequently used tables (something like a personal reference in the blurred image below), I wanted to come up with a list of tables and columns in the database. This is because 1. we are upgrading our customer information system (CIS) and I think it is great timing that I create a reproducible method for mapping and 2. I am scheduled to start internal training for SQL for the rest of the team and thought it would be beneficial if I can provide them with the reference list because it is foreseeable that they are going to start to ask for the columns they can use to join tables.

![mapping](/images/mapping.png)

### Trying a simpler way, hoping there is a list of constraints

Having worked on the current database that the column names for PK and FK are identical with no exception so far that I came across yet, I know that it is less likely that I find something when I dig into the constraint tables. However, it is always worth it to give it a try. Who knows this could have been a simpler task.

```SQL
SELECT TABLE_NAME
FROM ALL_CONSTRAINTS
WHERE CONSTRAINT_TYPE = 'R' -- "Referential integrity"
  AND R_CONSTRAINT_NAME IN
    (
      SELECT CONSTRAINT_NAME
      FROM ALL_CONSTRAINTS
      WHERE TABLE_NAME = 'CI_%' -- all t
        AND CONSTRAINT_TYPE IN ('U', 'P') -- "Unique" or "Primary key"
    );
```

As expected, no luck with this approach. I couldn’t find the list of the FK constraints in any table. I am sure that the constraints are saved somewhere else but just not in the database that I have access to.

### Gathering a list of all tables and columns

Alternatively, I came up with the idea of gathering all column information and identifying PK and FK myself. First off, get a list of all tables and columns from tables ALL_TABLES and ALL_TAB_COLUMNS, respectively.

```SQL
SELECT
    t.OWNER,
    t.TABLE_NAME,
    t.NUM_ROWS,
    c.COLUMN_ID,
    c.COLUMN_NAME,
    c.NUM_DISTINCT,
    c.DATA_TYPE,
    c.NULLABLE,
    c.NUM_NULLS
FROM
    ALL_TAB_COLUMNS c
INNER JOIN
    (
        SELECT *
        FROM ALL_TABLES
        WHERE ALL_TABLES.NUM_ROWS > 0
    ) t
    ON t.TABLE_NAME = c.TABLE_NAME
WHERE t.TABLE_NAME LIKE 'CI%'
ORDER BY
    t.TABLE_NAME ASC,
    c.COLUMN_ID ASC
```

Then I utilized the number of rows in each table (t.NUM_ROWS) and the number of distinct values in each row (c.NUM_DISTINCT) to identify the primary key. So, among many columns with identical names, I can tell it is the primary key of the table when the number of rows in the table is the same as the number of distinct values in the columns because there should be no duplicated values in the primary key column. The rest of the columns with the same name are foreign keys assuming that there is no exception to the column name convention that the FK column name is identical to that of the PK column

![PK-FK-formula](/images/PK-FK-formula.png)

Then I exported the query result to Excel and added some formulas to flag the primary keys and foreign keys as well as the parental table of the foreign key. Any fresh employee new to this database will be able to tell which table the foreign key originated from when they filter it by a certain column. I believe this would help them to identify columns they should use to join tables.

![PK-FK-result](/images/PK-FK-result.png)

I really don’t have any preference for how the PK and FK columns are named. They both worked and served their purposes. There obviously are some downsides to each option, especially from the perspective of efficiency but I would say they are addressed once the users get used to the data.
