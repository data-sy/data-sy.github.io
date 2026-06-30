---
title: "WHERE 조건절과 ORDER BY 정렬: 행을 거르고 줄 세우기"
date: 2025-02-06 21:30:00 +0900
categories: [CS, Database]
tags: [Database, SQL, Oracle, WHERE, 연산자, ORDER BY, 정렬]
description: "WHERE 절로 행을 거르는 원리부터 비교·BETWEEN·IN·IS NULL·LIKE·논리 연산자와 우선순위, 그리고 ORDER BY로 결과를 정렬하는 방법까지 SELECT의 핵심 조건·정렬절을 예제와 함께 정리했다."
---

> Oracle SQL/DB 학습 노트 시리즈의 세 번째 글이다. SELECT의 기본기에 이어, 이번에는 **원하는 행만 골라내는 WHERE 절**과 **결과를 줄 세우는 ORDER BY 절**을 정리한다. WHERE 절에서 쓰는 연산자 종류가 많지만, "조건식 = 컬럼 + 연산자 + 값"이라는 한 가지 원칙으로 모두 꿰뚫을 수 있다.

### \[ 목차 ]
- WHERE 절 — 행을 제한하는 절
- 비교 연산자
- BETWEEN — 범위로 찾기
- IN — 여러 값을 한 번에
- IS NULL — 빈 값 찾기
- LIKE — 일부 문자열로 찾기
- 논리 연산자 — AND·OR·NOT
- 연산자 우선순위
- ORDER BY — 결과 정렬
- 연산자 선택 가이드
- 종합 실습

<br/>

---

<br/>

### ㅇ WHERE 절 — 행을 제한하는 절

WHERE 절은 **조건에 맞는 행만 골라내는 절**이다. 조건식은 항상 `컬럼 + 연산자 + 값` 세 부분으로 이루어지고, 연산자가 컬럼과 값을 비교해 준다.

```sql
SELECT 컬럼
FROM   테이블
WHERE  조건식   -- 컬럼 + 연산자 + 값
```

![](/assets/img/posts/sql-where-and-orderby/01.jpeg)

WHERE 절은 항상 FROM 절 다음에 위치한다. 중요한 것은 **절의 실행 순서**다. SQL은 적힌 순서가 아니라 다음 순서로 실행된다.

- **1) FROM 절** : 물건을 찾으려면 먼저 물건이 있는 창고로 가야 한다 → 대상 테이블을 정한다
- **2) WHERE 절** : 조건에 맞는 **행**들만 가져온다(필터링)
- **3) SELECT 절** : 걸러진 행 안에서 원하는 컬럼을 뽑는다

즉 WHERE 절이 SELECT 절보다 **먼저** 실행되어 행을 제한한 뒤, 그 결과에 SELECT가 수행된다.

<br/>

#### 값을 쓸 때 지켜야 할 것

- **문자·날짜에는 작은따옴표** `' '`를 붙인다. 컬럼 이름·테이블 이름과 구분하기 위해서다.
- **대소문자를 구분한다.** 연산자는 값을 비교할 때 아스키코드를 사용하므로, **저장된 포맷 그대로** 써야 한다. 따라서 내가 가진 데이터의 형식(예: 날짜가 연·월·일인지 일·월·연인지)을 사전에 정확히 파악해야 한다.

<br/>

#### WHERE 절에는 ALIAS(별칭)를 쓸 수 없다

WHERE 절이 SELECT 절보다 먼저 실행되기 때문이다. ALIAS는 테이블의 컬럼 이름이 아니라, **출력될 때 보이는 열 머리글을 바꿔 주는 장치**다. WHERE 절이 돌아가는 시점에는 SELECT 절(별칭이 붙는 곳)이 아직 실행되지 않았으므로 별칭의 존재를 알 수 없다. 그래서 WHERE 절에는 반드시 **원래 컬럼 이름**을 써야 한다.

> 권장 사항: WHERE 절에 명시한 컬럼은 가급적 SELECT 리스트에도 함께 넣자. 그래야 결과만 보고도 어떤 조건으로 걸렀는지 알 수 있다. SELECT에 넣지 않으면 코드를 봐야만 결과를 해석할 수 있다.

<br/>

#### 첫 예제

90번 부서에 근무하는 사원의 이름과 급여를 출력해 보자. 풀이는 항상 같은 순서로 생각한다.

- SELECT 리스트 찾기 → `last_name, salary`
- 어디서 찾을지 → `employees`
- 어떤 조건인지 → 부서번호가 90

```sql
SELECT last_name, salary
FROM   employees
WHERE  department_id = 90;
```

값의 대소문자가 왜 중요한지 보여 주는 예제도 있다.

```sql
SELECT last_name, job_id, department_id
FROM   employees
WHERE  last_name = 'Whalen';
```

`last_name`은 첫 글자만 대문자이고 나머지는 소문자인 형식으로 저장돼 있다. `=` 연산자는 저장된 포맷을 그대로 비교하므로, `'whalen'`처럼 전부 소문자로 검색하면 `no rows selected`가 나온다.

<br/>

---

<br/>

### ㅇ 비교 연산자

값 하나를 비교하는 **단일 연산자**다. `=`, `>`, `>=`, `<`, `<=`, 그리고 "같지 않음"을 뜻하는 `<>`가 있다.

![](/assets/img/posts/sql-where-and-orderby/02.png)

- 단일 연산자이므로 값을 하나만 비교한다. `department_id = 90, 16`처럼 여러 값을 한 번에 넣을 수 없다.
- "같지 않음"은 `<>` 외에 `!=`로도 쓴다. 성능상으로는 `!=`가 살짝 더 빠르다. `<>`는 "크다 또는 작다"를 합친 비교인 반면, `!=`는 곧바로 부정을 뜻하기 때문이다.

```sql
WHERE salary >= 6000
WHERE last_name = 'Smith'
WHERE hire_date = '03/05/23'
```

#### 실습 — 연봉 조건과 별칭

연봉이 150000 이상인 사원의 이름과 연봉을 구하되, 이름은 `NAME`, 연봉은 `AnnSal`로 출력하라.

```sql
SELECT last_name AS name, salary*12 "AnnSal"
FROM   employees
WHERE  salary*12 >= 150000;
```

여기서 자주 하는 실수가 있다.

- WHERE 절에 별칭 `AnnSal`을 쓰면 안 된다. WHERE는 SELECT보다 먼저 실행되므로 별칭을 모른다. 그래서 조건에는 `salary*12`라고 원래 식을 그대로 써야 한다.
- `AnnSal`처럼 **대소문자를 살리고 싶은 별칭**은 큰따옴표 `" "`로 감싼다. 별칭은 기본적으로 대문자로 출력되기 때문이다. 반대로 `name`은 큰따옴표 없이 쓰면 어차피 대문자 `NAME`으로 나오므로, 대문자 출력만 원한다면 굳이 `"NAME"`처럼 감쌀 필요가 없다.

작업 순서는 SELECT → FROM → WHERE를 먼저 적은 뒤, 다시 SELECT로 올라가 별칭을 붙이는 식으로 하면 깔끔하다.

<br/>

---

<br/>

### ㅇ BETWEEN — 범위로 찾기

`BETWEEN A AND B`는 두 값 사이의 **범위**를 찾을 때 쓴다. 지정한 양 끝 값을 **포함**한다.

![](/assets/img/posts/sql-where-and-orderby/03.jpeg)

- 아스키코드로 비교하므로 문자에도 쓸 수 있지만, 실제로는 숫자와 날짜에서 더 많이 쓴다.
- 문법은 반드시 `컬럼 BETWEEN A AND B` 형태여야 한다. `150000 < salary*12 < 200000` 같은 수학식 표기는 오류다(`조건식 = 컬럼 + 연산자 + 값`이라는 원칙을 떠올리자).

```sql
SELECT last_name, salary
FROM   employees
WHERE  salary BETWEEN 2500 AND 3500;
```

날짜도 같은 방식으로 범위를 잡는다.

```sql
WHERE hire_date BETWEEN '2020/01/01' AND '2020/05/01'
```

<br/>

---

<br/>

### ㅇ IN — 여러 값을 한 번에

`IN`은 `=`의 확장판이다. `=`가 값 하나만 비교하는 단일 연산자라면, `IN`은 **여러 값을 동시에** 비교하는 복수 연산자다.

<div style="display:flex; gap:8px;">
<div style="flex:1;"><img src="/assets/img/posts/sql-where-and-orderby/04.jpeg"></div>
<div style="flex:1;"><img src="/assets/img/posts/sql-where-and-orderby/05.jpeg"></div>
</div>

`OR`를 여러 번 반복하는 것보다 `IN` 하나를 쓰는 편이 좋다.

```sql
SELECT employee_id, last_name, salary, manager_id
FROM   employees
WHERE  manager_id IN (100, 101, 201);
```

이 쿼리는 "매니저 아이디가 100 또는 101 또는 201인 사원"을 찾는다. `manager_id`는 나를 관리하는 관리자의 사원번호이므로, 의미상 "100·101·201번에게 보고하는 사원들의 정보를 보자"가 된다. 모든 데이터 타입에 쓸 수 있고, 문자나 날짜는 작은따옴표로 감싼다.

<br/>

---

<br/>

### ㅇ IS NULL — 빈 값 찾기

NULL(비어 있는 값)을 찾을 때 쓴다. NULL은 값이 아니라 "비어 있음"이므로 `= NULL`로는 찾을 수 없고, 전용 연산자 `IS NULL`을 써야 한다.

```sql
SELECT last_name, manager_id
FROM   employees
WHERE  manager_id IS NULL;
```

`manager_id`가 없다는 것은 곧 **관리자가 없는 사람**, 즉 조직의 최상위 한 명만 나온다는 뜻이다.

```sql
SELECT last_name, job_id, commission_pct
FROM   employees
WHERE  commission_pct IS NULL;
```

`commission_pct`가 NULL이면 **보너스를 받지 않는 사람**을 찾는 것이다.

<br/>

---

<br/>

### ㅇ LIKE — 일부 문자열로 찾기

`LIKE`는 일부 문자열이 포함된 데이터를 조회할 때 쓰는 연산자로, **와일드카드** `%`와 `_`를 함께 사용한다. 가장 헷갈리기 쉬운 연산자 중 하나다.

![](/assets/img/posts/sql-where-and-orderby/06.jpeg)

핵심은 와일드카드가 **`LIKE` 안에서만** 특수 문자로 작동한다는 점이다. 다른 연산자와 쓰면 그냥 보통 문자로 인식된다. 예를 들어 "이씨 성을 가진 자료를 찾아라"에서 `이름 = '이%'`라고 쓰면 오답이다. `=`는 값을 그대로 비교하므로, 글자 그대로 "이름이 `이%`인 사람"을 찾은 셈이 된다.

<br/>

#### % — 길이에 상관없는 모든 문자

`%`는 문자가 없는 경우까지 포함해, 길이에 상관없이 **모든 문자 데이터**를 의미한다.

```
'S%'   : S로 시작하는 것을 모두 찾기
'%S'   : S로 끝나는 것을 모두 찾기
'%S%'  : S를 포함하는 것을 모두 찾기
```

여기서도 **테이블에 저장된 포맷**을 정확히 알아야 한다. 예를 들어 날짜 `03/06/17`은 2003년 06월 17일을 의미하고, `/`는 날짜 포맷의 구분 옵션일 뿐이다. 그래서 `2006`을 넣으면 `20/06`(20월 06일)처럼 엉뚱하게 해석된다.

"2006년에 입사한 사원의 이름과 입사일 구하기"를 예로 들면 이렇게 갈린다.

- `'%2006'` → `20/06`으로 끝나는, 즉 20월 06일 입사 → `no rows selected`
- `'%06'` → 06으로 끝나는, 즉 06일 입사
- `'06%'` → 06으로 시작하는, 즉 **06년도 입사** → 정답

```sql
SELECT last_name, hire_date
FROM   employees
WHERE  hire_date LIKE '06%';
```

<br/>

#### _ (언더바) — 자리 하나를 차지하는 한 문자

`_`는 어떤 값이든 상관없이 **한 개의 문자**를 의미한다. 자리 수를 정확히 차지한다는 점이 `%`와 다르다.

```
'_o%'  : 앞에서 두 번째 글자가 o인 것을 모두 찾기
'__a%' : 앞에서 세 번째 글자가 a인 것을 모두 찾기
'%a_'  : 뒤에서 두 번째 글자가 a인 것을 모두 찾기
'b_'   : 두 글자짜리 중 첫 글자가 b인 것을 찾기
```

`'_o'`라고만 쓰면 정확히 두 글자짜리만 찾으므로, 뒤에 다른 글자가 더 오는 단어까지 찾으려면 `'_o%'`처럼 `%`를 붙여야 한다.

<br/>

---

<br/>

### ㅇ 논리 연산자 — AND·OR·NOT

조건이 여러 개일 때 WHERE 절에 조건을 이어 붙이는 연산자다.

#### AND — 모든 조건을 만족

- 조건을 **전부** 만족하는 행만 출력한다.
- 가장 많이 틀리는 부분: `AND` 뒤에는 반드시 **새로운 조건식 전체**를 다시 써야 한다. `did = 90 AND 50`(X)이 아니라 `did = 90 AND did = 50`(O)처럼 `컬럼 + 연산자 + 값`을 온전히 반복해야 한다.

"90번 부서에 근무하면서 급여가 5000 이상인 사람"은 이렇게 쓴다.

```sql
WHERE department_id = 90
AND   salary >= 5000
```

#### OR — 하나라도 만족

- 조건을 **하나라도** 만족하면 출력한다. 사용법은 AND와 같다.

"급여가 10000 이상이거나, 업무 이름에 man이 들어가는 사람"은 다음과 같다.

```sql
WHERE salary >= 10000
OR    job_id LIKE '%man%'
```

#### NOT — 부정

부정하는 연산자로, **연산자 앞**에 붙인다. 단, NULL의 경우에는 `IS NOT NULL`처럼 NULL 앞에 붙인다.

```sql
WHERE 컬럼 NOT IN (~, ~, ~)        -- 나열한 값이 아닌 것 찾기
WHERE 컬럼 NOT BETWEEN A AND B     -- 이상·이하를 부정 → 미만·초과
WHERE 컬럼 NOT LIKE '%A%'          -- A를 포함하지 않는 것 찾기
WHERE 컬럼 IS NOT NULL             -- 값이 있는 것 찾기
```

`=`, `<` 같은 단일 비교 연산자 앞에는 NOT을 쓰지 않는다. 이미 `<>`, `!=`라는 부정 연산자가 따로 있기 때문이다. 굳이 쓴다면 `WHERE NOT department_id = 90`처럼 **컬럼 앞에 붙여 조건 전체를 부정**하는 형태가 되는데, 오라클 전용 문법이 아니라 지양한다.

```sql
SELECT last_name, job_id
FROM   employees
WHERE  job_id NOT IN ('IT_PROG', 'ST_CLERK', 'SA_REP');
```

> `NOT IN`은 나열한 값이 **모두 아니어야** 한다. 드모르간 법칙으로 `¬(p ∨ q) = ¬p ∧ ¬q`이듯, `NOT IN (A, B, C)`는 "A도 아니고 B도 아니고 C도 아니다"라는 뜻이다.

<br/>

---

<br/>

### ㅇ 연산자 우선순위

여러 연산자가 섞이면 정해진 우선순위에 따라 처리된다. 전부 외울 필요는 없지만, **AND가 OR보다 우선순위가 높다**는 점은 반드시 기억해야 한다(곱하기가 더하기보다 먼저 계산되는 것과 같다).

![](/assets/img/posts/sql-where-and-orderby/07.png)

```sql
WHERE a
OR    b
AND   c
```

이렇게 적혀 있어도 `AND`(`b AND c`)가 먼저 처리된다. `OR`를 먼저 묶고 싶다면 **괄호**를 써야 한다.

```sql
WHERE (a
OR     b)
AND    c
```

#### 우선순위 실습 — 괄호의 유무

"업무가 `'AD_PRES'`이면서 급여가 15000 초과인 사람, 또는 업무가 `'SA_REP'`인 사람"을 의도하고 다음처럼 쓰면, AND가 먼저 묶여 의도와 다른 결과가 나온다.

```sql
SELECT last_name, job_id, salary
FROM   employees
WHERE  job_id = 'SA_REP'
OR     job_id = 'AD_PRES'
AND    salary > 15000;
```

![](/assets/img/posts/sql-where-and-orderby/08.png)

반대로 "업무가 `'SA_REP'` 또는 `'AD_PRES'`이면서, 급여가 15000을 초과하는 사람"을 원한다면 OR 부분을 괄호로 묶어야 한다.

```sql
SELECT last_name, job_id, salary
FROM   employees
WHERE  (job_id = 'SA_REP'
OR      job_id = 'AD_PRES')
AND    salary > 15000;
```

![](/assets/img/posts/sql-where-and-orderby/09.png)

같은 조건이라도 괄호 하나로 결과가 완전히 달라지므로, OR와 AND가 섞일 때는 괄호를 꼭 챙기자.

<br/>

---

<br/>

### ㅇ ORDER BY — 결과 정렬

ORDER BY 절은 추출된 결과를 **정렬**한다. SELECT 문의 가장 끝에 오며, **그 아래에는 어떤 명령문도 올 수 없다.**

![](/assets/img/posts/sql-where-and-orderby/10.jpeg)

```sql
SELECT DISTINCT 컬럼 ALIAS
FROM   테이블
WHERE  조건식
AND/OR 조건식
ORDER BY 컬럼;
```

- **ASC**(ascending) : 오름차순. **기본값**이라 생략할 수 있다.
- **DESC**(descending) : 내림차순.

오름차순의 기준은 데이터 타입마다 다르다.

- 문자 : a → z
- 숫자 : 0 → 100
- 날짜 : 과거 → 현재 (현재가 더 큰 값)

NULL의 위치도 알아 두자. **오름차순이면 맨 마지막**, 내림차순이면 맨 처음에 온다.

<br/>

#### ORDER BY에는 ALIAS를 쓸 수 있다

WHERE 절과 달리 ORDER BY 절에는 **별칭을 쓸 수 있다.** ORDER BY는 맨 마지막에 실행되기 때문이다. 앞 단계가 모두 실행되어 출력할 내용이 추출된 뒤, 그 결과를 정렬하는 단계이므로 SELECT 절에서 붙인 별칭(열 머리글)을 그대로 사용할 수 있다.

단, **SELECT에서 지정한 별칭과 똑같은 포맷**으로 써야 한다. 예를 들어 SELECT에서 `"annsal"`처럼 큰따옴표로 소문자를 살려 놓고, ORDER BY에서는 따옴표 없이 `annsal`(기본 대문자)로 쓰면 서로 다른 별칭으로 취급되어 에러가 난다.

![](/assets/img/posts/sql-where-and-orderby/11.png)

<br/>

#### 여러 컬럼으로 정렬

ORDER BY에 컬럼을 여러 개 지정할 수 있다. 첫 번째 컬럼으로 같은 값끼리 묶어 정렬하고, 그 그룹 안에서 다음 컬럼으로 다시 정렬하는 식으로 단계적으로 적용된다.

```sql
SELECT last_name, department_id, salary
FROM   employees
ORDER BY department_id, salary DESC;
```

"부서별(부서는 오름차순)로 묶고, 각 부서 안에서는 급여가 많은 순서로 출력하라"는 뜻이다.

> 컬럼 자리에 숫자를 써서 `ORDER BY 2`처럼 "SELECT 리스트의 2번째 컬럼 기준으로 정렬"하는 방식도 동작은 한다. 현업에서 더러 쓰이지만 정식 문법으로 권장되지는 않으니, 학습 단계에서는 컬럼 이름을 명시하는 정공법을 쓰자.

<br/>

---

<br/>

### ㅇ 연산자 선택 가이드

상황에 맞는 연산자를 고르는 것이 핵심이다.

| 상황 | 연산자 |
|---|---|
| 사잇값(범위)을 찾을 때 | `BETWEEN A AND B` |
| 여러 개의 값을 찾을 때 | `IN (...)` |
| 값이 비어 있는지 찾을 때 | `IS NULL` |
| 값 하나를 정확히 찾을 때 | `=` |
| 일부 문자열로 찾을 때 | `LIKE` |

`OR`로 풀어 쓸 수 있는 것은 대개 `IN`으로, `AND`로 풀어 쓸 수 있는 범위 조건은 `BETWEEN`으로 묶는 편이 깔끔하고 성능에도 유리하다. 비슷한 조건을 여러 줄로 늘어놓기 전에, 그 의미를 한 번에 표현하는 연산자가 있는지 먼저 떠올리자.

<br/>

---

<br/>

### ㅇ 종합 실습

지금까지의 연산자를 섞어 풀어 보는 문제들이다.

**1) 연봉이 120000 이상인 사원의 이름과 연봉**

```sql
SELECT last_name, salary*12
FROM   employees
WHERE  salary*12 >= 120000;
```

**2) 사원 번호가 176번인 사원의 이름과 부서 번호**

```sql
SELECT last_name, department_id
FROM   employees
WHERE  employee_id = 176;
```

`employee_id IN (176)`으로 써도 결과는 같지만, 값이 하나뿐이고 사원번호는 PK이므로 단일 연산자 `=`가 더 적절하다.

**3) 연봉이 150000~200000 범위를 벗어나는 사원의 이름과 연봉(연봉은 `AnnSal`)**

```sql
SELECT last_name, salary*12 "AnnSal"
FROM   employees
WHERE  salary*12 NOT BETWEEN 150000 AND 200000;
```

- `WHERE NOT BETWEEN 150000 < salary*12 < 200000` 같은 수학식 표기는 오류다(`ORA-00936`). 반드시 `컬럼 NOT BETWEEN A AND B` 형태여야 한다.
- `NOT`은 `BETWEEN` 앞에 붙인다. `WHERE NOT salary*12 BETWEEN ...`처럼 컬럼 앞에 붙이면 "연봉 자체가 아닌 것"을 부정하는 엉뚱한 의미가 된다.
- 숫자에 콤마(`150,000`)를 찍으면 안 된다.

**4) 2003/01/01 ~ 2005/05/30 사이에 고용된 사원의 이름·사번·고용일(고용일 역순 정렬)**

```sql
SELECT last_name, employee_id, hire_date
FROM   employees
WHERE  hire_date BETWEEN '03/01/01' AND '05/05/03'
ORDER BY hire_date DESC;
```

날짜는 문자처럼 작은따옴표로 감싸야 한다. 따옴표 없이 `030101`처럼 쓰면 숫자로 인식되어 `ORA-00932`(inconsistent datatypes) 오류가 난다. 연·월·일 포맷만 맞으면 `'030101'`이나 `'2005/01/01'`처럼 구분자가 있든 없든 자동 변환되어 동작한다. (이는 포맷을 그대로 비교하는 `LIKE`와 다른 점이다.)

**5) 20번·50번 부서에서 근무하는 사원의 이름·부서번호(이름 알파벳 순)**

```sql
SELECT last_name, department_id
FROM   employees
WHERE  department_id IN (20, 50)
ORDER BY last_name ASC;
```

"알파벳 순"은 숫자인 부서번호가 아니라 **이름** 기준이므로 `ORDER BY last_name`이다. `department_id = 20 OR department_id = 50`은 조건식이 두 개라 비효율적이니 `IN`으로 묶는다.

**6) 20번·50번 부서에 근무하며 연봉이 200000~250000 사이인 사원의 이름·연봉**

```sql
SELECT last_name, salary*12
FROM   employees
WHERE  department_id IN (20, 50)
AND    salary*12 BETWEEN 200000 AND 250000;
```

조건이 둘이므로 `AND`로 잇는다. 만약 결과가 `no rows selected`라면, 정말 데이터가 없는 경우도 있지만 대부분은 코드가 잘못된 것으로 보고 다시 점검하는 편이 좋다. 쿼리가 실행되는 것 자체보다 **결과 데이터를 제대로 분석하는 것**이 중요하다.

**7) 2006년도에 고용된 사원의 이름·고용일**

```sql
SELECT last_name, hire_date
FROM   employees
WHERE  hire_date LIKE '06%';
```

날짜이므로 작은따옴표, 06으로 시작하므로 `LIKE '06%'`다.

**8) 매니저가 없는 사원의 이름·업무**

```sql
SELECT last_name, job_id
FROM   employees
WHERE  manager_id IS NULL;
```

**9) 매니저가 있는 사원의 이름·업무·매니저 번호**

```sql
SELECT last_name, job_id, manager_id
FROM   employees
WHERE  manager_id IS NOT NULL;
```

**10) 커미션을 받는 사원의 이름·연봉·커미션(연봉은 `ANNSAL`, 연봉 역순 정렬)**

```sql
SELECT last_name, salary*12 AS ANNSAL, commission_pct
FROM   employees
WHERE  commission_pct IS NOT NULL
ORDER BY ANNSAL DESC;
```

WHERE 절에는 별칭을 못 쓰지만, **ORDER BY 절에서는 별칭 `ANNSAL`을 쓸 수 있다.**

**11) 이름의 네 번째 글자가 a인 사원의 이름**

```sql
SELECT last_name
FROM   employees
WHERE  last_name LIKE '___a%';
```

`'___a'`로 끝내면 정확히 네 글자이고 네 번째가 a인 사람만 찾는다. 뒤에 다른 글자가 더 오는 이름까지 포함하려면 마지막에 `%`를 붙여야 한다.

**12) 이름에 a와 e가 모두 들어가는 사원의 이름**

```sql
SELECT last_name
FROM   employees
WHERE  last_name LIKE '%a%'
AND    last_name LIKE '%e%';
```

`'%a%e%'`로 한 번에 쓰면 a가 e보다 앞에 오는 경우만 찾는다. `LIKE`는 적힌 포맷 그대로 비교하기 때문이다. 따라서 순서와 무관하게 둘 다 포함하려면, `AND`로 **각각 새로운 조건식**을 써야 한다.

**13) 30번·90번 부서 내의 모든 직업을 유일한 값으로, 직업 오름차순 출력**

```sql
SELECT DISTINCT job_id
FROM   employees
WHERE  department_id IN (30, 90)
ORDER BY job_id;
```

"유일한 값"은 중복 제거이므로 `DISTINCT`를 쓰고, 두 부서는 `AND`·`OR`가 아니라 하나의 연산자 `IN`으로 처리한다.

<br/>

---

<br/>

여기까지 WHERE 절로 행을 거르고 ORDER BY로 줄을 세우는 방법을 정리했다. 연산자가 많아 보이지만 결국 "조건식 = 컬럼 + 연산자 + 값"이라는 한 원칙으로 모두 이어진다. 다음 글에서는 SELECT에서 가장 까다롭다는 **여러 테이블을 엮는 조인(JOIN)**을 다룬다.
