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

**#1 CASE WHEN**
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
#### expr1 = expr2 이면 NULL 아니면 expr1 반환환

```MYSQL
SELECT NULLIF(1,1) #results: NULL

----

SELECT NULLIF(1,2)
#results: 1
```

---

# 비교, 논리 연산자

💡 `<=>` : NULL도 비교가능한 연산자

 ## 🌱 기본연산자들의 사용 예제
