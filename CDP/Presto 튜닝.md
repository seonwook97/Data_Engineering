# Presto 성능 튜닝

1.[튜닝 팁](#튜닝-팁)

---

## 튜닝 팁
- 항상 [TD_TIME_RANGE]('https://api-docs.treasuredata.com/en/tools/presto/api/#td_time_range') 또는 TD_INTERVAL 사용 (Scheduling Presto 쿼리에 대한 링크 포함)
- 다음과 같이 느린 메모리 소모 연산자를 사용하지 마십시오.
    - ORDER BY
    - COUNT(DISTINCT x)