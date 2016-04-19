# Problem 1

## (a) Select

```sql
SELECT count(*) FROM
(
SELECT count 
FROM Frequency 
WHERE docid = "10398_txt_earn"
)
x
```

## (b) Select Project

```sql
SELECT count(*) FROM
(
SELECT term
FROM Frequency 
WHERE docid = "10398_txt_earn" AND count = 1
)
x
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
x
```

(d) Count

```sql
SELECT count(*) FROM
(
SELECT docid FROM Frequency
WHERE term = "parliament"
)
x
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
x
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
x
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

TEST:
```sql
SELECT value_dxdt
FROM
(
SELECT d.docid AS row_num_dxdt , dt.docid AS col_num_dxdt , SUM(d.count * dt.count) AS value_dxdt
FROM Frequency d , (SELECT term, docid, count FROM Frequency) dt
WHERE d.term = dt.term
GROUP BY d.docid , dt.docid
)
x
WHERE row_num_dxdt = '10080_txt_crude' AND col_num_dxdt = '17035_txt_earn'
```

SOLUTION:

```sql
SELECT d.docid AS row_num_ddt , dt.docid AS col_num_ddt , SUM(d.count * dt.count) AS 
value_ddt
FROM Frequency d, Frequency dt
WHERE d.term = dt.term
AND row_num_ddt = '10080_txt_crude' 
AND col_num_ddt = '17035_txt_earn'
```

OR EVEN SHORTER:

```sql
SELECT SUM(d.count * dt.count)
FROM Frequency d, Frequency dt
WHERE d.term = dt.term
AND d.docid = '10080_txt_crude' 
AND dt.docid = '17035_txt_earn'
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
