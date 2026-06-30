---
title: "GROUP BY와 HAVING으로 집계하기"
date: 2025-02-10 21:30:00 +0900
categories: [CS, Database]
tags: [Database, SQL, Oracle, GROUP BY, HAVING, 집계함수]
description: "행을 묶어 집계하는 GROUP BY와 그 그룹에 조건을 거는 HAVING을 정리하고, WHERE와 HAVING의 차이·SELECT 실행 순서를 실습 문제로 익혔다."
---

> Oracle SQL/DB 학습 노트 시리즈의 다섯 번째 글이다. 그룹 함수로 값을 하나로 모으는 것을 넘어, 행을 **그룹 단위**로 묶어 집계하는 `GROUP BY`와 그 그룹에 조건을 거는 `HAVING`을 정리한다.

### \[ 목차 ]
- 왜 GROUP BY가 필요한가 — 그룹 함수의 한계
- GROUP BY 절
- HAVING 절
- WHERE와 HAVING — 두 조건절의 차이
- SELECT 문의 실행 순서
- 실습 — 집계 쿼리 만들어 보기

<br/>

---

<br/>

### ㅇ 왜 GROUP BY가 필요한가 — 그룹 함수의 한계

부서별 급여의 총합을 구하고 싶다. 즉, 결과를 **각 부서별로** 보고 싶다. 부서 번호를 함께 보려고 `department_id`를 SELECT 절에 적고 `SUM(salary)`를 실행하면 오류가 난다.

```sql
SELECT department_id, SUM(salary)
FROM employees;

ERROR at line 1:
ORA-00937: not a single-group group function
```

원인은 SELECT 절의 `department_id` 컬럼에 있다. 그룹 함수는 **그룹 당 하나의 결과**를 내는데, `department_id`는 그룹화되어 있지 않고 여전히 행 단위로 흩어져 있다. 하나로 모인 `SUM(salary)`과 여러 행짜리 `department_id`를 한 줄에 나란히 둘 수 없어 에러가 발생한 것이다.

부서별로 합을 구하려면 10번은 10번끼리, 20번은 20번끼리 미리 모아 둬야 한다. 즉, **그룹화**를 시켜 주자.

<br/>

---

<br/>

### ㅇ GROUP BY 절

`GROUP BY`는 그룹화되지 않은 컬럼들을 기준 컬럼별로 묶어 준다.

```sql
SELECT (그룹화해서 보여주고 싶은) 컬럼, 그룹함수(함수 실행할 컬럼)
FROM   컬럼을 소유한 테이블
WHERE  ...
GROUP BY 그룹화할 열을 지정
ORDER BY ...
```

규칙은 공식처럼 외워 두면 좋다.

- SELECT 리스트 절에서 **그룹 함수가 사용되지 않은 컬럼**들은 반드시 `GROUP BY` 절로 그룹화해 줘야 한다.
- 위치는 `WHERE` 절과 `ORDER BY` 절 **사이**에만 둔다.
- 그룹화할 열은 되도록 SELECT 리스트 절에도 명시한다. 표기하지 않아도 그룹화는 실행되지만, 무슨 기준으로 나온 결과인지 알 수 없으므로 적어 두는 편이 좋다. (`WHERE`·`ORDER BY` 절에서 그랬듯이)

<br/>

> **DISTINCT로 그룹화하면 안 될까?**
> `DISTINCT`와 `ORDER BY` 절도 내부적으로 그룹화 작업을 하긴 한다. 다만 그 뒤의 **목적**이 다르다.
> - **DISTINCT** : 똑같은 값끼리 모아 그중 하나를 도출(중복 제거)
> - **ORDER BY** : 똑같은 값끼리 모아 정렬해서 출력
> - **GROUP BY** : 그룹화해서 그룹 함수의 **인수**로 사용

<br/>

#### 여러 열에 GROUP BY 사용

기준 컬럼을 여러 개 줄 수 있다. 첫 번째 컬럼으로 먼저 그룹화하고, 그 안에서 두 번째 컬럼으로 다시 그룹화한 뒤 그룹 함수를 적용한다. 이때 **순서의 기준은 SELECT 절**이다(`GROUP BY` 절에 적는 순서는 상관없다).

```sql
-- 부서별, 그 안에서 직업별 급여 총합
SELECT department_id dept_id, job_id, SUM(salary)
FROM   employees
GROUP BY department_id, job_id
ORDER BY department_id;
```

<br/>

---

<br/>

### ㅇ HAVING 절

이번엔 **평균 급여가 8000보다 큰 부서**만 출력해 보자. 조건이 생겼으니 익숙한 `WHERE` 절에 넣으면 또 오류가 난다.

```sql
SELECT department_id, AVG(salary)
FROM   employees
WHERE  AVG(salary) > 8000
GROUP BY department_id;

ERROR at line 3:
ORA-00934: group function is not allowed here
```

`WHERE` 절로는 그룹을 제한할 수 없다. `WHERE`가 **행을 먼저 필터링**하는 절이기 때문이다. 그룹 함수의 값은 그룹화가 끝난 뒤에야 만들어지는, 테이블에 원래 존재하지 않는 **변형된 값**이라 `WHERE` 단계에서는 아직 존재하지도 않는다.

![](/assets/img/posts/sql-groupby-having/01.png)

그래서 그룹에 조건을 줄 때는 `WHERE`가 아니라 `HAVING`을 쓴다.

- 그룹에 대한 조건을 주고자 할 때 사용한다.
- `GROUP BY` 절과 `HAVING` 절은 순서가 바뀌어도 된다. `WHERE` 절과 `ORDER BY` 절 사이에만 있으면 된다. 다만 가독성을 위해 `GROUP BY`를 먼저 적기를 권한다.

```sql
-- 부서별 최대 급여를 구하되, 최대 급여가 10000보다 큰 부서만 도출
SELECT department_id, MAX(salary)
FROM   employees
GROUP BY department_id
HAVING MAX(salary) > 10000;
```

다음 쿼리는 무엇을 의미할까? `WHERE`가 먼저 실행되므로 직업명에 `REP`가 들어가지 않는 사원들만 추린 뒤, 직업별 급여 총합을 구하고, 그 총합이 13000보다 큰 직업만 출력한다.

```sql
SELECT job_id, SUM(salary)
FROM   employees
WHERE  job_id NOT LIKE '%REP%'
GROUP BY job_id
HAVING SUM(salary) > 13000
ORDER BY SUM(salary);
```

<br/>

---

<br/>

### ㅇ WHERE와 HAVING — 두 조건절의 차이

SELECT 문에는 조건을 거는 절이 두 가지 있다. 둘을 헷갈리지 않게 정리해 두자.

| 구분 | WHERE | HAVING |
|---|---|---|
| 대상 | 테이블에 있는 **행** | 그룹 함수로 도출된 **그룹** |
| 역할 | 행을 제한 / 행에 조건 | 그룹을 제한 / 그룹에 조건 |
| 사용 형태 | 컬럼 + 연산자 + 값 | 그룹 함수 + 연산자 + 값 |

- **`WHERE` 절에 그룹 함수를 쓰면 안 된다.** `GROUP BY` 절보다 `WHERE`가 먼저 실행되기 때문이다.
- **`HAVING` 절에는 일반 조건을 웬만하면 쓰지 말자.** 처리는 되지만 성능이 떨어진다. SELECT 절에서 값이 변형되어 떨어진 뒤 `HAVING`이 다시 역산으로 수행되기 때문이다. 행 단위 조건은 `WHERE`에서 미리 거르는 편이 낫다.

<br/>

---

<br/>

### ㅇ SELECT 문의 실행 순서

지금까지의 오류들은 모두 **실행 순서**로 설명된다. SELECT 문은 적는 순서와 실제 처리 순서가 다르다.

> `FROM` → `WHERE` → `GROUP BY` & `HAVING` → `SELECT` → `ORDER BY`

이 순서를 알면 ALIAS의 사용 범위도 자연스럽게 이해된다.

- `FROM` 절에서 **테이블**에 ALIAS를 주면, 그 뒤의 모든 절에서 접두어로 사용할 수 있다. (`FROM`이 가장 먼저 처리되므로)
- `SELECT` 절에서 **컬럼**에 ALIAS를 주면, `ORDER BY` 절에서만 사용할 수 있다. (`SELECT`가 `ORDER BY` 직전에 처리되므로)

<br/>

---

<br/>

### ㅇ 실습 — 집계 쿼리 만들어 보기

배운 내용을 문제로 굳혀 보자. 각 문제는 *문제 → 쿼리 → 풀이* 순서다.

#### 1) 자신의 매니저보다 먼저 입사한 사원

자신의 매니저보다 먼저 고용된 사원의 이름과 고용일을 출력한다.

```sql
SELECT e.last_name, e.hire_date
FROM   employees e, employees m
WHERE  e.manager_id = m.employee_id
AND    e.hire_date  < m.hire_date;
```

employees 테이블을 사원(`e`)과 매니저(`m`)로 두 번 부르는 셀프 조인이다. `e.manager_id = m.employee_id`로 사원과 그 매니저를 연결한 뒤, 내 입사일이 매니저 입사일보다 빠른 행(`e.hire_date < m.hire_date`)만 추린다.

<br/>

#### 2) 회사 전체의 급여 통계

회사 전체의 최대·최소·총합·평균 급여를 출력한다. 그룹 기준이 없으므로 `GROUP BY` 없이 전체를 한 그룹으로 집계한다.

```sql
SELECT MAX(salary), MIN(salary), SUM(salary), AVG(salary)
FROM   employees;
```

<br/>

#### 3) 직업별 급여 통계

각 직업별 최대·최소·총합·평균 급여를 출력하되, 컬럼 머리글을 각각 `MAX`·`MIN`·`SUM`·`AVG`로 하고 직업 기준 오름차순으로 정렬한다.

```sql
SELECT job_id,
       MAX(salary) MAX, MIN(salary) MIN,
       SUM(salary) SUM, AVG(salary) AVG
FROM   employees
GROUP BY job_id
ORDER BY job_id;
```

"최대 급여는 MAX로 출력"이라는 문장은 머리글을 ALIAS로 `MAX`라 붙이라는 뜻이다. 문제는 곡해하지 말고 그대로 읽자.

<br/>

#### 4) 동일한 직업을 가진 사원 수

직업이 같은 사원이 각각 몇 명인지 출력한다.

```sql
SELECT job_id, COUNT(employee_id)
FROM   employees
GROUP BY job_id;
```

직업이 그룹의 기준이 되고, 그 그룹 안의 행 수를 세야 하므로 `COUNT`를 쓴다. 셀 컬럼으로는 중복이 없는 PK인 `employee_id`를 넣는 편이 안전하다. 이름 컬럼은 동명이인으로 중복될 수 있다.

<br/>

#### 5) 매니저로 근무하는 사원 수

누군가의 매니저로 등록된 사원이 모두 몇 명인지 출력한다.

```sql
SELECT COUNT(DISTINCT manager_id)
FROM   employees;
```

`manager_id`에 등장하는 값이 곧 매니저다. 그런데 한 매니저는 여러 사원의 `manager_id`에 반복해서 나타나므로 `DISTINCT`로 중복을 제거해 센다. `COUNT`는 NULL을 자동으로 무시하므로 매니저가 없는 최상위 사원은 자연히 빠진다.

<br/>

#### 6) 최대 급여와 최소 급여의 차이

사내 최대 급여와 최소 급여의 차이를 출력한다.

```sql
SELECT MAX(salary) - MIN(salary)
FROM   employees;
```

그룹 함수끼리도, 또 그룹 함수와 상수 사이에도 산술 연산이 가능하다.

<br/>

#### 7) 매니저별 부하 사원의 최소 급여

매니저의 사번과, 그 매니저 밑 사원 중 최소 급여를 출력한다. 매니저가 없는 사원은 제외하고, 최소 급여가 5000 미만인 그룹도 제외하며, 급여 기준 역순으로 정렬한다.

```sql
SELECT manager_id, MIN(salary)
FROM   employees
WHERE  manager_id IS NOT NULL
GROUP BY manager_id
HAVING MIN(salary) >= 5000
ORDER BY MIN(salary) DESC;
```

조건이 두 종류라는 점이 핵심이다.

- "매니저 없는 사람 제외"는 **행 단위 일반 조건**이라 `WHERE manager_id IS NOT NULL`로 처리한다.
- "최소 급여 5000 미만 제외"는 **그룹 단위 조건**이라 `HAVING MIN(salary) >= 5000`으로 처리한다.

여기서 흔히 두 가지 실수를 한다. 첫째, `WHERE salary > 5000`처럼 급여 조건을 `WHERE`에 두면 그룹을 만들기도 전에 행이 빠져 결과가 달라진다. 둘째, `ORDER BY salary DESC`처럼 원본 컬럼으로 정렬하려 하면 안 된다. SELECT 리스트에 명시된 것은 이미 `MIN(salary)`로 변형된 값이라, 정렬도 `MIN(salary)` 기준으로 해야 한다.

<br/>

#### 8) 부서별 사원 수와 평균 급여

부서명·부서 위치 ID, 각 부서별 사원 총 수와 평균 급여를 출력하되 부서 위치 오름차순으로 정렬한다. 부서명·위치는 departments에, 사원 수·급여는 employees에 있으므로 두 테이블을 조인한다.

```sql
SELECT d.department_name, d.location_id,
       COUNT(e.employee_id), AVG(e.salary)
FROM   departments d, employees e
WHERE  d.department_id = e.department_id
GROUP BY d.department_name, d.location_id
ORDER BY d.location_id;
```

조인 조건(`WHERE d.department_id = e.department_id`)을 빠뜨리면 카테시안 곱이 생기니 반드시 넣는다. 그리고 두 테이블에 같은 이름의 컬럼이 있을 수 있으므로, 조인 환경에서는 `e.employee_id`·`e.salary`처럼 **테이블 ALIAS 접두어**를 붙여 어느 테이블의 컬럼인지 명확히 한다. `FROM` 절에서 ALIAS를 줬으니 다른 절에서 모두 접두어로 활용할 수 있다.

<br/>

---

<br/>

> **한 줄 정리**
> `GROUP BY`는 행을 그룹으로 묶어 집계하고, SELECT 절의 비집계 컬럼은 반드시 그룹화해야 한다. 그룹에 거는 조건은 `WHERE`가 아니라 `HAVING`을 쓴다. 모든 차이는 `FROM → WHERE → GROUP BY·HAVING → SELECT → ORDER BY`라는 실행 순서에서 비롯된다.

다음 글에서는 SELECT의 꽃이라 불리는 **서브쿼리**를 다룬다. 쿼리 안에 또 다른 쿼리를 넣어, `WHERE`에서 그룹 함수를 쓰지 못했던 한계까지 넘어서 본다.
</content>
</invoke>
