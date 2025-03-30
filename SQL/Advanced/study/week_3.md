# 복합 JOIN & Grouping

SELECT문과 DELETE, UPDATE문에서 JOIN문법을 지원함.


## MYSQL에서의 table_factor
### 1. 표준 SQL과 달리 괄호 안에 여러 개의 테이블을 쉼표로 나열할 수 있다.

#### **🌱예시**

```MYSQL
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```
에서 <u>**괄호 안의 쉼표는 CROSS JOIN으로 취급**</u>되어 아래와 같다.
```MYSQL
SELECT * FROM t1 LEFT JOIN (t2 CROSS JOIN t3 CROSS JOIN t4)
ON (조건들...)
```

### 2. MYSQL에서는 JOIN,CROSS JOIN, INNER JOIN이 서로 동일한 기능을 한다.

- 표준 SQL에서는 서로 다르게 취급한다.
    - `INNER JOIN`- ON 조건 필요
    - `CROSS JOIN`- ON 조건 필요없고 전부 조합합   

### 3. INNER JOIN이면 괄호 생략 가능
> INNER JOIN만 사용한다면 괄호 생략이 가능하다

#### **🌱예시**

```MYSQL
SELECT * FROM a JOIN b JOIN c;
```
### 4. 테이블 별칭 사용의 유무

- 테이블 별칭을 지정할 때 AS를 생략하고 지정 가능
- 서브쿼리에는 반드시 별칭이 필요


### 5. 쉼표(,)와 JOIN

```MYSQL
SELECT * FROM a, b;
SELECT * FROM a INNER JOIN b;
```
❗ 쉼표보다는 <u>**JOIN, LEFT JOIN등의 우선순위가 더 높아**</u> 쉼표와 JOIN을 같이 쓰면 오류발생 가능성 있음

→ 괄호를 사용하거나 쉼표를 사용하지 않고 JOIN을 직접 명시해 사용할 것 

```MYSQL
1안) SELECT * FROM (t1, t2) JOIN t3 ON (t1.i1 = t3.i3);
 
2안) SELECT * FROM t1 JOIN t2 JOIN t3 ON (t1.i1 = t3.i3);
```


- 조건이 없으면 둘 다 모든 조합(Cartesian)을 만든다.

### 6. USING(col1,col2)는 자동으로 컬럼임 매칭된다

```MYSQL
a LEFT JOIN b USING (c1, c2)
```
- a.c1 = b.c1 AND a.c2 = b.c2 가 자동 매칭되어서, 출력결과 중복된 컬럼만이 나타난다.

- 공동컬럼 이름이 동일해야 USING을 활용가능

#### **🌱만약 아래와 같다면?**
```MYSQL
a LEFT JOIN b 
ON a.c1=b.c1 AND a.c2=b.c2;
```
- 그러면 orders.customer_id와 customers.customer_id처럼 컬림이 2개가 출력됨

### 7. NATURAL JOIN은 자동으로 공통 컬럼만을 JOIN한다.

> 두 테이블의 이름이 동일한 컬럼이 있으면 자동으로 조건이 걸어진다.

#### **🌱 예시**
**상황예시**
```MYSQL
CREATE TABLE t1 (i INT, j INT);
CREATE TABLE t2 (k INT, j INT);
INSERT INTO t1 VALUES(1, 1);
INSERT INTO t2 VALUES(1, 1);
```
- j 컬럼만 공통으로 존재

#### **🌱 예시-(NATURAL JOIN과 USING동일)**
```MYSQL
SELECT * FROM t1 NATURAL JOIN t2;
```
**결과**
```MARKDOWN
+------+------+------+
| j    | i    | k    |
+------+------+------+
|  1   |  1   |  1   |
+------+------+------+
```
- 공통 컬럼 j는 자동으로 JOIN이 됨
- 결과에는 t1.j와 t2.j중 결측이가 아닌것으로 한 개 출력됨 (COALESCE(t1.j = t2.j))

### 8. NATURAL OUTER(LEFT,RIGHT) JOIN의 동작방식

#### **🌱 컬럼 설정정**
```
CREATE TABLE t1 (a INT, b VARCHAR(1));
CREATE TABLE t2 (a INT, c VARCHAR(1));

INSERT INTO t1 VALUES (1, 'x'), (2, 'y');
INSERT INTO t2 VALUES (2, 'z'), (3, 'w');
```

#### **🌱 NATURAL LEFT JOIN**

```MYSQL
SELECT * FROM t1 NATURAL LEFT JOIN t2;
```

```MARKDOWN
+------+------+------+
| a    | b    | c    |
+------+------+------+
|  1   | x    | NULL |
|  2   | y    | z    |
+------+------+------+
```

- 왼쪽 테이블을 기준으로, 왼쪽 테이블에는 있지만, 오른쪽에 없으면 NULL

#### **🌱 NATURAL RIGHT JOIN**
```MYSQL
SELECT * FROM t1 NATURAL RIGHT JOIN t2;
```

```MARKDOWN
+------+------+------+
| a    | c    | b    |
+------+------+------+
|  2   | z    | y    |
|  3   | w    | NULL |
+------+------+------+
```
- 오른쪽 테이블 기준으로 오늘쪽 테이블에는 있지만 왼쪽 테이블에 없으면 NULL처리

#### **🌱 ON을 사용했을 때**
```MYSQL
SELECT * FROM t1 LEFT JOIN t2 ON (t1.a = t2.a);
```
```MARKDOWN
+------+------+------+------+  
| a    | b    | a    | c    |  
+------+------+------+------+  
|  1   | x    | NULL | NULL |  
|  2   | y    |  2   |  z   |  
+------+------+------+------+  
```
- 공통 컬럼 a가 양쪽에서 모두 출력됨

----

# Group By
**GROUP BY 를 사용하는 경우, SELECT, HAVING, ORDER BY 절에 있는 컬럼은 반드시 GROUP BY에 명시되어있거나, 집계함수에 포함되어있어야한다.**

❌**잘못된 쿼리**
```MYSQL
SELECT o.custid, c.name, MAX(o.payment)
FROM orders o, customers c
WHERE o.custid = c.custid
GROUP BY o.custid;
```

- `c.name`을 GROUP BY 절에 포함해야한다.

하지만 `C.NAME`이 O.CUSTID와 종속적 관계라면, 실행가능한 쿼리가 된다. 

❓**기능적 종속**

어떤 컬럼(A)의 값이 결정되었을 때, 다른 컬럼(B)의 값이 항상 고정이라면, B는 A에 기능적으로 종속된다고 표현한다.

예: CUSTID가 고객 테이블의 PK라면, 그에 대응하는 NAME은 항상 하나 → NAME은 CUSTID에 종속

❗ 하지만 이 쿼리내에서는 CUSTID당 여러개의 이름이 있을 수 있다고 가정, 

이러한 쿼리에서는 CUSTID가 NAME에 종속되어있다고 판단하는 것이 더 보편적이어서 `GROUP BY C.NAME`을 쓰고 `O.CUSTID` 를 생략하는 방식으로 더 많이 쓴다.

- `o.payment`는 GROUP BY 절에 포함하지 않아도 된다.
    - 집계 함수 안에 있는 컬럼은 자동으로 내부에서 게산이 되기 때문에, 컬럼 자체를 GROUP BY에 명시할 필요가 없음.

## MYSQL의 확장 기능
### 1. HAVING절에서 SELECT절의 별칭 사용 허용

```MYSQL
SELECT NAME, COUNT(*) AS C
FROM ORDERS
GROUP BY NAME
HAVING C=1;
```

- 표준 SQL 에서는 `HAVING COUNT(*) = 1`처럼 함수 전체를 다시 써야 하지만, MYSQL에서는 별칭 사용이 가능하다

### 2. GROUP BY 표현식 허용
```MYSQL
SELECT ID, FLOOR(VALUE / 100)
FROM TBL_NAME
GROUP BY ID, FLOOR(VALUE / 100)
```

### 3.GROUP BY에서 별칭 사용 허용
```MYSQL
SELECT ID, FLOOR(VALUE/100) AS VAL
FROM TBL_NAME
GROUP BY ID, VAL;
```

-표준 SQL에서는 SELECT의 동작순서가 GROUP BY 보다 나중이기 때문에 불가하지만, MYSQL에서는 가능

### 4. 중복된 컬럼 별칭 사용
```MYSQL
SELECT COUNT(COL1) AS COL2 
FROM T
GROUP BY COL2
HAVING COL2 = 2;
```

- 테이블에 COL2라는 테이블이 존재한다는 가정하에, 기존 컬럼명과 동일한 별칭명이 생성
- 이러한 경우에는 <U>실제 컬럼을 우선으로 해석</U>

---

# HAVING

## SELECT문 구성요소 순서와 실행 흐름
```MYSQL
SELECT ...
FROM ...
[WHERE ...]
[GROUP BY ...]
[HAVING ...]
[ORDER BY ...]
[LIMIT ...]
```

## HAVING이란?
> GROUP BY로 그룹화된 결과에 조건을 거는 절

- 집계함수를 활용한 조건에 사용 

    (WHERE절에서는 집계함수 활용 못함)

## HAVING규칙
### 1. SELECT절에서 정의한 별칭 사용 가능
```MYSQL
SELECT O.CUSTID, MAX(O.PAYMENT) AS MAX_PAY
FROM ORDERS O
GROUP BY O.CUSTID
HAVING MAX_PAY > 10000;
```

- 표준 SQL에서는 SELECT의 실행순서가 HAVING 다음이라 HAVING에서 SELECT문의 별칭을 사용할 수 없지만, MYSQL에서는 가능하다
- 단 WHERE절에서는 여전히 SELECT문에서 정의한 별칭 사용 불가능

### 2. HAVING절에서 별칭/컬럼 충돌 시
```MYSQL
SELECT COUNT(COL1) AS COL2 
FROM TO 
GROUP BY COL2
HAVING COL2 = 2;
```

- COL2가 SELECT문에서 만든 별칭이기도하고, 실제 컬럼명일수도 있음
- MYSQL은 <U>GROUP BY의 컬럼을 우선 적용</U>

### 3. HAVING절은 그룹이 없는 경우에도 사용할 수 있음

```MYSQL
SELECT MAX(O.PAYMENT)
FROM ORDERS O
HAVING MAX(O.PAYMENT) > 10000;
```
- GROUP BY는 없지만 전체 테이블이 하나의 그룹으로 간주됨
- HAVING절은 집계부분에 조건을 주어, 만족하는 경우에만 결과를 반환함


# WEEK_3 정리 끝!
![alt text](<image/양치 핑구.jpg>)

