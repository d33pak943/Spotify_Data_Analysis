# Spotify_Data_Analysis SQL Project and Query Optimization

![Spotify Logo](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/spotify_logo.jpg)
![image](https://github.com/user-attachments/assets/5c0734ce-9904-4c6a-93cb-ee31dca1e05a)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries  and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 2. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data.
- Simple data retrieval, filtering, and basic aggregations.
- More complex queries involving grouping, aggregation functions, and joins.
- Nested subqueries, window functions, CTEs, and performance optimization.

### 3. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---
## Project Procedure

###  Database Setup

- **Database Creation**: The project starts by creating a database named `spotify_db`.
- **Table Creation**: A table named `spotify` is created to store the sales data. The table structure includes columns for Artist, Track, Album, Album_type, Danceability, Energy, Loudness, Speechiness, Acousticness, Instrumentalness, Liveness, Valence, Tempo, Duration_min, Title, Channel, Views, Likes, Comments, Licensed, official_video, Stream, EnergyLiveness,most_played_on


```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```

### Exploratory Data Analysis
```sql
SELECT COUNT(*) FROM spotify;
SELECT COUNT(DISTINCT artist) FROM spotify;
SELECT COUNT(DISTINCT album) FROM spotify;
SELECT DISTINCT album_type FROM spotify;
SELECT MAX(duration_min) FROM spotify;
SELECT MIN(duration_min) FROM spotify;
SELECT * FROM spotify
WHERE duration_min=0;
DELETE FROM spotify
WHERE duration_min=0;
SELECT DISTINCT channel FROM spotify;
SELECT DISTINCT most_played_on FROM spotify;
```

## Data Analysis & Findings
1.**Retrieve the names of all tracks that have more than 1 billion streams.**
```sql
SELECT 
   track 
FROM spotify
WHERE Stream > 1000000000;
```
2. **List all albums along with their respective artists.**
```sql
SELECT 
   DISTINCT album,
   artist 
FROM spotify;
```
3.**Get the total number of comments for tracks where `licensed = TRUE`.**
```sql
SELECT 
   SUM(comments) as total_comments 
FROM spotify
WHERE licensed = 'true';
```
4.**Find all tracks that belong to the album type `single`.**
```sql
SELECT 
   track  
FROM spotify
WHERE album_type = 'single' ;
```
5. **Count the total number of tracks by each artist.**
```sql
SELECT 
   artist,
   COUNT(*) as number_of_tracks 
FROM spotify
GROUP BY 1
ORDER BY 2 ;
```
6. **Calculate the average danceability of tracks in each album.**
```sql
SELECT 
   album,
   AVG(danceability) as avg_danceability
FROM spotify
GROUP BY 1
ORDER BY 2 DESC;
```
7.**Find the top 5 tracks with the highest energy values.**
```sql
SELECT 
   track,
   MAX(energy) 
FROM spotify 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```
8.**List all tracks along with their views and likes where `official_video = TRUE`.**
```sql
SELECT 
   track,
   SUM(views) as total_views,
   SUM(likes) as total_likes
FROM spotify
WHERE official_video='true'
GROUP BY 1;
```
9.**For each album, calculate the total views of all associated tracks.**
```sql
SELECT 
   album,
   track,
   SUM(views) as total_views 
FROM spotify
GROUP BY 1,2
ORDER BY 3 DESC;
```
10.**Retrieve the track names that have been streamed on Spotify more than YouTube.**
```sql
SELECT * FROM
(SELECT
    track,
	COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END),0) as streamed_on_youtube,
	COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END),0) as streamed_on_spotify
FROM spotify
GROUP BY 1
) as t1
WHERE 
   streamed_on_spotify > streamed_on_youtube
   AND
   streamed_on_youtube <>0;
```
11.**Find the top 3 most-viewed tracks for each artist using window functions.**
```sql
WITH ranking_artist
AS
(SELECT 
    artist,
    track,
	SUM(views) as total_views,
	DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM(views) DESC) AS rank
FROM spotify
GROUP BY 1,2
ORDER BY 1,3 DESC)
SELECT * FROM ranking_artist
WHERE rank <=3;
```
12.**Write a query to find tracks where the liveness score is above the average.**
```sql
SELECT
   track,
   artist,
   liveness
FROM spotify
WHERE liveness >(SELECT AVG(liveness) FROM spotify);
```
13. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC;
```
   
14.**Find tracks where the energy-to-liveness ratio is greater than 1.2.**
```sql
WITH ctee
AS
(SELECT
    track,
	SUM(energy) as total_energy,
	SUM(liveness) as total_liveness
FROM spotify
GROUP BY 1
)
SELECT
   track
FROM ctee
WHERE (total_energy/total_liveness)>1.2;
```
15.**Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.**
```sql
SELECT 
    track,
	views,
	likes,
	SUM(likes) OVER(PARTITION BY track ORDER BY views) as cumulative_likes
FROM spotify;
```

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **5.16 ms**
        - Planning time (P.T.): **0.11 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/1.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.132 ms**
        - Planning time (P.T.): **0.123 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/2.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/3.png)
      ![Performance Graph](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/4.png)
      ![Performance Graph](https://github.com/d33pak943/Spotify_Data_Analysis/blob/main/resources/5.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4, PostgreSQL 


---


## License
This project is licensed under the MIT License.
