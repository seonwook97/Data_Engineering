# Flattening nested arrays - 중첩 배열 평면화

1. [AWS Athena](#1-AWS-Athena)
2. [Presto Docs](#2-Presto-Docs)

---

## AWS Athena

### flatten
- 중첩 배열로 작업할 때 중첩 배열 요소를 단일 배열로 확장하거나 배열을 여러 행으로 확장해야 할 수 있음
- 중첩 배열의 요소를 값의 단일 배열로 병합하려면 `flatten`함수를 사용

```SQL
SELECT flatten(ARRAY[ ARRAY[1,2], ARRAY[3,4] ]) AS items
```

![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/29a95f21-4f6f-416e-92db-ad10c0acb861)

### cross join unnest
- 배열을 여러 행으로 평면화하려면 cross join과 unnest 함께 활용
  ```SQL
  WITH dataset AS (
  SELECT
    'data engineering' as department,
    ARRAY['seonwook', 'seonwook2', 'seonwook3', 'seonwook4'] as users
  )
  SELECT department, names FROM dataset
  CROSS JOIN UNNEST(users) as t(names)
  ```
  
  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/6fa54548-edd2-4354-b5ad-d73c7358dc14)

- 키-값 쌍의 배열을 평면화하려면 다음 예와 같이 선택한 키를 열로 바꿈
  ```SQL
  WITH
  dataset AS (
    SELECT
      'data engineering' as department,
       ARRAY[
        MAP(ARRAY['first', 'last', 'age'],ARRAY['seonwook', 'ko', '25']),
        MAP(ARRAY['first', 'last', 'age'],ARRAY['wookseon', 'ko', '26']),
        MAP(ARRAY['first', 'last', 'age'],ARRAY['sw', 'ko', '27'])
       ] AS people
    )
  SELECT names['first'] AS
   first_name,
   names['last'] AS last_name,
   department FROM dataset
  CROSS JOIN UNNEST(people) AS t(names)
  ```

  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/8ea4b3c2-4ef3-48d1-ba23-dfb60743b4a5)
  
---

## Presto Docs

### cross join
- cross join은 두 테이블의 데카르트 곱(모든 조합)을 반환
- 5행, 5행 -> 25개 행 반환

```SQL
select 
	sw.alpha as alp,
	sw2.alpha as alp2
from 
	test.test_sw as sw
cross join 
	test.test_sw2 as sw2
order by 1, 2;
```

![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/6678f747-6abb-44a3-ba65-ecad6241081b)

### unnest(cross join unnest)
- 'ARRAY' 또는 'MAP'으로 확장하는데 사용할 수 있음
- array는 단일 열로 확장되고 map은 두 개의 열(key, value)로 확장
- 여러 인수와 함께 사용 -> 가장 높은 카디널리티 인수만큼 행이 있는 여러 열로 확장(빈 값은 NULL)
- with ordinality는 unnest와 함께 사용할 수 있는 선택적 절 -> 배열의 각 요소의 순서를 포함하는 UNNEST 함수의 결과에 추가 열이 추가

- 단일 배열 열 활용
  ```SQL
  WITH dataset AS (
  SELECT
    'data engineering' as department,
    ARRAY['seonwook', 'seonwook2', 'seonwook3', 'seonwook4'] as users
  )
  SELECT department, names FROM dataset
  CROSS JOIN UNNEST(users) as t(names)
  ```

  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/2aec199b-26fd-4303-b294-0a574c4e70f7)
  
- 여러 배열 열 활용
  ```SQL
  WITH dataset AS (
  SELECT
    flatten(ARRAY[ARRAY['1', '2', '3', '4'], ARRAY['data engineering', 'data analytics', 'data architect']]) as department,
    ARRAY['seonwook', 'seonwook2', 'seonwook3', 'seonwook4'] as users
  )
  SELECT majors, names FROM dataset
  CROSS JOIN UNNEST(department, users) as t(majors, names)
  ```

  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/f4875235-1aca-4a8e-93bf-a4ccfe33fa88)

- `WITH ORDINALITY`절
  ```SQL
  SELECT 
    part, a, n
  FROM (
    VALUES
      (ARRAY['SW1', 'SW2']),
      (ARRAY['DA', 'BE', 'DE'])
  ) AS x (part)
  CROSS JOIN UNNEST(part) WITH ORDINALITY AS t(a, n)
  ```

  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/47cdbd2a-e708-444e-92a1-9d840a559e74)

- 단일 맵 열 활용
  ```SQL
  SELECT
    me, name, major
  FROM (
      VALUES
          (MAP(ARRAY['seonwook1', 'seonwook2', 'seonwook3'], ARRAY[1, 2, 3])), -- key, value
          (MAP(ARRAY['DA', 'DE'], ARRAY[4, 5]))
  ) AS x (me)
  CROSS JOIN UNNEST(me) AS t (name, major);
  ```

  ![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/4214e33f-558e-4663-898a-f8092e3f6134)
  
---
