# Subquery
> SELECT 명령문 내의 명령문

🫧**서브 쿼리 예시**
```
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
```
SELECT * FROM t1 
WHERE column1 = (SELECT MAX(column2) FROM t2);
```
- t2테이블 중 가장 큰 값과 동일한 값을 t1테이블의 column1에서 찾아 반환하는 코드(JOIN에서는 불가함함)


```
SELECT * FROM t1 AS t
WHERE 2 = (SELECT COUNT(*) FROM t1 WHERE ti.id =t.id)
```
- t1테이블에서 2번 작성된 데이터들을 조회
➡️ 집계가 이루어지는 다른 테이블과의 연결을 원한다면 JOIN이 아닌 서브쿼리를 사용한다.

-------

## ANY,IN, SOME을 포함한 하위쿼리

ANY, IN, SOME을 사용할 때의 비교 연산자 
```
= > < => <= <> !=
```

#### ANY
> 서브 쿼리에서 반환되는 값 중 하나라도 TRUE를 반환하면 TRUE를 반환한다.
```
SELECT s1 FROM t1 WHERE s1 > ANY(SELECT s1 FROM t2);
```
: t1테이블의 s1의 값이 t2테이블의 s1값 보다 하나라도 큰 s1을 조회

* t1테이블의 s1 = 10, t2테이블의 s1 = 21,14,7을 가지고 있으면 위의 표현은 TRUE를 반환한다.
t2테이블의 s1 = 10, 20 이거나, t2 테이블이 비어있다면 False를 반환한다.

❗t2테이블이 NULL을 포함하고 있으면 NULL을 반환한다.

❗서브쿼리에서의 IN은 ANY와 같은 작업을 한다.(하지만 동일하진 않다. ** ANY는 리스트를 가져올 수 없음.)
```
SELECT s1 FROM t1 WHERE s1 = ANY(SELECT s1 FROM t2)

SELECT s1 FROM t1 WHERE s1 IN (SELECT s1 FROM t2)
```
👩🏻‍🏫 salary <> ANY(6000,7000)은 salary의 값이 6000 또는 7000 둘 중 하나라도 다르면 TRUE로 반환된다. 

만약 A의 salary가 6000, B의 salary가 7000이면 A의 salary는 7000이 아니었고, B의 salary는 6000이 아니었기 때문에 둘다 TRUE로 조회값에 포함된다.

❗ANY와 SOME은 동일하게 동작하고 <>ANY와 <>SOME도 동일하게 동작한다. 

❗NOT IN 은 <> ALL 과 같다.

❗하지만 (NOT) IN 쿼리문 안에 FALSE가 포함되어있으면 항상 FASLE가 반환되기 때문에 ALL을 사용하는것이 안전한 방법이다.


## ALL을 포함한 하위쿼리
> 서브쿼리가 반환하는 열의 값을 모두 만족하는 경우에만 TRUE를 반환함

```
SELECT s1 FROM t1 WHERE s1 > ALL (SELECT s1 FROM t2);
```


* t1테이블의 s1 = 10, t2테이블의 s1 = -5, 0, 5를 가지고 있다면 s1이 t2의 s1값들보다 모두 큰 값을 가지고 있기 때문에 위의 표현은 TRUE를 반환한다.


❗ t2테이블이 NULL값을 포함하고 있다면 NULL반환


⚠️**주의**⚠️
```
SELECT * FROM t1 WHERE 1 > ALL (SELECT s1 FROM t2);
```
* t2가 빈 테이블이어도 TRUE

```
SELECT * FROM t1 WHERE 1 > (SELECT s1 FROM t2);
```
* 이 경우는 t2가 빈 테이블이면 NULL

---
#### + TABLE 문이란?

- 기존의 SELECT 방식
`SELECT * FROM 테이블 명`

- TABLE문을 사용한 조회방식
`TABLE 테이블 명`

❗ 테이블 문을 사용해서 IN, ALL, SOME, NOT IN 등을 간단하게 표현할 수 있다.

⭕ `SELECT * FROM t1 WHERE 1 > ALL (TABLE t2)` 

❗ 하지만 테이블 문에 MAX(), SUM(), AVG()과 같은 집계 함수를 포함하는 서브쿼리에서는 사용할 수 없다.

❌ `SELECT * FROM t1 WHERE 1 > ALL(MAX(TABLE t2))` 