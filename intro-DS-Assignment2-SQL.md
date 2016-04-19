# Problem 1

## (a) Select

```sql
SELECT count(*) FROM
(
SELECT count 
FROM Frequency 
WHERE docid = "10398_txt_earn"
)
```

## (b) Select and Project

```sql
SELECT count(*) FROM
(
SELECT term
FROM Frequency 
WHERE docid = "10398_txt_earn" AND count = 1
)
```

## (c) Union

```sql
SELECT count(*) FROM
(
SELECT term FROM Frequency
WHERE docid = "10398_txt_earn" AND count = 1

UNION

SELECT term FROM Frequency
WHERE docid = "925_txt_trade" AND count = 1
)
```

## (d) Count

```sql
SELECT count(*) FROM
(
SELECT docid FROM Frequency
WHERE term = "parliament"
)
```

## (e) Big document: GROUP BY and HAVING

```sql
SELECT count(*) FROM
(
SELECT docid, SUM(count)
FROM Frequency
GROUP BY docid
HAVING SUM(count) > 300
)
```

## (f) Two words: INTERSECT

```sql
SELECT count(*) FROM
(
SELECT docid FROM Frequency
WHERE term = "transactions" 

INTERSECT

SELECT docid FROM Frequency
WHERE term = "world" 
)
```

***

# Problem 2

## (g) Matrix multiplication

```sql
SELECT value_C
FROM
(
SELECT a.row_num AS row_num_C , b.col_num AS col_num_C , SUM(a.value * b.value) AS value_C
FROM a , b
WHERE a.col_num = b.row_num
GROUP BY a.row_num , b.col_num
)
x
WHERE row_num_C = 2 AND col_num_C = 3
```

## (h) Term-document matrix

```sql
SELECT d.docid AS row_num_ddt , dt.docid AS col_num_ddt , SUM(d.count * dt.count) AS 
value_ddt
FROM Frequency d, Frequency dt
WHERE d.term = dt.term
AND row_num_ddt = '10080_txt_crude' 
AND col_num_ddt = '17035_txt_earn'
```

## (i) Keyword search

```sql
SELECT d.docid, dt.docid, SUM(d.count * dt.count) AS similarity
FROM Frequency d, 
(
SELECT 'q' as docid, 'washington' as term, 1 as count 
UNION
SELECT 'q' as docid, 'taxes' as term, 1 as count
UNION 
SELECT 'q' as docid, 'treasury' as term, 1 as count
) dt
WHERE d.term = dt.term
GROUP BY d.docid
ORDER BY similarity DESC
```
