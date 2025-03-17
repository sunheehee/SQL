# Subquery
> SELECT 명령문 내의 명령문

🫧**서브 쿼리 예시**
```mysql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2)
```
- `SELECT * ~ WHERE column1` : 외부 쿼리
- `(SELECT ~ FROM t2)` : 서브쿼리

서브쿼리에서도 DISTINCT, GROUP BY, LIMIT, JOIN, INDEX, UNION, FUNCTION등의 기능이 가능하다

---

## 서브쿼리를 사용한 비교

🫧서브쿼리에 사용되는 비교 연산자

```
= > >= < <= <> != <=>
```
```
외부쿼리 LIKE 서브쿼리
```

🌱**서브쿼리를 사용한 비교문 예제**
```mysql
SELECT * FROM t1 
WHERE column1 = (SELECT MAX(column2) FROM t2);
```
- t2테이블 중 가장 큰 값과 동일한 값을 t1테이블의 column1에서 찾아 반환하는 코드(JOIN에서는 불가함함)


```mysql
SELECT * FROM t1 AS t
WHERE 2 = (SELECT COUNT(*) FROM t1 WHERE ti.id =t.id)
```
- t1테이블에서 2번 작성된 데이터들을 조회

    ➡️ JOIN에서는 집계가 이루어지는 다른 테이블과의 연결을 할 수 없다.서브쿼리를 사용한다.

-------

## ANY,IN, SOME을 포함한 하위쿼리

ANY, IN, SOME을 사용할 때의 비교 연산자 
```
= > < => <= <> !=
```

#### ANY
> 서브 쿼리에서 **반환되는 값 중 하나라도** TRUE를 반환하면 TRUE를 반환한다.
```mysql
SELECT s1 FROM t1 WHERE s1 > ANY(SELECT s1 FROM t2);
```
: t1테이블의 s1의 값이 t2테이블의 s1값 보다 하나라도 큰 s1을 조회

* t1테이블의 s1 = 10, t2테이블의 s1 = 21,14,7을 가지고 있으면 위의 표현은 TRUE를 반환한다.
t2테이블의 s1 = 10, 20 이거나, t2 테이블이 비어있다면 False를 반환한다.

❗t2테이블이 NULL을 포함하고 있으면 NULL을 반환한다.

❗서브쿼리에서의 IN은 ANY와 같은 작업을 한다.(하지만 동일하진 않다. ** ANY는 리스트를 가져올 수 없음.)
```mysql
SELECT s1 FROM t1 WHERE s1 = ANY(SELECT s1 FROM t2)

SELECT s1 FROM t1 WHERE s1 IN (SELECT s1 FROM t2)
```
👩🏻‍🏫 salary <> ANY(6000,7000)은 salary의 값이 6000 또는 7000 둘 중 하나라도 다르면 TRUE로 반환된다. 

만약 A의 salary가 6000, B의 salary가 7000이면 A의 salary는 7000이 아니었고, B의 salary는 6000이 아니었기 때문에 둘다 TRUE로 조회값에 포함된다.

❗ANY와 SOME은 동일하게 동작하고 <>ANY와 <>SOME도 동일하게 동작한다. 

❗NOT IN 은 <> ALL 과 같다.

❗하지만 (NOT) IN 쿼리문 안에 FALSE가 포함되어있으면 항상 FASLE가 반환되기 때문에 ALL을 사용하는것이 안전한 방법이다.

---

## ALL을 포함한 하위쿼리
> 서브쿼리가 반환하는 열의 값을 **모두 만족**하는 경우에만 TRUE를 반환함

```mysql
SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
```


* t1테이블의 s1 = 10, t2테이블의 s1 = -5, 0, 5를 가지고 있다면 s1이 t2의 s1값들보다 모두 큰 값을 가지고 있기 때문에 위의 표현은 TRUE를 반환한다.


❗ t2테이블이 NULL값을 포함하고 있다면 NULL반환


⚠️**주의**⚠️
```mysql
SELECT * FROM t1 WHERE 1 > ALL (SELECT s1 FROM t2);
```
➡️ t2가 빈 테이블이어도 TRUE

```mysql
SELECT * FROM t1 WHERE 1 > (SELECT s1 FROM t2);
```
➡️ 이 경우는 t2가 빈 테이블이면 NULL

#### + TABLE 문이란?

- 기존의 SELECT 방식
`SELECT * FROM 테이블 명`

- TABLE문을 사용한 조회방식
`TABLE 테이블 명`

❗ 테이블 문을 사용해서 IN, ALL, SOME, NOT IN 등을 간단하게 표현할 수 있다.

    ⭕ `SELECT * FROM t1 WHERE 1 > ALL (TABLE t2)` 

❗ 하지만 테이블 문에 MAX(), SUM(), AVG()과 같은 집계 함수를 포함하는 서브쿼리에서는 사용할 수 없다.

    ❌ `SELECT * FROM t1 WHERE 1 > ALL(MAX(TABLE t2))` 

---

## EXISTS / NOT EXISTS를 포함한 하위쿼리
> EXISTS는 SELECT가 가져오는 값의 리스트와는 상관없이 서브쿼리가 하나 이상의 행을 반환하면(즉, 행이 존재하기만 하면) TRUE를 반환

> NOT EXISTS는 서브쿼리가 결과를 반환하지 않으면 TRUE가 됨.

➡️ 즉 `EXISTS / NOT EXISTS`는 특정 조건에서 데이터 존재여부를 확인할 때 사용

🌱 **EXISTS, NOT EXISTS 사용 예시**

1. **하나 이상의 도시에 존재하는** 상점 찾기
```mysql
SELECT DISTINCT STORE_TYPE FROM STORES
WHERE EXISTS (
    SELECT * FROM CITIES_STORES 
    WHERE CITIES_STORES.STORE_TYPE = STORES.STORE_TYPE)
```

2. **어떤 도시에도 존재하지 않는** 상점 유형 찾기
```mysql
SELECT DISTINCT STORE_TYPE FROM STORES
WHERE NOT EXISTS (
    SELECT * FROM CITIES_STORES
    WHERE CITIES_STORES.STORE_TYPE=STORES.STORE_TYPE)
```

3. **모든 도시에 존재하는** 상점 유형 찾기
```mysql
SELECT DISTINCT STORE_TYPE FROM STORES
WHERE NOT EXISTS
(SELECT *
    FROM CITIES WHERE NOT EXISTS (
            SELECT * FROM CITIES_STORE
            WHERE CITIES_STORES.CITY=CITIES.CITY 
            AND CITIES_STORES.STORE_TYPE = STORES.STORE_TYPE))
```

✅NOT EXISTS의 이중사용 = EXISTS

----

## Subquery와 관련된 오류

### 1. 지원되지 않는 서브쿼리 문법
```mysql
SELECT * FROM t1 WHERE s1 IN (SELECT s2 FROM t2 ORDER BY s1 LIMIT 1);
```

```mysql
ERROR 1235 (ER_NOT_SUPPORTED_YET)
SQLSTATE = 42000
Message = "This version of MySQL doesn't yet support
'LIMIT & IN/ALL/ANY/SOME subquery'"
```

👩🏻‍🏫 IN, ALL, ANY, SOME 서브쿼리에서 ORDER BY 및 LIMIT을 사용할 수 없다. 

### 2. 서브쿼리에서 반환하는 **컬럼 개수**가 잘못된 경우
```mysql
SELECT (SELECT column1, column2 FROM t2) FROM t1;
```
```mysql
ERROR 1241 (ER_OPERAND_COL)
SQLSTATE = 21000
Message = "Operand should contain 1 column(s)"
```

❗ 서브쿼리는 기본적으로 하나의 컬럼만 반환한다. 

👩🏻‍🏫 여러 개의 컬럼을 비교하려면 `EXITS` 혹은 `JOIN`을 사용하는 것이 좋다.

#### ⭕**올바른 예제**

#### 2-1. 컬럼을 1개로 변경
```mysql
SELECT (SELECT column1 FROM t2) FROM t1;
``` 

#### 2-2. 여러 개의 컬럼 비교시 `EXISTS / IN / JOIN` 사용
```mysql
# IN 사용
SELECT * FROM t1 WHERE (column1, column2) IN (SELECT column1, column2 FROM t2);
```
```mysql
# EXISTS 사용
SELECT * FROM t1 WHERE EXISTS (
    SELECT * FROM t2 WHERE t1.column1 = t2.column1 AND t1.column2 = t2.column2
);
```
```mysql
# JOIN 사용
SELECT t1.*, t2.column1, t2.column2
FROM t1
JOIN t2 ON t1.id = t2.id;
```


### 3. 서브쿼리에서 **너무 많은 행**을 반환하는 경우
```mysql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```
```mysql
ERROR 1242 (ER_SUBSELECT_NO_1_ROW)
SQLSTATE = 21000
Message = "Subquery returns more than 1 row"
```

👩🏻‍🏫 서브쿼리가 최대 한 개의 행만 반환해야하는 경우, 여러 개의 행을 반환하면 오류가 발생한다.

👩🏻‍🏫 `= 연산자`는 단일 행의 값과 비교하기 때문에, 하위 쿼리에서 여러 개의 행을 반환하면 위와 같은 오류가 발생함.


#### ⭕**올바른 예제**

서브 쿼리에서 여러 개의 행을 반환할 가능성이 있다면 ANY 혹은 IN을 사용해서 비교한다. 
```mysql
SELECT * FROM t1 WHERE column1 = ANY (SELECT column1 FROM t2);
```

### 4. 서브 쿼리에서 동일한 테이블을 잘못 활용한 경우
```mysql
UPDATE t1 SET column2 = (SELECT MAX(column1) FROM t1);
```
```mysql
ERROR 1093 (ER_UPDATE_TABLE_USED)
SQLSTATE = HY000
Message = "You can't specify target table 'x'
for update in FROM clause"
```

👩🏻‍🏫같은 테이블을 서브쿼리에서 조회하면서 동시에 수정(`UPDATE`,`DELETE` 할 수 없다.)

➡️ CTE, DERIVED TABLE을 사용해서 해결할 수 있다. 

---

# CTE(Common Table Expression)
> 하나의 SQL문 내에서만 존재하는 이름이 지정된 임시 결과 집합 

## with절
- with 절에는 하나 이상의 쉼표로 구분된 서브쿼리가 포함된다. 
- 각 서브쿼리는 결과 집합을 생성하며, 특정 이름과 연결된다.

🌱**CTE 예제**
```mysql
WITH
    CTE1 AS (SELECT A,B FROM T1),
    CTE2 AS (SELECT C,D FRP, T2)

SELECT B,D FROM CTE1 JOIN CTE2
WHERE CTE1.A = CTE2.C
```
- `CTE1`, `CTE2`는 CTE로 정의된 **임시 결과 집합**이다.