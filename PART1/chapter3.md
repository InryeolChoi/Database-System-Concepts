# 3. SQL이란?

> SQL(Structured Query Language)은 관계형 데이터베이스 관리 시스템(RDBMS)에서 데이터를 관리하고 조작하기 위해 설계된 프로그래밍 언어.
> 

## SQL의 역사

- 1974년에 IBM에서 처음으로 개발.
- 계속 새로운 버전이 나오고 있으며, 여기서는 SQL2 버전을 기준으로 설명

## SQL의 특징

- 비절차적 언어. 사용자가 원하는 것만 명시
- 자연어에 가까운 구문을 사용해서 질의를 표현할 수 있음
- 두 가지 인터페이스 : 대화식(interactive) SQL & 내포된(embedded) SQL

## SQL의 구성요소

### DDL (데이터 정의 언어)

- 데이터베이스의 구조를 정의하거나 변경하는 데 사용
- 테이블, 뷰, 인덱스, 스키마 등의 데이터베이스 객체를 생성, 수정, 삭제할 때 사용
- 자동으로 **COMMIT**되며, 롤백 불가능.
- CREATE, ALTER, DROP, TRUNCATE

### DML (데이터 조작 언어)

- 데이터베이스에 저장된 데이터를 조회하거나 조작할 때 사용
- 명령어 실행 후 **COMMIT** 또는 **ROLLBACK**으로 트랜잭션을 제어할 수 있음.
- INSERT, SELECT, UPDATE, DELETE

### DCL (데이터 제어 언어)

- 데이터베이스 사용자와 권한을 관리하는 데 사용.
- 데이터베이스 보안과 접근 제어를 담당.
- GRANT, REVOKE 가 있음.
- 명령어 실행 후 자동 **COMMIT**.

# SQL 데이터 정의

## 기본 타입

```sql
/* 문자열 */
CHAR(n) -- 고정길이 문자
VARCHAR(n) -- 가변길이 문자

/* 숫자 */
int -- 정수
float(n) -- n개의 실수
bit (n) -- n개의 비트열

/* 기타 */
DATE -- 날짜형
BLOB -- 멀티미디어
```

## 기본 스키마 정의

```sql
-- department 테이블 (기존 테이블)
CREATE TABLE department (
    dept_name VARCHAR(20),
    building VARCHAR(15),
    budget NUMERIC(12, 2),
    PRIMARY KEY (dept_name)
);

-- instructor 테이블 (확장)
CREATE TABLE instructor (
    id INT,
    name VARCHAR(50) NOT NULL,      -- NOT NULL: name은 반드시 입력해야 함
    salary NUMERIC(10, 2),
    dept_name VARCHAR(20),          -- department의 dept_name을 참조
    PRIMARY KEY (id),               -- 주 키: 고유 식별자
    FOREIGN KEY (dept_name)         -- 외래 키: department 테이블의 dept_name 참조
        REFERENCES department(dept_name)
        ON DELETE CASCADE           -- 연관된 부서가 삭제되면 강사도 삭제
);
```

- 주 키 (primary key) : Null이 아니거나 유일해야 한다.
- 외래 키 (foreign key) : 어떤 테이블(릴레이션)의 어떤 속성을 따르는지 명시해야 한다.
- not null : 해당 속성의 Null 값을 허용하지 않는다.

릴레이션을 다루는 기본적인 파트들을 좀 더 소개하자면, 

- 맨 끝에는 세미콜론(`;`)을 작성한다.
- SQL은 무결성 제약 조건을 위반하는 모든 데이터베이스의 갱신을 막는다.
- 새로운 튜플을 삽입, 갱신, 삭제하는 명령문 : `insert, update, delete`
- 릴레이션 자체를 삭제하는 명령문 : `drop`

```sql
delete from univ; -- univ의 모든 튜플을 삭제한다.
drop table univ; -- univ의 모든 튜플과 스키마를 삭제한다.
```

- 릴레이션의 속성을 추가 / 삭제 : `alter table`

```sql
ALTER TABLE instructor ADD email VARCHAR(100);
ALTER TABLE instructor MODIFY name VARCHAR(100) NOT NULL;
ALTER TABLE instructor DROP COLUMN email;
```

# SQL 질의

## 단일 릴레이션

- update, delete, insert는 단일 릴레이션

## 복수 릴레이션

- select, from, where는 복수 릴레이션
- where은 조건을 의미
    - and, or, not을 붙여서 조건을 확장할 수 있다.
    - 
- select와 from은 필수

## 부가 연산

1. 재 명명하기 : `as`를 이용해 릴레이션의 별명을 지을 수 있다.
2. 문자열은 작은 따옴표를 사용한다.
3. 패턴 확인 : `like`를 사용해 패턴 일치를 수행할 수 있다
4. 모두 선택 : asterisk(`*`)를 사용해 모두 선택이 가능하다
5. 순서 : `order by` 를 사용해 오름차순 / 내림차순을 선택할 수 있다.
6. 집합연산은 `union, intersect, except`를 사용한다. 각각 합집합, 교집합, 차집합을 의미

## 집계 연산

1. 중복을 제거하는 경우, `distinct`를 사용한다.
2. 그룹단위로 집계를 하는 경우, `group by`를 사용한다.
3. 그룹단위로 집계를 하는 경우, `having` 을 이용해 조건을 걸 수 있다

## 중첩 하위 질의

하위 질의 : 다른 질의 안에 중첩된 `select, from, where` 

```sql
SELECT name, salary
FROM instructor
WHERE dept_name IN (
    SELECT dept_name
    FROM department
    WHERE budget >= 50000
);
```

`in, not in` 키워드를 사용해서 포함관계를 표현할 수 있음.

### 자주 쓰는 하위질의

- **집합끼리 비교할 때는** `SOME` 키워드를 사용.
- `exists, not exists`를 사용해 하위 질의(subquery)의 결과가 하나 이상의 행을 반환하는지 확인.
- `unique`를 이용해서 중복된 튜플이 있는지 없는지를 체크.
- 쿼리에서 **임시적인 결과 집합**을 생성하여 복잡한 쿼리를 간결하게 작성할 때 `with` 사용

## 데이터베이스의 변경

- 삭제 : `delete`
- 삽입 : `insert into`
- 갱신 : `update`