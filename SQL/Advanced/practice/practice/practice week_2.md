# 문제 풀이
## #1. DENSE_RANK() 관련 문제 풀이

### 요구사항
**점수**가 높은 사람이 1등으로 오게 **내림차순 정렬**, 동점자는 **동일한 순위**를 부여하며 **순위번호는 건너뛰지 않고 연속되**어야한다.

### 작성한 쿼리  
```MYSQL
SELECT 
    score,
    DENSE_RANK() OVER (ORDER BY score DESC)
FROM Scores 
ORDER BY score;
```
![alt text](image/DENSE_RANK().png)


### 🌱배운 점 + 헷갈렸던 것
#### OVER()절 안의 PARTITION BY
> 그룹별로 윈도우 함수를 계산하고 싶을 때 그룹을 나누는 기준 설정

#### OVER()절 안의 ORDER BY
> 윈도우 함수 작동 순서서
EX) 
- `DENSE_RANK`, `RANK`, `ROW_NUMBER` → 순위부여
- `SUM() OVER (ORDER BY col)` → 누적합 계산

#### OVER()절 밖의 ORDER BY
> 최종 출력 결과의 정렬

---
## #2. LEAD 관련 문제 - 서울숲의 미세먼지지

### 요구사항
measurements 테이블의 pm10 컬럼에는 다양한 대기오염도 측정 기준 중에서도 미세먼지(PM10) 농도가 기록되어 있습니다. 이 데이터를 이용하여 당일의 미세먼지 농도보다 바로 다음날의 미세먼지 농도가 더 안좋은 날을 찾아주세요

💡다음날과 오늘을 비교 **농도가 더 높은 다음날을 출력** 
➡️ LEAD() 활용 문제

### 작성한 쿼리
```MYSQL
SELECT today,next_day,pm10,next_pm10
FROM (SELECT    
        measured_at as 'today',
        LEAD(measured_at) OVER (ORDER BY measured_at) as 'next_day',
        pm10,
        LEAD(pm10) OVER (ORDER BY measured_at) as 'next_pm10'
     FROM measurements
     ) AS tmr
WHERE next_pm10 > pm10
``` 

![alt text](image/LEAD()문제.png)

### 🌱배운 점 + 헷갈렸던 것

- LEAD()의 문법
`LEAD(컬럼) OVER (ORDER BY 정렬기준)`

- 서브쿼리의 이용
LEAD()를 통해 새로운 컬럼을 만들고, 그 컬럼을 조회하는 것이었기 때문에, LEAD()를 통해 새로운 컬럼을 만드는 서브쿼리를 먼저 만들고, 서브쿼리 테이블의 값을 조회하는 형식으로 쿼리를 짜는 것이 훨씬 편했다.

## #3. 프로리뷰어의 리뷰 출력
### 요구사항
 MEMBER_PROFILE와 REST_REVIEW 테이블에서 리뷰를 가장 많이 작성한 회원의 리뷰들을 조회하는 SQL문을 작성해주세요. 회원 이름, 리뷰 텍스트, 리뷰 작성일이 출력되도록 작성해주시고, 결과는 리뷰 작성일을 기준으로 오름차순, 리뷰 작성일이 같다면 리뷰 텍스트를 기준으로 오름차순 정렬해주세요.

### 작성한 쿼리
```MYSQL
SELECT MEMBER_NAME, REVIEW_TEXT, DATE_FORMAT(REVIEW_DATE,'%Y-%m-%d') as REVIEW_DATE
FROM REST_REVIEW R
JOIN MEMBER_PROFILE M
ON R.MEMBER_ID = M.MEMBER_ID
WHERE R.MEMBER_ID = (SELECT MEMBER_ID
                    FROM REST_REVIEW
                    GROUP BY MEMBER_ID
                    ORDER BY COUNT(*) DESC
                    LIMIT1
                  )
ORDER BY REVIEW_DATE, REVIEW_TEXT
```

![alt text](<image/프로리뷰어의 리뷰 출력.png>)

### 🌱배운 점 + 헷갈렸던 것
- 리뷰를 가장 많이 작성한 회원을 조회하려고 서브쿼리의 SELECT문에서 COUNT 집계를 했는데, 그러면 원본 쿼리의 WHERE절에서 비교하는 컬럼의 수와 맞지 않아서 고민을 했다.
- ORDER BY에서도 COUNT집계를 사용할 수 있다는 걸 잊고 있었다.

# 모든 문제 해결!
![alt text](image/핑구에옹.png)
