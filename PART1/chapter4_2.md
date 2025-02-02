# 4. 중급 SQL (2)

# 무결성 제약조건

- 무결성이란? **완전성과 일관성을 유지하는 것**을 의미
- 무결성 제약조건 : 데이터베이스에서 데이터의 일관성과 정확성을 유지하기 위해 설정된 규칙이나 조건.
- 이를 통해 데이터가 유효하고 신뢰할 수 있는 상태를 유지하도록 보장함.

무결성 제약 조건은 보통 데이터베이스 스키마 설계 과정의 일부로 인식됨.

그래서 `create table` 명령어의 일부로 정의하나, 이미 만들어진 테이블에 추가하는 경우도 있다.

```sql
alter table (테이블 이름) add constraint (명령어)
```

## 제약 조건의 종류

- 주로 테이블 1개 (=단일 릴레이션)을 만들 때 붙여서 사용한다.
- 즉, `create table` 과 같이 사용한다는 의미이다.

### Not Null

- null이 아닌 경우를 막아주는 케이스
- 도메인 제약 조건의 한 예시

```sql
name varchar(20) not null
budget numeric(12.2) not null
```

### Unique

- 해당 열(column)에 **중복된 값이 들어오는 것을 허용하지 않는 제약 조건**.
- 예: 이메일 주소는 각 사용자마다 고유해야 할 경우 UNIQUE 제약을 사용.

```sql
CREATE TABLE Users (
    UserID INT,
    Email VARCHAR(100) UNIQUE
);
```

### Check

- 특정 조건(predicate)을 만족하는 값만 허용하도록 설정하는 제약 조건.
- 예: 나이는 0 이상이어야 한다는 조건을 CHECK로 설정할 수 있음.

```sql
CREATE TABLE Users (
    UserID INT,
    Age INT CHECK (Age >= 0)
);
```

## 참조 무결성

- 외래 키(foreign key)를 통해 서로 연관된 테이블 간의 데이터 일관성을 유지하기 위한 제약 조건.
- 데이터베이스의 신뢰성을 높이고, 잘못된 데이터나 논리적인 오류가 발생하지 않도록 보장.
- 예시 : course 테이블의 정의

```sql
create table course
	(course_id		varchar(8), 
	 title			varchar(50), 
	 dept_name		varchar(20),
	 credits		numeric(2,0) check (credits > 0),
	 primary key (course_id),
	 foreign key (dept_name) references department (dept_name)
		on delete set null
	);
```

여기서 외래 키 `dept_name`은 `department` 테이블의 제약조건을 의미.

이게 없으면 존재하지 않는 학과의 이름을 입력하는 것이 가능해진다.

### 참조 무결성의 특징

기본적으로 외래 키는 참조되는 테이블의 주 키 속성을 참고함.

또한 외래 키가 참조하는 속성(열)은 반드시 다음 조건 중 하나를 만족해야 함.

1. 기본 키(primary key)**로 선언된 열.
2. 유일 키(unique key)**로 선언된 열.

즉, 참조 대상이 되는 속성은 **중복될 수 없는 값**이어야 함.

추가적으로, 외래 키와 참조 열 간의 속성(열)에 대해 **데이터 타입**과 **열의 개수**가 동일해야 함.

- ex) 외래 키가 정수형(INT)이라면 참조하는 열도 정수형이어야 합니다.
- 다중 열 외래 키(multi-column foreign key)일 경우, 열의 개수와 순서도 일치해야 함.

만약 외래 키 제약 조건이 깨지면, 일반적으로 데이터베이스는 해당 작업을 거부하거나 트랜잭션을 롤백(rollback) 한다. 하지만 SQL에서는 이를 처리하기 위해 다양한 동작을 설정할 수 있다.

```sql
create table course (
    dept_name varchar(20) references department(dept_name)
    on delete cascade
    on update cascade
);
```

**ON DELETE CASCADE**

- 참조된 테이블(department)의 행이 삭제되면: 참조하는 테이블(course)의 관련 행도 **자동으로 삭제됨.**
- 위의 예시를 보면, `department`에서 `dept_name = 'Math'`가 삭제되면, `course` 테이블에서 `dept_name = 'Math'`를 참조하는 모든 행도 삭제됨.

**ON UPDATE CASCADE**

- 참조된 테이블(department)의 dept_name 값이 변경되면: 참조하는 테이블(course)의 관련 값도 **자동으로 업데이트됨**
- 위의 예시를 보면, `department`에서 `dept_name = 'CS'`가 `dept_name = 'CompSci'`로 변경되면, `course` 테이블에서도 해당 값을 업데이트됨

### **외래 키 연결 체인과 전파**

- 외래 키 관계가 여러 테이블에 걸쳐 연결되어 있을 때, 삭제 또는 업데이트 작업이 체인 전체에 영향을 미칠 수 있음.
- 예를 들어, 테이블들이 다음과 같이 연결되어 있다고 가정함:

```sql
department → course → enrollment
-- department가 최상위 테이블이고, course와 enrollment이 각각 이를 참조함.
```

**삭제 또는 업데이트의 전파 동작**

만약 department에서 특정 데이터를 삭제하면, 다음과 같은 동작이 전파됨:

1. course에서 해당 데이터를 참조하는 행이 삭제됨.
2. 이후, enrollment에서 course를 참조하는 행도 삭제됨.

이와 같이 삭제 또는 업데이트 작업이 테이블 간의 연쇄적(cascade)으로 작동함.

**문제점**

- 전파 작업 중 하나라도 오류가 발생하면, 전체 트랜잭션이 실패하고 모든 변경 사항이 **롤백됨**.
- 예를 들어 department에서 데이터를 삭제했는데, course 또는 enrollment에서 삭제 작업이 실패하면 전체 작업이 취소됨.

### **NULL 값과 외래 키**

- 외래 키는 NULL 값을 허용할 수 있음 (단, 속성이 **NOT NULL**로 선언되지 않은 경우임).
- NULL 값이 허용되면, 참조되는 값이 없어도 제약 조건을 만족한 것으로 간주됨.

**NULL 값의 동작**

- 외래 키 값 중 하나라도 NULL이면, 데이터베이스는 이를 자동으로 제약 조건을 만족한 것으로 처리함.
- 예를 들어, course 테이블의 dept_name 열이 NULL이면, 어떤 department의 값을 참조하지 않아도 무결성이 깨지지 않음.
- 그러나, NULL 값을 허용하면, 데이터 무결성이 약화될 가능성이 있음.
- 또한 참조 대상이 명확하지 않기 때문에, 데이터 간의 관계가 끊길 수 있음.
- 따라서 SQL에서는 NULL 값이 허용될 때 동작을 세부적으로 조정할 수 있는 옵션도 제공됨.

## 제약 조건 명명

무결성 제약 조건에 이름을 할당할 수 있다.

예를 들어, salary 열에 최소값 조건을 설정하고 이를 **minsalary**라는 이름으로 지정하려면 다음과 같이 작성

```sql
salary numeric(8,2),
constraint minsalary check (salary > 29000),
-- minsalary는 제약 조건 이름임.
-- check (salary > 29000)은 salary가 29,000보다 커야 한다는 조건임.
```

이렇게 지정된 이름을 사용하여 해당 제약 조건을 삭제할 수도 있다.

예를 들어, minsalary라는 이름의 제약 조건을 삭제하려면 다음과 같이 작성함:

```sql
alter table instructor drop constraint minsalary;
```

**이름이 없는 경우**

- 만약 제약 조건에 이름을 지정하지 않았다면, 시스템에서 자동으로 생성한 이름을 식별해야 함.
- 특정 DBMS(예: Oracle)에서는 시스템에서 생성한 이름을 확인하기 위해 `user_constraints` 테이블과 같은 시스템 테이블을 참조해야 함.

## 트랜잭션의 수행

> 트랜잭션 수행 중 무결성 제약조건이 위반된다면?
> 
- 하나의 트랜잭션은 여러 단계로 이루어질 수 있으며, 중간 단계에서 무결성 제약(integrity constraint)이 
**일시적으로 위반**될 수 있음.
- 하지만 트랜잭션의 마지막 단계에서 무결성 문제가 해결될 수 있음.

### **해결 방안 1 : 연기 가능**

테이블 person이 있고, 기본 키는 name이며, spouse라는 외래 키 속성이 있음.

- **제약 조건**: spouse 속성은 반드시 person 테이블에 존재하는 이름이어야 함.

John과 Mary가 결혼했다고 입력 시 John의 spouse를 Mary로, Mary의 spouse를 John으로 삽입해야 함.

- **먼저 John의 레코드를 추가하면, Mary가 아직 테이블에 없으므로 외래 키 제약 조건 위반 발생.**
- 이후 Mary의 레코드를 추가하면 제약 조건이 만족됨.

이렇게 나중에 제약 조건을 만족하는 것을  ”연**기 가능(deferrable)”** 이라고 함.

연기 가능한 제약 조건은 **이름**을 반드시 가져야 하며, 이름이 없는 경우 사용할 수 없음.

- 일부 데이터베이스는 이러한 연기 동작을 지원하지 않을 수 있음.

### 해결 방안 2 : **NULL 사용**

spouse 속성을 **NULL**로 설정한 후 데이터를 추가한 뒤 업데이트하는 방법도 있음.

- 예: 먼저 John과 Mary의 레코드를 각각 추가한 뒤, 나중에 spouse 속성을 업데이트함.
- 단, 속성이 **NULL**로 설정될 수 있어야 하며, 프로그래밍 작업이 더 복잡해질 수 있음.

## 복잡한 Check 조건과 주장

- SQL 표준에서는 CHECK 절(predicate)에 하위 쿼리를 포함하는 복잡한 조건을 정의할 수 있음.
- 예를 들어, time_slot 테이블에 존재하는 time_slot_id만 section 테이블에 추가되도록 강제하려면:

```sql
check (time_slot_id in (select time_slot_id from time_slot))
```

이 코드는 time_slot 테이블에 존재하지 않는 time_slot_id가 section 테이블에 추가되는 것을 방지함.

단, 이 조건은 section 데이터뿐 아니라, 참조 대상인 time_slot 데이터가 삭제되거나 수정될 때도 검증되어야 함.

**CHECK 조건의 한계**

- 데이터 무결성을 보장하는 데 유용하지만, 성능 비용이 클 수 있음.
- 하위 쿼리가 포함된 CHECK 조건은 단순 데이터 삽입뿐 아니라, 참조 테이블에 변경이 생길 때마다 평가되어야 하기 때문임.
- 일부 데이터베이스 시스템에서는 하위 쿼리를 포함하는 CHECK 조건을 지원하지 않음.

**Assertions (어설션)**

- **Assertion**은 데이터베이스가 항상 만족해야 하는 조건을 표현하는 방식임.
- CHECK보다 복잡한 조건을 정의할 수 있음

```sql
create assertion <assertion-name> check <predicate>;
```

ex) 학생의 총 학점(tot_cred)이 이수한 과목의 학점 합계와 같은지 확인하는 Assertion:

```sql
create assertion credits_earned_constraint check (
    not exists (
        select ID
        from student
        where tot_cred <> (
            select coalesce(sum(credits), 0)
            from takes natural join course
            where student.ID = takes.ID
              and grade is not null and grade <> 'F'
        )
    )
);
```

Assertion은 다음 조건을 만족해야 함

- 정의될 때 조건을 테스트함.
- 조건이 유효하면 데이터베이스에서 조건을 위반하는 작업은 허용되지 않음.

**또한 다음 사항들을 주의해야 함**

- Assertion은 테스트와 유지 관리 비용이 큼.
- 복잡한 Assertion이 많을수록 데이터베이스 성능에 부정적인 영향을 미칠 수 있음.
- 대부분의 데이터베이스 시스템은 Assertion을 지원하지 않음.
    - 이 경우 트리거(trigger)를 사용하여 동일한 기능을 구현할 수 있음.

# SQL의 데이터 타입

> SQL이 추가로 지원하는 데이터 타입과 사용자 정의 타입을 알아보자.
> 

## 날짜와 시간

- `date, time` 그리고 둘을 합친 `timestamp`가 존재
- 이외에도 몇 가지 유용한 함수들이 제공.

```sql
CREATE TABLE datetime_example (
    id SERIAL PRIMARY KEY,
    event_date DATE,
    event_time TIME,
    event_timestamp TIMESTAMP
);

-- 현재 날짜를 `event_date`, 현재 시간을 `event_time`, 현재 타임스탬프를 `event_timestamp`에 삽입
INSERT INTO datetime_example (event_date, event_time, event_timestamp)
VALUES (current_date, current_time, localtimestamp);

-- 특정 날짜와 시간을 지정해서 삽입
INSERT INTO datetime_example (event_date, event_time, event_timestamp)
VALUES ('2025-01-17', '12:30:00', '2025-01-17 12:30:00');

-- 현재 날짜에 7일을 더한 결과
SELECT current_date + INTERVAL '7 days' AS one_week_later;

-- 현재 타임스탬프에서 2시간을 뺀 결과
SELECT localtimestamp - INTERVAL '2 hours' AS two_hours_ago;

-- 날짜 차이를 계산
SELECT event_date, current_date - event_date AS days_difference
FROM datetime_example;

-- 시간 차이를 계산
SELECT event_time, current_time - event_time AS time_difference
FROM datetime_example;
```

## 타입 변환 및 서식 함수

1. CAST 함수

**데이터의 타입을 변환할 때 사용, 어떤 타입으로 바꿀지 사용자 맘대로!**

다양한 데이터 타입 간의 변환을 지원하며, ANSI 표준 SQL 함수입니다.

```sql
CAST(expression AS target_data_type)
```

예시는 다음과 같다.

```sql
-- 숫자를 문자열로 변환
SELECT CAST(123 AS VARCHAR(10)) AS string_value;

-- 문자열을 숫자로 변환
SELECT CAST('456' AS INTEGER) AS integer_value;

-- 날짜를 문자열로 변환
SELECT CAST(GETDATE() AS VARCHAR(20)) AS date_as_string;
```

1. 형식 지정 함수

**숫자를 문자로 바꾸거나, 문자를 숫자로 바꾸는 함수**

```sql
// MySQL의 format() 함수
-- FORMAT(number, decimal_places[, locale])

SELECT FORMAT(12345.6789, 2);  -- "12,345.68"
SELECT FORMAT(12345.6789, 2, 'de_DE');  -- "12.345,68" (독일 형식)
```

```sql
// PostgreSQL의 to_char() 함수
-- TO_CHAR(expression, format)

SELECT TO_CHAR(12345.6789, '99999.99') AS formatted_number;  -- "12345.68"
SELECT TO_CHAR(CURRENT_DATE, 'YYYY-MM-DD') AS formatted_date;  -- "2025-01-19"
```

1. **COALESCE 함수**

null 값을 어떻게 출력할지 선택해야 할 때 쓰는 함수.

```sql
SELECT ID, COALESCE(salary, 0) as salary from instructor  
```

그러나 이 함수의 경우 모든 인자가 동일한 타입이어야 한다. 즉 salary의 null 값을 숫자 0으로는 표기할 수 있어도, 문자열 “N/A”로는 표기할 수 없다.

1. **decode 함수**

조건에 따라 값을 반환. IF-THEN-ELSE와 유사.

```sql
SELECT ID, decode (salary, null, "N/A", salary) as salary from instructor  
```

위에서 설명했듯, 타입의 문제에 있어서 COALSCE 함수보다 자유롭다.

## 기본값

릴레이션을 만들 때`(create table)`, 각 속성의 기본값을 지정해 줄 수 있다.

```sql
create table student (
	ID varchar (5),
	name varchar (20) not null,
	dept_name varchar (20),
	**tot_cred numeric(3, 0) default 0,**
	primary key(ID)
); 
```

## 대형 객체 타입

이미지, 동영상 등 비정형 데이터를 릴레이션에 넣을 때 쓰는 타입

```sql
CREATE TABLE media_content (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description CLOB, -- 대용량 텍스트 데이터를 저장
    media_file BLOB   -- 이미지, 비디오와 같은 바이너리 데이터를 저장
);
```

- LOB는 Large OBject를 의미한다.
- BLOB = Binary Large OBject
- CLOB = Character Large OBject

이러한 대형 객체는 다루는데 있어서 효율적인 방법이 필요하다.

- 대형객체는 그 크기가 크기 때문에, DB에서 내 컴퓨터의 메모리로 가져오면 메모리 부족 문제 또는 성능 저하를 초래할 수 있음.

따라서 Locator(위치자)의 개념을 도입한다.

- Locator는 대용량 객체의 **참조 포인터(위치 정보)** 역할을 합니다.
- SQL 쿼리를 통해 대용량 객체 전체가 아닌 **locator**만 가져옵니다.
- 운영체제에서 `read()` 시스템 콜을 이용하는 것과 비슷한 원리라고 생각하면 된다.

## 고유 타입

- 사용자가 직접 정의하는 사용자 정의 타입의 일종
- create type 절을 사용한다.

```sql
create type Dollars as numeric(12, 2) final;
create type Pounds as numeric(12, 2) final;
```

만약, Dollars와 Pounds를 이용해서 릴레이션을 만든다고 해보자.

```sql
CREATE table department (
	dept_name varchar (20),
	building varchar (15),
	budget Dollars
);
```

이때 budget의 값을 Pounds형의 새 변수에 할당하려고 하면 오류가 날 수밖에 없다.

또한 budget의 값에 숫자를 직접 더하는 것 또한 오류의 위험성이 존재한다.

만약 사용자가 만든 타입을 삭제하거나 변경하고 싶으면, `drop type`과 `alter type`을 쓰면 된다.

```sql
-- 값 'excited'를 'happy' 뒤에 추가
ALTER TYPE mood ADD VALUE 'excited' AFTER 'happy';

-- 'sad'를 'unhappy'로 이름 변경
ALTER TYPE mood RENAME VALUE 'sad' TO 'unhappy';
```

과거에는 사용자 정의 타입이 없었고, 도메인이라는 것이 존재.

```sql
create domain DDollars as numeric(12,2) not null;

create domain YearlySalary numeric(8,2)
constraint salary_value_test
check (value >= 29000.00);

create domain degree_level varchar(10)
constraint degree_level_test
check (value in ('Bachelors', 'Masters', 'Doctorate'));
```

- **도메인**은 기본 타입에 제약 조건을 추가하여 데이터 무결성을 보장하며, 테이블 속성 타입으로 활용 가능.
- **사용자 정의 타입**은 제약 조건을 정의할 수 없으나, 프로시저나 함수 등 SQL 확장에서 유용하게 사용.
- 도메인은 check 및 in 절을 활용하여 값의 범위와 조건을 지정 가능.

## 고유 키 값의 생성

**유일 키와 데이터 타입**

- 일부 기본 키(primary key)는 실제 현실 세계 데이터를 저장 (dept_name과 같은 속성).
- 일부는 시스템에서 식별을 위해 생성된 값 (ID와 같은 속성).
- 따라서 새로운 값을 생성할 때, 기존의 키 값과 중복되지 않도록 관리하는 것이 중요.

**수동 관리 방식의 한계**

- 새로운 ID 값을 생성하려면 기존의 모든 ID 값을 검사해야 하며, 이는 시스템 성능을 저하시킬 수 있음.
- 별도의 테이블을 두고 현재까지 사용된 가장 큰 ID 값을 기록한 후, 새로운 값이 필요할 때 이를 증가시키는 방식도 있음. 그러나 이러한 방식은 관리에 추가적인 복잡성을 야기함.

**자동 관리 방식**

현대 데이터베이스 시스템에서는 유일 키 값을 자동으로 관리할 수 있는 기능을 제공한다. 이는 데이터베이스 시스템과 버전에 따라 구문이 다를 수 있음.

**데이터베이스별 키 생성 방식**

```sql
// 예시 : postgreSQL**
-- serial 데이터 타입 사용.
-- 자동으로 고유 식별자를 생성.

ID serial
```

```sql
// MySQL
-- auto increment 사용.	

ID int auto_increment
```

**Sequence 객체를 활용한 고유 값 생성**

- 많은 데이터베이스는 create sequence 구문을 지원.
- Sequence는 특정 릴레이션과 독립된 카운터 객체로, 다음 값을 생성하여 여러 릴레이션에서 고유 식별자를 생성 가능.

```sql
create sequence id_sequence
  start with 1
  increment by 1;

select nextval('id_sequence');
```

## Create Table의 확장

SQL에서는 Create Table에 추가적인 기능을 부여하는 경우도 있다.

1. `Create Table like` 

기존 테이블과 동일한 스키마를 가진 테이블을 생성하려면 `CREATE TABLE LIKE` 구문을 쓴다.

```sql
-- temp_instructor는 instructor와 동일한 스키마를 가진 새 테이블로 생성됨.
create table temp_instructor like instructor
```

1. `create table … as` 

임시 테이블을 만들 때 사용. 주로 복잡한 쿼리 결과를 저장하기 위한 용도

```sql
CREATE TABLE t1 AS
(SELECT *
 FROM instructor
 WHERE dept_name = 'Music')
WITH DATA;
```

1. `with data`
- **WITH DATA**: 데이터를 포함하여 테이블 생성.
    - WITH DATA가 생략되면 구현에 따라 데이터 없이 테이블만 생성될 수도 있음.
    - 컬럼 이름과 데이터 유형은 기본적으로 쿼리 결과에서 자동으로 추론됨.
    - 명시적으로 컬럼 이름을 지정하려면 테이블 생성 시 정의 가능.
- **CREATE VIEW와의 차이점**
    - CREATE TABLE ... AS는 테이블의 내용을 생성 시 고정.
    - 반면 CREATE VIEW는 쿼리 결과를 동적으로 반영.

## 스키마, 카탈로그, 환경

초기 데이터베이스 시스템은 단일 구조를 가지고 있었음.

그러나 점점 시스템이 커지면서, 이름 충돌 방지를 위해 사용자 간 조정이 필요해짐

따라서 현대의 데이터베이스 시스템(DBMS)는 다음과 같은 구조를 가지게 됨.

**현대 데이터베이스의 3단계 계층 구조**

- **Catalog**(카탈로그): 최상위 계층.
- **Schema**(스키마): 카탈로그 내에 포함. SQL 객체(테이블, 뷰 등)가 저장됨.

(일부 DBMS에서는 “Catalog” 대신 “Database”라는 용어를 사용.)

- **Relations**(관계): 스키마 내에서 관리.

그렇다면, 계층구조를 사용한 릴레이션의 진짜 이름은?

```sql
catalog5.univ_schema.course

/*
카탈로그를 생략하면, 현재 연결의 기본 카탈로그가 사용됨.
예: 기본 카탈로그가 catalog5이면 univ_schema.course로 동일한 관계 식별 가능.

기본 스키마에 있는 관계라면, 스키마 이름도 생략 가능.
예: 기본 카탈로그가 catalog5, 기본 스키마가 univ_schema일 경우, 
단순히 course로 관계 식별 가능.
*/
```

 ****

**계층 구조의 특징과 이점**

- 사용자가 데이터베이스에 연결하려면 사용자 이름과 비밀번호로 인증 필요.
- 각 사용자에게는 기본 카탈로그와 스키마가 설정됨.
    - 사용자가 데이터베이스에 연결하면, 기본 카탈로그와 스키마가 연결 컨텍스트에 설정됨.
- 이는 운영체제의 홈 디렉터리 개념과 유사.

- 이러한 다중 계층구조는 서로 다른 애플리케이션이나 사용자가 이름 충돌 걱정 없이 독립적으로 작업할 수 있도록 만들어줌.
- 동일 데이터베이스 시스템에서 여러 버전(예: 프로덕션 및 테스트 버전)의 애플리케이션 운영 가능.

- **기본 카탈로그와 스키마, 그리고 사용자 식별자**(권한 식별자)는 기본적으로 각 데이터베이스 연결을 설정하는 SQL 환경의 일부임.
- 모든 SQL 명령문 (DDL, DML 포함)은 지정된 스키마 컨텍스트 내에서 동작.

**스키마와 카탈로그 생성/삭제**

- **스키마**: CREATE SCHEMA, DROP SCHEMA 명령어로 생성/삭제 가능.
    - 대부분의 DBMS에서는 사용자 계정 생성 시 자동으로 스키마도 생성.
    - 스키마 이름은 사용자 계정 이름과 동일하며, 기본 카탈로그에 생성.
- **카탈로그**: 생성 및 삭제는 SQL 표준이 아닌, DBMS 구현에 따라 달라짐.