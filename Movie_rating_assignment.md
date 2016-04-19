# Movie-rating database

You've started a new movie-rating website, and you've been collecting data on reviewers' ratings of various movies. There's not much data yet, but you can still try out some interesting queries. Here's the schema: 

Movie ( mID, title, year, director ) 
English: There is a movie with ID number mID, a title, a release year, and a director. 

Reviewer ( rID, name ) 
English: The reviewer with ID number rID has a certain name. 

Rating ( rID, mID, stars, ratingDate ) 
English: The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate. 

## Query exercises

### Q1
Find the titles of all movies directed by Steven Spielberg. 

```sql

SELECT title 
FROM Movie 
WHERE director = 'Steven Spielberg'
```

### Q2
Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order. 

```sql

SELECT DISTINCT year 
FROM Movie JOIN Rating 
WHERE Movie.mID = Rating.mID 
    AND Rating.stars >= 4
ORDER BY year
```

### Q3
Find the titles of all movies that have no ratings. 

```sql

SELECT title 
FROM Movie 
EXCEPT
SELECT DISTINCT title 
FROM Movie JOIN Rating
WHERE Movie.mID = Rating.mID
```

### Q4
Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date. 

```sql

SELECT DISTINCT name 
FROM Reviewer, Rating 
WHERE Reviewer.rID = Rating.rID 
    AND Rating.ratingDate IS NULL
```

### Q5
Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars. 

```sql

SELECT DISTINCT name, title, stars, ratingDate 
FROM Movie, Reviewer, Rating
WHERE Movie.mID = Rating.mID AND Reviewer.rID = Rating.rID
ORDER BY name, title, stars
```

### Q6
For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie. 

```sql

SELECT DISTINCT name, title
FROM Movie, Reviewer, Rating, Rating R2
WHERE Movie.mID = Rating.mID AND Reviewer.rID = Rating.rID
AND Movie.mID = R2.mID AND R2.rID = Rating.rID
AND Rating.stars < R2.stars AND Rating.ratingDate < R2.ratingDate
```

### Q7
For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title. 

```sql

SELECT DISTINCT title, MAX(stars)
FROM Movie JOIN Rating USING(mID)
GROUP BY title
ORDER BY title
```

### Q8
For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title. 

```sql

SELECT title, (MAX(stars) - MIN(stars)) AS spread
FROM Rating JOIN Movie USING(mID)
GROUP BY mID
ORDER BY spread DESC, title
```

### Q9
Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.) 

```sql

SELECT (-AVG(after.avgafter80) + AVG(before.avgbefore80))
FROM
(
SELECT AVG(stars) as avgbefore80
FROM Rating JOIN Movie USING(mID)
WHERE year < 1980
GROUP BY title
) AS before
,
(
SELECT AVG(stars) as avgafter80
FROM Rating JOIN Movie USING(mID)
WHERE year > 1980
GROUP BY title
) AS after
```

***

## Query exercises Extras

### Q1
Find the names of all reviewers who rated Gone with the Wind. 

```sql

SELECT DISTINCT Reviewer.name
FROM (Movie JOIN Rating USING(mID)) JOIN Reviewer USING(rID)
WHERE Movie.title = 'Gone with the Wind'
```

### Q2
For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars. 

```sql

SELECT DISTINCT Reviewer.name, Movie.title, Rating.stars
FROM (Movie JOIN Rating USING(mID)) JOIN Reviewer USING(rID)
WHERE Movie.director = Reviewer.name
```

### Q3
Return all reviewer names and movie names together in a single list, alphabetized. (Sorting by the first name of the reviewer and first word in the title is fine; no need for special processing on last names or removing "The".) 

```sql

SELECT DISTINCT Reviewer.name
FROM Reviewer
UNION
SELECT DISTINCT Movie.title
FROM Movie 
ORDER BY Reviewer.name, Movie.title
```

### Q4
Find the titles of all movies not reviewed by Chris Jackson. 

```sql

SELECT title
FROM MOVIE
WHERE mID IN
(
SELECT DISTINCT mID
FROM Movie
EXCEPT
SELECT DISTINCT mID
FROM Rating JOIN Reviewer USING(rID)
WHERE Reviewer.name = 'Chris Jackson'
)
```

### Q5
For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once. For each pair, return the names in the pair in alphabetical order. 

```sql

SELECT DISTINCT T1.name, T2.name
FROM 
(Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)) AS T1,
(Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)) AS T2
WHERE T1.mID = T2.mID AND T1.name <> T2.name AND T1.name < T2.name
```

### Q6
For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars. 

```sql

SELECT DISTINCT name, title, stars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
WHERE stars IN 
(
SELECT MIN(stars) FROM Rating  
)
GROUP BY title
```

### Q7
List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order. 

```sql

SELECT DISTINCT title, AVG(stars) AS avgstars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY title
ORDER BY avgstars DESC, title
```


### Q8
Find the names of all reviewers who have contributed three or more ratings. (As an extra challenge, try writing the query without HAVING or without COUNT.) 

```sql

SELECT DISTINCT name
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY name
HAVING COUNT(stars) >= 3
```


### Q9
Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title. (As an extra challenge, try writing the query both with and without COUNT.) 

```sql

SELECT title, director
FROM Movie
WHERE director IN
(
SELECT director
FROM Movie
GROUP BY director
HAVING COUNT(director) > 1
)
ORDER BY director, title
```


### Q10
Find the movie(s) with the highest average rating. Return the movie title(s) and average rating. (Hint: This query is more difficult to write in SQLite than other systems; you might think of it as finding the highest average rating and then choosing the movie(s) with that average rating.) 

```sql

SELECT DISTINCT title, avgstars
FROM
(
SELECT DISTINCT title, AVG(stars) AS avgstars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY title
)
WHERE avgstars IN
(
SELECT DISTINCT MAX(avgstars)
FROM
(
SELECT DISTINCT title, AVG(stars) AS avgstars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY title
)
)
```


### Q11
Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating. (Hint: This query may be more difficult to write in SQLite than other systems; you might think of it as finding the lowest average rating and then choosing the movie(s) with that average rating.) 

```sql

SELECT DISTINCT title, avgstars
FROM
(
SELECT DISTINCT title, AVG(stars) AS avgstars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY title
)
WHERE avgstars IN
(
SELECT DISTINCT MIN(avgstars)
FROM
(
SELECT DISTINCT title, AVG(stars) AS avgstars
FROM Movie JOIN (Rating JOIN Reviewer USING(rID)) USING(mID)
GROUP BY title
)
)
```

***

## Modification exercises

### Q1
Add the reviewer Roger Ebert to your database, with an rID of 209. 

```sql

INSERT INTO Reviewer VALUES(209, 'Roger Ebert')
```

### Q2
Insert 5-star ratings by James Cameron for all movies in the database. Leave the review date as NULL. 

```sql

INSERT INTO Rating
SELECT Rating.rID, Movie.mID, 5, NULL
FROM Movie, Reviewer, Rating
WHERE Rating.rID = Reviewer.rID AND Reviewer.name = 'James Cameron'
```

### Q3
For all movies that have an average rating of 4 stars or higher, add 25 to the release year. (Update the existing tuples; don't insert new tuples.) 

```sql

UPDATE Movie 
SET year = year + 25
WHERE mID in 
(
SELECT mID
FROM Rating 
GROUP BY mID
HAVING AVG(stars) >= 4.0
)
```

### Q4
Remove all ratings where the movie's year is before 1970 or after 2000, and the rating is fewer than 4 stars. 

```sql

DELETE
FROM Rating
WHERE mID IN
(
SELECT mID
FROM Movie 
WHERE year NOT BETWEEN 1970 AND 2000
)
AND stars < 4
```
