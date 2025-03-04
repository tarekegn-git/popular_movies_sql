# Data Preparation: Top 1000 Popular Hollywood Movies
In this project, we will do  data cleaning and preparation for further analysis
## Dataset Description
This dataset obtained from the [Kaggle](https://www.kaggle.com/datasets/sanjeetsinghnaik/top-1000-highest-grossing-movies) contains information about the top 1000 highest-grossing Hollywood movies. The database is organized with 14 columns and over 970 movie entries. Columns contain information about the movie title, release year, release date, budget, worldwide sales, license, etc. Each row represents a movie entry.
## Project Plan
In this project, we will prepare the dataset to make it suitable for analysis. The raw data is not ideal for immediate analysis due to several issues such as column naming issues, corrupted data entries, duplicate values, datatype issues etc. 
## 1. Loading the Dataset Renaming the Table
  The original name for the table is like `highest holywood grossing movies`   
  which is inconvenient to work with because of spaces and long name, so let's rename it into top_movies:
```sql
RENAME TABLE `highest holywood grossing movies` TO `top_movies`;
```
Now we have renamed the original data into `top_movies` let's look into the dataset for the first time:
```sql
SELECT *
FROM top_movies;
```
Result:  
![top_movies_1](https://github.com/user-attachments/assets/9b083628-d631-4014-9c85-e58b571d25b4)  
## 2. Creating a Copy of `top_movies` Dataset Table
To preserve the original table while working with this dataset, let's create a copy of the `top_movies` table and name it as `highest_grossing_movies`. Second, we can confirm if all columns are copied correctly. Third all data entries from `top_movies` are inserted into the new table.
```sql
CREATE TABLE highest_grossing_movies
LIKE top_movies;
-- Confirm Columns

SELECT *
FROM highest_grossing_movies;

-- Insert all data entries from the original into the new table  
INSERT highest_grossing_movies
SELECT *
FROM top_movies;

-- lets see the data in new table

SELECT * 
FROM highest_grossing_movies;
```
Result:  
![highest_grossing_1](https://github.com/user-attachments/assets/02060426-d29f-45c1-8920-e5872ea3323f)  
## 3. Renaming Columns
From section 2, we have a new table to work with. It is safe to work with it since we have `top_movies` as a backup. As we can see from the Section 2 result, column names are inconsistent and we should rename them to follow the uniform naming standard. However, the default datatype won't be changed yet. To do so we write SQL query as follows:  
```sql
ALTER TABLE highest_grossing_movies
CHANGE COLUMN `MyUnknownColumn` `s_num` INT,
CHANGE COLUMN `Title` `title` VARCHAR(255),
CHANGE COLUMN `Movie Info` `movie_info` TEXT,
CHANGE COLUMN `Year` `year` INT,
CHANGE COLUMN `Distributor` `distributor` VARCHAR(255),
CHANGE COLUMN `Budget (in $)` `budget` TEXT,
CHANGE COLUMN `Domestic Opening (in $)` `domestic_opening` TEXT,
CHANGE COLUMN `Domestic Sales (in $)` `domestic_sales` TEXT,
CHANGE COLUMN `International Sales (in $)` `international_sales` TEXT,
CHANGE COLUMN `World Wide Sales (in $)` `worldwide_sales` TEXT,
CHANGE COLUMN `Release Date` `release_date` TEXT,
CHANGE COLUMN `Genre` `genre` VARCHAR(255),
CHANGE COLUMN `Running Time` `running_time` TEXT,
CHANGE COLUMN `License` `license` VARCHAR(255);

-- let's check it
SELECT *
FROM highest_grossing_movies;
```

Result:
![highest_grossing_column_renamed](https://github.com/user-attachments/assets/f57ca9e5-8b60-4752-8316-c20369d7da40)  
Now, all columns follow the same naming standard and this is helpful during subsequent tasks that involve complex queries.
## 4. Duplicate Entries
Before any further analysis, checking for duplicate data entries is very important. In this section, we will look into duplicate entries and fix them accordingly. For this purpose, we use SQL `DISTINCT` and `COUNT` keywords to return the total number of movie titles.  
```sql
SELECT COUNT(title) AS entire_title_count
FROM highest_grossing_movies;
-- Result: 954
-- Now let's get the count of distinct movies

SELECT COUNT(DISTINCT title) AS distinct_title_count
FROM highest_grossing_movies;
-- Result: 943
```
The results for the total number of movie titles and distinct count of movies show some differences. Below we will see which movies have similar titles and check if they are actually duplicate entries. First we give a row number for each movie, and then we get the same movies that appear in more than one row.  
```sql
SELECT title,
	ROW_NUMBER() OVER(
    PARTITION BY title
    ) AS row_num
FROM highest_grossing_movies;

-- Let's check movies with duplicate entries

SELECT *
FROM(
SELECT title,
	ROW_NUMBER() OVER(
    PARTITION BY title
    ) AS row_num
FROM highest_grossing_movies
) duplicates
WHERE row_num > 1;
```

Result:  
![highest_grossing_duplicates](https://github.com/user-attachments/assets/e65cea25-9691-4949-97d3-79ee2e618ce0)
The result shows that some movies have duplicate entries, but we can't confirm that since some movies may have similar titles with different release years. To check this scenario, let's get the title and release year of the movies `Aladdin` and `Beauty and the Beast using the` using `WHERE` keyword as follows:  
```sql
-- Let's check the Aladdin movie with 2-row entries to confirm
SELECT title, year
FROM highest_grossing_movies
WHERE title = 'Aladdin';

# title, year
Aladdin, 2019
Aladdin, 1992

-- there are two distinct movies titled Aladdin with different release years. Let's check others

SELECT title, year
FROM highest_grossing_movies
WHERE title = 'Beauty and the Beast';

-- title, year
Beauty and the Beast, 2017
Beauty and the Beast, 1991

```
Therefore, we need to rename the title of one of the pair movies so they won't be counted as duplicates. To do so, we assume the latest movie is a sequel or remake of an older movie. For instance, we assume Aladdin (2019 is a remake of Aladdin (1992. So we will rename the latest movie by adding 2 in front of its title( ex. Aladdin, 2019 renamed as Aladdin 2, and so on).  The following query does just that:
```sql
UPDATE highest_grossing_movies h1
JOIN (SELECT `title`, MIN(`year`) AS first_release_year
      FROM highest_grossing_movies
      GROUP BY `title`) h2
ON h1.`title` = h2.`title`
AND h1.`year` > h2.first_release_year
SET h1.`title` = CONCAT(h1.`title`, ' 2');
```
We can now confirm the duplicate entries problem has been fixed. Note: this is for demonstration only.  
```sql
-- If we check Beauty and the Beast movie again, it returns 1 row with the earliest release year(1991)

SELECT title, year
FROM highest_grossing_movies
WHERE title = 'Beauty and the Beast';

-- If we check Beauty and the Beast 2 movie, it returns 1 row with the latest release year(2017)

SELECT title, year
FROM highest_grossing_movies
WHERE title = 'Beauty and the Beast 2';

-- The first movie and the second movie are now counted as distinct movies
SELECT COUNT(DISTINCT title) AS number_of_movies
FROM highest_grossing_movies;
--Result 954

SELECT *
FROM highest_grossing_movies
WHERE title LIKE ('Beauty and the Beast%');
```
Result:  
![highest_grossing_duplicates_fixed](https://github.com/user-attachments/assets/db5e607e-373f-470d-a0ff-be810a7cf6aa)	
## 5. Removing Corrupt Data Entries
Some columns may have corrupt data entries which is incompatible for later analysis. In this section, we will check fincial columns such as `budget`, `domestic_opening`,` domestic_sales`, and `international_sales`.
```sql
-- There are some incorrect values inside some columns, such as column name budget

SELECT title, year, budget, domestic_opening, domestic_sales, international_sales
FROM highest_grossing_movies;

```
Result:	
![highest_grossing_corrupted_entries](https://github.com/user-attachments/assets/dfc79886-2439-4988-9a38-fbf245016026)	

The `budget` column has some incorrect values. Let's check how many movies have this problem. It is important to have this column with correct values for any future analysis. If that is not the case, we have to remove those movies from our table so that we can gain valuable insight into the remaining movies.	
```sql
SELECT *
FROM highest_grossing_movies
WHERE budget NOT REGEXP '^[0-9]+$';
-- 204 rows have corrupted fields for the budget column

-- for the sake of later analysis, lets remove movies with corrupt entries for the budget column

DELETE FROM highest_grossing_movies
WHERE budget NOT REGEXP '^[0-9]+$';
```

## 6. Simple Insights into Movies Dataset
Before some example analysis, let's make sure that columns `domestic_sales` and `international_sales` sum up as column `worldwide_sales`
Given this data, let's check if there is any movie in which there was no profit or the filmmaker lost. This means the movie `budget` has to be greater than the sum of `domestic_opening` and `worldwide_sales`.
Check if there are any `NULL` values
```sql
SELECT *
FROM highest_grossing_movies
WHERE domestic_sales IS NULL;
```
Result:	
![image](https://github.com/user-attachments/assets/fa20fda6-fd95-4b58-b3a4-53859442f53e)


```sql
SELECT *
FROM highest_grossing_movies
WHERE budget > (domestic_opening + worldwide_sales);

```
Result:	
![highest_grossing_profit_loss](https://github.com/user-attachments/assets/3e5336a6-e53b-4f03-9ae1-654364662f00)

We can check for every possible business insight using a large pool of SQL queries.  The following are just few of them.

```sql
SELECT title, (domestic_opening + domestic_sales + international_sales) AS total_sales
FROM highest_grossing_movies;

SELECT *, ((domestic_opening + domestic_sales + international_sales) - budget) AS profit
FROM highest_grossing_movies
ORDER BY profit DESC
LIMIT 10;
```
Result: Top ten high-profit movies	
![highest_grossing_profit_loss_top10](https://github.com/user-attachments/assets/b4984599-c92c-4563-8f13-b902552af6e9)

Some movies may have the same release date
```sql
-- Date data type in realease_date
SELECT release_date, GROUP_CONCAT(title SEPARATOR ', ') AS movies_released_inthis_date
FROM highest_grossing_movies
GROUP BY release_date
HAVING COUNT(*) > 1;
```
Result:
![image](https://github.com/user-attachments/assets/7558f10c-19e5-44e1-8f6b-d5ff9639b2a3)

```sql
SELECT *
FROM highest_grossing_movies
WHERE title IN ('The Others',' American Pie 2');

```
Result:	
![image](https://github.com/user-attachments/assets/1d8c3656-5ab1-45e8-9e4e-99e9a897bc5a)

## 7. Conclusion
In this project, we have demonstrated some of the main data preparation tasks using SQL queries and SQL data manipulation techniques to alter the data table and performed some data access queries to analyse the top movies database. For this project, we used MySQL database management software. 
