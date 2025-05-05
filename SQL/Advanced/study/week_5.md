# GROUP_CONCAT()

### 그룹 내의 NULL이 아닌 값들을 연결해서 문자열로 반환

**예제 1**
```MYSQL
SELECT student_name,
    GROUP_CONCAT(test_score)
FROM student
GROUP BY student_name;
```


**예제 2**
```MYSQL
SELECT student_name,
    GROUP_CONCAT(DISTINCT test_score
    ORDER BY test_score DESC SEPARATOR ' ')
FROM student
GROUP BY student_name;
```


### 예시1과 예시2의 차이점
| 항목             | 예시 1                                                                 | 예시 2                                                                                       |
|------------------|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| SQL 문           | `GROUP_CONCAT(test_score)`                                             | `GROUP_CONCAT(DISTINCT test_score ORDER BY test_score DESC SEPARATOR ' ')`                 |
| DISTINCT 사용  | ❌ 중복 허용                                                          | ✅ 중복 제거                                                                                 |
| 정렬 기준        | ❌ 없음 (입력 순서대로 나열됨)                                       | ✅ test_score 내림차순 정렬 (`ORDER BY test_score DESC`)                                |
| 구분자 지정      | ❌ 없음 → 기본 구분자 ,(콤마) 사용됨                                     | ✅ ' ' (공백)을 구분자로 명시 → `SEPARATOR ' '`                                          |
| 출력 예시 (가정) | 85,90,85                                                            | 90 85                                                                                     |
| 용도 차이        | 단순 나열                                                             | 중복 제거 + 정렬 + 깔끔한 포맷의 문자열 생성용                                              |

### 반환값
결과 문자열은  `group_concat_max_len`값만큼 최대 길이 제한이 있음
- 기본값: 1024
- 더 길게 설정이 가능하지만, `max_allowed_packet`값보다 클 수는 없음

- group_concat_max_len변경방법: 
    ```
    SET GLOBAL (OR SESSION) group_concat_max_len = 변경할 값;
    ```

### 반환타입
- nonbinary문자열 → nonbinary문자열
- binary문자열 → binary문자열
- 기본적으로는 TEXT / BLOB 타입으로 바나환
    (group_concat_max_len이 512 이하이면, VARCHAR / VARBINARY로 반환)

------

# WITH RECURSIVE

### 반복되는 작업을 자동으로 처리하고 싶을 때 사용하는 SQL 문법

```MYSQL
WITH RECURSIVE cte AS (
  -- 1단계: 처음에 시작할 값 (비재귀)
  SELECT ...
  UNION ALL
  -- 2단계: 반복해서 값을 만들어가는 부분 (재귀)
  SELECT ... FROM cte WHERE 조건
)
SELECT * FROM cte;

```
- 처음 SELECT문 : 처음 한 번만 실행되면서 초기 값 집합을 생성성
- 재귀 SELECT문: 자기 자신을 계속 부루는 SELECT문 → 여러 번 반복됨

**🌱예시 (숫자 1~5까지 출력)**

```MYSQL
WITH RECURSIVE NUMBER AS(
    SELCT 1
    UNION ALL
    SELECT n + 1
    FROM NUMBER
    WHERE n < 5
)

SELECT * FROM numbers;
```

## 📝WITH RECURSIVE 문법

### 데이터 타입과 NULL
- 어떤 데이터 타입을 쓸 지는 비재귀 SELECT가 기준 (첫번째 SELECT)
- 모든 열은 NULL을 가질 수 있는 것이 기본값
- 재귀 SELECT에서 데이터 길이가 길어지면 잘릴 수 있음
    
    ➡️ 문자열이 길어질 수 있는 경우에는 `CAST()`를 이용해서 열 길이를 지정해야함

    ```MYSQL
    SELECT CAST ('abc' AS CHAR(20))
    ```

    ```markdown
    🔎CAST()란?
    : 데이터의 타입을 바꿔주는 함수

    - WITH RECURSIVE에서는 열 타입이나 길이를 명확히 지정할 때 자주 사용됨
    ```
### 무한 루프 방지
WITH RECURSIVE는 재귀가 무한히 반복되지 않도록 

✨ WHERE절을 사용해서 종료조건이 명시한다. 

✨ `UNION DISTINCT`를 사용하면 중복 행이 제거되면서, 무한 루프 방지에 유용하다.

✨ 기본 1000번까지 `cte_max_recursion_depth`변수로 최대 반복을 제한할 수 있다.
```mysql
SET SESSION cte_max_recursion_depth = 10;

WITH RECURSIVE cte AS (
    SELECT 1 
    UNION ALL
    SELECT n+1 FROM cte
)
SELECT * FROM cte;
```
-  10번 이상 반복되면 mysql자동 중단됨

✨ `max_execution_time`을 이용해 너무 오래 걸리는 쿼리는 자동으로 정해진 시간 후 종료시킬 수 있다.
```mysql
SET max_execution_time = 1000;

WITH RECURSIVE cte AS (
    SELECT 1 
    UNION ALL
    SELECT n+1 FROM cte
)
SELECT * FROM cte;
```
- max_execution_time의 기본단위는 ms로 1000ms = 1초
- 즉, 쿼리 실행 시간이 1초가 넘으면 sql이 자동 중단된다.

✨ LIMIT으로 행 개수 제한
```mysql
WITH RECURSIVE cte (n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte LIMIT 10000
)
SELECT * FROM cte;
```
- 재귀로 생성되는 행 수를 10000개로 제한

### 반복 작동 방식
각 재귀 반복은 **직전 반복에서 생성된 행에만 작동**한다.

- 예를 들어 직전 반복에서 n+1해서 3이 나왔다면, 1이나 2가아닌 3에 대해서만 다시 반복

재귀 SELECT가 여러 쿼리 블록을 포함하는 경우, 각 블록은 독립적으로 실행되며, 순서는 지정되지 않는다. 

### 열 참조 방식
열은 위치가 아니라 이름으로 참조된다.
- 즉, 재귀 SELECT와 비재귀 SELCT의 열 순서가 달라도 이름에 대해 작업이 수행된다.

```MYSQL
WITH RECURSIVE cte AS (
  SELECT 1 AS n, 1 AS p, -1 AS q
  UNION ALL
  SELECT n + 1, q * 2, p * 2 FROM cte WHERE n < 5
)
SELECT * FROM cte;
```

**결과**
| n | p  | q   |
| - | -- | --- |
| 1 | 1  | -1  |
| 2 | -2 | 2   |
| 3 | 4  | -4  |
| 4 | -8 | 8   |
| 5 | 16 | -16 |

### ❌ 제약 조건

`재귀 SELECT안`에서는 아래와 같은 기능을 사용할 수 없음
- 집계 함수
- 윈도우 함수
- GROUP BY
- ORDER BY
- DISTINCT

`기타 제약`
- 재귀 SELECT는 CTE이름을 FROM 절에서만 한 번 사용해야한다.
- CTE외의 다른 데이블을 조인할 수 있지만, LEFT JOIN 을 했을 때, CTE가 우측에 위치하면 안됨

## WITH RECURSIVE 주 사용 예시

### 1. 피보나치 수열 생성

피보나치 수열을 0과 1 (혹은 1과 1)로 시작해서, 그 이후의 각 숫자는 앞의 두 숫자를 더해가는 수열을 말한다.

ex: 0,1,1,2,3,5,8 ...

따라서 재재귀 쿼리에서 현재값과 이전값을 계속 저장하면서 이를 이용해 피보나치 수열을 만들 수 있다. 

```MYSQL
WITH RECURSIVE fibonacci (n,fib_n, next_fib_n) AS(
    SELECT 1, 0, 1
    UNION ALL
    SELECT n + 1, next_fib_n, fib_n + next_fib_n
    FROM fibonacci 
    WHERE n <10>
)
SELECT * FROM fibonacci;
```

**결과**
```markdown
+------+-------+------------+
| n    | fib_n | next_fib_n |
+------+-------+------------+
|    1 |     0 |          1 |
|    2 |     1 |          1 |
|    3 |     1 |          2 |
|    4 |     2 |          3 |
|    5 |     3 |          5 |
|    6 |     5 |          8 |
|    7 |     8 |         13 |
|    8 |    13 |         21 |
|    9 |    21 |         34 |
|   10 |    34 |         55 |
+------+-------+------------+
```

- n: 몇 번째 항인지 알려주는 인덱스
- fib_n: n번째의 피보나치 수
- next_fib_n: 다음 피보나치 수 (n+1번째)

#### 비재귀 SELECT(초기 행)
```mysql
SELECT 1,0,1
```

- 1번째 항 : fib_n = 0, next_fib_n = 1

#### 재귀 SELECT(반복 실행되는 행)
```MYSQL
SELECT n+1, next_fib_n, fib_n+ next_fib_n
```
- n+1 : 항 번호 증가
- 형재 항의 next_bin_n이 다음 항에서는 fib_n이 됨
- 다음 항의 next_fib_n은 `fib_n + next_fib_n`

#### 재귀 종료 조건
```MYSQL
WHERE n < 10
```
n이 10보다 작을 때까지만 반복하므로, 총 10개의 항을 생성함함

#### 특정 항만 조회하기
```MYSQL
SELECT fib_n 
FROM fibonacci 
WHERE n = 8;
```
```
| fib_n |
| ------ |
| 13     |
```
8번째 피보나치 수가 무엇인지 한 번에 조회할 수 있음


### 2. 날짜 시퀀스 생성

매일의 매출을 요약하는 보고서를 만들 때, 매출이 없는 데이터는 누락될 수가 있다.

이런 경우를 대비해서, 모든 날짜를 생성하고, 거기에 매출 데이터를 JOIN하면 누락없이 날짜별 매출을 확인할 수 있다.


#### 📊 매출 데이터
```MYSQL
SELECT * FROM sales ORDER BY date, price;
```
| date       | price  |
| ---------- | ------ |
| 2017-01-03 | 100.00 |
| 2017-01-03 | 200.00 |
| 2017-01-06 | 50.00  |
| 2017-01-08 | 10.00  |
| 2017-01-08 | 20.00  |
| 2017-01-08 | 150.00 |
| 2017-01-10 | 5.00   |

#### 📊 날짜별 매출 요약 쿼리
```MYSQL
SELECT date, SUM(price) AS sum_price
FROM sales
GROUP BY date
ORDER BY date;
```
| date       | sum\_price |
| ---------- | ---------- |
| 2017-01-03 | 300.00     |
| 2017-01-06 | 50.00      |
| 2017-01-08 | 180.00     |
| 2017-01-10 | 5.00       |

✔️ 1월 4일,5일,7일,9일은 매출이 없어서 누락된 것으로 보임

#### 📊해결방법: 날짜 시퀀스 생성 + LEFT JOIN 

```MYSQL
WITH RECURSIVE dates (date) AS (
    SELECT MIN(date) FROM sales
    UNION ALL
    SELECT date + INTERVAL 1 DAT FROM dates
    WHERE date + INTERVAL 1 DAY <= (SELECT MAX(date) FROM sales)
)

SELECT dates.date, COALESCE(SUM(price), 0) AS sum_price
FROM dates
LEFT JOIN sales ON dates.date = sales.date
GROUP BY dates.date
ORDER BY dates.date;
```
- SELECT MIN(date) : 가장 이른 날짜로 시작
- UNION ALL SELCT date + 1 DAY : 하루씩 더한 날짜 생성
- WHERE date <= MAX(date) : 마지막 날짜까지만 생성
- LEFT JOIN sales: 날짜 시퀀스에 매출 데이터를 연결(없으면 NULL값 처리됨)
- COALESCE(SUM(price),0): 매출이 없는 날짜는 0으로 처리

#### 📊 최종결과
| date       | sum\_price |
| ---------- | ---------- |
| 2017-01-03 | 300.00     |
| 2017-01-04 | 0.00       |
| 2017-01-05 | 0.00       |
| 2017-01-06 | 50.00      |
| 2017-01-07 | 0.00       |
| 2017-01-08 | 180.00     |
| 2017-01-09 | 0.00       |
| 2017-01-10 | 5.00       |


### 3. 조직 구조 탐색

재귀적 CTE는 계층 구조를 가진 데이터를 탐색하는데 유용하다. 

조직도, 카테고리 트리, 폴더구조처럼 상-하위 관계가 있는 데이터를 한 번에 쭉 타고 내려갈 때 `WITH RECURSIVE`를 사용하면 효율적이다.

#### 📊예시 테이블 - employees
```mysql
CREATE TABLE employees (
  id         INT PRIMARY KEY NOT NULL,
  name       VARCHAR(100) NOT NULL,
  manager_id INT NULL,
  INDEX (manager_id),
  FOREIGN KEY (manager_id) REFERENCES employees (id)
);
```
```mysql
INSERT INTO employees VALUES
(333, "Yasmina", NULL),  -- CEO
(198, "John", 333),
(692, "Tarek", 333),
(29, "Pedro", 198),
(4610, "Sarah", 29),
(72, "Pierre", 29),
(123, "Adil", 692);
```
| id   | name    | manager\_id |
| ---- | ------- | ----------- |
| 333  | Yasmina | NULL        |
| 198  | John    | 333         |
| 692  | Tarek   | 333         |
| 29   | Pedro   | 198         |
| 4610 | Sarah   | 29          |
| 72   | Pierre  | 29          |
| 123  | Adil    | 692         |

#### 📊 CEO → 각 지원까지의 경로 생성

```
WITH RECURSIVE employee_paths (id,name,path) AS (
    SELECT id, name, CAST(id AS  CHAR(200))
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
    FROM employee_paths AS ep
    JOIN employees AS e ON ep.id = e.manager_id
)
SELECT * FROM employee_paths
ORDER BY path;
```
`SELECT id, name, CAST(id AS  CHAR(200)) FROM employees   WHERE manager_id IS NULL`
- manager_id IS NULL이라는 것은 CEO라는 것.

    ➡️ CEO 정보

`JOIN employees AS e ON ep.id = e.manager_id`
- ep : 상사의 정보
- ep.id: 상사의 ID
- e.manager_id: 직원이 보고 있는 상사의 ID

    ➡️ 누가 ep를 상사로 갖고 있는가를 e에서 찾아서 join하는 과정

`SELECT ~ CONCAT(ep.path, ',', e.id)`
- 경로에 현재 직원 ID를 붙임

#### 출력결과
| id   | name    | path            |
| ---- | ------- | --------------- |
| 333  | Yasmina | 333             |
| 198  | John    | 333,198         |
| 29   | Pedro   | 333,198,29      |
| 4610 | Sarah   | 333,198,29,4610 |
| 72   | Pierre  | 333,198,29,72   |
| 692  | Tarek   | 333,692         |
| 123  | Adil    | 333,692,123     |


#### 조직도 구조
```SCSS
Yasmina(333)
├── John(198)
│   └── Pedro(29)
│       ├── Sarah(4610)
│       └── Pierre(72)
└── Tarek(692)
    └── Adil(123)
```

#### 특정 직원만 조회
```
SELECT * FROM employee_paths
FROM id IN (692,4610)
ORDER BY path;
```
| id   | name  | path            |
| ---- | ----- | --------------- |
| 4610 | Sarah | 333,198,29,4610 |
| 692  | Tarek | 333,692         |

✔️ ID가 692, 4610에 속하는 직원들의 상사경로를 포함하는 정보를 출력하는 쿼리
