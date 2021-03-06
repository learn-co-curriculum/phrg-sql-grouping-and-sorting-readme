# Grouping and Sorting Data 

## Objectives 

1. Explain the importance of grouping and sorting data stored in a database
2. Group and sort data with the `GROUP BY()` and `ORDER BY()` keywords
3. Craft advanced queries using aggregator functions along with sorting keywords and other conditional clauses 

## Grouping and Sorting Data

SQL isn't picky about how it returns data to you, based on your queries. It will simply return the relevant table rows in the order in which they exist in the table. This is often insufficient for the purposes of data analysis and organization. 

How common is it to order a list of items alphabetically? Or numerically from least to greatest? 

We can tell our SQL queries and aggregate functions to group and sort our data using a number of clauses:

* `ORDER BY()`
* `LIMIT`
* `GROUP BY()`
* `HAVING` and `WHERE`
* `ASC`/`DESC`


Let's take a closer look at how we use these keywords to narrow our search criteria as well as to order and group it.

## Setting up the Database

Some cats are very famous, and accordingly very wealthy. Our Pets Database will have a `cats` table in which each cat has a name, age, breed, and net worth. Our database will also have an `owners` table and `cats_owners` join table so that a cat can have many owners and an owner can have many cats.

**Creating the Database:**

Create the database in your terminal with the following: 

```bash
sqlite3 pets_database.db
```

**Creating the tables:**

In the `sqlite3>` prompt in your terminal:

**`cats` table:**

```sql
CREATE TABLE cats (
id INTEGER PRIMARY KEY,
name TEXT,
age INTEGER,
breed TEXT, 
net_worth INTEGER
);
```

**`owners` Table:**

```sql
CREATE TABLE owners (id INTEGER PRIMARY KEY, name TEXT);
```

**`cats_owners` Table:**

```sql
CREATE TABLE cats_owners (
cat_id INTEGER,
owner_id INTEGER
);
```

**Inserting the values:**

**`cats`:**

```sql
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (1, "Maru", 3, "Scottish Fold", 1000000);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (2, "Hana", 1, "Tabby", 21800);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (3, "Grumpy Cat", 4, "Persian", 181600);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (4, "Lil\' Bub", 2, "Tortoiseshell", 2000000);
```

**`owners`:**

```sql
INSERT INTO owners (name) VALUES ("mugumogu");
INSERT INTO owners (name) VALUES ("Sophie");
INSERT INTO owners (name) VALUES ("Penny");
```

**`cats_owners`:**

```sql
INSERT INTO cats_owners (cat_id, owner_id) VALUES (3, 2);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (3, 3);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (1, 2);
```

### Code Along I: `ORDER BY()`

#### Syntax

```sql
SELECT column_name, column_name
FROM table_name
ORDER BY column_name ASC|DESC, column_name ASC|DESC;
```

`ORDER BY()` will automatically sort the returned values in ascending order. Use the `DESC` keyword, as above, to sort in descending order. 

#### Exercise

Imagine you're working for an important investment firm in Manhattan. The investors are interested in investing in a lucrative and popular cat. They need your help to decide which cat that will be. They want a list of famous and wealthy cats. We can do that with a basic `SELECT` statement:

```sql
SELECT * FROM cats WHERE net_worth > 0;
```

This will return:

```bash
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Maru             3           Scottish Fold  1000000   
Hana             1           Tabby          21800     
Grumpy Cat       4           Persian        181600    
Lil\' Bub         2           Tortoiseshell  2000000   
```   

Our investors are busy people though. They don't have time to manually sort through this list of cats for the best candidate. They want you to return the list to them with the cats sorted by net worth, from greatest to least.  

We can do so with the following lines:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC;
```

This will return:

```bash
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Lil\' Bub        2           Tortoiseshell  2000000   
Maru             3           Scottish Fold  1000000   
Grumpy Cat       4           Persian        181600    
Hana             1           Tabby          21800     
```

### Code Along II: The `LIMIT` Keyword

Turns out our investors are very impatient. They don't want to review the list themselves, they just want you to return to them the wealthiest cat. We can accomplish this by using the `LIMIT` keyword with the above query:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC LIMIT 1;
```

Which will return:

```bash
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Lil\' Bub        2           Tortoiseshell  2000000   
```

The `LIMIT` keyword specifies how many of the records resulting from the query you'd like to actually return.

### Code Along III: `GROUP BY()`

The `GROUP BY()` keyword is very similar to `ORDER BY()`. The only difference is that `ORDER BY()` sorts the resulting data set of basic queries while `GROUP BY()` sorts the result sets of aggregate functions. 

#### Syntax

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

#### Exercise

Let's calculate the sum of the net worth of all of the cats that belong to our second owner:

```sql
SELECT SUM(cats.net_worth)
FROM owners
INNER JOIN cats_owners
ON owners.id = cats_owners.owner_id
JOIN cats ON cats_owners.cat_id = cats.id
WHERE cats_owners.owner_id = 2;
```

This should return:

```sql
SUM(cats.net_worth) 
--------------------
1181600 
```

In the above query, we use the `SUM(cats.net_worth)` aggregator. `SUM` looks at all of the values in the `net_worth` column of the `cats` table (or whichever column you specify in parentheses) and takes the sum of the those values.  

### Code Along IV: `HAVING` vs `WHERE` clause<sup>1</sup>  
Suppose we have a table called `employee_bonus` as shown below. Note that the table has multiple entries for employees Abigail and Matthew. 

**`employee_bonus`**:

Employee   | Bonus
-----------|-------
Matthew    |1000|
Abigail    |2000|
Matthew    |500|
Tom     	  |700|
Abigail 	  |1250|  

To calculate the total bonus that each employee received, we would write a SQL statement like this:  

```sql
SELECT employee, sum(bonus) from employee_bonus group by employee;
```  

This should return:  

Employee   | Bonus
-----------|-------
Matthew    |1500|
Abigail    |3250|
Tom        |700|

Now, suppose we wanted to find the employees who received more than $1,000 in bonuses for the year of 2007. You might think that we could write a query like this:  

```sql  
BAD SQL:
SELECT employee, SUM(bonus) FROM employee_bonus 
GROUP BY employee WHERE SUM(bonus) > 1000;
```  

Unfortunately the above will not work because the `WHERE` clause doesn’t work with aggregates – like `SUM`, `AVG`, `MAX`, etc. What we need to use is the `HAVING` clause. The `HAVING` clause was added to SQL so that we could compare aggregates to other values – just how the `WHERE` clause can be used with non-aggregates. Now, the correct SQL will look like this:

```sql
GOOD SQL:
SELECT employee, SUM(bonus) FROM employee_bonus 
GROUP BY employee HAVING SUM(bonus) > 1000;
```  

#### Difference between `HAVING` and `WHERE` clause
The difference between the `HAVING` and `WHERE` clause in SQL is that the `WHERE` clause can not be used with aggregates but the `HAVING` clause can. One way to think of it is that the `HAVING` clause is an additional filter to the `WHERE` clause.  


## Resources: 
* [`HAVING` vs `WHERE` clauses](http://www.programmerinterview.com/index.php/database-sql/having-vs-where-clause/)

* [Video Review- SQL Joins Overview](https://www.youtube.com/watch?v=qfB1MRnzk4g) 

## Does this need an update?

Please open a [GitHub issue](https://github.com/learn-co-curriculum/phrg-sql-grouping-and-sorting-readme/issues) or [pull-request](https://github.com/learn-co-curriculum/phrg-sql-grouping-and-sorting-readme/pulls). Provide a detailed description that explains the issue you have found or the change you are proposing. Then "@" mention your instructor on the issue or pull-request, and send them a link via Connect.

<p data-visibility='hidden'>PHRG Grouping and Sorting Data</p>
