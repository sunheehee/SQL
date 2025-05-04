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

