# 5. 고급 SQL

# 프로그래밍 언어로 SQL

데이터베이스를 다루는 프로그래머는 SQL로 쿼리를 치는 것 뿐만 아니라, 다른 프로그래밍 언어로도 데이터베이스에 접근할 수 있어야 한다. 왜?

1. SQL로는 모든 질의를 다 표현할 수 없기 때문.
2. 비선언적 동작(ex. 보고서 출력, 사용자와의 의사소통, 질의 결과를 시각화)는 SQL로 표현할 수 없기 때문

## 접근 방식

**1. 내장 SQL (embedded SQL)**

SQL 문장이 프로그램 코드에 정적으로 포함되어 있는 형태.

- SQL 구문이 소스 코드에 하드코딩되어 있어, 실행 시 수정이 불가능하다.
- SQL이 컴파일 시점에 검증되므로 실행 속도가 빠르고 안정성이 높다.
- 주로 고정된 SQL 작업(예: 간단한 SELECT, INSERT, UPDATE, DELETE)에 사용된다.
- 단, SQL을 수정하려면 전체 프로그램을 다시 컴파일해야 함.

예시) 자바 스프링에서의 내장 SQL

```java
@Repository
public class UserRepository {
    private final JdbcTemplate jdbcTemplate;

		// 생성자
    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // 내장 SQL: 고정된 SQL 쿼리
    public User findUserById(int userId) {
        String sql = "SELECT * FROM users WHERE user_id = ?";
        return jdbcTemplate.queryForObject(
            sql, new BeanPropertyRowMapper<>(User.class), userId
        );
    }
}
```
**2. 동적 SQL (dynamic SQL)** 

- 실행 시점에 SQL 문장이 동적으로 생성되거나 실행되는 형태
- SQL을 실행 시점에 생성할 수 있어 유연성이 높다.
- 사용자 입력, 조건 등에 따라 SQL 문장이 변경될 수 있다.
- 주로 복잡한 조건이나 다양한 사용자 입력을 처리할 때 사용된다.
- 그러나 실행 시점에 오류가 발생할 가능성이 높고, 보안 문제와 성능 문제가 존재한다.

대표적인 동적 SQL의 예시는 다음이 있다.

- 자바와 DB를 연결하는 JDBC
- 파이썬의 Python Database API
- 여러가지 언어를 위한 ODBC

예시) 자바 스프링에서의 동적 SQL

```sql
@Mapper
public interface UserMapper {

    // 동적 SQL: 조건에 따라 SQL 쿼리가 동적으로 변경
    @Select("<script>" +
            "SELECT * FROM users " +
            "WHERE 1=1 " +
            "<if test='name != null'>AND name = #{name}</if> " +
            "<if test='email != null'>AND email = #{email}</if> " +
            "</script>")
    List<User> findUsers(@Param("name") String name, @Param("email") String email);
}
```

## 내장 SQL

> SQL 문장이 프로그램 코드에 정적으로 포함되어 있는 형태.
> 

SQL 질의를 내장한 언어를 호스트 언어라고 함. 

그런데 호스트 언어와 SQL은 처리 방식에서 차이가 있다.

- 호스트 언어 : 단일 변수 / 레코드 위주의 처리
- SQL : 데이터 레코드 위주의 처리

둘의 처리 단위가 맞지 않기 때문에 해결방법을 모색해야 했음.

따라서 반복문을 이용하고, 하나의 단일 변수 / 레코드를 가리키기 위해 **커서(cursor)**라는 것을 사용.

- **커서(cursor) : 한 번에 한 튜플을 가지고 오는 수단.**

```c
#include <stdio.h>      // 표준 입출력 헤더
#include <sqlca.h>      // SQLCA(SQL Communication Area) 헤더

EXEC SQL BEGIN DECLARE SECTION; // 커서 정의
char name[] = "박영권";
char title[10];
EXEC SQL END DECLARE SECTION;

EXEC SQL
    DECLARE title_cursor CURSOR FOR
    SELECT title FROM employee WHERE empname = :name;

int main() {
    // 데이터베이스에 연결 (실제 환경에서는 사용자/비밀번호 필요)
    EXEC SQL CONNECT TO database_name USER username IDENTIFIED BY password;

    // 커서 열기
    EXEC SQL OPEN title_cursor;

    // 데이터 가져오기
    EXEC SQL FETCH title_cursor INTO :title;

    // 결과 출력
    printf("직책: %s\n", title);

    // 커서 닫기
    EXEC SQL CLOSE title_cursor;

    // 데이터베이스 연결 해제
    EXEC SQL COMMIT WORK RELEASE;

    return 0;

```

갱신할 튜플들에 대해 커서를 정의할 때는 다음과 같이 작성한다.

```sql
FOR UPDATE OF title;
```

**SQLCA (SQL 통신 영역)**

- SQL과 C를 통신해 줄 수 있는 영역
- C 프로그램에 내포된 SQL문에 발생하는 에러들을 사용자에게 알려줌
- 예시 : SQLCA 중 가장 널리 사용되는 SQLCODE 변수

```c
#include <stdio.h>
#include <sqlca.h> // 오라클DB 전용 내장 SQL을 위한 헤더

EXEC SQL DECLARE cl CURSOR FOR
    SELECT empno, empname, title, manager, salary, dno
    FROM employee;

EXEC SQL OPEN cl;
while (SQLCODE == 0)
{
    /* 데이터를 성공적으로 가져올 수 있으면 SQLCODE의 값이 0이다. */
    EXEC SQL
        FETCH cl INTO :eno, :name, :title, :manager, :salary, :dno;

    if (SQLCODE == 0)
        printf("%4d %12s %12s %4d %8d %2d\n",
               eno, name, title, manager, salary, dno);
}
EXEC SQL CLOSE cl;
```

## 동적 SQL : JDBC

자바와 데이터베이스를 연결해주는 API. 직접 SQL 구문을 작성해야 한다.

여기서는 JDBC를 어떻게 이용하는지를 간단하게 살펴본다.

먼저, 데이터베이스에 접속하려면 다음과 같은 코드를 사용한다.

```java
import java.sql.Connection;
import java.sql.DriverManager;

public class DatabaseConnectionExample {
    public static Connection getConnection() {
        String url = "jdbc:mysql://localhost:3306/your_database";
        String username = "your_username";
        String password = "your_password";
        Connection connection = null;

        try {
            connection = DriverManager.getConnection(url, username, password);
            System.out.println("Database connected successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }

        return connection;
    }

    public static void main(String[] args) {
        getConnection();
    }
}
```

데이터베이스에 SQL을 전달하는 코드

```java
import java.sql.Connection;
import java.sql.Statement;

public class ExecuteSQLExample {
    public static void executeSQL() {
        Connection connection = DatabaseConnectionExample.getConnection();

        try (Statement statement = connection.createStatement()) {
            **String sql = "CREATE TABLE IF NOT EXISTS users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(50), email VARCHAR(100))";
            statement.execute(sql);**
            System.out.println("SQL executed successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        executeSQL();
    }
}
```

예외를 처리하고, 자원을 관리하는 코드 (자바의 `try-with-resource`를 사용)

```java
import java.sql.Connection;
import java.sql.Statement;

public class ResourceManagementExample {
    public static void manageResources() {
		    // java.sql.Connection 클래스는 AutoCloseable 인터페이스를 상속받음.
		    // 따라서 try가 끝나면 내부의 close()가 알아서 작동함.
        **try (Connection connection = DatabaseConnectionExample.getConnection();
             Statement statement = connection.createStatement()) {

            String sql = "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com')";
            statement.executeUpdate(sql);
            System.out.println("Data inserted successfully!");**

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        manageResources();
    }
}
```

**질의 결과를 검색하는 코드**

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class QueryResultExample {
    public static void queryResult() {
        try (Connection connection = DatabaseConnectionExample.getConnection();
             Statement statement = connection.createStatement()) {

            String sql = "SELECT * FROM users";
            ResultSet resultSet = statement.executeQuery(sql);

						// ResultSet을 하나하나 돌려가며 검색
            **while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                String email = resultSet.getString("email");
                System.out.printf("ID: %d, Name: %s, Email: %s%n", id, name, email);
            }**
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        queryResult();
    }
}
```

**일부 값을 “?“로 대체한 준비된 구문 (PreparedStatement 사용)**

```java
import java.sql.Connection;
import java.sql.PreparedStatement;

public class PreparedStatementExample {
    public static void usePreparedStatement(String name, String email) {
        **String sql = "INSERT INTO users (name, email) VALUES (?, ?)";**

        try (Connection connection = DatabaseConnectionExample.getConnection();
            PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
						**// ?에 해당하는 부분들의 값을 채운다.
            preparedStatement.setString(1, name);
            preparedStatement.setString(2, email);
            
            // 채운 것을 실제 데이터베이스에 반영**
            preparedStatement.executeUpdate();
            System.out.println("Prepared statement executed successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        usePreparedStatement("Bob", "bob@example.com");
    }
}
```

**호출할 수 있는 구문 (CallableStatement 사용)**

```java
import java.sql.CallableStatement;
import java.sql.Connection;

public class CallableStatementExample {
    public static void callStoredProcedure() {
        String sql = "{CALL InsertUser(?, ?)}";

        try (Connection connection = DatabaseConnectionExample.getConnection();
             CallableStatement callableStatement = connection.prepareCall(sql)) {

            callableStatement.setString(1, "Charlie");
            callableStatement.setString(2, "charlie@example.com");
            callableStatement.execute();
            System.out.println("Stored procedure called successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        callStoredProcedure();
    }
}
```

**메타데이터 관련 메소드**

```java
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.ResultSet;
import java.sql.Statement;

public class MetaDataExample {
    public static void getMetaData() {
        try (Connection connection = DatabaseConnectionExample.getConnection();
             Statement statement = connection.createStatement()) {

            // DatabaseMetaData
            DatabaseMetaData metaData = connection.getMetaData();
            System.out.println("Database Product Name: " + metaData.getDatabaseProductName());
            System.out.println("Database Product Version: " + metaData.getDatabaseProductVersion());

            // ResultSetMetaData
            String sql = "SELECT * FROM users";
            ResultSet resultSet = statement.executeQuery(sql);

            System.out.println("\nTable Metadata:");
            for (int i = 1; i <= resultSet.getMetaData().getColumnCount(); i++) {
                System.out.println("Column " + i + ": " + resultSet.getMetaData().getColumnName(i));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        getMetaData();
    }
}
```

# 함수와 프로시저

> SQL에서는 어떻게 사용자 정의 함수와 프로시저를 만드는가?
> 
- 프로시저(Procedure) : 데이터베이스에 저장된 SQL문과 로직을 미리 정의한 코드 블록
- 대부분의 사용자 정의 함수와 프로시저는 SQL 표준을 따르지 않음.
- 이러한 사용자 정의 함수와 프로시저를 만들 때는 성능을 고려하는 것이 굉장히 중요

## 사용자 정의 함수 : 선언과 호출

테이블을 반환하는 사용자 정의 함수를 선언하는 방법

ex. 특정한 부서에 있는 모든 강사(instructor)가 담긴 테이블을 반환하는 함수

```sql
-- 함수의 매개변수는 함수 이름으로 접두어를 붙여 참조한다. (예: instructor_of.dept_name)
CREATE FUNCTION instructor_of (dept_name VARCHAR(20))
RETURNS TABLE (
    ID VARCHAR(5),
    name VARCHAR(20),
    dept_name VARCHAR(20),
    salary NUMERIC(8, 2)
)
RETURN TABLE (
    SELECT ID, name, dept_name, salary
    FROM instructor
    WHERE instructor.dept_name = instructor_of.dept_name
);
```

이 함수는 다음과 같은 방식으로 쿼리에서 사용할 수 있다 : 

```sql
SELECT * FROM TABLE(instructor_of('Finance'));
```

이 쿼리는 ‘Finance’ 학과의 모든 강사를 반환한다. 이렇게 간단한 경우에는 테이블 함수 없이도 쉽게 쿼리를 작성할 수 있다. 그러나 일반적으로 테이블 함수는 매개변수를 지원하는 뷰(Parameterized View)로 생각할 수 있으며, 일반적인 뷰의 개념을 확장하여 매개변수를 사용할 수 있도록 해줌.

## 프로시저 : 선언과 호출

프로시저(procedure)를 선언하는 예시.

ex. 특정 학과(dept_name)에 속한 교수(instructor)의 수를 계산하는 SQL **저장 프로시저.**

```sql
CREATE PROCEDURE dept_count_proc (
    IN dept_name VARCHAR(20),
    OUT d_count INTEGER
)
BEGIN
    SELECT COUNT(*) INTO d_count
    FROM instructor
    WHERE instructor.dept_name = dept_count_proc.dept_name;
END;
```

- IN : 입력 파라미터로, 조회하려는 학과의 이름을 전달.
- OUT : 출력 파라미터로, 계산된 교수 수를 반환.

이러한 프로시저는 다음과 같이 CALL문을 이용해 호출할 수 있다.

```sql
DECLARE d_count INTEGER;
**CALL dept_count_proc('Physics', d_count);**
```

## SQL의 절차적 언어 기능

SQL은 일반 프로그래밍 언어에 가까운 기능을 제공하며, 이를 `Persistent Storage Module(PSM)`이라 함.

1. **변수 선언과 할당**

```sql
-- **변수 선언**: declare 문을 사용하며, SQL 데이터 타입으로 선언 가능.
declare student_count int; -- 변수 선언
declare max_capacity int default 50; -- 기본값 포함 변수 선언

-- **값 할당**: set 문으로 변수에 값 할당.
set student_count = (select count(*) from takes where course_id = 'CS101');

-- 조건문과 함께 사용
if student_count < max_capacity then
    set student_count = student_count + 1; -- 값 증가
else
    signal sqlwarning set message_text = 'Class is full!';
end if;
```

1. **복합문**
- **형식**: `begin ... end`
    - 내부에 여러 SQL 문을 포함할 수 있음.
    - `begin atomic ... end`: 모든 문장이 하나의 트랜잭션으로 실행됨.
- 로컬 변수를 선언할 수 있음.

```sql
begin
    declare student_count int;
    declare max_capacity int default 30;

    -- 현재 수강 인원 계산
    set student_count = (select count(*) from takes where course_id = 'CS101');

    -- 수강 가능 여부 확인
    if student_count < max_capacity then
        insert into takes (student_id, course_id) values ('S001', 'CS101');
        commit; -- 트랜잭션 커밋
    else
        rollback; -- 트랜잭션 롤백
    end if;
end;
```

1. **반복문 / 조건문**

```sql
-- while
while boolean expression do sequence of statements;
end while

-- repeat
repeat sequence of statements;
until boolean expression
end repeat

-- for loop
declare n integer default 0;
for r as select budget from department where dept name = ‘Music‘
do set n = n− r.budget
end for

-- if-then-else
if boolean expression
	then statement or compound statement
elseif boolean expression
	then statement or compound statement
else statement or compound statement
end if
```

1. 예외 처리
- **예외 선언은** `declare condition`를 사용
- **핸들러**: 예외 발생 시 동작 정의. `declare exit handler`

```sql
-- 예외 선언 및 처리
declare **class_full** condition; -- 예외 선언

-- 예외 핸들러 정의
**declare exit handler** for **class_full** 
begin
    signal sqlwarning set message_text = 'Class is full!';
end;
```

- 예외처리가 구현된 로직

```sql
-- 로직 구현
begin
    declare student_count int;
    declare max_capacity int default 30;

    -- 현재 수강 인원 계산
    set student_count = (select count(*) from takes where course_id = 'CS101');

    -- 수강 가능 여부 확인
    if student_count >= max_capacity then
        **signal class_full; -- 예외 발생**
    else
        insert into takes (student_id, course_id) values ('S001', 'CS101'); -- 등록 처리
    end if;
end;
```

## SQL의 외부 언어 연계

> SQL이 절차적 언어 기능을 가지고 있다고 해도 한계가 있음.
그렇다면, 다른 언어와 SQL을 연계해서 기능을 구현할 수는 없을까?
> 
- SQL의 절차적 확장은 매우 유용할 수 있지만, 아쉽게도 데이터베이스 간 표준 방식으로 지원되지 않음.
- 따라서 명령형 프로그래밍 언어에서 절차를 정의하고 이를 SQL 쿼리 및 트리거 정의에서 호출할 수 있게 하는 방식이 대안이 될 수 있음.
- 특히, SQL로는 할 수 없는 계산을 코드를 통해 할 수 있음.

예시코드

```sql
create procedure dept_count_proc( 
    in dept_name varchar(20),
    out count integer
)
language C
external name '/usr/avi/bin/dept_count_proc';

create function dept_count(dept_name varchar(20))
returns integer
language C
external name '/usr/avi/bin/dept_count';
```

- 외부 언어로 만든 코드가 예외상황을 처리하지 못하는 경우, DB에 오류가 발생할 수 있다.
- 따라서 이를 극복하기 위해 다음과 같은 방법을 쓴다.
    - SQL문에 `parameter style general`이라는 추가 줄을 선언에 포함하여 외부 절차/함수가 표시된 인수만 받고 null 값이나 예외를 처리하지 않는다는 것을 나타낼 수 있음.
    - 다른 프로세스에서 실행시키고, 그 결과를 가지고 오기. (프로세스 간 통신 이용)
    - 샌드박스(sandbox) 내에서 실행시키기

# 트리거

- 명시된 이벤트 (=데이터베이스의 갱신)이 발생할 때마다 자동으로 실행되는 동작
- 데이터베이스의 무결성을 유지하기 위한 일반적이고 강력한 도구
- 언제, 어떤 동작을 실행할지를 미리 정해야 함.

## 언제 필요한가?

- **SQL의 제약 조건(constraint)으로 표현할 수 없는 무결성 제약 조건을 구현**하는 데 유용.
- 또한, 특정 조건이 충족되었을 때 **알림을 보내거나 자동으로 작업을 시작**하는 메커니즘으로도 활용.
- ex.  **어떤 학생이 수강 신청(takes 테이블)에 등록될 때** 해당 학생의 총 학점(student 테이블의 total credits)을 업데이트하도록 트리거를 설계할 수 있음.
- 만약 트리거가 무언가를 변경한다고 할 때, 외부 시스템이 필요하다고 하면 다음과 같은 프로세스를 거친다.
    - 트리거가 특정한 릴레이션을 변경
    - 특정한 릴레이션을 외부 시스템(백엔드 등)이 읽고 실제로 변경
    - 외부 시스템을 통해 실제 데이터베이스에 반영
- **즉, 실제 트리거는 외부 시스템과 직접적으로 상호작용하지는 않음.**

**어떻게 트리거로 무결성을 유지할까?**

**ex. timeslot_check1 트리거**

- **목적**: section 테이블에 새로운 튜플이 **삽입**될 때, 해당 시간 슬롯(time_slot_id)이 유효한지 확인합니다.
- **동작**: 삽입된 시간 슬롯이 유효하지 않으면 **롤백**(삽입 취소)합니다.

```sql
create trigger timeslot_check1
after insert on section
referencing new row as nrow
for each row
when (nrow.time_slot_id not in (
    select time_slot_id
    from time_slot)) /* time_slot_id가 time_slot 테이블에 없는 경우 */
begin
    rollback
end;
```

**ex. timeslot_check2 트리거**

- **목적**: time_slot 테이블에서 시간 슬롯이 **삭제**될 때, 해당 시간 슬롯이 section 테이블에서 여전히 참조되고 있는지 확인합니다.
- **동작**: 시간 슬롯이 삭제되었지만 section 테이블에서 여전히 참조 중인 경우, **롤백**(삭제 취소)합니다.

```sql
create trigger timeslot_check2
after delete on time_slot
referencing old row as orow
for each row
when (
    orow.time_slot_id not in (
        select time_slot_id
        from time_slot) /* time_slot_id가 time_slot 테이블에 마지막으로 삭제된 경우 */
    and orow.time_slot_id in (
        select time_slot_id
        from section)) /* 하지만 section 테이블에서 여전히 참조 중인 경우 */
begin
    rollback
end;
```

## Check와의 차이

```sql
// Check 제약조건
CREATE TABLE Employees (
    emp_id INT PRIMARY KEY,
    salary DECIMAL(10, 2) CHECK (salary > 0) -- 급여가 0보다 커야 한다는 조건
);
```

```sql
// 트리거
CREATE TRIGGER UpdateBonus
AFTER UPDATE ON Employees
FOR EACH ROW
WHEN (NEW.salary > 10000) -- 급여가 10,000을 초과할 때만 실행
BEGIN
    INSERT INTO Bonuses (emp_id, bonus_amount)
    VALUES (NEW.emp_id, 1000); -- 보너스 기록 테이블에 데이터 추가
END;
```

- Check 제약 조건: 간단한 데이터 검증(값의 범위, 조건 확인)에 적합합니다.
- 트리거: 데이터 무결성 유지뿐만 아니라, 복잡한 비즈니스 로직(자동화 작업, 다른 테이블과의 상호작용 등)을 처리하는 데 적합합니다.

따라서, 검증만 필요한 경우에는 Check 제약 조건을 사용하고, 자동화된 작업이나 복잡한 로직이 필요한 경우에는 트리거를 사용하는 것이 좋습니다.

## **트리거 활용 시 주의사항**

1. **무한 루프 방지**: 
- 트리거의 동작이 다른 트리거를 실행하고, 이 과정이 반복될 경우 무한 루프가 발생할 수 있습니다. 예를 들어, 삽입 트리거가 동일한 테이블에 또 다른 삽입을 유발하면 문제가 생길 수 있음.
- 대부분의 데이터베이스는 이러한 루프를 방지하기 위해 트리거 체인의 길이를 제한하거나, 자기 참조 트리거를 금지함.
1. **비표준 문법**:
- SQL:1999 이전에는 트리거가 SQL 표준에 포함되지 않았기 때문에, 데이터베이스 시스템마다 문법이 다릅니다. 이로 인해 이식성에 문제가 생길 수 있음.
1. **대안 활용**:
- 트리거는 복잡한 작업을 자동화하는 데 유용하지만, 경우에 따라 다른 기술로 대체하는 것이 더 적합함.
- 예를 들어, **참조 무결성 유지**는 트리거 대신 SQL의 **ON DELETE CASCADE** 같은 기능으로 구현할 수 있음.
1. **성능 문제**:
- 트리거는 데이터 변경 시마다 실행되므로, 과도하게 사용하면 성능에 영향을 줄 수 있음.

# 재귀 쿼리

- 재귀 쿼리란? **스스로를 반복적으로 호출**하면서 원하는 데이터를 찾아가는 쿼리
- 왜 쓰는가? 데이터베이스에서 **계층적 데이터**(트리 구조나 그래프 구조)를 처리하거나, **직접 또는 간접 관계**를 반복적으로 탐색해야 할 때 사용한다.

## 예시 : 선이수 과목 (반복문)

각 대학 과목들의 선이수 과목들을 정리해놓은 `prereq` 릴레이션이 있다고 가정하자.

```sql
course_id	| prereq_id
CS-347 | CS-319
CS-319 | CS-315
CS-319 | CS-101
CS-315 | CS-190
CS-190 | CS-101
```

만약 이번 학기에 CS-347를 듣으려면, 무슨 과목들을 선수강해야 하는가?

- 간접적인 전제 조건은 전부 포함한다.
- 중복 조건은 1번만 반영한다.

우리는 반복문을 집어넣은 사용자 정의 함수 `findAllPrereqs`으로 이를 해결할 수 있다.

```sql
create function findAllPrereqs(cid varchar(8))
returns table (course id varchar(8))
begin
    -- 임시 테이블 생성
    CREATE TEMPORARY TABLE c_prereq (course_id VARCHAR(8));  -- 최종 결과
    CREATE TEMPORARY TABLE new_c_prereq (course_id VARCHAR(8)); -- 새로 찾은 전제 조건
    CREATE TEMPORARY TABLE temp (course_id VARCHAR(8)); -- 중간 결과

    -- 1단계: 직접 전제 조건 찾기
    INSERT INTO new_c_prereq
    SELECT prereq_id
    FROM prereq
    WHERE course_id = cid;
    -- 반복적으로 전제 조건 찾기
    REPEAT
        -- 새롭게 발견된 전제 조건을 최종 결과에 추가
        INSERT INTO c_prereq
        SELECT course_id
        FROM new_c_prereq;

        -- 간접 전제 조건 찾기
        INSERT INTO temp
        SELECT prereq.prereq_id
        FROM new_c_prereq, prereq
        WHERE new_c_prereq.course_id = prereq.course_id
        EXCEPT -- 이미 발견된 전제 조건은 제외
        SELECT course_id
        FROM c_prereq;

        -- 다음 반복을 위해 temp 내용을 new_c_prereq로 복사
        DELETE FROM new_c_prereq;
        INSERT INTO new_c_prereq
        SELECT *
        FROM temp;

        -- temp 초기화
        DELETE FROM temp;

    -- 더 이상 새로운 전제 조건이 없으면 종료
    UNTIL NOT EXISTS (SELECT * FROM new_c_prereq)
    END REPEAT;

    -- 최종 결과 반환
    RETURN TABLE c_prereq;
END;
```

## 예시 : 선이수 과목 (재귀쿼리)

그런데, 위의 코드는 너무 길다는 단점이 있다! 

이때 SQL의 재귀 쿼리를 도입해서 코드의 양을 줄일 수 있다.

```sql
WITH RECURSIVE rec_prereq(course_id, prereq_id) AS (
    -- 기본 쿼리: 직접 전제 조건
    SELECT course_id, prereq_id
    FROM prereq

    UNION

    -- 재귀 쿼리: 간접 전제 조건
    SELECT p.course_id, r.prereq_id
    FROM prereq p
    INNER JOIN rec_prereq r
    ON p.prereq_id = r.course_id
)
SELECT *
FROM rec_prereq
WHERE course_id = 'CS-347';
```

SQL의 **WITH RECURSIVE** 문법을 사용하면, 반복적인 작업을 간단히 작성할 수 있다!

## 재귀 쿼리의 특징

- 항상 두 가지 하위 쿼리의 **UNION**으로 정의되어야 함 :
    1. **기본 쿼리(base query)**: 재귀를 사용하지 않는 쿼리.
    2. **재귀 쿼리(recursive query)**: 재귀적으로 정의된 뷰를 참조하는 쿼리.
- 다음과 같은 과정을 반복한다고 생각하면 된다.

```
- 기본 쿼리를 실행.
- 재귀 쿼리를 실행
    - 그 안에 기본 쿼리를 실행하고,
	  - 재귀 쿼리를 실행하고
			  - 그 안에 기본 쿼리를 실행하고,
			  - 재귀 쿼리를 실행하고
....

이 구조가 고정점을 찾을 때까지 계속 이어짐.
```

- 최종적으로 만들어지는 뷰를 고정점(fixed point)라고 함.
- 또한 재귀 쿼리는 **단조(monotonic)**이어야 한다. 즉, 뷰에 새로운 튜플이 추가될수록 이전 결과의 상위 집합이 되어야 함.
- 즉, 다음과 같은 작업은 허용되지 않는다는 것.
    1. **재귀 뷰에 대한 집계 연산**.
    2. **재귀 뷰를 포함하는 하위 쿼리에서** NOT EXISTS **사용**.
    3. **재귀 뷰가 오른쪽에 포함된 차집합(**EXCEPT**) 사용**.

# 고급 통계 기능

> 기타 고급 통계 기능을 알아보자.
> 

## 랭킹 (Ranking)

- 순위 매기기에는 order by를 사용한다.
- 중복 순위를 허용(ex. 1위가 2명인 경우)하려면 rank, 그렇지 않으면 dense rank
- NULLS FIRST 또는 NULLS LAST를 사용해 NULL 값의 순서를 지정
- 다른 순위 함수들의 예시

```
•	PERCENT_RANK: 순위를 백분율로 나타냅니다.
•	CUME_DIST: 누적 분포를 계산합니다.
•	ROW_NUMBER: 동일한 정렬 값이 있어도 각 튜플에 고유한 번호를 부여합니다.
•	NTILE(n): 각 파티션을 n개의 동일한 크기로 나눕니다.

```

## 윈도우 기능 (Windowing)

특정 튜플 범위에 대해 집계 함수(aggregate function)를 계산.

고정된 시간 범위의 집계를 계산하는 데 유용하며, 이 시간 범위를 윈도우(window)라고 부른다. 윈도우는 겹칠 수 있으며, 이 경우 하나의 튜플이 여러 윈도우에 포함될 수 있다.

- 주로 추세분석을 할 때 이 기능을 사용

**예시 : 고정된 범위의 윈도우**

다음 쿼리는 정렬 순서에 따라 지정된 범위(이 경우 3개의 이전 튜플)에 대해 평균을 계산한다:

```sql
SELECT year, AVG(num_credits) 
OVER (ORDER BY year ROWS 3 PRECEDING) AS avg_total_credits
FROM tot_credits;
```

이 쿼리는 지정된 정렬 순서에 따라 3개의 이전 튜플에 대한 평균을 계산한다. 

예를 들어, 2019년에 대해, 2018년과 2017년의 튜플이 존재한다면, 2017년, 2018년, 2019년 값을 평균내는 결과를 반환한다. 

가장 초기 연도에 대해서는 해당 연도만 평균에 포함하며, 두 번째 연도는 2개의 연도에 대해 평균을 계산한다.

이 예는 각 연도가 관계에서 한 번만 나타나기 때문에 의미가 있음. 

만약 동일한 연도에 여러 튜플이 있다면, 튜플 순서가 여러 가지로 가능하다.

## 피벗 (Pivoting)

- 피벗(Pivot)은 데이터를 다른 방식으로 재구성하여 분석에 유용한 형태로 변환하는 기능이다.
- 예를 들어, 한 상점이 어떤 종류의 옷이 인기 있는지 확인하고자 한다고 가정해보자. 옷 데이터는 다음 속성으로 구성된다 :

```sql
	•	item name: 옷의 종류 (예: skirt, dress, shirt, pants)
	•	color: 색상 (dark, pastel, white)
	•	clothes size: 크기 (small, medium, large)
	•	quantity: 해당 조합으로 판매된 총 수량
```

### **피벗 테이블 생성**

**피벗 테이블**(cross-tabulation 또는 pivot-table)은 주어진 데이터에서 특정 속성 값이 새로운 열 이름으로 변환된 형태의 테이블이다. 예를 들어, **color** 속성의 값(dark, pastel, white)이 새로운 속성 이름으로 변환해 다음과 같은 테이블을 만들 수 있다 : 

| item name | size | dark | pastel | white |
| --- | --- | --- | --- | --- |
| dress | S | 2 | 4 | 0 |
| dress | M | 6 | 3 | 0 |

피벗 테이블을 생성하는 쿼리는 다음과 같이 작성할 수 있다 : 

```sql
SELECT * FROM sales
PIVOT (
    SUM(quantity)
    FOR color IN ('dark', 'pastel', 'white')
);
```

**피벗의 동작**

- **단일 튜플**: color 값이 “dark”인 튜플이 하나뿐이라면 해당 튜플의 quantity 값이 새 열의 값으로 사용.
- **다중 튜플**: 동일한 조건에 여러 튜플이 있다면, SUM 함수와 같은 집계 함수로 값을 합산한다.

피벗 없이도 기본 SQL 구문으로 동일한 결과를 얻을 수 있지만, 피벗 구문은 이러한 쿼리를 훨씬 간결하게 작성할 수 있도록 도와준다.

### **롤업(Rollup)과 큐브(Cube)**

- 롤업(Rollup)과 큐브(Cube)는 GROUP BY 구문의 확장을 제공하여 다중 그룹화 결과를 한 번의 쿼리로 생성할 수 있다.
- 롤업은 점진적인 집계를 생성한다. 예를 들어:

```sql
SELECT item_name, color, SUM(quantity)
FROM sales
GROUP BY ROLLUP(item_name, color);

/*
이 쿼리는 다음과 같은 그룹화를 생성한다:

1. (item_name, color)
2. (item_name)
3. () (전체 합계)

결과는 각 그룹화 수준별로 SUM(quantity) 값을 계산하며, 속성이 없는 그룹화는 NULL로 표시된다.
*/
```

- 큐브는 모든 속성의 **모든 부분 집합**에 대해 그룹화를 생성한다. 예를 들어:

```sql
SELECT item_name, color, clothes_size, SUM(quantity)
FROM sales
GROUP BY CUBE(item_name, color, clothes_size);

/*
이 쿼리는 다음 그룹화를 생성합니다:
	1.	(item_name, color, clothes_size)
	2.	(item_name, color)
	3.	(item_name, clothes_size)
	4.	(color, clothes_size)
	5.	(item_name)
	6.	(color)
	7.	(clothes_size)
	8.	() (전체 합계)

큐브는 롤업보다 더 많은 그룹화를 생성합니다.
*/
```

### **그룹화 세트 (Grouping Sets)**

특정 그룹화만 필요할 경우, GROUPING SETS 구문을 사용할 수 있다.

```sql
SELECT item_name, color, clothes_size, SUM(quantity)
FROM sales
GROUP BY GROUPING SETS ((color, clothes_size), (clothes_size, item_name));

/*
이 쿼리는 color, clothes_size와 clothes_size, item_name에 대해서만 그룹화를 수행합니다.
*/
```

### 그룹화된 NULL 값 처리

롤업과 큐브로 생성된 NULL 값을 명확히 구분하고자 할 때, GROUPING() **함수**를 사용할 수 있다:

```sql
SELECT 
    CASE WHEN GROUPING(item_name) = 1 THEN 'all' ELSE item_name END AS item_name,
    CASE WHEN GROUPING(color) = 1 THEN 'all' ELSE color END AS color,
    SUM(quantity) AS quantity
FROM sales
GROUP BY ROLLUP(item_name, color);
```

위 쿼리는 롤업으로 생성된 NULL 값을 all로 표시하며, 데이터베이스에 저장된 실제 NULL 값과 구분할 수 있다.