# Presto 성능 튜닝

1. [튜닝 팁](#튜닝-팁)
2. [필요한 열 지정](#필요한-열-지정)
3. [시간 기반 파티셔닝 활용](#시간-기반-파티셔닝-활용)
4. [GROUP BY 절 내의 카디널리티 고려](#GROUP-BY-절-내의-카디널리티-고려)
5. [ORDER BY와 함께 LIMIT 사용](#ORDER-BY와-함께-LIMIT-사용)
6. [count(distinct x) -> approx_distinct()](#count(distinct-x)-->-approx_distinct())
7. [like -> regexp_like()](#like-->-regexp_like())
8. [Presto 조인 및 정렬 알고리즘 선택](#Presto-조인-및-정렬-알고리즘-선택)

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

### 시간을 정수로 지정
- 필드가 WHERE 절 내에 지정 되면 `time`쿼리 파서는 처리해야 하는 파티션을 자동으로 감지합니다. 대신에 시간을 지정하면 이 자동 감지가 작동 하지 않습니다 .`float` `int`
    ```SQL
    SELECT field1, field2, field3 FROM tbl WHERE time > 1349393020 -- good
    SELECT field1, field2, field3 FROM tbl WHERE time > 1349393020 + 3600 -- good
    SELECT field1, field2, field3 FROM tbl WHERE time > 1349393020 - 3600 -- good
    SELECT field1, field2, field3 FROM tbl WHEREtime > ${last_proceessed_time} -- good 
    SELECT field1, field2, field3 FROM tbl WHERE time > 13493930200 / 10 -- bad
    SELECT field1, field2, field3 FROM tbl WHERE time > 1349393020.00 -- bad
    SELECT field1, field2, field3 FROM tbl WHERE time BETWEEN 1349392000 AND 1349394000 -- bad
    SELECT field1, field2, field3 FROM tbl WHEREtime > (SELECT MAX(last_updated) FROM tbl2) -- bad 
    ```

### TD-TIME-RANGE 사용
- TD_TIME_RANGE를 사용하여 데이터를 분할할 수 있음
    ```SQL
    SELECT ... WHERE TD_TIME_RANGE(time, '2013-01-01 PDT') -- good
    SELECT ... WHERE TD_TIME_RANGE(time, '2013-01-01','PDT', NULL) -- good
    SELECT ... WHERE TD_TIME_RANGE(time, '2013-01-01', TD_TIME_ADD('2013-01-01', '1day', 'PDT')) -- good
    ```
- 그러나 TD_TIME_RANGE에서 나누기를 사용하면 시간 파티션 최적화가 작동하지 않습니다. 
    - 예를 들어 Treasure Data는 최적화를 비활성화하므로 SQL 구성을 권장하지 않습니다.

## GROUP BY 절 내의 카디널리티 고려
- GROUP BY 내의 필드 목록을 높은 카디널리티 순서로 신중하게 정렬하여 GROUP BY 기능의 성능을 향상시킬 수 있습니다.

    ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/2e3dddcc-dac0-4aa5-97fd-c2e7c2a45b66)

- 또는 GROUP BY 열에 문자열 대신 숫자를 사용하십시오. 숫자는 문자열보다 적은 메모리를 필요로 하고 비교가 더 빠르기 때문입니다.

## ORDER BY와 함께 LIMIT 사용
- ORDER BY는 모든 행을 단일 작업자에게 보낸 다음 정렬하도록 요구합니다. ORDER BY는 종종 Presto 작업자에 많은 메모리를 요구할 수 있습니다. 상위 또는 하위 N개의 레코드를 조회하려면 정렬 비용과 메모리 압력을 줄일 수 있는 LIMIT를 사용하십시오.
    
    ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/236fc25b-5a34-4843-9b0a-5646fe1961b9)

## count(distinct x) -> approx_distinct()
- Presto 에는 상당한 성능 향상을 제공하지만 약간의 오류가 있는 몇 가지 근사 집계 함수가 있습니다. 예를 들어 approx_distinct() 함수를 사용하면 표준 오차가 2.3%인  COUNT(DISTINCT x)의 근사값을 얻을 수 있습니다. 다음 예는 전날 순 사용자의 대략적인 수를 제공합니다.

    ```SQL    
    SELECT
      approx_distinct(user_id)
    FROM
      access
    WHERE
      TD_TIME_RANGE(time,
        TD_TIME_ADD(TD_SCHEDULED_TIME(), '-1d', 'PDT'),
        TD_SCHEDULED_TIME())
    ```

## like -> regexp_like()
- Presto의 쿼리 최적화 프로그램은 많은 LIKE 절이 포함된 쿼리를 개선할 수 없습니다. 결과적으로 쿼리 실행이 예상보다 느려질 수 있습니다. 성능을 향상시키기 위해 OR 조건으로 연결된 일련의 LIKE 절을 Presto 고유의 단일 regexp_like 절로 대체할 수 있습니다.

    ```SQL
    -- like    
    SELECT
      ...
    FROM
      access
    WHERE
      method LIKE '%GET%' OR
      method LIKE '%POST%' OR
      method LIKE '%PUT%' OR
      method LIKE '%DELETE%'
    
    -- regexp_like() 
    SELECT
      ...
    FROM
      access
    WHERE
      regexp_like(method, 'GET|POST|PUT|DELETE')
    ```

## Presto 조인 및 정렬 알고리즘 선택

### 조인 알고리즘
- 조인 배포에는 두 가지 유형이 있습니다.
    - **Partitioned**: 쿼리에 참여하는 각 노드는 데이터의 일부에서 해시 테이블을 작성합니다.
    - **Broadcast**: 쿼리에 참여하는 각 노드는 모든 데이터에서 해시 테이블을 작성합니다(데이터는 각 노드에 복제됨).

- Presto에서 사용되는 기본 조인 알고리즘은 분산된 PARTITION 조인입니다. 이 알고리즘은 조인 키의 해시 값을 사용하여 왼쪽 테이블과 오른쪽 테이블을 모두 분할합니다. 분할된 조인은 쿼리를 처리하기 위해 여러 작업자 노드를 사용합니다. 일반적으로 더 빠르고 더 적은 메모리를 사용합니다.

- 조인 테이블 중 하나가 매우 작은 일부 경우 분산 PARTITION 조인을 사용하여 네트워크를 통해 데이터를 분할하는 오버헤드가 조인 작업에 참여하는 모든 노드에 전체 테이블을 브로드캐스트하는 이점을 초과할 수 있습니다. 이러한 경우 'BROADCAST' 조인이 더 잘 수행될 수 있습니다. BROADCAST 조인을 사용하는 경우 조인 절에서 먼저 큰 테이블을 지정하십시오. 다음 매직 코멘트를 지정하여 'BROADCAST' 조인을 활성화할 수 있습니다.

    ```SQL    
    -- set session join_distribution_type = 'BROADCAST'
    ```
    - 이 옵션은 올바른 조인 테이블이 모든 노드에 복사되므로 더 많은 메모리를 사용합니다.
