# 문제풀이
## #1. 
### 요구사항
식사 금액이 테이블 당 평균 식사 금액보다 더 많은 경우를 모두 출력하는 쿼리를 작성해주세요

### 작성한 쿼리
```
SELECT *
FROM tips
WHERE total_bill > (SELECT AVG(total_bill) FROM tips);
```

## #2.
### 요구사항
### 작성한 쿼리