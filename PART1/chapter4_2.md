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

### 참조 무결성

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