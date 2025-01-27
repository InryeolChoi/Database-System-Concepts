# 중급 SQL (3)

# SQL의 인덱스 정리

## 인덱스란?

- 데이터베이스에서 특정 속성 값으로 튜플을 효율적으로 찾을 수 있도록 도와주는 데이터 구조.
- 전체 데이터를 순차적으로 검색하지 않고, 원하는 값을 빠르게 찾을 수 있음.
- 예시: Physics 학과의 강사를 찾을 때, 모든 튜플을 확인하지 않고 dept_name 속성에 인덱스를 사용하여 바로 접근 가능.

## **인덱스의 특징**

- **중복 데이터 구조**: 인덱스는 논리 스키마(logical schema)가 아닌 물리 스키마(physical schema)의 일부로, 데이터베이스의 정확성에 반드시 필요한 것은 아님.
- **효율성**: 쿼리 및 업데이트 작업의 처리 속도를 향상.
- 기본 키(primary key) 및 외래 키(foreign key)와 같은 무결성 제약 조건을 효율적으로 강제.
- **비용**: 인덱스는 추가적인 저장 공간을 사용하며, 업데이트 시 성능에 영향을 줄 수 있음.

## **인덱스 생성 및 삭제**

**인덱스 생성 명령어**

```sql
create index <인덱스 이름> on <테이블 이름> (<속성 리스트>);
```

- 예를 들어, dept_name 속성에 인덱스를 생성한다고 해보자.

```sql
create index dept_index on instructor (dept_name);
```

**고유 인덱스(unique index)**

- 특정 속성이나 속성 조합이 후보 키(candidate key)임을 선언.

```sql
create unique index <인덱스 이름> on <테이블 이름> (<속성 리스트>);
```

- 예시

```sql
create unique index dept_index on instructor (dept_name);
```

후보 키가 아닌 속성에 고유 인덱스를 선언하면 에러 발생.

**인덱스 삭제 명령어**

```sql
drop index <인덱스 이름>;
```

## **인덱스 활용**

- SQL 쿼리에서 인덱스가 도움이 되는 경우, 쿼리 프로세서가 자동으로 인덱스를 사용하여 성능을 최적화.
- 예: dept_name = 'Music' 조건을 만족하는 튜플을 찾을 때, 인덱스를 활용하여 전체 데이터를 읽지 않음.

## **인덱스 유형 및 클러스터링**

- **인덱스 유형**: B+-트리, 해시 인덱스 등 다양한 인덱스 유형을 명시적으로 지정 가능(시스템에 따라 다름).
- **클러스터링 인덱스(Clustered Index)**: 테이블의 튜플을 특정 인덱스의 검색 키(search key)를 기준으로 정렬.
- 하나의 테이블에 하나의 클러스터링 인덱스만 생성 가능.

## **자동 인덱스 관리**

- 데이터베이스 시스템은 자동으로 일부 인덱스를 생성할 수 있음.
- 그러나 인덱스 생성 및 삭제는 대부분의 시스템에서 프로그래머가 명시적으로 제어.

# 권한

데이터베이스는 사용자에게 여러 형태의 권한을 부여해 줄 수 있음.

- 데이터 읽기
- 새로운 데이터 삽입
- 데이터 갱신
- 데이터 삭제

이러한 유형의 각 권한을 **특권**이라고 부름. 

사용자가 질의나 갱신을 수행할 때, DBMS는 우선 그 사용자의 권한을 토대로 해당 질의나 갱신을 수행할 권한이 있는지 실제로 확인하고, 없으면 질의나 갱신을 실행하지 않음.

데이터에 대한 권한과 함께 데이터베이스 스키마 변경 권한도 사용자에게 부여할 수 있음. 일부 형태의 권한을 사용자는 타 사용자에게 이를 넘기거나, 철회할 수 있음.

가장 높은 권한은 DBA(데이터베이스 관리자) 권한. DBA는 타 사용자에게 권한을 주거나 데이터베이스를 재구성할 수 있음. 이는 운영체제의 superuser와 비슷.

여러가지 권한을 부여하는 예시들을 알아보자.

## 권한의 부여 및 취소

다음과 같은 명령어를 이용해 권한을 부여할 수 있음.

```sql
GRANT <privilege list> ON <relation/view name> TO <user/role list>;
```

예시는 다음과 같다.

```sql
-- Amit, Satoshi에게 department 릴레이션에 대한 select 권한을 부여한다.
GRANT SELECT ON department TO Amit, Satoshi;

-- Amit, Satoshi에게 department 릴레이션의 budget 속성을 update할 권한을 부여한다.
GRANT UPDATE (budget) ON department TO Amit, Satoshi;
```

**Update 권한을 Grant 문에 포함할 때는 반드시 Update 키워드 다음에 갱신 권한을 포함할 속성 목록을 괄호를 통해 써서 포함하게 해준다.**

권한 부여의 다른 특징들을 적어보자면…

- 새로 생성된 테이블 / 뷰의 소유자는 자동으로 모든 권한을 부여받는다.
- Delete 권한을 이용해 릴레이션의 튜플을 삭제할 권한을 준다
- public이라는 사용자 이름은 모든 사용자를 이야기한다.

권한을 취소하는 명령어는 다음과 같다.

```sql
REVOKE <privilege list> ON <relation/view name> FROM <user/role list>;
```

예시는 다음과 같다.

```sql
REVOKE SELECT ON department FROM Amit, Satoshi;
```

## 역할과 권한

데이터베이스 시스템은 사용자마다 역할을 정해줄 수 있다.

ex) 사용자 = 교수면 권한을 더 많이 부여하고, 사용자 = 학생이면 권한을 더 적게 부여

다음과 같은 명령어를 보자.

```sql
CREATE ROLE instructor;  
GRANT SELECT ON takes TO instructor;  
GRANT instructor TO dean;  
GRANT dean TO Satoshi;
```

1. instructor라는 역할을 만든다
2. takes 릴레이션에 대한 select 권한을 Instructor에게 준다.
3. instructor 역할을 dean 역할에 부여
4. Satoshi에게 dean 역할을 부여

## 뷰에 대한 권한

뷰에 대한 권한 역시 설정할 수 있다. 

예를 들어, 특정 데이터를 제한적으로 제공하기 위해 뷰를 만든다고 하자.

```sql
CREATE VIEW geo_instructor AS  
SELECT * FROM instructor WHERE dept_name = 'Geology';
```

- 이때 geo_instructor의 사용자는 뷰에 대한 권한만을 부여받아야 하며, 기본 테이블(instructor)에 직접 접근할 수 있는 권한이 없어야 한다.

또한 사용자가 geo_instructor 뷰를 통해 쿼리를 실행하면, 내부적으로는 뷰가 정의된 SQL로 변환된다.

예를 들어서, 사용자가 아래의 쿼리를 실행시키면…

```sql
SELECT * FROM geo_instructor;
```

실제 데이터베이스 내부에서는 다음과 같은 쿼리가 작동한다.

```sql
SELECT * FROM instructor WHERE dept_name = 'Geology';
```

**만약 사용자가** 뷰를 생성하려면 기본 테이블에 대해 필요한 권한이 있어야 한다.

예를 들어, `geo_instructor` 뷰를 생성하려면 instructor 테이블에 대해 SELECT 권한이 필요.

또한, 뷰 생성자가 추가적인 권한을 얻는 것은 제한된다. 예를 들어 뷰를 업데이트할 권한을 얻으려면, 기본 테이블에 대해 업데이트 권한이 있어야 함.

## 스키마와 권한

SQL은 사용자가 릴레이션을 생성할 때 외래 키를 선언할 수 있도록 허가하는 references 특권을 가지고 있으며, 이는 특정 속성에 수여한다.

다음 예시는 mariano라는 사용자가 department 릴레이션의 key인 dept_name 속성을 외래 키로 참조하는 릴레이션을 생성할 수 있게 한다.

```sql
grant references (dept_name) on department to Mariano;
```

이런게 왜 필요한가?

- 외래 키 참조는 **참조된 테이블**에 대한 삭제와 업데이트 작업에 제약을 가함.
- 예시: Mariano가 dept_name 속성을 외래 키로 참조하는 관계 **r**을 생성하고, Geology 부서에 대한 데이터를 삽입했을 경우:
    - 이제 다른 사용자가 Geology 부서를 department 테이블에서 삭제하려면 r 관계를 수정해야 함.
- 즉 외래 키 선언은 참조된 테이블의 다른 사용자의 작업을 제한할 수 있으므로 REFERENCES **권한**이 필요함.

같은 이유로, CHECK **제약 조건**이 하위 쿼리를 통해 다른 테이블(예: department)을 참조할 경우에도 마찬가지로 REFERENCES **권한**이 필요함.

## 권한 양도 / 취소

권한의 양도는 다음과 같이 작성하면 된다.

```sql
-- Amit에게 department 릴레이션의 select 권한을 주고, Amit이 이를 양도할 수 있게 함.
grant select on department to Amit with grant option;
```

- 적절한 grant 문에 with grant option을 추가하면 양도 권한을 부여할 수 있다.
- 권한의 양도가 여러 번 이뤄지는 과정을 그래프 구조를 통해서 표현할 수 있으며 이를 권한 그래프라고 함

권한의 취소는 위에서 봤던 대로 revoke를 사용하면 된다.

그런데, 하나의 권한 취소는 연쇄적인 권한 취소를 불러올 수 있다.

따라서 다음과 같이 restrict를 명시할 수 있다.

```sql
revoke select on department from Amit, Satoshi restrict;
```

이러한 연쇄 취소에 대처하기 위해 사용자보다는 주로 역할(Role)에 특권을 주는 경우가 많다.

특정 사용자가 권한을 부여했을 때, 해당 사용자에게서 권한이 회수되면 그 사용자가 부여했던 권한도 함께 취소되는 경우가 있음.

- 예시: Satoshi가 dean 역할을 가지며, Amit에게 instructor 역할을 부여함. 이후 Satoshi의 dean 역할이 회수되었더라도, Amit은 여전히 교수직원으로 남아야 하므로 instructor 역할을 유지해야 함.
- 하지만 기본적인 **Cascading Revocation** 메커니즘에서는 Satoshi의 역할 회수가 Amit의 권한까지 취소하는 부적절한 상황을 초래함.

따라서 이를 해결하기 위해  SQL에서는 권한을 사용자(User)가 아닌 역할(Role)로 부여할 수 있는 메커니즘을 제공함. 세션(Session)에 연관된 현재 역할(current role)을 설정하여 이를 통해 권한을 부여 가능.

**현재 역할 설정**: 기본적으로 세션에 연결된 현재 역할은 null로 설정됨. 

이를 다음 명령어를 통해 특정 역할을 세션에 설정할 수 있음:

```sql
SET ROLE role_name;
```

단 사용자가 해당 역할을 소유하지 않을 경우 명령어는 실패함.

**역할을 통한 권한 부여**: 권한 부여 시 granted by current role 구문을 사용하여 현재 역할로 권한 부여 가능.

```sql
GRANT privilege_name TO user_name GRANTED BY CURRENT ROLE;
```

단, 현재 역할이 **null**이 아니어야 함.

## 행 수준 권한

- 일부 데이터베이스는 행 수준에서 세분화된 권한을 제공해줌
- 오라클의 가상 사설 데이터베이스(VPD)가 대표적인 예시