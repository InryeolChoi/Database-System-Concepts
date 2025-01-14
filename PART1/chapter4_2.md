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