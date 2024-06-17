## Array Functions and Operators

```SQL
select
	array[1, 2, 3, 4][1], -- first element
	ARRAY [1] || ARRAY [2], -- [1, 2] -- concat, concat(array[1], array[2])
	ARRAY [1] || 2, -- [1, 2], 	1 || ARRAY [2], -- [1, 2]
	array_sort(ARRAY [3, 2, 5, 1, 2], (x, y) -> IF(x < y, 1, IF(x = y, 0, -1))), -- descending order [5, 3, 2, 2, 1]
	array_sort(ARRAY ['bc', 'ab', 'dc'], (x, y) -> IF(x < y, 1, IF(x = y, 0, -1))), -- descending order ['dc', 'bc', 'ab']
	array_sort(ARRAY [3, 2, null, 5, null, 1, 2], -- sort null first with descending order
	                  (x, y) -> CASE WHEN x IS NULL THEN -1
	                                 WHEN y IS NULL THEN 1
	                                 WHEN x < y THEN 1
	                                 WHEN x = y THEN 0
	                                 ELSE -1 END), -- [null, null, 5, 3, 2, 2, 1]
	array_sort(ARRAY [3, 2, null, 5, null, 1, 2], -- sort null last with descending order
	                  (x, y) -> CASE WHEN x IS NULL THEN 1
	                                 WHEN y IS NULL THEN -1
	                                 WHEN x < y THEN 1
	                                 WHEN x = y THEN 0
	                                 ELSE -1 END), -- [5, 3, 2, 2, 1, null, null]
	array_sort(ARRAY ['a', 'abcd', 'abc'], -- sort by string length
	                  (x, y) -> IF(length(x) < length(y),
	                               -1,
	                               IF(length(x) = length(y), 0, 1))), -- ['a', 'abc', 'abcd']
    cardinality(array[1, 2, 3, 4, 5]),
	array_sort(ARRAY [ARRAY[2, 3, 1], ARRAY[4, 2, 1, 4], ARRAY[1, 2]], -- sort by array length
	                  (x, y) -> IF(cardinality(x) < cardinality(y),
	                               -1,
	                               IF(cardinality(x) = cardinality(y), 0, 1))), -- [[1, 2], [2, 3, 1], [4, 2, 1, 4]]
    array_union(array[1, 2, 3, 4], array[5, 6, 7, 8]), -- {1, 2, 3, 4, 5, 6, 7, 8}
	contains(array[1, 2, 3, 4], 3), -- [v]
	element_at(array[1, 2, 5, 4], 3) -- 5
```

![image](https://github.com/seonwook97/Data-Engineering/assets/92377162/4926aa42-5b3b-4ec0-a66a-8f7777665484)
