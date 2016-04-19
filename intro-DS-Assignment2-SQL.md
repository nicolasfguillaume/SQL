Problem 1
---------

# (a) select: s10398_txt_earn(frequency)

```sql
SELECT count(*) FROM
(
SELECT count 
FROM Frequency 
WHERE docid = "10398_txt_earn"
)
x
```

# (b) select project: pterm(sdocid=10398_txt_earn and count=1(frequency))

```sql
SELECT count(*) FROM
(
SELECT term
FROM Frequency 
WHERE docid = "10398_txt_earn" AND count = 1
)
x
```

(c) union: pterm(sdocid=10398_txt_earn and count=1(frequency)) U pterm(sdocid=925_txt_trade and count=1(frequency))

SELECT count(*) FROM
(
SELECT term FROM Frequency
WHERE docid = "10398_txt_earn" AND count = 1

UNION

SELECT term FROM Frequency
WHERE docid = "925_txt_trade" AND count = 1
)
x

(d) count:

SELECT count(*) FROM
(
SELECT docid FROM Frequency
WHERE term = "parliament"
)
x

(e) big document: GROUP BY and HAVING

SELECT count(*) FROM
(
SELECT docid, SUM(count)
FROM Frequency
GROUP BY docid
HAVING SUM(count) > 300
)
x

(f) two words: INTERSECTION

SELECT count(*) FROM
(
SELECT docid FROM Frequency
WHERE term = "transactions" 

INTERSECT

SELECT docid FROM Frequency
WHERE term = "world" 
)
x

Problem 2
---------

(g) Matrix multiplication

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

(h) Term-document matrix

TEST:

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


SOLUTION:

SELECT d.docid AS row_num_ddt , dt.docid AS col_num_ddt , SUM(d.count * dt.count) AS 
value_ddt
FROM Frequency d, Frequency dt
WHERE d.term = dt.term
AND row_num_ddt = '10080_txt_crude' 
AND col_num_ddt = '17035_txt_earn'

OR EVEN SHORTER:

SELECT SUM(d.count * dt.count)
FROM Frequency d, Frequency dt
WHERE d.term = dt.term
AND d.docid = '10080_txt_crude' 
AND dt.docid = '17035_txt_earn'

(i): Keyword search:


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
