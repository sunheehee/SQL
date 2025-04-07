# CASE & IF
# CASE

## CASE문법

**#1 Case value**
```MYSQL
CASE value
    WHEN compare_value1 THEN result1
    [WHEN compare_value2 THEN result2...]
    [ELSE result]
END
```
- value = compare_value를 만족하는 result값을 반환
- 일치하는 값이 없으며 ELSE에 해당하는 result값 반환
- ELSE도 없으면 NULL반환

**#2 CASE WHEN**
```MYSQL
CASE 
    WHEN 조건 THEN result
    [WHEN whrjs THEN result]
    [ELSE result]
END
```
- 조건이 TRUE를 만족하는 첫번째 case문의 result를 반환
- 모두 FALSE라면 ELSE의 값을 반환
- ELSE없으면 NULL반환

## CASE 반환 타입 결정 규칙

- 모두 숫자 → 숫자
    - 하나라도 DOUBLE → DOUBLE
    - 하나라도 DECIMAL → DECIMAL
    - 모두 정수이면 → 가장 큰 범주의 정수형
    
        ✅ 모두 섞여있다면 타입결정의 우선순위는 `DOUBLE > DECIMAL > 정수 > 문자열... 순서`로 결정됨

- 모두 문자열 → VARCHAR(가장 긴 문자열 길이 기준)
- 하나라도 BLOB → BLOB
- DATE/TIME/TIMESTAMP → DATETIME
- 그 외는 기본적으로 VARCHAR
- NULL은 타입 결정에서 무시됨

--- 

# IF

## IF(expr1, expr2, expr3)
####  expr1이 참이면  expr2 반환 / 거짓이면 expr3 반환


```MYSQL
SELECT IF (1>2,2,3)  #results: 3
```

⚠️**헷갈리지 말아야할 것**

IF()는 함수이고, `IF 조건 THEN RESULT ... ELSE...END` 는 제어문(즉,문장!)이라서 다르다
-  `IF()`는 이 쿼리문을 통해서 <U>**값을 반환하는것이 목적인 반환용 함수**</U>
- `IF제어문`은 <U>**흐름을 제어**하는 것으로 스토어드 프로시저, 트리거등에서 사용되</U>고, 일반 SELECT안에서는 사용할 수 없다.

    🌱 **IF문 사용예시**
    ```MYSQL
    # 프로지서
    DELIMITER $$
    CREATE PROCEDURE gradeCheck(score INT)
    BEGIN
    IF score >= 90 THEN
        SELECT 'A';
    ELSEIF score >= 80 THEN
        SELECT 'B';
    ELSE
        SELECT 'C';
    END IF;
    END //
    DELIMITER ;
    ```


## IFNULL(expr1, expr2)
#### expr1이 NULL이 아니면 expr1 반환 / NULL이면 expr2반환

```MYSQL
SELECT IFNULL(1,0) 
# results: 1
```
```MYSQL
SELECT IFNULL(NULL,10)
#results: 10
```
```MYSQL
SELECT(1/0, 'YES')
#results: YES
```

## NULLIF(expr1, expr2)
#### expr1 = expr2 이면 NULL 아니면 expr1 반환

```MYSQL
SELECT NULLIF(1,1) #results: NULL

----

SELECT NULLIF(1,2)
#results: 1
```

---

# 비교, 논리 연산자

 ## 🌱 기본연산자들의 사용 예제

 ### 💡 <=> : NULL도 비교가능한 연산자
```MYSQL
SELECT 1<=>1, NULL <=> NULL, 1<=>NULL #results: 1,1,0

#기존 등호 연산자 사용시와의 차이점
SELECT 1=1, NULL=NULL, 1=NULL #results: 1, NULL, NULL
```
- 두 피연산자가 모두 NULL인 경우, 1을 반환
- 하나의 피연산자만 NULL인경우에는 0을 반환
- 일반 등호(=)로는 NULL을 비교하는데 사용할 수 없다 

++ 행 비교의 경우, `(a,b) <=> (x,y)`는 다음과 같다
```MYSQL
(a<=>x) AND (b<=>y)
```
---

 ### 💡 <>, != : 같지 않음 비교
 ```MYSQL
 SELECT '01' <> '0.01'; 
#results: 1
```
- 1:참 ➡️ 같지않음이 참이니까 다르다.
- 두 피연산자는 모두 문자열로, 문자 자체가 다르기 때문에 1이 반환

```mysql
SELECT .01 <> '0.01'
#results:0
```
- 0: 거짓 ➡️ 같지 않음이 거짓이니까 같다!
- .01은 숫자형, 0.01은 문자형이지만, 문자열 0.01은 암묵적으로 숫자로 변환되어 비교가 됨(MYSQL)

++ 행 비교의 경우, `(a,b) <> (x,y)`는 다음과 같다
```
(a<>x) OR (b<>y)
```

---

 ### 💡 BETWEEN
 `expr BETWEEN min AND max`
 #### expr이 min보다 크거나 같고, max보다 작거나 같으면 1 반환 OR 0

 ```mysql
 SELECT 2 BETWEEN 3 AND 1;
-- 결과: 0
 ```
 - BETWEEN은 항상 왼쪽 값이 작고, 오른쪽 값이 커야한다.
 
 📝 BETWEEN을 날짜나 시간값과 함께 사용할 때, CAST()를 사용해서 값을 원하는 데이터 타입으로 명시적으로 변환하는것이 좋다.

 **CAST사용 예시**
 ```MYSQL
 -- 테이블 생성
CREATE TABLE events (
    event_id INT,
    event_time DATETIME
);

-- 데이터 삽입
INSERT INTO events VALUES
(1, '2025-04-01 10:30:00'),
(2, '2025-04-02 14:00:00'),
(3, '2025-04-03 09:00:00');

-- 날짜 범위로 DATETIME 비교 (CAST 없이 비교 시 오류 또는 잘못된 결과 가능)
-- 명시적으로 DATE → DATETIME으로 변환해서 비교
SELECT *
FROM events
WHERE event_time BETWEEN
      CAST('2025-04-01' AS DATETIME)
  AND CAST('2025-04-02' AS DATETIME);
```
**🫧정리**

CAST()를 이용해서 비교하고자하는 입력값을 기존 테이블의 형식에 맞추어 변환시키면 오류 혹은 잘못된 결과 출력없이 잘 비교할 수 있음. 

---

### 💡 COALESCE(value,...)
#### 리스트 안에서 NULL이 아닌 첫 번째 값을 반환한다.

```MYSQL
SELECT COALESCE(NULL, 1);
-- 결과: 1

SELECT COALESCE(NULL, NULL, NULL);
-- 결과: NULL
```
- 리스트 안의 값이 모두 NULL이면 NULL을 반환함

---

### 💡 EXISTS(query) 
#### 주어진 쿼리의 결과에 행이 하나라도 존재하면 1 반환

```MYSQL
CREATE TABLE t (col VARCHAR(3));
INSERT INTO t VALUES ('aaa', 'bbb', 'ccc', 'eee');

SELECT EXISTS (SELECT * FROM t WHERE col LIKE 'c%');
-- 결과: 1 (행 있음)

SELECT EXISTS (SELECT * FROM t WHERE col LIKE 'd%');
-- 결과: 0 (행 없음)
```

---

### 💡 NOT EXISTS(query)
#### EXISTS와 반대로 쿼리의 결과에 행이 하나라도 존재하면 0 반환 

```MYSQL
SELECT NOT EXISTS (SELECT * FROM t WHERE col LIKE 'c%');
-- 결과: 0 (행 있음 -> 거짓)

SELECT NOT EXISTS (SELECT * FROM t WHERE col LIKE 'd%');
-- 결과: 1 (행 없음 -> 참)
```

---

### 💡 GREATEST(val1, val2, ...)
#### 둘 이상의 인자 중 가장 큰 값을 반환함

```MYSQL
SELECT GREATEST(2, 0);
-- 결과: 2

SELECT GREATEST(34.0, 3.0, 5.0, 767.0);
-- 결과: 767.0

SELECT GREATEST('B', 'A', 'C');
-- 결과: 'C'
```
- 하나라도 NULL이 포함되면 NULL 반환

---

### 💡 expr IN (value, ...)
#### expr이 괄호 안 리스트 중 하나와 일치하면 1 (true)
```mysql
SELECT 2 IN (0, 3, 5, 7);
-- 결과: 0
```

✅ 튜플 비교 가능
```mysql
SELECT (3, 4) IN ((1, 2), (3, 4));
-- 결과: 1

SELECT (3, 4) IN ((1, 2), (3, 5));
-- 결과: 0
```

⚠️ 문자형과 숫자형의 혼합 사용 주의
```mysql
SELECT val1 FROM tbl1 WHERE val1 IN (1,2,'a');  -- ❌ 혼합 사용
SELECT val1 FROM tbl1 WHERE val1 IN ('1','2','a');  -- ✅ 일관된 문자열 사용
```
```mysql
SELECT 'a' IN (0), 0 IN ('b');
-- 결과: 1, 1
```
- 'a'와 'b'는 숫자로 변환불가 → 0으로 간주되기 때문에 결과가 1로 반환

---

### 💡 INTERVAL(N, N1, N2, N3, ... )
#### N이 Nn이하이면 n-1 반환

```mysql
SELECT INTERVAL(23, 1, 15, 17, 30, 44, 200);
-- 결과: 3  (23은 17보다 크고 30 이하)

SELECT INTERVAL(10, 1, 10, 100, 1000);
-- 결과: 2  (10은 10 이하 → 인덱스 2)

SELECT INTERVAL(22, 23, 30, 44, 200);
-- 결과: 0  (22는 23보다 작음)
```

**추가설명**

- N이 N1 이하이면 0
- N이 N2 이하이면 1
- N이 Nn 이하이면 n-1
- N이 NULL이면 -1
- 모든 인자는 정수 처리됨

⚠️오름차순 형태로 정렬되어있어야 정상작동함.

---

### 💡 IS (TRUE/FALSE/UNKNOWN)
#### value가 TRUE, FALSE, UNKNOWN 중 하나일 때, 값이 불린과 일치하는지 테스트

```MYSQL
SELECT 1 IS TRUE, 0 IS FALSE, NULL IS UNKNOWN;
-- 결과: 1, 1, 1
```
---
### 💡 ISNULL(expr)
#### expr이 NULL이면 1을 반환하고, 그렇지 않으면 0 반환 
```MYSQL
SELECT ISNULL(1+1);
-- 결과: 0

SELECT ISNULL(1/0);
-- 결과: 1   (0으로 나누면 NULL이 되니까)
```