# Subquery
> SELECT 명령문 내의 명령문

🫧**서브 쿼리 예시**
```
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2)
```
- `SELECT * ~ WHERE column1` : 외부 쿼리
- `(SELECT ~ FROM t2)` : 서브쿼리

서브쿼리에서도 DISTINCT, GROUP BY, LIMIT, JOIN, INDEX, UNION, FUNCTION등의 기능이 가능하다

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
: t2테이블의 s1값보다 t1테이블의 s1의 값이 큰 s1을 조회

가정)
t1테이블에 value:10
t2테이블이 value=21,14,7을 가지고 있으면 위의 표현은 TRUE를 반환한다.

```


