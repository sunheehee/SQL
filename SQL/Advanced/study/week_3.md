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