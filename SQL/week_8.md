# 총 정리 문제 풀이 1~3

## 문제 1.
각 트레이너별로 가진 포켓몬의 평균 레벨을 계산하고, 그 중 평균레벨이 높은 TOP3 트레이너의 이름과 보유한 포켓몬의 수, 평균 레벨을 출력해주세요.

```
1. 트레이너가 보유한 포켓몬의 평균 레벨, 포켓몬의 수

SELECT
  trainer_id,
  ROUND(AVG (level)) As avg_level,
  count (id) As pokemon_cnt

FROM basic.trainer_pokemon
WHERE
  status != "Released" # released를 제외하고
Group By
  trainer_id  # 트레이너'별'


2. 1번과정의 식 WITH문化

WITH trainer_avg_level AS(
SELECT
  trainer_id,
  ROUND(AVG (level)) as avg_level,
  count (id) as pokemon_cnt

FROM basic.trainer_pokemon
WHERE
  status != "Released"
Group By
  trainer_id
)

3. 1번에서 만든 테이블 + TRAINER 테이블 합쳐서 TRAINER의 이름 출력, 평균레벨 순 3위의 트레이너 이름만 출력

SELECT
  t.name,
  tal.avg_level,
  tal.pokemon_cnt
FROM basic. trainer AS t
LEFT JOIN trainer_avg_level AS tal 👾with 문에서 정의한 이름 가져오기기
ON t.id=tal.trainer_id  👾Join할 기준 
ORDER BY 
  avg_level DESC
Limit 3
```

### 쿼리 결과

![alt text](<../image/8주차/문제1 쿼리 결과.png>)

---

## 문제 2
각 포켓몬 타입1을 기준을 가장 많이 포획된(방출여부 상관없음) 포켓몬의 타입1, 포켓몬의 이름과 포획횟수를 출력해주세요

