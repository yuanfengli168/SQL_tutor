# Window functions from online courses:

## 1st course:

- platform: datacamp
- Name:PostgreSQL Summary Stats and Window Functions
- url:https://app.datacamp.com/learn/courses/postgresql-summary-stats-and-window-functions
- start date: 10/28/2021
- end date: pending.

### Aha-moment:

1. [The LAG window function returns the values for a row at a given offset above (before) the current row in the partition.](https://www.sqlservertutorial.net/sql-server-window-functions/sql-server-lag-function/)

### Contents and Codes:

-**Course Description**:
Have you ever wondered how data professionals use SQL to solve real-world business problems, like generating rankings, calculating moving averages and running totals, deduplicating data, or performing time intelligence? If you already know how to select, filter, order, join and group data with SQL, this course is your next step. By the end, you will be writing queries like a pro! You will learn how to create queries for analytics and data engineering with window functions, **the SQL secret weapon!** Using flights data, you will discover how **simple it is to use window functions, and how flexible and efficient they are.**

- **Chapters**:

1. Introduction to window functions:
2. Fetching, ranking, and paging:
3. Aggregate window functions and frames:
4. Beyong window functions:

### Chapter 1:

#### Part 1 out of 4:

- **class videos:**

  1. example:

  ```
  SELECT
      YEAR, Event, Country
  FROM Summer_Medals
  WHERE
      Medal = 'Gold'

  <!-- now we would like to add row numbers very quickly, using window functions -->
  SELECT
      Year, Even, Country
      ROW_NUMBER() OVER () AS Row_N
      FROM Summer_Medals
      WHERE
          Medal = 'Gold';
  ```

  **Few key points:**

  1. The over clause indicates that it is a window function
  2. the () after the OVER can be empty, but can also contain subclauses, such as ORDER BY, PARTITION BY, ROWS or RANGE,, PRECEDING, FOLLOWING, or UNBOUNDED
     - ORDER BY
     - PARTITION BY
     - ROW/RANGE PRECEDING/FOLLOWING/UNBOUNDED

- **Practices**:

  - Q1: Window functions vs GROUP BY Which of the following is FALSE?
  - Possible Answers:

    - Unlike GROUP BY results, window functions don't reduce the number of rows in the output.
    - Window functions can fetch values from other rows into the table, whereas GROUP BY functions cannot.
    - Window functions can open a "window" to another table, whereas GROUP BY functions cannot.
    - Window functions can calculate running totals and moving averages, whereas GROUP BY functions cannot.

    False one is 3rd, which because it happens in place.

  - Codings:

        Numbering rows

    The simplest application for window functions is numbering rows. Numbering rows allows you to easily fetch the nth row. For example, it would be very difficult to get the 35th row in any given table if you didn't have a column with each row's number.

        Numbering Olympic games in ascending order

    The Summer Olympics dataset contains the results of the games between 1896 and 2012. The first Summer Olympics were held in 1896, the second in 1900, and so on. What if you want to easily query the table to see in which year the 13th Summer Olympics were held? You'd need to number the rows for that.

  ```
  SELECT
  *,
  -- Assign numbers to each row
  ROW_NUMBER() OVER () AS Row_N
  FROM Summer_Medals
  ORDER BY Row_N ASC;


    SELECT
    Year,

    -- Assign numbers to each year
    ROW_NUMBER() OVER () AS Row_N
    FROM (
    SELECT DISTINCT Year
    FROM Summer_Medals
    ORDER BY Year ASC
    ) AS Years
    ORDER BY Year ASC;



  ```

#### Part 2 out of 4:

- **class videos:**

  1. example:

  ```
  SELECT Year, Even, Country,
    ROW_NUMBER() OVER (ORDER BY Year DESC) AS Row_N
  FROM Summer_Medals
  WHERE
    Medal = 'Gold'


  SELECT Year, Even, Country,
    ROW_NUMBER() OVER (ORDER BY Year DESC, Event ASC) AS Row_N
  FROM Summer_Medals
  WHERE
    Medal = 'Gold'

  <!-- using order by with two columns for window functiona and out table at same time -->



  <!-- Reigning Champions -->
  WITH Discus_Gold AS (
      SELECT
        YEAR, Country AS Champion
      FROM Summer_Medals
      WHERE
        Year IN (1996, 2000, 2004, 2008, 2012)
        AND Gender = 'Men' AND Medal = 'Gold'
        AND Event = 'Discus Throw'
  )
  SELECT
    Year, Champion,
    LAG(Champion, 1) OVER (ORDER BY Year ASC) AS Last_Champion
  FROM Discus_Gold
  ORDER BY Year ASC;
  ```

  **Key points**:

  1. window functions' order is taking before the outer oder in the ORDER BY clause
  2. so in other words: the window functions is always having more priority than other aggregating clauses.

- **Practices**:
  - Q1:You've already numbered the rows in the Summer Medals dataset. What if you need to reverse the row numbers so that the most recent Olympic games' rows have a lower number?
    - Instruction: Assign a number to each year in which Summer Olympic games were held so that rows with the most recent years have lower row numbers.

q2:Numbering Olympic athletes by medals earned
Row numbering can also be used for ranking. For example, numbering rows and ordering by the count of medals each athlete earned in the OVER clause will assign 1 to the highest-earning medalist, 2 to the second highest-earning medalist, and so on.
i2:For each athlete, count the number of medals he or she has earned.

i3:
Having wrapped the previous query in the Athlete_Medals CTE, rank each athlete by the number of medals they've earned.

```
SELECT
  Year,
  -- Assign the lowest numbers to the most recent years
  ROW_NUMBER() OVER (ORDER BY Year DESC) AS Row_N
FROM (
  SELECT DISTINCT Year
  FROM Summer_Medals
) AS Years
ORDER BY Year;


SELECT
  -- Count the number of medals each athlete has earned
  Athlete,
  Count(*) AS Medals
FROM Summer_Medals
GROUP BY Athlete
ORDER BY Medals DESC;


WITH Athlete_Medals AS (
  SELECT
    -- Count the number of medals each athlete has earned
    Athlete,
    COUNT(*) AS Medals
  FROM Summer_Medals
  GROUP BY Athlete)

SELECT
  -- Number each athlete by how many medals they've earned
  Athlete,
  ROW_NUMBER() OVER (ORDER BY Medals DESC) AS Row_N
FROM Athlete_Medals
ORDER BY Medals DESC;

```

q3:
Reigning weightlifting champions
A reigning champion is a champion who's won both the previous and current years' competitions. To determine if a champion is reigning, the previous and current years' results need to be in the same row, in two different columns.

Return each year's gold medalists in the Men's 69KG weightlifting competition.

```
SELECT
  -- Return each year's champions' countries
  Year,
  country AS champion
FROM Summer_Medals
WHERE
  Discipline = 'Weightlifting' AND
  Event = '69KG' AND
  Gender = 'Men' AND
  Medal = 'Gold';





<!-- bug1, forgot to write order by -->
  WITH Weightlifting_Gold AS (
  SELECT
    -- Return each year's champions' countries
    Year,
    Country AS champion
  FROM Summer_Medals
  WHERE
    Discipline = 'Weightlifting' AND
    Event = '69KG' AND
    Gender = 'Men' AND
    Medal = 'Gold')
SELECT
  Year, Champion,
  -- Fetch the previous year's champion
  LAG(Champion, 1) OVER
    (ORDER BY Year ASC) AS Last_Champion     <!-- bug1, forgot to write order by -->
FROM Weightlifting_Gold
ORDER BY Year ASC;


<!-- if we need next year, you can use DESC like this -->
WITH Weightlifting_Gold AS (
  SELECT
    -- Return each year's champions' countries
    Year,
    Country AS champion
  FROM Summer_Medals
  WHERE
    Discipline = 'Weightlifting' AND
    Event = '69KG' AND
    Gender = 'Men' AND
    Medal = 'Gold')

SELECT
  Year, Champion,
  -- Fetch the previous year's champion
  LAG(Champion, 1) OVER
    (ORDER BY Year DESC) AS Last_Champion
FROM Weightlifting_Gold
ORDER BY Year ASC;
```

Part 3 out of 4:

- class videos:

  - Major concepts:

    1. Learned order by last two classes
    2. Now this class will focus on Partition by subclass!

  - Detail in Partition By
    1. similar to group by
    2. PARTITION BY splits the table into partions based on a column's unique values
       - but the results aren't rolled into one column
    3. Operated on separately by the window function
       - ROW_NUMBER will reset for each partition
       - LAG will only fetch a row's previous value if its previous row is in the same partition

- TWO CODING EXAMPLES IN QUERY:

```
<!-- WRONG FOR TRIPLE JUMP, WHERE LAST CHAMPION FOR SWE SHOULD BE NULL -->
WITH Discus_Gold AS (
  SELECT
    Year, Event, Country AS Champion
  FROM Summer_Medals
  WHERE
    Year IN (2004, 2008, 2012)
    AND Gender = 'Men'
    AND Medal = 'Gold'
    AND Event IN ('Discus Throw, 'Triple Jump')
)
SELECT
  Year, Event, Champion, LAG(Champion) OVER (ORDER BY Event ASC, Year ASC) AS Last_Champion
FROM Discus_Gold
ORDER BY Event ASC, Year ASC;
```

![](./chap1_3_wrongLastChampion.png)

```
<!-- Correct version can leverage the help of partition by) -->
WITH Discus_Gold AS (
  SELECT
    Year, Event, Country AS Champion
  FROM Summer_Medals
  WHERE
    Year IN (2004, 2008, 2012)
    AND Gender = 'Men'
    AND Medal = 'Gold'
    AND Event IN ('Discus Throw, 'Triple Jump')
)
SELECT
  Year, Event, Champion,
  LAG(Champion) OVER
    (PARTITION BY Event ORDER BY Event ASC, Year ASC) AS Last_Champion
FROM Discus_Gold
ORDER BY Event ASC, Year ASC;

```

![](./chap1_3_correct.png)

```
<!-- A more complex problem that let you partition by two columns (multiple columns) at the same time-->
WITH Country_Gold AS (
  SELECT
    DISTINCT Year, Country, Event
  FROM Summer_Medals
  WHERE
    Year IN (2008, 2012)
    AND Country IN ('CHN', 'JPN')
    AND Gender = 'Women' AND Medal = 'Gold'
)
SELECT Year, Country, Event,
  ROW_NUMBER() OVER (PARTITION BY Year, Country)
FROM Country_Gold;

<!-- you can see that the Row_number is starting for each partition -->
```

![chinese, japan](./chap1_3.png)

- **excersice: Practices**
  Reigning champions by gender
  You've already fetched the previous year's champion for one event. However, if you have multiple events, genders, or other metrics as columns, you'll need to split your table into partitions to avoid having a champion from one event or gender appear as the previous champion of another event or gender.

- instructions:
  Return the previous champions of each year's event by gender.

- MY CODES

```
WITH Tennis_Gold AS (
  SELECT DISTINCT
    Gender, Year, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Event = 'Javelin Throw' AND
    Medal = 'Gold')

SELECT
  Gender, Year,
  Country AS Champion,
  -- Fetch the previous year's champion by gender
  LAG(Country) OVER (PARTITION BY Gender
            ORDER BY Year ASC) AS Last_Champion
FROM Tennis_Gold
ORDER BY Gender ASC, Year ASC;
```

Return the previous champions of each year's events by gender and event.

```
WITH Athletics_Gold AS (
  SELECT DISTINCT
    Gender, Year, Event, Country
  FROM Summer_Medals
  WHERE
    Year >= 2000 AND
    Discipline = 'Athletics' AND
    Event IN ('100M', '10000M') AND
    Medal = 'Gold')

SELECT
  Gender, Year, Event,
  Country AS Champion,
  -- Fetch the previous year's champion by gender and event
  LAG(Country) OVER (PARTITION BY Gender, Event ORDER BY Year ASC) AS Last_Champion
FROM Athletics_Gold
ORDER BY Event ASC, Gender ASC, Year ASC;

```
Good job! You can partition by more than one column in case your groups are spread over several columns.