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
