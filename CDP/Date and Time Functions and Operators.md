# Date and Time Functions and Operators

- 현재 CDP presto 버전: 350
- Presto docs: 282
- Trino docs: 422

---

## Time Zone Conversion
```SQL
-- UTC = KST - 9
select timestamp '2023-07-18 01:00 UTC' at time zone 'Asia/Seoul' as KST
```

![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/0f8f069d-3ff6-48df-8427-81ac9345da32)

## Date and Time Functions
```SQL
select 
	current_date as currentdate -- 현재 날짜
	, current_time as currenttime-- 현재 시간
	, current_timestamp as currenttimestamp-- 현재 날짜 및 시간
	, current_timezone() as currenttimezone-- 현재 표준시
	, last_day_of_month(date('2023-07-18')) as last_day_of_month -- 월 말일
	, from_iso8601_timestamp('2023-07-18') as string_to_timestamp -- ISO 8601 형식 문자열을 시간대가 있는 타임스탬프로 변환
	, from_iso8601_date('2023-07-18') as string_to_date -- ISO 8601 형식 문자열을 날짜형으로 변환
	, from_unixtime(1689654000) as unixtime -- unixtime -> timestamp(UTC)
	, localtime as local_time -- 현재 시간(현지기준)
	, localtimestamp as local_timestamp
	, now() as currenttimestamp2 -- current_timestamp	
```

![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/25cd915a-889b-4ffd-a73f-b18c217c70a8)
