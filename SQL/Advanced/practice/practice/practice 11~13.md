# 우유와 요거트가 담긴 장바구니 찾기
## 요구사항
우유와 요거트를 동시에 구입한 장바구니의 아이디를 조회하는 SQL 문을 작성해주세요. 이때 결과는 장바구니의 아이디 순으로 나와야 합니다.

## 작성한 쿼리

```MYSQL
SELECT CART_ID
FROM CART_PRODUCTS
GROUP BY CART_ID
HAVING GROUP_CONCAT(NAME) LIKE '%Yogurt%' AND
    GROUP_CONTCAT(NAME) LIKE '%Milk%
ORDER BY CART_ID;
```

## 배운 점(오답노트)

**처음 작성한 코드**
```MYSQL
SELECT CART_ID
FROM CART_PRODUCTS
WHERE GROUP_CONCAT(NAME) IN 'Yogurt' AND
    GROUP_CONTCAT(NAME) IN 'Milk'
GROUP BY CART_ID
ORDER BY CART_ID;
```

📝 `GROUP_CONCAT()`은 집계함수

- 일반적으로는 `GROUP BY`와 함께 사용됨됨
- 그룹단위 함수 → GROUP BY 이후에 HAVING절에서 사용
    - WHERE절은 행단위 조건

📝 IN은 **리스트와 함께 사용**해서 리스트 안에 원하는 값이 있는지 확인 + 문자열 전체가 정확히 일치하는지를 파악할 때 사용 
  
```MYSQL 
A IN ('Milk','Yogurt')
```
- MILK만 있어도 TRUE
- Yogurt만 있어도 TRUE
- MILK, YOGURT → ❌

내가 원하는 것은 GROUP_CONCAT한 값 안에 MILK와 YOUGURT가 동시에  ❗포함❗되어있는지를 확인하는 것

➡️ LIKE사용

--------

# 언어별 개발자 분류하기

## 요구사항

## 작성한 쿼리

## 배운점

### 📝비트연산자 정리(복습)

| 연산자 | 이름     | 예시 (a=5, b=3)               | 결과 비트값 |
|--------|----------|-------------------------------|--------------|
| &    | AND      | 5 & 3 = 101 & 011 = 001 | 1            |
| `|`    | OR       | 5 \| 3 = 101 \| 011 = 111 | 7            |
| ^    | XOR      | 5 ^ 3 = 101 ^ 011 = 110 | 6            |


