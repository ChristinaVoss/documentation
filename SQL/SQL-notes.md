# SQL how to guide

### Create a table

**Syntax**

```sql
CREATE TABLE <table_name> (<col1_name> <COL1_TYPE>, <col2_name> <COL2_TYPE>, ... );
```

**Example**

```sql
CREATE TABLE groceries (id INTEGER PRIMARY KEY, name TEXT, quantity INTEGER );
```

If you don't want to manually handle the id (which could easily go wrong....), use AUTOINCREMENT:

```sql
CREATE TABLE groceries (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, quantity INTEGER );
```

### Insert values into a table

**Syntax**

```sql
INSERT INTO <table_name> VALUES (<value_for_col1>, <value_for_col2>, ...);
```

**Example**

```sql
INSERT INTO groceries VALUES (1, "Bananas", 4);
```

### Insert values into a table with specified columns (select which you add to)

**Syntax**

```sql
INSERT INTO <table_name>(col2_name, col3_name,...) VALUES (<value_for_col2>, <value_for_col3>, ...);
```

**Example**

```sql
INSERT INTO exercise_logs(type, minutes, calories, heart_rate) VALUES ("biking", 30, 100, 110);
```
__Notice in the above, that the id column is not inserted__

### Display table (all values)

**Syntax**

```sql
SELECT * FROM <table_name>;
```

**Example**

```sql
SELECT * FROM groceries;
```

### Display values from single column (all rows)

**Syntax**

```sql
SELECT <col_name> FROM <table_name>;
```

**Example**

```sql
SELECT price FROM groceries;
```

### Display table (all values), then filter (WHEN) and order (ORDERBY)

**Syntax**

```sql
SELECT * FROM <table_name> WHERE <some_predicate> ORDER BY <column>;
```

**Example**

```sql
SELECT * FROM groceries WHERE aisle > 5 ORDER BY aisle;
```

## Aggregating data

### Show count (SUM) of items in a column

**Syntax**

```sql
SELECT SUM(<col_name>) FROM <table_name>;
```

**Example**

```sql
SELECT SUM(quantity) FROM groceries;
```

### Show count (SUM) of items in a column, grouped by a column

**Syntax**

```sql
SELECT <col_to_group_by>, SUM(<col_name>) FROM <table_name> GROUP BY <col_to_group_by>;
```

**Example**

```sql
SELECT aisle, SUM(quantity) FROM groceries GROUP BY aisle;
```



### Show maximum (MAX) value in a column

**Syntax**

```sql
SELECT MAX(<col_name>) FROM <table_name>;
```

**Example**

```sql
SELECT MAX(quantity) FROM groceries;
```

### More complex queries with AND/OR

**Syntax**

```sql
/* AND */
SELECT * FROM <table_name> WHERE <some_predicate> AND <some_predicate>;
/* OR */
SELECT * FROM <table_name> WHERE <some_predicate> OR <some_predicate>;
```

**Example**

```sql
/* AND */
SELECT * FROM exercise_logs WHERE calories > 50 AND minutes < 30;
/* OR */
SELECT * FROM exercise_logs WHERE calories > 50 OR heart_rate > 100;
```

## IN and Subqueries

### Using IN with list of values

**Syntax**

```sql
SELECT * FROM <table_name> WHERE type IN (<string1_or_value_to_match>, <string2_or_value_to_match>, ...);
```

**Example**

```sql
SELECT * FROM exercise_logs WHERE type IN ("biking", "hiking", "tree climbing", "rowing");

/* Identical to: */
SELECT * FROM exercise_logs WHERE type = "biking" OR type = "hiking" OR type = "tree climbing" OR type = "rowing";
```

### Using IN with subquery

Imagine you have two tables, exercise_logs and drs_favourites. Instead of manually listing the values found in a row from one table as the lsit in another (as used above), you can use a SELECT from one table as the list in the second.

**Syntax**

```sql
SELECT * FROM <table1_name> WHERE type IN (SELECT <col_name> FROM <table2_name>);
```

**Example**

```sql
SELECT * FROM exercise_logs WHERE type IN (
    SELECT type FROM drs_favorites);
```

### Using LIKE to allow fuzzier match

IN matches exact, so if you miss out a single letter from a string it will fail. LIKE matches like a regex pattern.

**Syntax**

```sql
SELECT * FROM <table1_name> WHERE type IN (
    SELECT <col_name> FROM <table2_name> WHERE <col_name> LIKE <%pattern%>);
```

**Example**

```sql
SELECT * FROM exercise_logs WHERE type IN (
    SELECT type FROM drs_favorites WHERE reason LIKE "%cardiovascular%");
```

## Restricting grouped results with HAVING

Results are grouped when using aggregate functions (SUM, MAX, AVG, MIN, ...). HAVING is similar to WHERE, but is used on the grouped result, wheras WHERE checks every individual row (and does not check the grouped value).

### HAVING

**Syntax**

```sql
SELECT <col_to_show(label_for_aggregate)>, <aggregate_function> 
    AS <alias_name_for_aggregate_function_result>
    FROM <table_name>
    GROUP BY <col_to_group_by>
    HAVING <some_predicate>;
```

**Example**

```sql
SELECT type, SUM(calories)
    AS total_calories 
    FROM exercise_logs
    GROUP BY type
    HAVING total_calories > 150;
```

Simpler example (doesn't show the result of the aggregate function, only the values from the column/rows that matched the result):

**Example**

```sql
SELECT type FROM exercise_logs GROUP BY type HAVING COUNT(*) >= 2;
```

### CASE STATEMENTS

CASE lets us actually create a new column based on certain calculations or other results (works similarly to a switch statement).

```sql
SELECT <col1_to_show>, <col2_to_show>,
    CASE 
        WHEN <some_predicate> THEN <a_name_for_this_value>
        WHEN <some_predicate> THEN <a_name_for_this_value>
        WHEN <some_predicate> THEN <a_name_for_this_value>
        ELSE <a_name_for_this_value>
    END as <new_col_name>
FROM <table_name>;
```

**Example**

```sql
SELECT type, heart_rate,
    CASE 
        WHEN heart_rate > 220-30 THEN "above max"
        WHEN heart_rate > ROUND(0.90 * (220-30)) THEN "above target"
        WHEN heart_rate > ROUND(0.50 * (220-30)) THEN "within target"
        ELSE "below target"
    END as "hr_zone"
FROM exercise_logs;
```

## Splitting data into related tables

### JOIN

Shows three types of joins, from bad to good. First, the cross join, is the simplest, but not very useful. Will do n*m rows from crossing a table with n rows with another table with m rows.

The second, and implicit inner join, creates something closer to the result you may want, but is considered bad practice compared to the third, an explicit inner join (where you fully control the output).

**Syntax**

```sql
/* cross join */
SELECT * FROM <table1_name>, <table2_name>;

/* implicit inner join */
SELECT * FROM <table1_name>, <table2_name>
    WHERE <table1_name>.<table1_id_col> = <table2_name>.<table2_id_col>;
    
/* explicit inner join - JOIN */
SELECT <table1_name>.<table1_col1_to_show>, <table1_name>.<table1_col2_to_show>, <table2_name>.<table2_col1_to_show>, ... /* using table name is optional, but increases control */
    FROM <table1_name>
    JOIN <table2_name>
    ON <table1_name>.<table1_id_col> = <table2_name>.<table2_id_col>
    WHERE <some_predicate>; /*optional*/
```

**Example**

```sql
/* cross join */
SELECT * FROM student_grades, students;

/* implicit inner join */
SELECT * FROM student_grades, students
    WHERE student_grades.student_id = students.id;
    
/* explicit inner join - JOIN */
SELECT students.first_name, students.last_name, students.email, student_grades.test, student_grades.grade FROM students
    JOIN student_grades
    ON students.id = student_grades.student_id
    WHERE grade > 90;
```

### LEFT OUTER JOIN

Inner joins only show rows where the object exists in both tables. If you want results that include objects what may have null values as they only matched one table, use outer joins.

**Syntax**

```sql   
/* explicit inner join - JOIN */
SELECT <table1_name>.<table1_col1_to_show>, <table1_name>.<table1_col2_to_show>, <table2_name>.<table2_col1_to_show>, ... /* using table name is optional, but increases control */
    FROM <table1_name>
    LEFT OUTER JOIN <table2_name>
    ON <table1_name>.<table1_id_col> = <table2_name>.<table2_id_col>
```

**Example**

```sql
/* outer join */ 
SELECT students.first_name, students.last_name, student_projects.title
    FROM students /* will include students from here */
    LEFT OUTER JOIN student_projects /* even if they don't exist here */
    ON students.id = student_projects.student_id;
```

### SELF JOIN

Sometimes you want to join the same table with itself (if it has values that are linked to another column in the same table). When doing this you can get problems with ambiguity, as the interpreter won't know which table you are referring to when naming columns, so you can add an _alias_ to the second (or first if you prefer) to keep them apart.

**Syntax**

```sql   
/* explicit inner join - JOIN */
SELECT <table1_name>.<table1_col1_to_show>, <table1_name>.<table1_col2_to_show>, <aliastable_name>.<aliastable_col1_to_show>, ... /* using table name is optional, but increases control */
    FROM <table1_name>
    JOIN <table1_name> <alias>
    ON <table1_name>.<table1_id_col> = <aliastable_name>.<aliastable_id_col>
```

**Example**

```sql
/* self join */
SELECT students.first_name, students.last_name, buddies.email as buddy_email
    FROM students
    JOIN students buddies
    ON students.buddy_id = buddies.id;
```
## CHANGING ENTRIES IN THE DATABASE
### UPDATE

Change the value(s) of an existing row in a table

**Syntax**

```sql   
UPDATE <table_name> SET <col_name> = <a_value> WHERE <some_predicate>;
```

**Example**

```sql
UPDATE diary_logs SET content = "I had a horrible fight with OhNoesGuy" WHERE id = 1;
```

### DELETE

Remove a row in a table

**Syntax**

```sql   
DELETE FROM <table_name> WHERE <some_predicate>;
```

**Example**

```sql
DELETE FROM diary_logs WHERE id = 1;
```

## ALTER TABLES
### Add a row to a table

**Syntax**

```sql   
ALTER TABLE <table_name> ADD <col_name> <col_type>;
```

**Example**

```sql
/* basic - will add NULL values to existing rows*/
ALTER TABLE diary_logs ADD emotion TEXT;

/* with specified default value to use instead of NULL */
ALTER TABLE diary_logs ADD emotion TEXT default "unknown";
```

### Delete a table (dangerous!)

**Syntax**

```sql   
DROP TABLE <table_name>;
```

**Example**

```sql
DROP TABLE diary_logs;
```