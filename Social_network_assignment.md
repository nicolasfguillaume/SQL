# Social Network database
Students at your hometown high school have decided to organize their social network using databases. So far, they have collected information about sixteen students in four grades, 9-12. Here's the schema: 

Highschooler ( ID, name, grade ) 
English: There is a high school student with unique ID and a given first name in a certain grade. 

Friend ( ID1, ID2 ) 
English: The student with ID1 is friends with the student with ID2. Friendship is mutual, so if (123, 456) is in the Friend table, so is (456, 123). 

Likes ( ID1, ID2 ) 
English: The student with ID1 likes the student with ID2. Liking someone is not necessarily mutual, so if (123, 456) is in the Likes table, there is no guarantee that (456, 123) is also present. 

## Query exercises

### Q1
Find the names of all students who are friends with someone named Gabriel. 

```sql

SELECT DISTINCT H2.name
FROM Highschooler H1, Friend, Highschooler H2
WHERE H1.ID = Friend.ID1 AND H2.ID = Friend.ID2
AND H1.name = 'Gabriel'
```

### Q2
For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like. 

```sql

SELECT DISTINCT H1.name, H1.grade, H2.name, H2.grade 
FROM Highschooler H1, Likes, Highschooler H2
WHERE H1.ID = Likes.ID1 AND H2.ID = Likes.ID2
AND (H1.grade - H2.grade) >= 2
```

### Q3
For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order. 

```sql

SELECT DISTINCT H1.name, H1.grade, H2.name, H2.grade
FROM Highschooler H1, Highschooler H2,
( SELECT DISTINCT L1.ID1, L1.ID2 FROM Likes L1,  Likes L2
WHERE L1.ID1 = L2.ID2 AND L1.ID2 = L2.ID1 ) Pairs
WHERE H1.ID = Pairs.ID1 AND H2.ID = Pairs.ID2 AND H1.name < H2.name
```

### Q4
Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade. 

```sql

SELECT name, grade
FROM Highschooler
WHERE ID NOT IN
(SELECT DISTINCT ID1 FROM Likes
UNION
SELECT DISTINCT ID2 FROM Likes)
ORDER BY grade, name
```

### Q5
For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades. 

```sql

SELECT DISTINCT H1.name, H1.grade, H2.name, H2.grade
FROM Highschooler H1, Likes L, Highschooler H2
WHERE H1.ID = L.ID1 AND H2.ID = L.ID2
AND H2.ID NOT IN (SELECT DISTINCT ID1 FROM Likes)
```

### Q6
Find names and grades of students who only have friends in the same grade. Return the result sorted by grade, then by name within each grade. 

```sql

SELECT name, grade FROM Highschooler
WHERE ID NOT IN (
SELECT DISTINCT H1.ID
FROM Highschooler H1, Friend F, Highschooler H2
WHERE H1.ID = F.ID1 AND H2.ID = F.ID2
AND H1.grade <> H2.grade )
ORDER BY grade, name
```

### Q7
For each student A who likes a student B where the two are not friends, find if they have a friend C in common (who can introduce them!). For all such trios, return the name and grade of A, B, and C. 

```sql

SELECT DISTINCT H1.name,H1.grade, H2.name, H2.grade, H3.name, H3.grade
FROM Highschooler H1, Likes, Highschooler H2, Friend Fof1, Friend Fof2, Highschooler H3
WHERE H1.ID = Likes.ID1 AND Likes.ID2 = H2.ID
AND H2.ID NOT IN (SELECT ID2 FROM Friend WHERE ID1 = H1.ID )
AND H3.ID = Fof1.ID2 AND H3.ID = Fof2.ID2 
AND H1.ID = Fof1.ID1 AND H2.ID = Fof2.ID1 
```

### Q8
Find the difference between the number of students in the school and the number of different first names. 

```sql

SELECT nbStudent - nbName
FROM
(SELECT COUNT(DISTINCT ID) AS nbStudent FROM Highschooler),
(SELECT COUNT(DISTINCT name) AS nbName FROM Highschooler) 
```

### Q9
Find the name and grade of all students who are liked by more than one other student. 

```sql

SELECT H2.name, H2.grade
FROM Highschooler H1, Likes, Highschooler H2
WHERE H1.ID = Likes.ID1 AND Likes.ID2 = H2.ID
GROUP BY H2.ID
HAVING COUNT(DISTINCT H1.ID) > 1
```

***

## Query exercises Extras

### Q1
For every situation where student A likes student B, but student B likes a different student C, return the names and grades of A, B, and C. 

```sql

SELECT HA.name, HA.grade, HB.name, HB.grade, HC.name, HC.grade
FROM Highschooler HA, Likes LikesAB, Highschooler HB, Likes LikesBC, Highschooler HC
WHERE HA.ID = LikesAB.ID1 AND LikesAB.ID2 = HB.ID
AND HB.ID = LikesBC.ID1 AND LikesBC.ID2 = HC.ID
AND HA.ID <> HC.ID
```

### Q2
Find those students for whom all of their friends are in different grades from themselves. Return the students' names and grades. 

```sql

SELECT DISTINCT H.name, H.grade
FROM Highschooler H

EXCEPT

SELECT DISTINCT H1.name, H1.grade
FROM Highschooler H1, Friend, Highschooler H2
WHERE H1.ID = Friend.ID1 AND Friend.ID2 = H2.ID
AND H1.grade = H2.grade
```

### Q3
What is the average number of friends per student? (Your result should be just one number.) 

```sql

SELECT AVG(friendCount)
FROM
(
SELECT COUNT(H2.ID) AS friendCount
FROM Highschooler H1, Friend, Highschooler H2
WHERE H1.ID = Friend.ID1 AND Friend.ID2 = H2.ID
GROUP BY H1.ID
)
```

### Q4
Find the number of students who are either friends with Cassandra or are friends of friends of Cassandra. Do not count Cassandra, even though technically she is a friend of a friend. 

```sql

SELECT nbFriend + nbFriendofFriend
FROM
(
SELECT COUNT(DISTINCT H2.ID) AS nbFriend
FROM Highschooler H1, Friend, Highschooler H2
WHERE H1.ID = Friend.ID1 AND Friend.ID2 = H2.ID
AND H1.name = 'Cassandra' ),
(
SELECT COUNT(DISTINCT H3.ID) AS nbFriendofFriend
FROM Highschooler H1, Friend Friend1_2, Highschooler H2, Friend Friend2_3, Highschooler H3
WHERE H1.ID = Friend1_2.ID1 AND Friend1_2.ID2 = H2.ID
AND H2.ID = Friend2_3.ID1 AND Friend2_3.ID2 = H3.ID
AND H1.name = 'Cassandra' AND H3.name <> 'Cassandra' )
```

### Q5
Find the name and grade of the student(s) with the greatest number of friends. 

```sql

SELECT friendCount_Table.name, friendCount_Table.grade
FROM
( SELECT H1.ID, H1.name,H1.grade, COUNT(DISTINCT H2.ID) AS friendCount
FROM Highschooler H1, Friend, Highschooler H2
WHERE H1.ID = Friend.ID1 AND Friend.ID2 = H2.ID
GROUP BY H1.ID ) AS friendCount_Table
WHERE friendCount = (
    SELECT MAX(friendCount)
    FROM ( SELECT H1.ID, H1.grade, COUNT(DISTINCT H2.ID) AS friendCount
        FROM Highschooler H1, Friend, Highschooler H2
        WHERE H1.ID = Friend.ID1 AND Friend.ID2 = H2.ID
        GROUP BY H1.ID )
) 
```

***

## Modification exercises

### Q1
It's time for the seniors to graduate. Remove all 12th graders from Highschooler. 

```sql

DELETE 
FROM Highschooler
WHERE grade = '12'
```

### Q2
If two students A and B are friends, and A likes B but not vice-versa, remove the Likes tuple. 

```sql

DELETE 
FROM Likes
WHERE ID2 IN
( 
SELECT Friend.ID2
FROM Friend
WHERE Friend.ID1 = Likes.ID1
)
AND ID2 NOT IN
(
SELECT L.ID1
FROM Likes L
WHERE L.ID2 = Likes.ID1
)
```
