---
title: "서브쿼리와 ANY·ALL — 모르는 값을 쿼리 안에서 구하기"
date: 2025-02-11 21:30:00 +0900
categories: [CS, Database]
tags: [Database, SQL, Oracle, 서브쿼리, subquery, ANY, ALL, 단일행, 다중행]
description: "특정 값을 모를 때 SELECT 안에 SELECT를 넣어 한 번에 해결하는 서브쿼리를 정리한다. WHERE·HAVING절에서의 사용법, 단일 행/다중 행 구분, ANY·ALL 연산자, 그리고 NULL이 만드는 함정까지 실습 문제로 다진다."
---

> Oracle SQL/DB 학습 노트 시리즈의 여섯 번째 글이다. 부트캠프에서 정리했던 노트를 다시 다듬어, "값을 모를 때 쿼리 안에서 그 값을 구하는" 서브쿼리를 처음부터 정리한다.

### \[ 목차 ]
- 서브쿼리란 무엇인가
- WHERE절·HAVING절에서의 서브쿼리
- 단일 행 / 다중 행 서브쿼리
- 단일 행 서브쿼리 실습
- 단일 행 서브쿼리에서의 NULL
- 다중 행 서브쿼리 — ANY·ALL
- 다중 행 서브쿼리에서의 NULL
- 종합 실습 — 다시 풀어보기

<br/>

> 참고자료 — [《Do it! 오라클로 배우는 데이터베이스 입문》](https://search.daum.net/search?w=bookpage&bookId=3938139&tab=introduction&DA=LB2&q=Do%20it!%20%EC%98%A4%EB%9D%BC%ED%81%B4%EB%A1%9C%20%EB%B0%B0%EC%9A%B0%EB%8A%94%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%20%EC%9E%85%EB%AC%B8)

<br/>

---

<br/>

### ㅇ 서브쿼리란 무엇인가

다음 문제를 보자.

> Abel이라는 사원보다 급여를 더 많이 받는 사원의 이름과 급여를 출력하시오.

Abel의 급여를 모르니까, 먼저 그 값을 찾아야 한다. 그래서 처음 떠올리는 방식은 SELECT를 두 번 실행하는 것이다.

1. 1차 — `WHERE last_name = 'Abel'`로 Abel의 급여를 조회한다. 결과는 `11000`.
2. 2차 — 그 값을 손으로 옮겨 `WHERE salary > 11000`으로 다시 조회한다.

문장을 두 번 실행해야 하는 불편함이 생긴다. 이걸 **한 번의 문장으로** 처리하는 방법이 바로 **서브쿼리**다.

```sql
SELECT last_name, salary
FROM employees
WHERE salary > ( SELECT salary
                 FROM employees
                 WHERE last_name = 'Abel' );
```

괄호 안의 `SELECT`(서브쿼리)가 먼저 실행되어 `11000`이라는 값을 만들고, 바깥의 `SELECT`(메인쿼리)가 그 값을 받아서 사용한다.

![](/assets/img/posts/sql-subquery-any-all/01.jpeg)

정리하면 서브쿼리의 성격은 이렇다.

- 서브쿼리가 **먼저** 실행되고, 그 결과값을 메인쿼리가 받아서 사용한다.
- 서브쿼리 안에 서브쿼리가 또 들어갈 수 있다. 현업에서는 두 단계 정도 더 타고 들어가기도 한다.
- **조인을 대체**할 수 있다. 특히 셀프조인은 웬만하면 서브쿼리로 풀린다. 조인은 두 테이블을 하나로 매칭하는 작업이 추가되어 성능이 떨어지므로, 서브쿼리로 풀 수 있으면 서브쿼리가 낫다.

> 조인과 정렬은 일종의 **필요악**이다. 어쩔 수 없으면 쓰지만, 안 써도 되면 안 쓰는 편이 성능에 유리하다.

사용할 수 있는 위치도 알아두자. 서브쿼리는 **GROUP BY 절을 제외한** 거의 모든 절에서 쓸 수 있다. 이 글에서는 가장 기본이 되는 **WHERE절·HAVING절**을 다루고, FROM절 서브쿼리는 다음 단계에서 이어간다. SELECT절·ORDER BY절 서브쿼리는 난도가 높아 이 과정에서는 다루지 않는다.

<br/>

---

<br/>

### ㅇ WHERE절·HAVING절에서의 서브쿼리

WHERE절과 HAVING절은 둘 다 **조건문**이라는 공통점이 있다. 구성도 같다.

```
(컬럼 또는 그룹함수) + 연산자 + 값
```

이때 서브쿼리는 맨 오른쪽의 **'값'** 자리를 차지한다. 즉 WHERE·HAVING절에서 서브쿼리는 **값의 역할**을 한다. 특정 값을 모르고 있을 때 그 자리를 서브쿼리로 채우는 것이다.

![](/assets/img/posts/sql-subquery-any-all/02.png)

작성할 때 지켜야 할 지침은 다음과 같다.

- **서브쿼리는 괄호로 묶는다.** 메인쿼리와 구분하고, 하나의 값임을 드러내기 위해서다.
- **비교 연산자의 오른쪽**에 서브쿼리를 둔다. (`컬럼 + 연산자 + 서브쿼리`)
- 실행 측면에서는 넘겨주는 컬럼과 받는 컬럼의 **타입만 같으면** 된다. 컬럼 이름이 달라도 처리는 된다. 다만 보통은 의미가 통하도록 같은 컬럼끼리 비교한다(사원번호 = 급여 같은 비교는 처리는 돼도 의미가 없다).
- **ORDER BY절에는 가급적 서브쿼리를 쓰지 않는다.** ORDER BY는 출력되는 결과값을 정렬하는 역할인데, 서브쿼리는 출력되는 값이 아니라 '사용'되는 값이다. 출력되지도 않는 값을 정렬할 이유가 없다. (단, FROM절 서브쿼리에서는 ORDER BY가 Top-N 분석 등으로 매우 중요해진다.)
- **단일 행 서브쿼리에는 단일 행 연산자를, 다중 행 서브쿼리에는 다중 행 연산자를 쓴다.** 연산자를 짝에 맞게 쓰는 것이 핵심이다. ★

<br/>

---

<br/>

### ㅇ 단일 행 / 다중 행 서브쿼리

서브쿼리는 **돌려주는 결과의 개수**로 나뉜다.

| 구분 | 정의 | 사용 연산자 |
|---|---|---|
| **단일 행 서브쿼리** | 결과를 하나만 돌려줌 | `=`, `<`, `>` 등 단일 행 연산자 |
| **다중 행 서브쿼리** | 결과를 여러 개 돌려줌 | `IN`, `>ANY`, `<ALL` 등 다중 행 연산자 |

더 크게 묶으면, 한 컬럼만 넘기는 **단일 컬럼 서브쿼리**(그 안에 단일 행·다중 행이 포함된다)와, 여러 컬럼을 한 행으로 묶어 넘기는 **다중 컬럼 서브쿼리**로도 나눌 수 있다. 다중 컬럼 서브쿼리는 실무에서 자주 쓰이진 않는다.

서브쿼리를 작성할 때는 늘 다음 세 가지를 순서대로 짚으면 막히지 않는다.

> **서브쿼리 작성 패턴**
> 1. **내가 뭘 모르는가** — 모르는 값을 어떤 컬럼으로 찾을 것인가 (서브쿼리)
> 2. **누가 받는가** — 그 값을 어떤 컬럼이 받아들이는가 (메인쿼리의 비교 대상)
> 3. **어떤 연산자인가** — 단일 행인가 다중 행인가

<br/>

---

<br/>

### ㅇ 단일 행 서브쿼리 실습

**1) 141번 사원과 업무(job_id)가 같은 사원들**

141번의 업무가 무엇인지 모르므로, 그 값을 서브쿼리로 구한다. 141번은 한 명이므로 결과가 하나라 `=`를 쓴다.

```sql
SELECT last_name, job_id
FROM employees
WHERE job_id = ( SELECT job_id
                 FROM employees
                 WHERE employee_id = 141 );
```

**2) 141번과 업무가 같으면서, 143번보다 급여를 많이 받는 사원들**

모르는 값이 둘(141번의 업무, 143번의 급여)이므로 서브쿼리를 두 개 둔다.

```sql
SELECT last_name, job_id, salary
FROM employees
WHERE job_id = ( SELECT job_id
                 FROM employees
                 WHERE employee_id = 141 )
AND salary > ( SELECT salary
               FROM employees
               WHERE employee_id = 143 );
```

**3) 서브쿼리에서 그룹함수 쓰기 — 최소급여를 받는 사원**

WHERE절에는 그룹함수를 직접 쓸 수 없다. 그래서 그룹함수가 만든 값을 조건으로 쓰고 싶을 때 서브쿼리가 제격이다. 회사 전체 최소급여를 서브쿼리로 구해, 그 급여를 받는 사원을 찾는다.

```sql
SELECT last_name, job_id, salary
FROM employees
WHERE salary = ( SELECT MIN(salary)
                 FROM employees );
```

> 이 쿼리는 "부서별 최소급여"가 아니라 **회사 전체의 최소급여**를 받는 사원을 찾는다. 서브쿼리가 값 하나(전체 최소급여)를 돌려주므로 단일 행 서브쿼리다.

**4) HAVING절에서의 서브쿼리 — 50번 부서의 최저급여보다 최저급여가 높은 부서들**

HAVING절도 `그룹함수(컬럼) + 연산자 + 값` 구조이고, 그 '값' 자리에 서브쿼리를 둔다. HAVING절의 서브쿼리가 꼭 그룹함수를 쓸 필요는 없지만, 여기서는 50번 부서의 최저급여를 그룹함수로 구한다.

```sql
SELECT department_id, MIN(salary)
FROM employees
GROUP BY department_id
HAVING MIN(salary) > ( SELECT MIN(salary)
                       FROM employees
                       WHERE department_id = 50 );
```

<br/>

---

<br/>

### ㅇ 단일 행 서브쿼리에서의 NULL

서브쿼리가 값을 **하나도 찾지 못하면**, 연쇄적으로 메인쿼리도 값을 찾지 못한다. 다음 쿼리를 보자.

```sql
SELECT last_name, job_id
FROM employees
WHERE job_id = ( SELECT job_id
                 FROM employees
                 WHERE last_name = 'Haas' );
```

```
no rows selected
```

`Haas`라는 사원이 없으므로 서브쿼리 결과가 비고, 메인쿼리도 `no rows selected`가 된다.

> NULL은 `=`로 찾을 수 없다. (`IS NULL`로만 찾을 수 있다.) 그래서 단일 행 서브쿼리는 **NULL이 나오지 않도록** 신경 써야 한다.

<br/>

---

<br/>

### ㅇ 다중 행 서브쿼리 — ANY·ALL

서브쿼리 결과가 **여러 개** 나오면 단일 행 연산자를 쓸 수 없다. 다음은 잘못된 예다.

```sql
SELECT employee_id, last_name
FROM employees
WHERE salary = ( SELECT MIN(salary)
                 FROM employees
                 GROUP BY department_id );
```

부서별 최소급여이므로 서브쿼리가 여러 값을 돌려준다. 이때 `=`를 쓰면 `single-row subquery returns more than one row` 류의 에러가 난다. 이런 메시지는 그냥 넘기지 말고 읽어야 한다. 에러 메시지에 답이 있다. 여기서는 `=` 대신 `IN`을 써야 한다.

다중 행 연산자에는 `IN` 외에 **ANY**와 **ALL**이 있다. 단일 행 연산자는 값 하나하고만 비교하지만, ANY·ALL은 서브쿼리가 돌려준 **여러 값 전체**를 기준으로 비교한다.

![](/assets/img/posts/sql-subquery-any-all/03.jpeg)

서브쿼리 결과가 `30, 40`이라고 할 때(원본 집합은 10~60), 각 연산자의 의미는 다음과 같다.

| 연산자 | 의미 | 결과 |
|---|---|---|
| `IN` | 결과와 같은 값 | 30, 40 |
| `< ALL` | **전부보다** 작은 값 → 최솟값(30)보다 작은 값 | 10, 20 |
| `> ALL` | **전부보다** 큰 값 → 최댓값(40)보다 큰 값 | 50, 60 |
| `< ANY` | **하나라도보다** 작은 값 → 최댓값(40)보다 작은 값 | 10, 20, 30 |
| `> ANY` | **하나라도보다** 큰 값 → 최솟값(30)보다 큰 값 | 40, 50, 60 |

요령으로 정리하면 **ALL은 최솟값·최댓값을 기준**으로(전부와 비교), **ANY는 반대쪽 극값을 기준**으로(하나라도와 비교) 동작한다. ALL은 서브쿼리 결과 자체를 포함하지 않고, ANY는 포함할 수 있다.

#### 네 연산자로 직접 실습하기

> `IT_PROG` 업무를 하는 사원들의 급여를 기준으로, 급여가 더 작은/큰 사원을 구하여라.

먼저 서브쿼리만 실행해 `IT_PROG`의 급여 분포를 확인해 두면 답을 검산할 수 있다.

| SALARY |
|:---:|
| 9000 |
| 6000 |
| 4800 |
| 4800 |
| 4200 |

**`< ALL` — IT_PROG 최소급여(4200)보다도 작은 사원들**

```sql
SELECT employee_id, last_name, job_id, salary
FROM employees
WHERE salary < ALL ( SELECT salary
                     FROM employees
                     WHERE job_id = 'IT_PROG' );
```

**`> ALL` — IT_PROG 최대급여(9000)보다도 큰 사원들**

```sql
SELECT employee_id, last_name, job_id, salary
FROM employees
WHERE salary > ALL ( SELECT salary
                     FROM employees
                     WHERE job_id = 'IT_PROG' );
```

**`< ANY` — IT_PROG 최대급여(9000)보다 작은 사원들**

```sql
SELECT employee_id, last_name, job_id, salary
FROM employees
WHERE salary < ANY ( SELECT salary
                     FROM employees
                     WHERE job_id = 'IT_PROG' );
```

**`> ANY` — IT_PROG 최소급여(4200)보다 큰 사원들**

```sql
SELECT employee_id, last_name, job_id, salary
FROM employees
WHERE salary > ANY ( SELECT salary
                     FROM employees
                     WHERE job_id = 'IT_PROG' );
```

> `< ANY`는 `MAX`로, `> ANY`는 `MIN`으로 바꿔 풀 수도 있다. 다만 `MAX`/`MIN`은 그룹함수라 계산 작업이 추가되므로, ANY가 성능 면에서 낫다. 그리고 `job_id`는 반드시 `'IT_PROG'`처럼 **언더바까지 정확히** 적어야 한다. `'IT PROG'`로 적으면 서브쿼리가 빈 결과가 되어 의도와 다른 행이 나온다.

<br/>

---

<br/>

### ㅇ 다중 행 서브쿼리에서의 NULL

NULL은 다중 행에서도 함정을 만든다. "매니저가 아닌(누구의 매니저로도 등록되지 않은) 사원"을 찾으려는 의도로 다음을 작성했다고 하자.

```sql
SELECT e.last_name
FROM employees e
WHERE e.employee_id NOT IN ( SELECT m.manager_id
                             FROM employees m );
```

```
no rows selected
```

서브쿼리는 106개의 `manager_id`와 함께 **NULL 하나**를 돌려준다(매니저가 없는 King의 행 때문이다). `NOT IN`은 "이 집합에 없는 값을 찾아라"는 뜻인데, 집합 안에 NULL이 섞이면 비교 자체가 불능이 되어 아무 행도 나오지 않는다.

![](/assets/img/posts/sql-subquery-any-all/04.jpg)

> 서브쿼리가 값을 넘길 때, 그 안의 NULL을 `IN`/`NOT IN`으로 걸러내는 것은 불가능하다. ★ NULL을 찾고 싶다면 `IS NULL`을 써야 한다.

<br/>

---

<br/>

### ㅇ 종합 실습 — 다시 풀어보기

수업에서 푼 문제들을 혼자 다시 풀며 자주 했던 실수를 함께 정리한다.

**25) Zlotkey와 같은 부서에 근무하는 *다른* 모든 사원들의 사번·고용일자**

```sql
SELECT employee_id, hire_date
FROM employees
WHERE department_id IN ( SELECT department_id
                         FROM employees
                         WHERE last_name = 'Zlotkey' )
AND last_name != 'Zlotkey';
```

- "**다른** 사원"이므로 Zlotkey 본인을 빼야 한다 → `AND last_name != 'Zlotkey'`.
- Zlotkey가 여러 명일 수 있으므로 `=`가 아니라 `IN`을 쓴다. (이 데이터에서는 우연히 한 명이라 `=`로도 에러는 안 났지만, 습관적으로 다중 행 연산자를 쓰는 게 안전하다.)
- 흔한 실수 — 서브쿼리의 마지막 괄호 `)`를 닫지 않으면 `ORA-00907: missing right parenthesis`가 난다.

**26) 회사 전체 평균급여보다 더 많이 받는 사원들의 사번·이름**

```sql
SELECT employee_id, last_name
FROM employees
WHERE salary > ( SELECT AVG(salary)
                 FROM employees );
```

- 출력할 것(사번·이름)은 메인쿼리 SELECT에, 모르는 값(평균급여)은 서브쿼리에 둔다.
- WHERE절에 그룹함수를 직접 못 쓰니 서브쿼리로 빼는 패턴이다.
- 평균값은 하나이므로 단일 행 서브쿼리다. `ALL`은 필요 없다(GROUP BY가 없으므로).

**27) 이름에 U가 들어가는 사원들과 같은 부서에 근무하는 사원들의 사번·이름**

```sql
SELECT employee_id, last_name
FROM employees
WHERE department_id IN ( SELECT department_id
                         FROM employees
                         WHERE last_name LIKE '%U%'
                            OR last_name LIKE '%u%' );
```

- 부분 문자열은 `=`가 아니라 `LIKE`로 찾는다. `last_name = '%u%'`로 쓰면 그런 이름은 없으니 `no rows selected`다.
- 해당 부서가 여러 개일 수 있으므로 `IN`을 쓴다.

**28) Seattle에 근무하는 사원 중 커미션을 받지 않는 사원들의 이름·부서명·지역번호**

도시 이름 `Seattle`은 `locations` 테이블의 `city` 컬럼에 있다. 조인으로 풀면 다음과 같다.

```sql
SELECT e.last_name, d.department_name, d.location_id
FROM employees e, departments d, locations l
WHERE e.department_id = d.department_id
AND d.location_id = l.location_id
AND l.city = 'Seattle'
AND e.commission_pct IS NULL;
```

서브쿼리를 배웠으니, `locations`까지 조인하지 않고 Seattle의 지역번호만 서브쿼리로 구해 `departments`에 연결할 수 있다.

```sql
SELECT e.last_name, d.department_name, d.location_id
FROM employees e, departments d
WHERE e.department_id = d.department_id
AND d.location_id = ( SELECT location_id
                      FROM locations
                      WHERE city = 'Seattle' )
AND e.commission_pct IS NULL;
```

- 서브쿼리 안의 `Seattle`은 메인쿼리의 테이블에서 가져온 게 아니므로, 조인 접두어를 붙일 필요가 없다.
- 자주 했던 실수 두 가지 — 테이블 이름은 `department`가 아니라 **`departments`**(`ORA-00942`), 그리고 **조인 조건**(`e.department_id = d.department_id`)을 빼먹지 말 것(★). `department_name`은 `employees`가 아닌 `departments`에 있으니 접두어도 정확히 붙여야 한다(`ORA-00904`).

**29) Davies보다 나중에 고용된 사원들의 이름·고용일자 (고용일자 역순)**

```sql
SELECT last_name, hire_date
FROM employees
WHERE hire_date > ALL ( SELECT hire_date
                        FROM employees
                        WHERE last_name = 'Davies' )
ORDER BY hire_date DESC;
```

- Davies가 여러 명일 수 있으므로 단일 행 연산자 `>`가 아니라 **`> ALL`**을 써야 한다. "모든 Davies보다 나중"이라는 의미가 정확히 살아난다.
- 이름은 전체를 이미 알고 있으니 `LIKE`로 검색할 필요 없이 `=`로 충분하다.

**30) King을 매니저로 두고 있는 모든 사원들의 이름·급여**

```sql
SELECT last_name, salary
FROM employees
WHERE manager_id IN ( SELECT employee_id
                      FROM employees
                      WHERE last_name = 'King' );
```

- "King을 매니저로 둔다" = King의 사원번호(`employee_id`)를 자신의 `manager_id`로 가진다는 뜻이다. 넘기는 컬럼(`employee_id`)과 받는 컬럼(`manager_id`)의 **이름은 다르지만 타입이 같아** 정상 동작한다.
- King이 여러 명일 수 있으므로 `IN`을 쓴다.

**31) 회사 평균급여보다 많이 받으면서 이름에 u가 든 사원이 근무하는 부서의 모든 사원들의 사번·이름·급여**

문제를 단계로 풀면 ① 평균보다 많이 받고 ② 이름에 u가 든 사원을 찾고 → ③ 그 사원들이 근무하는 부서를 구해 → ④ 그 부서의 모든 사원을 출력한다.

```sql
SELECT employee_id, last_name, salary
FROM employees
WHERE department_id IN ( SELECT department_id
                         FROM employees
                         WHERE salary > ( SELECT AVG(salary)
                                          FROM employees )
                         AND last_name LIKE '%u%' );
```

- 평균급여는 그냥 값 하나이므로 `( SELECT AVG(salary) FROM employees )`로 충분하다. 여기에는 **GROUP BY가 필요 없다.** `AVG`가 보이면 반사적으로 GROUP BY를 붙이기 쉬운데, 회사 전체 평균 하나만 필요한 경우엔 쓰지 않는다.
- 코드를 읽을 때는 **안쪽 서브쿼리부터** 읽는다. 서브쿼리가 값의 역할을 하고, 그 값을 근거로 바깥 WHERE절이 실행되기 때문이다.
- 조건의 **위치**가 의미를 바꾼다. `salary > ...`와 `last_name LIKE '%u%'`를 같은 서브쿼리 안에 두면 "평균보다 많이 받으면서 이름에 u가 든 사원이 속한 부서"를 구하지만, `last_name` 조건을 바깥 메인쿼리로 빼면 "그 부서들 중 이름에 u가 든 사원만" 출력하는 전혀 다른 쿼리가 된다.

<br/>

---

<br/>

여기까지가 서브쿼리의 기본기다. 핵심은 늘 같다 — **모르는 값을 괄호 안 SELECT로 구하고**, 그 결과가 하나면 단일 행 연산자, 여럿이면 `IN`·`ANY`·`ALL`을 쓰고, **NULL을 조심하는 것**.

다음 글에서는 검색(SELECT)을 넘어 데이터를 직접 바꾸는 **DML(INSERT·UPDATE·DELETE)**을 다룬다.
