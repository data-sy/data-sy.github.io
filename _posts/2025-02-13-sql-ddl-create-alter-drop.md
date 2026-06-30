---
title: "DDL: CREATE·ALTER·DROP과 제약조건"
date: 2025-02-13 21:30:00 +0900
categories: [CS, Database]
tags: [Database, SQL, Oracle, DDL, CREATE, ALTER, DROP, 제약조건, PK, FK, 무결성]
description: "테이블을 만드는 CREATE부터 구조를 바꾸는 ALTER, 지우는 DROP·TRUNCATE, 그리고 무결성을 지키는 5가지 제약조건까지 — DDL 전반을 정리하고 DML/DDL 실습으로 마무리한다."
---

> Oracle SQL/DB 학습 노트 시리즈의 여덟 번째 글이다. 앞에서 SELECT와 DML로 데이터를 다뤘다면, 이번에는 테이블 그 자체를 만들고 고치고 지우는 **DDL**과, 테이블의 무결성을 지켜 주는 **제약조건**을 정리한다. 마지막에는 DML/DDL 실습으로 DB 구축 과정을 직접 밟아 본다.

### \[ 목차 ]
- DDL 개요 — DB 구축의 흐름
- CREATE — 테이블 생성
- AS 서브쿼리로 테이블 만들기
- ALTER — 테이블 구조 변경
- DROP·TRUNCATE — 지우는 명령어들
- 제약조건 — 무결성을 지키는 5가지 키
- 제약조건 선언 — 테이블 레벨 vs 컬럼 레벨
- 명령어 모음 (레퍼런스)
- 스크립트 파일로 작업하기
- DML/DDL 실습 — DB 구축

<br/>

---

<br/>

### ㅇ DDL 개요 — DB 구축의 흐름

DB 구축은 결국 세 단계로 요약된다.

1. **테이블 생성** : CREATE
2. **데이터 삽입** : INSERT
3. **데이터 저장** : COMMIT

이 틀이 잡히면 그 안에서 SELECT와 DML로 DB를 운영한다. 이번 글의 주인공인 **DDL(Data Definition Language)**은 그중 1단계, 테이블이라는 **객체 자체를 생성·수정·삭제**하는 영역이다(CREATE / ALTER / DROP). DML이 행 단위로 데이터를 다룬다면, DDL은 테이블의 구조를 다룬다.

> DDL은 **한 번 실행할 때마다 자동으로 저장(auto-commit)**된다. 롤백으로 되돌릴 수 없으니 조심해서 사용해야 한다.

<br/>

---

<br/>

### ㅇ CREATE — 테이블 생성

테이블을 만들려면 두 가지 전제가 필요하다. 아무나 만들 수 없으므로 **생성 권한**이 있어야 하고, 데이터를 담을 **저장 공간**도 충분히 확보돼 있어야 한다. 그리고 테이블을 만들 때는 반드시 **테이블 이름**과 **각 열의 이름·데이터 타입·크기**를 지정한다.

![CREATE 개요](/assets/img/posts/sql-ddl-create-alter-drop/01.png)

기본 형태는 다음과 같다.

```sql
CREATE TABLE 테이블이름
(컬럼이름 데이터타입(길이),
 컬럼이름 데이터타입(길이),
 컬럼이름 데이터타입(길이)
)
```

예를 들어 부서 테이블 `dept`를 만들어 보자.

```sql
CREATE TABLE dept
( deptno number(2),
  dname  varchar2(14),
  loc    varchar2(13)
)
```

테이블을 만든 뒤 실제 데이터를 INSERT 했다면, 잊지 말고 **COMMIT**으로 저장한다.

<br/>

---

<br/>

### ㅇ AS 서브쿼리로 테이블 만들기

컬럼을 일일이 나열하는 대신, **서브쿼리의 결과를 그대로 테이블로** 만들 수도 있다. 서브쿼리가 실행되어 출력되는 결과물이 곧 하나의 테이블이 되는 것이다. 이때 **데이터 타입·길이·제약조건·데이터까지 전부** 따라온다. 주로 테이블 내용을 복사할 때 쓴다.

```sql
CREATE TABLE dept80
AS
SELECT employee_id, last_name, salary*12 annsal, hire_date
FROM employees
WHERE department_id = 80
```

```text
SQL> desc dept80
 Name           Null?    Type
 -------------- -------- --------------
 EMPLOYEE_ID             NUMBER(6)
 LAST_NAME      NOT NULL VARCHAR2(25)
 ANNSAL                  NUMBER
 HIRE_DATE      NOT NULL DATE
```

이 방식의 장점은 명확하다. 예컨대 "50번 부서 사원의 이름과 연봉을 저장하는 `dept50` 테이블을 생성하라"는 작업은 원래 ① 테이블 생성 → ② 대상 데이터 조회 → ③ INSERT → ④ COMMIT의 네 단계를 거쳐야 하는데, CREATE ~ AS 서브쿼리를 쓰면 이걸 **한 번에** 끝낼 수 있다.

단, 주의할 점도 있다. 이렇게 만든 테이블은 원본과 **PK/FK로 연결된 게 아니라 독립된 테이블**이다. employees의 급여가 바뀌어도 복사해 둔 `dept50`의 값은 따라 변하지 않아, 시간이 지나면 의미가 없어진다. 그래서 이 방식은 **주 단위로 갱신하는 통계성 자료(매출 내역 등)**에 적합하다(이런 용도는 뒤에서 배울 **뷰**를 더 자주 쓴다).

#### 컬럼 이름(컬럼 헤딩) 정하기

서브쿼리의 컬럼 헤딩이 그대로 새 테이블의 컬럼 이름이 된다. 소문자로 입력해도 실제로는 **대문자**로 저장된다. 컬럼 이름을 정하는 방법은 두 가지다.

```sql
-- 방법 1) 테이블 이름 뒤에 컬럼 이름을 직접 명시 (개수가 정확히 맞아야 함)
CREATE TABLE avgsal(name, deptid, sal)
AS
SELECT last_name, department_id, salary
FROM employees
WHERE salary*12 >= 150000
```

```sql
-- 방법 2) SELECT 절에서 ALIAS로 이름 부여 (권장)
CREATE TABLE avgsal
AS
SELECT last_name name, department_id deptid, salary sal
FROM employees
WHERE salary*12 >= 150000
```

특히 **연산식이나 그룹 함수가 들어가면 ALIAS가 필수**다. 컬럼 이름에는 오라클 명령어나 괄호 같은 특수기호가 들어갈 수 없기 때문이다.

```sql
-- ALIAS 없이 그룹 함수를 쓰면 에러
CREATE TABLE avgdept
AS
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
/
-- ORA-00998: must name this expression with a column alias

-- avgsal ALIAS를 붙여 해결
CREATE TABLE avgdept
AS
SELECT department_id, AVG(salary) avgsal
FROM employees
GROUP BY department_id
```

> 앞 예제에서 `salary*12`에 `annsal` ALIAS를 붙인 것도 같은 이유다. 연산식(`*`)은 컬럼 이름으로 쓸 수 없으므로 반드시 별칭을 줘야 한다.

#### CREATE AS 서브쿼리 vs INSERT 서브쿼리

INSERT 문에서도 VALUES 대신 서브쿼리를 쓸 수 있다.

```sql
INSERT INTO avgdept
SELECT department_id, salary
FROM employees
```

둘의 차이는 **무엇을 가져오느냐**에 있다.

| 구분 | 가져오는 것 |
|---|---|
| **CREATE ~ AS 서브쿼리** | 테이블 구조 + 컬럼 데이터 타입·길이 + 데이터까지 전부 |
| **INSERT ~ 서브쿼리** | 서브쿼리의 결과(데이터)만 그대로 삽입 |

INSERT 서브쿼리는 데이터를 **복사**할 때 간편하다. 107개 행을 일일이 INSERT 하는 대신 서브쿼리 한 문장으로 한꺼번에 삽입할 수 있다.

<br/>

---

<br/>

### ㅇ ALTER — 테이블 구조 변경

이미 생성된 테이블의 구조를 바꿀 때는 `ALTER TABLE`을 쓴다. 새 열을 추가하거나, 기존 열의 데이터 타입·길이를 바꾸거나, 열을 삭제할 수 있다. 명령문은 세 가지다.

- **ADD** : 컬럼 추가
- **MODIFY** : 데이터 타입 변경, 길이 수정
- **DROP** : 컬럼 삭제

#### ADD — 열 추가

추가한 열은 항상 맨 마지막(오른쪽)에 붙고, 기존 행들의 그 열 값은 모두 **NULL**이 된다.

```sql
ALTER TABLE dept
ADD (job_id varchar2(9))
```

새로 생긴 `job_id`에 값을 채우고 싶다면, **INSERT가 아니라 UPDATE**를 써야 한다. INSERT는 행 단위 작업이라 새 행이 추가되어 버리기 때문이다.

```sql
UPDATE dept
SET job_id = 'AA'
WHERE deptno = 10
```

#### MODIFY — 열 수정

```sql
ALTER TABLE 테이블이름
MODIFY (컬럼이름 데이터타입(바꿀길이))
```

길이를 늘리는 것은 데이터가 있든 없든 항상 가능하지만, **줄이거나 타입을 바꾸는 것은 데이터 유무에 따라 달라진다.**

- **길이 늘리기** : 데이터 유무와 무관하게 가능
- **길이 줄이기** : 저장된 데이터보다 작게는 못 줄인다(데이터가 없으면 가능)
- **타입 변경** : 데이터가 있으면 불가, 비어 있어야 가능

```sql
-- ITCENTER(7자)가 들어 있는 dname을 5자로 줄이면 에러
ALTER TABLE dept MODIFY (dname varchar2(5))
/
-- ORA-01441: cannot decrease column length because some value is too big

-- 데이터가 있는 컬럼의 타입을 바꿔도 에러
-- ORA-01439: column to be modified must be empty to change datatype
```

이는 **컬럼 무결성**(컬럼의 타입·길이에 맞는 데이터만 있어야 한다) 때문이다.

> 사실 현업에서 ALTER는 자주 쓰지 않는다. 구조를 자꾸 바꿔야 한다는 건 설계가 처음부터 잘못됐다는 뜻이기 때문이다. 그래서 설계 단계에서 철저히 해 수정이 없도록 한다.

<br/>

---

<br/>

### ㅇ DROP·TRUNCATE — 지우는 명령어들

#### 컬럼 삭제와 DROP의 문제점

열을 삭제하는 명령은 다음과 같다.

```sql
ALTER TABLE dept
DROP COLUMN job_id
```

그런데 이 명령에는 **동시성** 문제가 있다. DML이 **행 단위로 락**을 거는 반면, DDL은 **테이블 전체에 락**을 건다. 더구나 DROP은 데이터만 지우는 게 아니라 **데이터가 저장된 공간까지 함께 삭제**한다. 파일이 크면 삭제가 오래 걸리듯, 100만 행짜리 컬럼을 DROP 하면 그 작업이 끝날 때까지(예: 10분) 테이블 전체에 락이 걸려 아무도 접근하지 못한다. 그래서 운영 중인 DB에는 적합하지 않다.

#### SET UNUSED — 운영 중 안전하게 지우기

이 문제를 피하려고 만든 명령이 `SET UNUSED`다.

```sql
ALTER TABLE dept
SET UNUSED (loc)
```

데이터 딕셔너리(DB의 모든 정보를 저장하는 곳)에는 컬럼의 상태 정보가 기본값 `USED`로 저장돼 있다. `SET UNUSED`는 이 상태를 `UNUSED`로 바꿀 뿐, **실제 데이터와 저장 공간은 그대로 둔다.** 지우는 작업을 하지 않으니 매우 빠르다. 다만 쓸모없어진 데이터와 공간이 남아 있다는 게 단점이다(`SET USED`라는 명령은 없다 — 오로지 DROP을 극복하려고 만든 명령이라서).

남겨 둔 공간까지 실제로 정리하려면 별도로 다음을 실행한다.

```sql
ALTER TABLE dept
DROP UNUSED COLUMNS
```

#### DROP TABLE — 테이블 삭제

```sql
DROP TABLE 테이블이름
```

이때 **FK로 참조당하는 부모 테이블은 바로 지워지지 않는다.** 예컨대 employees가 departments를 참조하고 있으면, departments를 먼저 지울 수 없고 employees를 먼저 지워야 한다. (참고로 `RENAME`으로 테이블 이름을 바꿀 수 있다는 것도 알아 두자.)

#### TRUNCATE — 행 절단

```sql
TRUNCATE TABLE 테이블이름
```

DELETE / TRUNCATE / DROP은 모두 "지운다"는 점에서 헷갈리기 쉬워, 표로 정리하면 다음과 같다.

| 명령어 | 분류 | 지우는 범위 | 공간 | 복구 |
|---|---|---|---|---|
| **DELETE** | DML | 행 삭제 | 데이터 공간·테이블 구조 공간 모두 남음 | 롤백으로 복구 가능 |
| **TRUNCATE** | DDL | 행 삭제 | 데이터 저장 공간 삭제, 구조 공간은 남음 | auto-commit이라 복구 불가 |
| **DROP** | DDL | 테이블 삭제 | 모든 공간 삭제(데이터·구조·딕셔너리) | auto-commit이라 복구 불가 |

DELETE와 TRUNCATE는 둘 다 행을 지우지만, **DELETE는 WHERE 절을 쓸 수 있고 TRUNCATE는 쓸 수 없다.** 따라서 조건에 맞는 일부 행만 지울 때는 DELETE, 테이블의 전체 행을 한 번에 지울 때는 TRUNCATE를 쓴다. (TRUNCATE·DROP은 auto-commit이라 복구가 안 되지만, 백업이 있으면 복구할 수 있다.)

> CREATE TABLE은 자주 쓰는 명령이지만, 나머지 DDL 명령은 현업에서 거의 쓰지 않는다. DB 구조를 자주 바꾼다는 건 설계가 잘못됐다는 뜻이고, 구조를 바꾸면 연결된 운영 프로그램을 전부 손봐야 하기 때문이다. DBA의 일은 구조 변경이 아니라 **성능이 떨어지지 않도록 모니터링하고 튜닝**하는 것이다.

<br/>

---

<br/>

### ㅇ 제약조건 — 무결성을 지키는 5가지 키

지금까지 만든 테이블에는 사실 문제가 있다. 같은 INSERT를 두 번 실행하면 똑같은 행이 중복으로 쌓인다. **중복이 허용되는 = 무결성이 깨진** 테이블이다.

```text
    DEPTNO DNAME      LOC
---------- ---------- ------
        10 ITCENTER   1700
        20 ADMIN      1800
        10 ITCENTER   1700   <- 중복!
        20 ADMIN      1800   <- 중복!
```

무결성(결점이 없음)을 보장하려면, 컬럼에 **키(제약조건)**를 설치해 "조건에 맞는 데이터만 저장하겠다"고 정해 둬야 한다. 가장 이상적인 테이블은 **컬럼 1개에 하나의 PK가 지정된** 형태다(컬럼이 10개여도 그중 하나에 PK).

유효한 제약조건은 다음 다섯 가지이며, **모두 컬럼별로 설치**한다.

| 제약조건 | 의미 |
|---|---|
| **NOT NULL** | 반드시 값이 있어야 한다 |
| **UNIQUE** | 중복을 허용하지 않는다 (NULL은 허용) |
| **PRIMARY KEY** | UNIQUE + NOT NULL. 테이블을 대표하는 컬럼에 설치 |
| **FOREIGN KEY** | 다른 테이블의 값을 참조(관계 설정). PK나 UK로 지정된 컬럼만 참조 가능 |
| **CHECK** | 사용자 정의 조건 (예: `salary > 0`) |

하나의 컬럼에 제약조건을 **여러 개** 걸 수도 있다. 예컨대 학번은 PK로, 주민번호는 NOT NULL + UNIQUE 두 개를 함께 줄 수 있다.

> 참고로 키(KEY)라는 단어가 들어가는 건 **PRIMARY KEY**와 **FOREIGN KEY**뿐이다. NOT NULL·UNIQUE·CHECK에는 KEY가 붙지 않는다.

#### FOREIGN KEY의 옵션

FK는 다른 테이블의 컬럼을 참조하므로 **참조할 테이블과 컬럼**을 지정해야 한다(`REFERENCES 테이블이름(컬럼이름)`). FK에는 특히 중요한 동작이 있다.

- **REFERENCES 절만 사용** : 종속적인 삭제를 **방지**한다(기본 방침). 자식이 참조 중인 부모 행은 못 지운다.
- **ON DELETE CASCADE** : 부모 행을 지우면 그걸 참조하는 **자식 행도 함께 삭제**된다. 매우 강력하니 주의.
- **ON DELETE SET NULL** : 부모 행을 지우면 자식의 **참조 값을 NULL로 변경**한다(행은 남음).

테이블을 통째로 지울 때도 비슷한 옵션이 있다.

- `DROP TABLE departments CASCADE` : 참조된 테이블까지 함께 삭제
- `DROP TABLE departments CASCADE CONSTRAINTS` : FK(관계)를 끊어, 관계가 사라진 덕에 테이블이 삭제됨

DROP은 실행과 동시에 auto-commit이 발생하니 **신중하게** 다뤄야 한다.

<br/>

---

<br/>

### ㅇ 제약조건 선언 — 테이블 레벨 vs 컬럼 레벨

제약조건에는 **이름을 지정**하는 게 좋다. 생략하면 오라클이 `SYS_001`처럼 알아서 이름을 붙이지만, 수많은 제약조건을 관리하려면 직관적인 이름이 훨씬 효과적이다. 관례적으로 **`테이블이름_컬럼이름_제약조건유형`** 형태로 짓는다.

생성 시기는 두 가지다. 테이블을 만들 때 CREATE 안에서 함께 선언하거나(현업에서 주로 이 방법), 만든 뒤 ALTER TABLE로 추가한다. 선언하는 위치에 따라 두 가지 방식이 있다.

#### 테이블 레벨 — 권장

컬럼을 모두 선언한 **뒤**, 콤마로 구분해 제약조건을 별도로 선언한다. 가독성·편집 측면에서 유리하다.

```sql
CREATE TABLE dept
(did   number(5),
 dname varchar2(10),
 CONSTRAINT dept_did_pk PRIMARY KEY(did)
)
```

`CONSTRAINT 제약조건이름 제약조건유형(컬럼)` 형태로, 위 예는 "`dept_did_pk`라는 이름의 PK 제약조건을 `did` 컬럼에 설치"한다는 뜻이다. 나머지 키도 PRIMARY KEY 부분만 바꾸면 똑같은 규칙을 따른다.

#### 컬럼 레벨

컬럼을 선언하면서 그 **자리에서 바로** 제약조건을 붙인다(컬럼이 앞에 위치).

```sql
CREATE TABLE dept
(did   number(5) CONSTRAINT dept_did_pk PRIMARY KEY,
 dname varchar2(10)
)
```

#### 두 방식의 사용 관례

- PK·UK·FK·CHECK 네 가지는 **컬럼 레벨·테이블 레벨 모두** 선언 가능
- **NOT NULL은 컬럼 레벨에서만** 선언 가능

그래서 현업에서는 **NOT NULL은 컬럼 레벨에서, 나머지 키 4개는 테이블 레벨에서** 선언하는 식으로 나눠 쓰는 경우가 대부분이다. 또한 NOT NULL에는 보통 제약조건 이름을 따로 명시하지 않아, 이름이 `SYS_...`인 제약조건은 NOT NULL이겠구나 하고 짐작할 수 있다.

<br/>

---

<br/>

### ㅇ 명령어 모음 (레퍼런스)

여기까지 배운 명령어를 한곳에 정리해 둔다.

![명령어 모음](/assets/img/posts/sql-ddl-create-alter-drop/02.png)

#### SELECT 문

```sql
SELECT DISTINCT 컬럼
FROM 테이블
WHERE 조건식 연산자 값
GROUP BY 그룹함수를 쓸 때, 그룹함수에 안 쓰인 컬럼들
HAVING 그룹화한 결과에 대한 조건
ORDER BY 정렬
```

- 문자와 날짜는 작은따옴표(`' '`)로 감싼다
- 값·조건절(WHERE·HAVING)에 서브쿼리를 쓸 수 있다 (WHERE 절에서 그룹 함수를 쓰고 싶을 때도 서브쿼리)
- 이름을 찾을 때는 중복을 고려해 `=` 대신 `IN`을 쓴다
- 테이블을 조인하면 **반드시 조인 조건**을 빠뜨리지 않는다

#### DML — 행 단위 삽입·변경·삭제 (커밋으로 저장)

```sql
-- 행 삽입 (컬럼 목록은 선택사항, NULL은 NULL로)
INSERT INTO 테이블 (컬럼1, 컬럼2, 컬럼3)
VALUES (값1, 값2, 값3);

-- 행 변경 (WHERE는 선택, 결과적으로 특정 컬럼의 값이 바뀜)
UPDATE 테이블
SET 컬럼이름 = 변경값
WHERE 조건;

-- 행 삭제 (WHERE 생략 시 모든 행 삭제)
DELETE FROM 테이블
WHERE 조건;
```

- VALUES 대신 서브쿼리를 쓸 수 있다
- **자식 테이블의 FK** : 참조하는 PK 값이 존재해야 삽입 가능
- **부모 테이블의 PK** : 자식 FK가 그 값을 쓰고 있으면 수정·삭제 불가. 또 PK는 중복이 안 되므로 이미 쓰인 값인지 주의

#### DDL — 테이블 생성 및 관리 (실행마다 자동 저장)

```sql
-- 테이블 생성
CREATE TABLE 테이블명
(컬럼이름 데이터타입(크기),
 deptno number(2),
 dname  varchar2(14),
 loc    varchar2(13)
);

-- 서브쿼리로 생성 (통계성 자료)
CREATE TABLE dept80
AS
SELECT employee_id, last_name, salary*12 annsal, hire_date
FROM employees
WHERE department_id = 80;

-- 열 추가 (데이터는 NULL, 값 채울 땐 UPDATE)
ALTER TABLE 테이블명 ADD (컬럼이름 데이터타입(크기));

-- 열 수정
ALTER TABLE 테이블명 MODIFY (컬럼이름 바꿀타입(바꿀길이));

-- 열 삭제 (시간이 걸림)
ALTER TABLE 테이블명 DROP COLUMN 컬럼이름;

-- 상태를 UNUSED로 만들어 삭제한 것처럼 사용
ALTER TABLE 테이블명 SET UNUSED (컬럼이름);

-- 테이블 삭제 / 테이블 절단
DROP TABLE 테이블명;
TRUNCATE TABLE 테이블명;
```

#### 제약조건 주기

```sql
-- 테이블 레벨에서 선언 (제약조건이름 = 테이블_컬럼_유형)
CREATE TABLE 테이블이름
(컬럼이름 데이터타입(길이),
 dname varchar2(10) NOT NULL,                          -- NOT NULL은 컬럼 레벨
 CONSTRAINT dept_did_pk    PRIMARY KEY(did),
 CONSTRAINT dept_email_uk  UNIQUE(email),
 CONSTRAINT dept_dname_fk  FOREIGN KEY(dname)           -- 자식 테이블
   REFERENCES 부모테이블(참조컬럼)                        -- 부모 테이블
   ON DELETE CASCADE,        -- 부모 행 삭제 시 자식 종속 행도 삭제
-- ON DELETE SET NULL          부모 행 삭제 시 참조 값을 NULL로
 CONSTRAINT dept_salary_min CHECK(salary > 0)
);

-- 컬럼 레벨에서 선언
CREATE TABLE dept
(did   number(5) CONSTRAINT dept_did_pk PRIMARY KEY,
 dname varchar2(10)
);
```

<br/>

---

<br/>

### ㅇ 스크립트 파일로 작업하기

SQL\*Plus는 한 줄씩 입력하기 불편하다. 코드를 **`.sql` 스크립트 파일**로 저장해 두면 여러 명령을 한 번에 재사용할 수 있어 적극 권장된다.

```text
SQL> SAVE test.sql        -- 버퍼의 마지막 명령을 파일로 저장
SQL> @test.sql            -- 스크립트 파일 실행
```

저장된 `.sql` 파일은 오라클 설치 경로의 `bin` 폴더(예: `내PC > C드라이브 > oraclexe > app > oracle > product > 11.2.0 > server > bin`)에 생기며, 메모장으로 열어 편집할 수 있다. 여기서 ED(버퍼)와 스크립트 파일의 역할을 구분해 두자.

| 도구 | 정체 | 실행 명령 | 용도 |
|---|---|---|---|
| **ED (버퍼)** | 마지막 SQL 1개만 담기는 메모리 | `/` 또는 `run` | 방금 명령 수정·편집 |
| **`.sql` (스크립트)** | 여러 SQL을 담는 파일 | `@파일이름.sql` | 여러 코드 동시 실행 |

스크립트 파일을 작성할 때 주의할 점이 있다.

- 모든 명령문 끝에 **세미콜론(`;`)**을 붙인다. 빠지면 `ORA-00933: SQL command not properly ended` 에러가 난다.
- 끝에 **`/`는 쓰지 않는다.** `/`는 버퍼를 한 번 더 실행하라는 뜻이라, 마지막 명령이 중복 실행된다(테이블이 두 개씩 생기는 원인).
- 같은 스크립트를 다시 실행하면 이미 있는 테이블에서 `ORA-00955: name is already used...` 에러가 난다. 그래서 보통 맨 위에 `DROP TABLE`을 먼저 둔다.

```sql
DROP TABLE a;            -- 처음엔 테이블이 없어 에러가 나지만 무시하고 진행

CREATE TABLE a ( a number(3) );

INSERT INTO a VALUES (10);
INSERT INTO a VALUES (20);
INSERT INTO a VALUES (30);
INSERT INTO a VALUES (40);
COMMIT;

SELECT * FROM a;
```

<br/>

---

<br/>

### ㅇ DML/DDL 실습 — DB 구축

이제 배운 걸 모아 테이블을 직접 만들어 본다. DB 구축의 과정은 앞서 말한 그대로다 — **① 테이블 생성(CREATE) → ② 데이터 삽입(INSERT) → ③ 데이터 저장(COMMIT)**. 이 안에서 SELECT·DML로 운영한다.

주어진 **테이블 정의서**와 **데이터**는 다음과 같다.

<div style="display:flex; gap:8px;">
<div style="flex:1;"><img src="/assets/img/posts/sql-ddl-create-alter-drop/03.png"></div>
<div style="flex:1;"><img src="/assets/img/posts/sql-ddl-create-alter-drop/04.png"></div>
</div>

#### 제약조건을 포함한 두 테이블 만들기

부모 역할을 할 `test` 테이블과, 그걸 참조하는 자식 `test1` 테이블을 스크립트로 작성한다. 참조 관계가 있으므로 **부모(`test`)를 먼저 생성**해야 자식(`test1`)이 참조할 수 있다.

```sql
DROP TABLE test  cascade constraints;
DROP TABLE test1 cascade constraints;

CREATE TABLE test
(a number(5),
 b number(5) not null,
 c number(5),
 CONSTRAINT test_a_pk PRIMARY KEY(a)
);

CREATE TABLE test1
(d number(5),
 e number(5),
 f number(5),
 a number(5),
 CONSTRAINT test1_d_pk  PRIMARY KEY(d),
 CONSTRAINT test1_e_uk  UNIQUE(e),
 CONSTRAINT test1_f_ck  CHECK(f IN (0, 1)),
 CONSTRAINT test1_a_fk  FOREIGN KEY(a)
   REFERENCES test(a)
);
```

이 실습에서 한참 막혔던 부분이 있다. FK의 참조 키워드는 `REFERENCE`가 아니라 **`REFERENCES`**(끝에 `s`)다. 빠뜨리면 엉뚱하게도 `ORA-00907: missing right parenthesis`(오른쪽 괄호 누락) 에러로 나타나, 원인을 찾기 어렵다.

> 정리하면, 제약조건이 있는 여러 테이블을 만들 때는 **참조 방향(자식 → 부모)**을 따져 부모 테이블을 먼저 생성하고, FK 키워드 `REFERENCES`의 철자에 주의한다.

<br/>

---

<br/>

다음 글에서는 CREATE ~ AS 서브쿼리의 한계를 메워 주는 **뷰(View)** — 실제 데이터를 복사하지 않고도 통계성 자료를 다루는 가상 테이블 — 를 살펴본다.
