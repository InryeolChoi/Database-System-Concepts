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