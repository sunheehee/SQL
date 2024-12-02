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

```
SELECT
  type1,
  kor_name,
  count(tp.id) AS cnt --- ❓왜 tp.pokemon_id가 아닌 tp.id를 보는건지?(결과는 같던데)
FROM basic.trainer_pokemon AS tp
LEFT JOIN basic.pokemon AS p
ON tp.pokemon_id = p.id
GROUP BY
  type1,
  kor_name
LIMIT 1
```

### 쿼리 결과

![alt text](<../image/8주차/문제2 쿼리 결과.png>)

---
## 문제 3

전설의 포켓몬을 보유한 트레이너들은 전설의 포켓몬과 일반 포켓몬을 얼마나 보유하고 있을까요? (트레이너의 이름을 같이 출력해주세요)

```
👾FLOW
1. trainer_pokemon + pokemon => 전설T/F -> 전설 여부에 따른 개수 파악(COUNT) 

2. + trainer JOIN해서 trainer 이름 출력

👾JOIN KEY
pokemon.id = trainer_pokemon.pokemon_id, (이전table).trainer_id = trainer.id
```
```
1. 전설,일반별 개수 구하는 식

SELECT
  tp.trainer_id,
  SUM(CASE
    WHEN P.is_legendary IS TRUE THEN 1 ELSE 0 END)AS legendary_cnt,
  SUM(CASE
    WHEN not P.is_legendary THEN 1 ELSE 0 END)AS normal_cnt
FROM basic.trainer_pokemon AS tp 👾--- trainer&pokemon 의 id가 동시에 있는 컬럼 >> 기준이 되는 컬
LEFT JOIN basic.pokemon AS p
ON tp.pokemon_id=p.id
WHERE  
  tp.status in ("Active","Training")
GROUP BY
  tp.trainer_id

❗ SUM(CASE WHEN)함수는 집계함수이므로 GROUP BY와 함께 사용
❗ CASE WHEN을 작성할 수 있는 경우가 많이 있음
1) CASE WHEN ~ (IS TRUE) THEN
2) CASE WHEN NOT ~ THEN 은 CASE WHEN ~ IS NOT TRUE THEN 과 동일함
```
```
2. with 문으로 묶기

SELECT
  t.name as trainer_name,
  c.legendary_cnt,
  c.normal_cnt
FROM basic.trainer as t 👾-- 정보가 필요한 것만, 최소한으로 갖추고 있는 테이블을 기준 테이블로 정함
LEFT JOIN legendary_cnt as c
ON t.id = c.trainer_id
WHERE 
  c.legendary_cnt >= 1 👾-- '전설포켓몬을 보유하고 있는'이라는 조건을 만족하기 위해서 
```

### 쿼리 결과

![alt text](<../image/8주차/문제3 쿼리 결과.png>)

# 총 정리 문제 풀이 4~6
## 문제 4
가장 승리가 많은 트레이너 ID, 트레이너 이름, 승리한 횟수, 보유한 포켓몬의 수, 평균 포켓몬의 레벨을 출력해주세요. 단, 포켓몬의 레벨은 소수점 둘째자리에서 반올림해주세요.



```
👾FLOW

1. battle 테이블에서  winner_id를 count,

2. + trainer테이블에서 이름 

3. + trainer_pokemon 테이블에서 포켓몬 수, 포켓몬의 평균 레벨 출력(round)

👾Join KEY
 battle.winner_id = trainer.id, 
 => (). trainer_id=trainer_pokemon.trainer_id
```
**가장 승리가 많은**
  1) 이름을 추가한 결과에서 필터링 후, 가장 승리가 많은 trainer_id 1개만 뽑고, 그 id의 평균 포켓몬 레벨, 포켓몬 수 출력하는 방법
  2)  평균 포켓몬 레벨, 포켓몬 수 까지 다 구한 후의 trainer_id 1개만 뽑는 방법

  - 데이터가 많으면 1번 방법을 권장
  - 결과를 어떻게 사용할까에 따라서(추후에 top3도 보고싶고~) 바뀐다면 2번 추천
```
1️⃣ battle 테이블에서  winner_id를 count

SELECT
  winner_id,
  count(winner_id) as win_cnt
FROM basic.battle as b
where
  winner_id IS NOT NULL --- 👾winner_id가 null인 경우는 무승부인 경우라 제외.
group by
  winner_id

2️⃣ with문 형성 + trainer테이블에서 이름추가

WITH win_cnts AS(
  SELECT
    winner_id,
    count(winner_id) as win_count
  FROM basic.battle as b
  where
    winner_id IS NOT NULL --- winner_id가 null인 경우는 무승부인 경우라 제외.
  group by
    winner_id
)

--- (2) 트레이너 이름을 추가

SELECT
  wc.winner_id as trainer_id,
  wc.win_count,
  t.name as trainer_name
FROM win_cnts AS wc
LEFT JOIN basic.trainer AS t
ON wc.winner_id = t.id


3️⃣ 2번의 쿼리문 WITH화 + trainer_pokemon 테이블에서 포켓몬 수, 포켓몬의 평균 레벨 출력(round)

trainer_win AS (
  SELECT
    wc.winner_id as trainer_id,
    wc.win_count,
    t.name as trainer_name
  FROM win_cnts AS wc
  LEFT JOIN basic.trainer AS t
  ON wc.winner_id = t.id
  ORDER BY
    win_count DESC
  LIMIT 1 
)

SELECT
  tw.trainer_id,
  tw.trainer_name,
  tw.win_count,
  count(tp.pokemon_id) AS pokemon_cnt,
  ROUND(AVG(tp.level),2) AS avg_level

FROM trainer_win as tw
LEFT JOIN basic.trainer_pokemon as tp
ON tw.trainer_id = tp.trainer_id
WHERE 
  tp.status in ("ACTIVE","Training")
Group by
  tw.trainer_id,
  tw.trainer_name,
  tw.win_count
```

### 쿼리 결과

![alt text](<../image/8주차/문제4 쿼리 결과.png>)

## 문제 5
트레이너가 잡았던 포켓몬의 총 공격력(attack)과 방어력(defense)의 합을 계산하고, 이 합이 가장 높은 트레이너를 찾아라

![alt text](<../image/8주차/문제5 문제풀이과정.png>)

- attack컬럼과 defense컬럼을 합쳐 stat컬럼을 만드는 것은 단순 `attack + defense AS stat` 으로 표현 가능

- 하지만 1열과 2열을 합치는것은 `집계`기능을 활용해야함

  ![alt text](<../image/8주차/문제5 문제풀이과정2.png>)

  ```
  SELECT
  trainer_id,
  -- p.attack,
  -- p.defense,
  SUM(p.attack + p.defense) as total_stat
  FROM basic.trainer_pokemon as tp
  LEFT JOIN basic.pokemon as p
  ON tp.pokemon_id=p.id
  group by  
    trainer_id
  ```

```
위의 식 + trainer테이블과 JOIN해서 트레이너 이름 출력


SELECT
 t.name, 
 trainer_id,
 total_stat
FROM total_stats as ts
LEFT JOIN basic.trainer as t
ON ts.trainer_id = t.id
ORDER BY
  total_stat DESC
LIMIT 1
```

### 쿼리 결과

![alt text](<../image/8주차/문제5 쿼리 결과.png>)

## 문제 6
각 포켓몬의 최고 레벨과 최저 레벨을 계산하고, 레벨 차이가 가장 큰 포켓몬의 이름을 출력하세요.


```
1.포켓몬의 레벨 차이 구하기

SELECT
  p.kor_name,
  pokemon_id,
  Max(tp.level) AS max_level,
  MIN(tp.level) AS min_level,
  Max(tp.level) - MIN(tp.level) as level_diff
FROM basic.trainer_pokemon as tp
LEFT JOIN basic.pokemon as p
ON tp.pokemon_id = p.id

Group by
  p.kor_name,
  pokemon_id

❗WHERE조건으로 id를 지정해서 데이터 구조를 확인하면서 쿼리문을 작성하면 더 쉽게 이해할 수 있음


2. 1번 내용 WITH 문화 + 마지막 쿼리문은 간단하게

SELECT
  kor_name,
  level_diff
FROM level_difference as ld
ORDER BY
  level_diff DESC
LIMIT 1
```
### 쿼리 결과

![alt text](<../image/8주차/문제6 쿼리 결과.png>)

##  문제7.
각 트레이너가 가진 포켓몬 중에서 공격력(attack)이 100이상인 포켓몬과 100미만인 포켓몬의 수를 각각 계산해주세요. 트레이너의 이름과 두 조건에 해당하는 포켓몬의 수를 출력해주세요.

**👾-- 공격력이 100이상, 100미만인 것은 조건(where)인 것 같지만
열수준으로 합쳐야하는 것이기 때문에 COUNTIF를 사용함**

```
WITH not_released as (
  SELECT
      *
  FROM basic.trainer_pokemon as tp
  WHERE status != "Released"
),

trainer_high_and_low_attack_cnt AS (
  SELECT
    nr.trainer_id,
    countif(p.attack >= 100 ) as high_attack_cnt,
    countif(p.attack <= 100 ) as low_attack_cnt
    
  FROM not_released as nr
  LEFT JOIN basic.pokemon as p
  ON nr.pokemon_id = p.id
  GROUP BY
    nr.trainer_id
)

SELECT
 t.name,
 cnt.*
FROM trainer_high_and_low_attack_cnt as cnt
LEFT JOIN basic.trainer as t
ON cnt.trainer_id = t.id
```

### 쿼리 결과

![alt text](<../image/8주차/문제7 쿼리 결과.png>)

```
맞게 집계된 것인지 확인하는 쿼리문

SELECT
  nr.trainer_id,
  nr.pokemon_id,
  p.attack
FROM not_released as nr
LEFT JOIN basic.pokemon AS p
ON nr.pokemon_id = p.id
Where 
  trainer_id = 5

```
👾위의 쿼리를 시행한 후 결과를 보고 id하나를 골라서 where조건식에 적용해보는 코드임

# 강의 수강 완료 인증샷

![alt text](<../image/8주차/강의 수강완료 인증샷.png>)



![alt text](../image/8주차/헤ㅔㅎ.png)