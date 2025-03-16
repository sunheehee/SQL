# 문제풀이
## #1. 많이 주문한 테이블 찾기
### 요구사항
식사 금액이 테이블 당 평균 식사 금액보다 더 많은 경우를 모두 출력하는 쿼리를 작성해주세요

### 작성한 쿼리
```
SELECT *
FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips);
```

## #2. 레스토랑의 대목
### 요구사항
요일별 매출액 합계를 구하고, 매출이 1500 달러 이상인 요일의 결제 내역을 모두 출력하는 쿼리를 작성해주세요.

### 작성한 쿼리
```
SELECT * 
FROM tips
WHERE day IN (SELECT day
              FROM tips 
              GROUP BY day
              HAVING sum(total_bill)>=1500)
```




