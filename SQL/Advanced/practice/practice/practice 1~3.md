# 문제풀이
## #1. 많이 주문한 테이블 찾기
### 요구사항
식사 금액이 테이블 당 평균 식사 금액보다 더 많은 경우를 모두 출력하는 쿼리를 작성해주세요

### 작성한 쿼리
```mysql
SELECT * 
FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips);
```
![alt text](<image/1 많이 주문한 테이블 찾기.png>)


## #2. 레스토랑의 대목
### 요구사항
요일별 매출액 합계를 구하고, 매출이 1500 달러 이상인 요일의 결제 내역을 모두 출력하는 쿼리를 작성해주세요.

### 작성한 쿼리
```mysql
SELECT * 
FROM tips
WHERE day IN (SELECT day
              FROM tips 
              GROUP BY day
              HAVING sum(total_bill)>=1500)
```
![alt text](<image/2 레스토랑의 대목.png>)


## #3.식품분류별 가장 비싼 식품의 정보 조회하기
### 요구사항
FOOD_PRODUCT 테이블에서 식품분류별로 가격이 제일 비싼 식품의 분류, 가격, 이름을 조회하는 SQL문을 작성해주세요. 이때 식품분류가 '과자', '국', '김치', '식용유'인 경우만 출력시켜 주시고 결과는 식품 가격을 기준으로 내림차순 정렬해주세요.

### 작성한 쿼리
><subquery 버전>
```mysql
SELECT CATEGORY, PRICE, PRODUCT_NAME
FROM FOOD_PRODUCT
WHERE (CATEGORY, PRICE) IN (SELECT CATEGORY, MAX(PRICE)
                    FROM FOOD_PRODUCT
                    WHERE CATEGORY IN ('과자','국','김치','식용유')
                    GROUP BY CATEGORY)
ORDER BY PRICE DESC;
```

><WTIH 버전>
```mysql
WITH MAX_PRICE_PRODUCT AS (
        SELECT CATEGORY, MAX(PRICE) AS MAX_PRICE
        FROM FOOD_PRODUCT 
        WHERE CATEGORY IN ('과자','국','김치','식용유')
        GROUP BY CATEGORY
)

SELECT FP.CATEGORY, FP.PRICE, FP.PRODUCT_NAME
FROM FOOD_PRODUCT FP
JOIN MAX_PRICE_PRODUCT MPR
ON FP.CATEGORY = MPR.CATEGORY AND FP.PRICE = MPR.MAX_PRICE
ORDER BY FP.PRICE DESC;
```
![alt text](<image/3-1 식품분류별 가장 비싼 식품의 정보 조회하기.png>)

❗with로 가상의 테이블을 만든 후에, 사용할 때에는 Join을 이용해서 두 개의 테이블을 묶어줘야한다. ON에서는 WITH문에서 조회하는 컬럼을 모두 이어줘야한다.

개인적으로는 서브쿼리를 이용한 쿼리문이 생각의 흐름과 유사하게 흘러가는 플로우라 더 직관적이고 이해하기 쉬운 것 같다.

![alt text](image/week_1_sql.png)
