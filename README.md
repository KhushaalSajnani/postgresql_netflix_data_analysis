
# Netflix Data Analysis

- **Dataset:** [Netflix Dataset - Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
- **Tool Used:** *_PostgreSQL_* 

```sql
CREATE TABLE IF NOT EXISTS NETFLIX 
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);
```

> 1. Count the number of Movies and TV Shows
```sql
SELECT type, COUNT(*) 
FROM netflix
GROUP BY type;
```
> 2. Find the most common rating for movies and TV shows

```sql
WITH grouped_cte AS (
	SELECT rating,type, COUNT(*) as cnt
	FROM netflix 
	GROUP BY 1, 2
),
ranked_grouped AS (
	SELECT *,
	RANK() OVER(PARTITION BY type ORDER BY cnt DESC)
	FROM grouped_cte
)
SELECT * FROM ranked_grouped WHERE rank = 1;
```

> 3. List all movies released in a specific year (e.g., 2020)

```sql
SELECT title
FROM netflix 
WHERE release_year = 2020;
```

> 4. Find the top 5 countries with the most content on Netflix

```sql
SELECT UNNEST(STRING_TO_ARRAY(country, ',')), COUNT(*)
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

> 5. Identify the longest movie

```sql
SELECT title, COALESCE(duration, '0 min') 
FROM netflix
WHERE type = 'Movie' AND duration <> '0 min'
ORDER BY duration DESC;
```

> 6. Find content added in the last 5 years

```sql
SELECT * FROM netflix
WHERE release_year >= EXTRACT(year from CURRENT_DATE) - 5
ORDER BY release_year;
```

> 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

```sql
SELECT title, type, director FROM netflix
where director LIKE '%Rajiv Chilaka%';
```

> 8. List all TV shows with more than 5 seasons

```sql
SELECT title
FROM netflix
WHERE type<>'Movie' AND CAST(SPLIT_PART(duration, ' ',1) AS INTEGER) > 5
```

> 9.  Count the number of content items in each genre

```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in,',')) as Genre, 
COUNT(show_id) 
FROM netflix 
GROUP BY 1 
ORDER BY 2 DESC;
```

> 10. Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!
```sql
SELECT EXTRACT(year from to_date(date_added, 'Month DD, YYYY')) as year, 
COUNT(*) as total_cnt,
ROUND((COUNT(*)::numeric/ (SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric * 100), 2) as avg_cnt
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 3 DESC
LIMIT 5;
```
> 11. List all movies that are documentaries
```sql
SELECT title, listed_in
FROM netflix
WHERE type = 'Movie' AND (listed_in = 'Documentaries' OR listed_in LIKE '%Documentaries%')
```
> 12. Find all content without a director

```sql
SELECT title 
FROM netflix 
WHERE COALESCE(director, 'NULL') = 'NULL'
```

> 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT * 
FROM netflix 
WHERE casts like '%Salman Khan%' AND release_year >= EXTRACT(year from CURRENT_DATE) - 10;
```
> 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')), COUNT(show_id) 
FROM netflix 
WHERE country = 'India' AND type = 'Movie' GROUP BY 1 
ORDER BY 2 DESC 
LIMIT 10; 
```
> 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
```sql
SELECT category, COUNT(*) FROM (SELECT *,
	CASE
		WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
		ELSE 'Good'
	END as category
FROM netflix) GROUP BY category
```



