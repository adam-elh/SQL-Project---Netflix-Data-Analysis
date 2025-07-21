# SQL-Project-Netflix-Data-Analysis

-- Personal Study - SQL Project Netflix Dataset Data Analysis

-- Create Table Query
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);

-- Exploratory Data Analysis

SELECT *
FROM netflix; -- 8807 movies & shows

SELECT COUNT(*) as total_count
FROM netflix; -- 8807 movies & shows

SELECT DISTINCT(type)
FROM netflix; -- 2 types (movies & tv shows)

-- 15 Basic Questions

-- 1. Count the Number of Movies vs TV Shows.

SELECT type, COUNT(*) as type_total
FROM netflix
GROUP BY type; -- 6131 Movies, 2676 TV Shows

-- 2. Find the Most Common Rating for Movies and TV Shows.

SELECT
     type,
	 rating
FROM
(SELECT 
     type, 
	 rating, 
	 COUNT(*), 
	 RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
FROM netflix
GROUP BY type, rating  
) as t1
WHERE ranking = 1;

-- 3. List All Movies Released in a Specific Year (e.g., 2020).

SELECT * FROM netflix
WHERE release_year = 2020 AND type = 'Movie';

-- 4. Find the Top 5 Countries with the Most Content on Netflix.

SELECT UNNEST(STRING_TO_ARRAY(country,',')) as new_country, COUNT(show_id) as total FROM netflix
GROUP BY new_country ORDER BY total DESC LIMIT 5; -- String to Array & Unnest Fuctions

-- 5. Identify the Longest Movie.

SELECT * 
FROM netflix
WHERE type = 'Movie' AND duration = (SELECT MAX(duration) FROM netflix);

-- 6. Find Content Added in the Last 5 Years.

SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'; -- 2326 Movies & TV Shows

-- 7. Find All Movies/TV Shows by Director 'Mike Flanagan'

SELECT * FROM netflix
WHERE director ILIKE '%Mike Flanagan%'; -- ILike Fuction includes all strings that include name & not case sensitive

-- 8. List All TV Shows with More Than 5 Seasons.

SELECT *
FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::numeric > 5; -- SPLIT_PART Fuction to extract number from duration string

-- 9. Count the Number of Content Items in Each Genre.

SELECT 
     UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre, 
	 COUNT(show_id) as total
FROM netflix
GROUP BY genre ORDER BY total DESC;

-- 10. Find each year and the average number of content release in the US on netflix.

SELECT 
     EXTRACT(YEAR FROM TO_DATE(date_added,'MONTH DD, YYYY')) as year,
	 COUNT(*) AS yearly,
	 ROUND(COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'United States')::numeric * 100, 2) as avg_per_year
FROM netflix
WHERE country = 'United States'
GROUP BY year ORDER BY yearly DESC;

-- 11. List All Movies that are Documentaries.

SELECT * 
FROM netflix 
WHERE listed_in ILIKE '%Documentaries%'; -- 689 total records.

-- 12. Find All Content Without a Director.

SELECT * 
FROM netflix
WHERE director IS NULL; -- 2634 Movies & TV Show with a missing director.

-- 13. Find How Many Movies Actor 'Aaron Paul' Appeared in the Last 10 Years.

SELECT * 
FROM netflix
WHERE casts ILIKE '%Aaron Paul%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10; -- 6 Movies & TV Shows

-- 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in the US.

SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) as actors, COUNT(*) as total_roles
FROM netflix
WHERE country ILIKE '%United States%'
GROUP BY actors ORDER BY total_roles DESC LIMIT 10;

-- 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords as 'Mature' Content and if not name it 'Safe' Content.

WITH new_table
AS
(
SELECT 
     *, 
	 CASE 
	 WHEN 
	    description ILIKE '%kill%'
		OR 
		description ILIKE '%violence%'
	 THEN 'Mature Content'
	 ELSE 'Safe Content'
	 END category
FROM netflix
)
SELECT
     category,
	 COUNT(*) as total_content
FROM new_table
GROUP BY category; -- 8465 Movies & TV Shows are considered safe and 342 considered mature. 
