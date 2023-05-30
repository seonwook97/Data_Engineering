# Presto 성능 튜닝

1.[튜닝 팁](#튜닝-팁)

---

## 튜닝 팁
- 항상 `TD_TIME_RANGE` 또는 `TD_INTERVAL` 사용 (Scheduling Presto 쿼리에 대한 링크 포함)
    - https://api-docs.treasuredata.com/en/tools/presto/api/#td_time_range
- 다음과 같이 느린 메모리 소모 연산자를 사용하지 마십시오.
    - `ORDER BY`
    - `COUNT(DISTINCT x)`
- `COUNT(DISTINCT x)`를 사용하는 대신 `approx_distinct(x)`를 사용
    - Join processing
        - 큰 테이블부터 작은 테이블 순으로 테이블을 조인해야 합니다.
        - 동등하지 않은 조인 조건을 사용하면 쿼리 처리 속도가 느려집니다.
    - Columnar storage characteristics
        - 너무 많은 열을 선택하면 쿼리 처리 속도가 느려집니다.
    - Query result size
        - 너무 많은 행을 생성하려면 시간이 걸립니다. 대신 `CREATE TABLE AS...` 또는 `INSERT INTO` 또는 `result_output_redirect` 매직 코멘트를 사용하십시오 .

## 필요한 열 지정
- Treasure Data의 실제 데이터는 특정 컬럼만을 사용하는 쿼리에 최적화된 컬럼 스토리지 형식으로 저장됩니다. 액세스되는 열을 제한하면 쿼리 성능이 크게 향상될 수 있습니다. 와일드카드(*)를 사용하는 대신 필요한 열만 지정하십시오.
```SQL
SELECT time,user,host FROM tbl -- good
SELECT * FROM tbl -- bad
```

## 시간 기반 파티셔닝 활용
- 가져온 모든 데이터는 각 데이터 레코드 내의 필드를 기반으로 시간별 버킷으로 자동 분할됩니다 `time`.
- 쿼리에서 시간 범위를 지정하면 불필요한 데이터 읽기를 피할 수 있으므로 쿼리 속도를 크게 높일 수 있습니다.
