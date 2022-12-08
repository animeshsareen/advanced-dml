# advanced-dml
#subquery-practice
## A collection of SQL scripts using advanced DML syntax

Run [this DDL script](https://github.com/animeshsareen/registration-sql/files/10147123/ddl.script.txt) to practice + test your own queries. You can see how we arrived at that relational database design over in [this](https://github.com/animeshsareen/registration-database) repo.

- Don't forget that my scripts were written in Oracle, so they may not be the same as yours.

In that case, check out [this website](http://www.sqlines.com/online) to convert between any two query languages.
___
Run [this](https://github.com/animeshsareen/advanced-dml/blob/main/ddl-script.sql) Oracle SQL script to initialize the student registration database!

*Beware of the first PL/SQL block -* **it drops all user-objects.**

Head to the [practice](https://github.com/animeshsareen/advanced-dml/tree/main/practice) folder to check out various DML topics + my practice scripts...

Or keep reading!
___
### Introducing SELECT, JOIN, and Subqueries

``` SQL 
--Starting with some basic SELECT queries
-- Read the syntax and predict what the output might be!

-- working with functions in SELECT
SELECT
    COUNT(*)  AS count_of_student,
    MIN(cgpa) AS min_student_cgpa,
    MAX(cgpa) AS max_student_cgpa
FROM
    student;
    
--using JOINs with aliases
SELECT
    t.last_name,
    c.course_code,
    COUNT(sc.section_id) AS number_of_classes
FROM
         section sc
    JOIN teacher t ON t.uteid = sc.instructor_uteid
    JOIN course  c ON sc.course_id = c.course_id
WHERE
        semester_code = 'FA22'
    AND substr(c.course_code, 1, 3) IN ( 'ACC', 'MIS' )
GROUP BY
    t.last_name,
    c.course_code
ORDER BY
    1,
    2;

/* 
This is a decently advanced query, since it uses aggregate functions (AVG)
To execute it, we did a multi-join and used GROUP BY to ensure SQL knew what to aggregate by
*/

SELECT
    m.major_code,
    s.classification,
    round(AVG(s.cgpa),
          2) AS average_cgpa
FROM
         student s
    JOIN major_student_linking msl ON s.uteid = msl.uteid
    JOIN major                 m ON msl.major_id = m.major_id
GROUP BY
    m.major_code,
    s.classification
ORDER BY
    AVG(s.cgpa) DESC;



/* 
The rest of these are relatively advanced SELECT queries, 
but they're great as reference points to keep your DML skills sharp
*/

SELECT
    s.primary_college_code AS "College Code",
    first_name
    || ' '
    || last_name           AS "Name",
    COUNT(e.uteid)         AS number_of_course
FROM
         student s
    JOIN enrollment e ON s.uteid = e.uteid
    JOIN section    sc ON e.section_id = sc.section_id
WHERE
    sc.section_day IN ( 'M', 'W', 'MW' )
    AND sc.semester_code = 'FA22'
GROUP BY
    s.first_name
    || ' '
    || s.last_name,
    s.primary_college_code
ORDER BY
    1,
    2;

 SELECT
            s.primary_college_code AS "College Code",
            first_name
            || ' '
            || last_name           AS "Name",
            COUNT(e.uteid)         AS number_of_course
        FROM
                 student s
            JOIN enrollment e ON s.uteid = e.uteid
            JOIN section    sc ON e.section_id = sc.section_id
        WHERE
            sc.section_day IN ( 'M', 'W', 'MW' )
            AND sc.semester_code = 'FA22'
        GROUP BY
            s.first_name
            || ' '
            || s.last_name,
            s.primary_college_code
            HAVING COUNT(e.uteid)>1
        ORDER BY
            1 ASC,
            2 ASC;

--Some ROLLUP practice
SELECT
    a.city            AS home_city,
    a.zip_postal_code AS home_zip_code,
    COUNT(sal.uteid)    AS number_of_students
FROM
         student_address_linking sal
    JOIN address a ON sal.address_id = a.address_id
    JOIN student s ON sal.uteid = s.uteid
WHERE
    city NOT IN ( 'New Delhi', 'Beijing' )
    AND address_type_code = 'H'
GROUP BY
    rollup(city,
           zip_postal_code)
ORDER BY
    1;

/* 

ROLLUP vs CUBE: what's the difference?

CUBE returns subtotals for all column permutations, while rollup 
only computes hierarchical subtotals as we "rollup" into a grand sum (ex. city -> zip); 

CUBE is often useful in business contexts (ex. analyzing global iPhone sales) 
since it'll show crosstabular subtotals across dimensions 
—  i.e. city, country, model, AND city + country, city + model, country + model... so on. 

*/


SELECT
    m.major_code,
    COUNT(msl.uteid) AS number_students
FROM
         student s
    JOIN major_student_linking msl ON s.uteid = msl.uteid
    JOIN major                 m ON msl.major_id = m.major_id
WHERE
    s.international_flag = 'N'
GROUP BY
    m.major_code
HAVING
    COUNT(msl.uteid) > 1
ORDER BY
    2 DESC;


--practicing with NOT operator
SELECT
    s.uteid
FROM
    student s
WHERE
    s.uteid NOT IN (
        SELECT DISTINCT
            msl.uteid
        FROM
            major_student_linking msl
    );

SELECT DISTINCT
    s.uteid,
    m.uteid,
    date_declared
FROM
    student               s
    LEFT JOIN major_student_linking m ON s.uteid = m.uteid
WHERE
    m.uteid IS NULL;
    
SELECT
    s.first_name,
    s.last_name,
    s.email,
    s.cgpa                                      AS cgpa,
    round((sysdate - s.date_of_birth) / 365, 0) AS age
FROM
    student s
WHERE
    s.cgpa > (
        SELECT
            AVG(cgpa)
        FROM
            student
    )
ORDER BY
    5 ASC,
    4 DESC;

SELECT
    iv.uteid,
    iv.number_of_sections,
    iv.total_minutes,
    round(total_minutes / number_of_sections, 1) AS avg_minutes_per_section
FROM
    (
        SELECT
            e.uteid,
            COUNT(e.section_id)    AS number_of_sections,
            SUM(sc.length_minutes) AS total_minutes
        FROM
            enrollment e
            LEFT JOIN section    sc ON e.section_id = sc.section_id
        GROUP BY
            uteid
    ) iv
ORDER BY
    avg_minutes_per_section;
    
SELECT
    uteid,
    first_name,
    last_name,
    primary_dept,
    office_phone
FROM
    teacher
WHERE
        uteid IN (
        SELECT DISTINCT
            instructor_uteid
        FROM
            section
        WHERE
            section_day = ANY ( 'M', 'W', 'MW', 'MWF' )
    )
    OR primary_dept = 'COE'
ORDER BY
    4,
    2;

SELECT
    s.first_name,
    s.last_name,
    s.email,
    s.classification,
    s.cgpa,
    iv.total_section
FROM
         student s
    JOIN (
        SELECT
            uteid,
            COUNT(uteid) AS total_section
        FROM
            enrollment
        GROUP BY
            uteid
    ) iv ON s.uteid = iv.uteid
WHERE
        classification = 4
    AND cgpa < 2.5
ORDER BY
    5,
    6;
```
___
### Advanced functions and data types
``` SQL

-- Manipulating query output using TRIM, to_char, and other functions
SELECT
    sysdate,
    rtrim(to_char(sysdate, 'DAY'))
    || ', '
    || ltrim(to_char(sysdate, 'MONTH'))    AS day_month,
    to_char(sysdate, 'MM/DD/YY - ')
    || 'hour:'
    || to_char(sysdate, 'HH')              AS date_with_hours,
    round(TO_DATE('31-DEC-22') - sysdate,
          0)                               AS days_til_end_of_year,
    lower(to_char(sysdate, 'mon dy yyyy')) AS lowercase,
    to_char(100000, '$999,999')            AS my_ideal_starting_salary
FROM
    dual;

-- using NVL2 for boolean logic and column parsing
SELECT
    s.uteid,
    nvl2(msl.major_id, first_name
                       || ' '
                       || last_name
                       || ' '
                       || 'has declared a major.', first_name
                                                   || ' '
                                                   || last_name
                                                   || ' '
                                                   || 'has not declared a major.') AS "Student Information",
    nvl2(date_declared,
         'Date declared:'
         || ' '
         || to_char(date_declared, 'Mon-dd-yyyy'),
         'Date declared:')          AS "Date Declared"
FROM
    student               s
    FULL OUTER JOIN major_student_linking msl ON s.uteid = msl.uteid
ORDER BY
    s.uteid ASC;
    
    -- Using DML to asemble dynamic query outputs
SELECT DISTINCT
    upper(substr(first_name, 1, 1))
    || '. '
    || last_name AS teacher_name,
    unique_number,
    seat_limit
FROM
         teacher t
    JOIN section s ON t.uteid = s.instructor_uteid
WHERE
        section_mode = 'F'
    AND seat_limit > 40
order by 1;

--Unique constraint + DML activity
SELECT
    uteid,
    first_name
    || ' '
    || last_name AS "Name",
    international_flag,
    CASE
        WHEN international_flag = 'Y' THEN
            to_char(46498, '$99,999')
        ELSE
            to_char(13570, '$99,999')
            || ' to '
            || TRIM(to_char(46498, '$99,999'))
    END          AS "Tuition"
FROM
    student
ORDER BY
    3 DESC,
    1 ASC;

--5.

SELECT
    s.uteid            AS "UT ID",
    major_id,
    length(first_name) AS first_name_length,
    round(TO_NUMBER(sysdate - date_declared),
          0)           AS days_since_major_declared
FROM
         student s
    INNER JOIN major_student_linking msl ON s.uteid = msl.uteid
WHERE
    classification IN ( '3', '4' )
    AND TO_NUMBER(sysdate - date_declared) > 50
ORDER BY
    4 ASC;
    
--More string manipulation, but this time with instr()
--It's  similar to substr () but with a few strategic differences

SELECT
    uteid,
    last_name,
    email,
    substr(email,
           1,
           instr(email, '@') - 1) AS emailname,
    substr(email,
           instr(email, '@') + 1,
           length(email))         AS emaildomain
FROM
    student
ORDER BY
    5 ASC;
    

--Manipulating long strings with substr()
SELECT
    first_name
    || ' '
    || last_name   AS student_name,
    major_code,
    date_declared,
    substr(phone, 1, 3)
    || '-***-****' AS redacted_phone_number
FROM
         student s
    JOIN major_student_linking msl ON s.uteid = msl.uteid
    JOIN major                 m ON msl.major_id = m.major_id
WHERE
    date_declared > '01-Jan-22'
ORDER BY
    2,
    3;


--Practice with CASE WHEN
SELECT
    CASE
        WHEN cgpa >= 3.9 THEN
            '1st Class Honor'
        WHEN cgpa >= 3.7
             AND cgpa < 3.9 THEN
            '2nd Class Honor'
        WHEN cgpa >= 3.5
             AND cgpa < 3.7 THEN
            '3rd Class Honor'
        ELSE
            'On track to graduate'
    END AS "Honor Level",
    first_name,
    last_name,
    email
FROM
    student
ORDER BY
    1,
    3;


SELECT
    first_name,
    last_name,
    classification,
    email,
    cgpa,
    DENSE_RANK()
    OVER(
        ORDER BY
            cgpa DESC
    ) AS student_rank
FROM
    student
WHERE
    classification = '4'
;

--This final query was the most challenging by far.

--Updated the previous query by making it in-line join subquery - i.e. selecting from the query itself.  
--Then, added in a row filter (ROWNUM) to the outer query to only pull rows <=5 

-- Also utilized DENSERANK

SELECT
    *
FROM
    (
    SELECT
        first_name,
        last_name,
        classification,
        email,
        cgpa,
        DENSE_RANK()
        OVER(
            ORDER BY
                cgpa DESC
        ) AS student_rank
    FROM
        student
    WHERE
        classification = '4'
        )
where rownum <=5;

SELECT
    *
FROM
    (
    SELECT
        first_name,
        last_name,
        classification,
        email,
        cgpa,
        DENSE_RANK()
        OVER(
            ORDER BY
                cgpa DESC
        ) AS student_rank
    FROM
        student
    WHERE
        classification = '4'
        )
where student_rank <= 3;```


