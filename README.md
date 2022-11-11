# RECURSIVE CTEs — A step by step guide on generating number series and building a Calendar Lookup Table using Recursive CTES

If you were to fill an empty column with a date range of 2022-01-01 to 2022-01-07 for example, you can easily use the query below

```
INSERT INTO table_1 (date)
VALUES  ('2022-01-01'),
        ('2022-01-02'),
        ('2022-01-03'),
        ('2022-01-04'),
        ('2022-01-05'),
        ('2022-01-06'),
        ('2022-01-07');
 ```
Good! 
But what if the date range you need is between January 1, 2022 to December 31, 2030, how would you do that? That’s where we can leverage the power of RECURSIVE CTEs. 

Before we start building the Calendar Lookup Table, let us quickly look at how a RECURSIVE CTE works with a couple of examples.

Example 1: Generating a number series between 1 to 100

```
WITH RECURSIVE number_series AS
(
  SELECT 1 AS my_number  -- } Anchor Member

  UNION ALL

  SELECT my_number + 1   
  FROM number_series     -- } Recursive Member
  WHERE my_number < 100  -- } Terminator (Asake Lol!)
)
SELECT my_number
FROM number_series ;	   -- } Invocation Statement
```

Example 2: Building on the query in example 1, we can also get an output of odd numbers between 1 to 100. This is achieved by add a condition in the Invocation Statement (WHERE my_number % 2 = 1)

```
WITH RECURSIVE odd_number_series AS
(
  SELECT 1 AS my_number

  UNION ALL

  SELECT my_number + 1
  FROM odd_number_series
  WHERE my_number < 100
)
SELECT my_number
FROM odd_number_series
WHERE my_number % 2 = 1;  -- } This conditions returns only odd numbers
```

Alright! Now that it’s “Enchanté” between you and RECURSIVE CTE, let’s build the Calendar Lookup Table.

So, we start by writing the RECURSIVE CTE query to generate our desired date range. 

# WARNING! 

In MySQL, the default maximum number of rows that can be displayed as output when using RECURSIVE CTE is 1000. 
Hence, since output will exceed 1000 rows, we need to reset the maximum limit by first running the query below

```
SET SESSION cte_max_recursion_depth = 10000;
```

Now let’s generate our date_series

```
WITH RECURSIVE date_series AS
(
  SELECT '2022-01-01' AS my_date

  UNION ALL

  SELECT DATE_ADD(my_date, INTERVAL 1 DAY) 
  FROM date_series
  WHERE my_date < '2030-12-31'
)
SELECT my_date
FROM date_series;
```

The next thing is to create an empty table to insert the generated date series. Let’s create the table

```
DROP TABLE IF EXISTS Calendar_Lookup;
CREATE TABLE Calendar_Lookup 
(
  Date DATE,
  Day_of_week_no INT,
  Day_of_week_name VARCHAR(32),
  Month_no INT,
  Month_name VARCHAR(32),
  Year INT,
  Weekend_Weekday VARCHAR(32)
);
```
Now that we have our table, let’s fill the first column, “Date” with results from the generated series

```
INSERT INTO Calendar_Lookup (date)
WITH RECURSIVE date_series AS
(
  SELECT '2022-01-01' AS my_date

  UNION ALL

  SELECT DATE_ADD(my_date, INTERVAL 1 DAY) 
  FROM date_series
  WHERE my_date < '2030-12-31'
)
SELECT my_date
FROM date_series;
```

We can run the query below to confirm our date column is filled in correctly
```
SELECT * FROM calendar_lookup;
```
Awesome!
Now that we have the date column populated, we can use the Update() function to fill the remaining parts of the table.

```
UPDATE calendar_lookup
SET 
  day_of_week_no = dayofweek(date),
  day_of_week_name = DATE_FORMAT(date, '%W'),
  Month_no = MONTH(date),
  Month_name = DATE_FORMAT(date, '%M'),
  Year = YEAR(date),
  Weekend_Weekday = CASE
                      WHEN dayofweek(date) IN (1, 7) THEN 'Weekend'
                      ELSE 'Weekday'
                    END;
```
Run the query again to confirm we have our complete Calendar_Lookup table
```
SELECT
	*
FROM calendar_lookup;
```

VOILA!!!
We have our completed Calendar Lookup Table.
