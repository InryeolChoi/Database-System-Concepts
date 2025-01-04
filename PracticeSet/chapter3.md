# 3단원 연습문제
문제에서는 대학교 스키마가 자주 언급된다. `db-book.com`에 들어가면 대학교 스키마를 실제로 따올 수 있으며, 이를 활용해서 문제를 풀 수 있다.

## 1번

> 대학교 스키마를 활용해 다음 질의를 SQL로 작성하라
> 

```sql
-- a) 컴퓨터 과학과의 3학점짜리 과목의 제목을 찾아라
SELECT title 
FROM course 
WHERE dept_name = 'Comp. Sci.' AND credits = 3;

-- b) Einstein 교수가 가르치는 과목을 듣는 모든 학생의 아이디를 찾아라
SELECT DISTINCT id 
FROM takes
WHERE (course_id, sec_id, semester, year) IN 
(
    SELECT course_id, sec_id, semester, year 
    FROM teaches INNER JOIN instructor 
        ON teaches.id = instructor.id
    WHERE instructor.name = 'Einstein'
);

-- c) 교수의 가장 높은 급여를 구하라
SELECT MAX(salary) 
FROM instructor

-- d) 가장 높은 급여를 받는 모든 교수를 찾아라
SELECT id, name
FROM instructor
WHERE salary = (SELECT MAX(salary) FROM instructor)

-- e) 2017년 가을에 개설된 각 분반에 걸친 최대 등록자 수를 구하라
SELECT course_id, sec_id, (
    SELECT COUNT(id)
    FROM takes
    WHERE takes.year = section.year
        AND takes.semester = section.semester
        AND takes.course_id = section.course_id 
        AND takes.sec_id = section.sec_id
) AS enrollment 
FROM section 
WHERE semester = 'Fall' AND year = 2017

-- f) 2017년 가을의 모든 분반에 걸친 최대 등록자 수를 구하라
WITH enrollment_in_fall_2017(course_id, sec_id, enrollment) AS (
    SELECT course_id, sec_id, COUNT(id)
    FROM takes
    WHERE semester = 'Fall' AND year = 2017
    GROUP BY course_id, sec_id
) 
SELECT CASE 
        WHEN MAX(enrollment) IS NOT NULL THEN MAX(enrollment)
        ELSE 0
       END
FROM enrollment_in_fall_2017;

-- g) 2017년 가을의 최대 등록자 수를 찾는 분반을 구하라.
WITH enrollment_in_fall_2017(course_id, sec_id, enrollment) AS (
    SELECT course_id, sec_id, COUNT(id) 
    FROM takes
    WHERE semester = 'Fall' AND year = 2017
    GROUP BY course_id, sec_id
) 
SELECT course_id, sec_id
FROM enrollment_in_fall_2017
WHERE enrollment = (SELECT MAX(enrollment) FROM enrollment_in_fall_2017);
```

## 2번

> takes 릴레이션의 문자로 된 등급을 숫자로 바꾼 grade_points(grade, points)라는 릴레이션이 있다고 가정하자. 예를 들어, ‘A’는 4점, ‘A-’는 3.7점, ‘B+’은 3.3점이다. 위 릴레이션과 대학교 스키마가 있을 때, 다음 질의를 SQL로 작성해보자.
> 

```sql
-- a. ID가

-- b. 

-- c.

```