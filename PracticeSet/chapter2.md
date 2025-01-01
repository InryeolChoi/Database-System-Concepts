# 2단원 연습문제
## 2.1

> 다음 데이터베이스를 생각해보자. 어떤 속성이 주 키로 가장 적합한가?
> 

```
employee (person_name, street, city)
works (person_name, company_name, salary)
company (company_name, salary)
```

employee의 경우, 주 속성은 person_name. (단 동일인이 있다면 person_name) 

works의 경우, 주 속성은 person_name, company_name.

company의 경우, 주 속성은 company_name

## 2.2

> `instructor` 테이블의 `dept_name` 속성과 `department` 테이블 간의 외래 키 제약조건을 고려해보자. 이러한 테이블들에 대해 외래 키 제약조건을 위반할 수 있는 삽입과 삭제의 예시를 들어보자.
> 

```
instructor (ID, name, dept_name, salary)
department (dept_name, building, budget)

instructor -> department
```

크게 두 가지 예시가 있다.

1. 없는 값을 삽입
- 다음과 같은 튜플을 instructor에 삽입 : `(10111, Ostrom, Economics, 110000)`
- 근데, dept_name에 Economics가 없으면 ‘외래 키 제약조건’을 위배한 것

1. 있어야 하는 값을 삭제
- 다음과 같은 튜플을 department에서 삭제 : `(Biology, Watson, 90000)`
- 근데 instructor에 Biology 값이 있는 튜플이 1개라도 존재할 경우, ‘외래 키 제약조건’을 위배한 것.

## 2.3

> time_slot 테이블을 고려해보자. 특정 시간 슬롯이 한 주에 여러 번 만날 수 있으므로, day와 start_time을 이 테이블의 주 키에 포함시키고, end_time은 포함하지 않는 이유를 설명하자.
> 

```
time_slot (time_slot_id, day, start_time, end_time)
```

day는 수업이 진행되는 요일. 같은 수업이라도 월요일과 화요일에는 다른 행으로 기록됨.

start_time은 수업이 시작하는 시간입니다. 같은 요일에 같은 수업이 두 번 진행될 수 있으므로, 시작 시간이 고유한 식별에 필요. 따라서 day와 start_time은 고유성을 가지고 있다고 할 수 있음

그러나 end_time은 start_time에 종속적인 값이므로 고유성을 제공하지 않음. 수업이 시작한 시간이 동일하다면 종료 시간은 항상 동일하게 결정되기 때문.

따라서 end_time은 주 키에 포함되지 못함.

## 2.4

> 그림 2.1에 나타난 instructor 테이블의 예시에서, 두 명 이상의 교수가 동일한 이름을 갖고 있지 않다. 이를 통해 name을 instructor 테이블의 슈퍼키(또는 기본 키)로 사용할 수 있다고 결론지을 수 있을까?
> 

그렇지 않음. 같은 이름의 교수가 언제든지 추가될 수 있기 때문임.

## 2.5

> 학생(student) 테이블과 지도교수(advisor) 테이블의 데카르트 곱(Cartesian product)을 수행한 후, 결과에 대해 조건 $S\_{id} ~=~ ID$*를 사용하여 선택(selection) 연산을 수행하면 어떤 결과가 나오는가? (관계 대수의 기호 표현을 사용하면 이 쿼리는*  $\sigma_{s\_{id} = ID}(student \times advisor)$ 로 나타낼 수 있다.)
> 

```
student (ID, name, dept_name, tot_cred)
advisor (S_ID, i_ID)
```

각 튜플에는 student의 모든 속성이 들어간 뒤 student의 id와 동일한 S_ID, 지도 교수의 i_id가 나온다. 

지도교수가 없는 학생의 경우, 이 결과에서 제외된다. 또한 한 지도교수가 여러 명의 학생을 가질 경우, 이는 중복되어서 나오게 된다. 

## 2.6

> 다음 릴레이션들을 보고, 조건에 맞는 관계대수를 쓰시오.
> 
> 
> 1. “Miami” 도시에 거주하는 각 직원의 이름을 찾아라.
> 2. 급여가 $100,000보다 높은 각 직원의 이름을 찾아라.
> 3. “Miami”에 거주하며 급여가 $100,000보다 높은 각 직원의 이름을 찾아라.
> 

```
employee (person_name, street, city)
works (person_name, company_name, salary)
company (company_name, city)
```

1. $\Pi_{person\_name}
(\sigma_{city ~=~ Miami} 
(\text{employee}))$
2. $\Pi_{person\_name}
(\sigma_{salary ~>~ 100000}
(\text{works}))$
3. $\Pi_{person\_name}
(\sigma_{city = Miami ~\land~ salary ~>~ 100000}(\text{employee} \Join \text{works}))$

## 2.7

> 다음 릴레이션들을 보고, 조건에 맞는 관계대수를 쓰시오.
> 
> 
> 1. “Chicago”에 위치한 각 지점(branch)의 이름을 찾아라.
> 2. “Downtown” 지점에서 대출을 받은 각 대출자(borrower)의 ID를 찾아라.
> 

```
branch (branch_name, branch_city, assets)
customer (ID, customer_name, customer_street, customer_city)
loan (loan_number, branch_name, amount)
borrower (ID, loan_number)
account (account_number, branch_name, balance)
depositor (ID, account_number)
```

1. $\Pi_{branch\_name}(\sigma_{branch\_city ~=~ Chicago}(\text{branch}))$
2. $\Pi_{ID}(\sigma_{branch\_name ~=~ Downtown}(loan \bowtie~_{loan.branch\_name ~=~ branch.branch\_name}~(branch)))$

## 2.8

> 다음 질의에 맞는 관계대수를 제시하라.
1. “BigBank”에서 근무하지 않는 각 직원의 ID와 이름을 찾아라.
2. 데이터베이스에 있는 모든 직원보다 적어도 동일한 급여를 받는 각 직원의 ID와 이름을 찾아라.
> 

```
employee (person_name, street, city)
works (person_name, company_name, salary)
company (company_name, city)
```

1. $\Pi_{\text{ID, person\_name}}~(\text{employee}) - 
\Pi_{\text{ID, person\_name}}~(\text{employee} \bowtie~_
{\text{employee.ID} ~=~ \text{works.ID}}
(\sigma_{company\_name = BigBank})(works))$
2. $\Pi_{\text{ID}}~(\text{employee)} - \Pi_{\text{A.ID, A.person\_name}}
(\rho_A(\text{employee}) \bowtie
~_{\text{A.salary < B.salary}} 
\rho_B(\text{employee}))$