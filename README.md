# DEFORESTATION-Exploration
This is aproject in SQL degree program from UDACITY.

## Introduction

You’re a data analyst for ForestQuery, a non-profit organization, on a mission to reduce deforestation around the world and which raises awareness about this important environmental topic.

Your executive director and her leadership team members are looking to understand which countries and regions around the world seem to have forests that have been shrinking in size, and also which countries and regions have the most significant forest area, both in terms of amount and percent of total area. The hope is that these findings can help inform initiatives, communications, and personnel allocation to achieve the largest impact with the precious few resources that the organization has at its disposal.

You’ve been able to find tables of data online dealing with forestation as well as total land area and region groupings, and you’ve brought these tables together into a database that you’d like to query to answer some of the most important questions in preparation for a meeting with the ForestQuery executive team coming up in a few days. Ahead of the meeting, you’d like to prepare and disseminate a report for the leadership team that uses complete sentences to help them understand the global deforestation overview between 1990 and 2016.

## Table of contents
1. The pdf file **deforestation-exploration-project.pdf** is the project report.
2. **images** folder contains images of the outputs of all the queries which I have ran.

## CREATE VIEW

```sql
-- Create a View called “forestation” by joining all three tables - forest_area, land_area and regions.
DROP VIEW IF EXISTS forestation;

CREATE VIEW forestation  
AS
(SELECT f.country_code, f.country_name, f.year, f.forest_area_sqkm,
(l.total_area_sq_mi*2.59) AS total_area_sqkm,
r.region, r.income_group,
((SUM(forest_area_sqkm)/SUM(total_area_sq_mi*2.59))*100) AS percent_forest 
FROM forest_area AS f
JOIN land_area AS l
ON f.country_code = l.country_code
AND f.year = l.year
JOIN regions AS r
ON r.country_code = l.country_code
GROUP BY 1,2,3,4,5,6,7);
 
SELECT * 
FROM forestation;

```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/1.png)

## PART 1 : GLOBAL SITUATION

a) _What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as "World" in the region table.?_

```sql
WITH total_forest_area_1990 AS 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 1990 AND country_name = 'World')

SELECT * FROM total_forest_Area_1990;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/GS_a.png)

b) _What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can use the country record in the table is denoted as "World"?_

```sql
WITH total_forest_area_2016 AS 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 2016 AND country_name = 'World')

SELECT * FROM total_forest_Area_2016;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/GS_b.png)

c) _What was the change (in sq km) in the forest area of the world from 1990 to 2016?_

```sql
SELECT 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 1990 AND country_name = 'World') - 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 2016 AND country_name = 'World') AS change_in_forest_area 
FROM forestation
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/GS_c.png)

d) _What was the percent change in forest area of the world between 1990 and 2016?_

```sql
SELECT 
(((SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 1990 AND country_name = 'World') -
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 2016 AND country_name = 'World')) /
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 1990 AND country_name = 'World'))*100 AS percent_change_in_forest_area
FROM forestation
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/GS_d.png)

e) _If you compare the amount of forest area lost between 1990 and 2016, to which country's total area in 2016 is it closest to?_

```sql
SELECT country_name, total_area_sqkm, 
ABS((total_area_sqkm - (SELECT 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 1990 AND country_name = 'World') - 
(SELECT SUM(forest_area_sqkm) AS total_forest_area_sqkm
FROM forest_area
WHERE year = 2016 AND country_name = 'World')))) AS forest_area_lost
FROM forestation
WHERE year = 2016
GROUP BY 1,2
ORDER BY 3
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/GS_e.png)

## PART 2 : REGIONAL OUTLOOK

a) _What was the percent forest of the entire world in 2016? Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2 decimal places?_

```sql
WITH forest_percentage_2016
AS
(SELECT region, 
ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forest_area
FROM forestation
WHERE year = 2016
GROUP BY 1
ORDER BY 2 DESC)

-- the percent forest of the entire world in 2016
SELECT * FROM forest_percentage_2016
WHERE region = 'World';
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_a1.png)

```sql
-- Which region had the HIGHEST percent forest in 2016
SELECT * FROM forest_percentage_2016
ORDER BY 2 DESC
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_a2.png)

```sql
-- which had the LOWEST, to 2 decimal places
SELECT * FROM forest_percentage_2016
ORDER BY 2
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_a3.png)

b) _What was the percent forest of the entire world in 1990? Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2 decimal places?_

```sql
WITH forest_percentage_1990
AS
(SELECT region, 
ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forest_area
FROM forestation
WHERE year = 1990
GROUP BY 1
ORDER BY 2 DESC)

-- the percent forest of the entire world in 1990
SELECT * FROM forest_percentage_1990
WHERE region = 'World';
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_b1.png)

```sql
-- Which region had the HIGHEST percent forest in 1990
SELECT * FROM forest_percentage_1990
ORDER BY 2 DESC
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_b2.png)

```sql
-- which had the LOWEST, to 2 decimal places
SELECT * FROM forest_percentage_1990
ORDER BY 2
LIMIT 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_b3.png)

c) _Based on the table you created, which regions of the world DECREASED in forest area from 1990 to 2016?_

```sql
WITH forest_percentage_2016
AS
(SELECT region, 
ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forest_area
FROM forestation
WHERE year = 2016
GROUP BY 1),
forest_percentage_1990
AS
(SELECT region, 
ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forest_area
FROM forestation
WHERE year = 1990
GROUP BY 1)

SELECT fp1990.region,
ROUND(fp1990.percent_forest_area::NUMERIC, 2) AS percent_fa_1990,
ROUND(fp2016.percent_forest_area::NUMERIC, 2) AS percent_fa_2016
FROM forest_percentage_1990 AS fp1990
JOIN forest_percentage_2016 AS fp2016
ON fp1990.region = fp2016.region
WHERE fp1990.percent_forest_area > fp2016.percent_forest_area
GROUP BY 1,2,3;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/RO_c.png)

## PART 3 : COUNTRY-LEVEL DETAILS

a) _Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What was the difference in forest area for each?_

```sql
WITH forest_change_1990
AS
(SELECT region, country_name, SUM(forest_area_sqkm) AS total_forest_area1
FROM forestation
WHERE year = 1990
GROUP BY 1,2),

forest_change_2016
AS
(SELECT region, country_name, SUM(forest_area_sqkm) AS total_forest_area2
FROM forestation
WHERE year = 2016
GROUP BY 1,2)

SELECT f1.country_name, f1.region, ROUND((f1.total_forest_area1-f2.total_forest_area2)::NUMERIC,2) AS forest_change_sqkm
FROM forest_change_1990 f1
JOIN forest_change_2016 f2                                         ON f1.country_name = f2.country_name
AND (f1.total_forest_area1 IS NOT NULL AND f2.total_forest_area2 IS NOT NULL) AND f1.region != 'World'
ORDER BY 3 DESC                                                     
LIMIT 5; 
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/CL_a.png)

b) _Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each?_

```sql
WITH forest_change_1990
AS
(SELECT region, country_name, SUM(forest_area_sqkm) AS total_forest_area1
FROM forestation
WHERE year = 1990
GROUP BY 1,2),

forest_change_2016
AS
(SELECT region, country_name, SUM(forest_area_sqkm) AS total_forest_area2
FROM forestation
WHERE year = 2016
GROUP BY 1,2)

SELECT f1.country_name, f1.region, ROUND((((f1.total_forest_area1-f2.total_forest_area2)/(f1.total_forest_area1))*100)::NUMERIC,2) AS percent_forest_change
FROM forest_change_1990 f1
JOIN forest_change_2016 f2                                         ON f1.country_name = f2.country_name
AND (f1.total_forest_area1 IS NOT NULL AND f2.total_forest_area2 IS NOT NULL) AND f1.region != 'World'
ORDER BY 3 DESC                                                     
LIMIT 5;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/CL_b.png)

c) _If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016?_

```sql
WITH fc2016
AS
(SELECT region, country_name, ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forestation
FROM forestation
WHERE year = 2016 AND region != 'World' AND forest_area_sqkm IS NOT NULL AND total_area_sqkm IS NOT NULL
GROUP BY 1,2),
quartiles
AS
(SELECT fc2016.region, fc2016.country_name,
 CASE WHEN fc2016.percent_forestation >= 75 THEN '75%-100%'
 	  WHEN fc2016.percent_forestation BETWEEN 50 AND 75 THEN '50%-75%'       
      WHEN fc2016.percent_forestation BETWEEN 25 AND 50 THEN '25%-75%'
      WHEN fc2016.percent_forestation <= 25 THEN '0-25%'
 	  END AS percentile
 FROM fc2016
 ORDER BY 3 DESC)
 
 SELECT quartiles.percentile AS Quartiles,
 		COUNT(quartiles.percentile) AS number_of_countries
FROM quartiles
GROUP BY 1
ORDER BY 2 DESC;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/CL_c.png)

d) _List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016._

```sql
WITH fc2016
AS
(SELECT region, country_name, ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forestation
FROM forestation
WHERE year = 2016 AND region != 'World' AND forest_area_sqkm IS NOT NULL AND total_area_sqkm IS NOT NULL
GROUP BY 1,2),
quartiles
AS
(SELECT fc2016.region, fc2016.country_name,
 CASE WHEN fc2016.percent_forestation >= 75 THEN '75%-100%'
 	  WHEN fc2016.percent_forestation BETWEEN 50 AND 75 THEN '50%-75%'       
      WHEN fc2016.percent_forestation BETWEEN 25 AND 50 THEN '25%-75%'
      WHEN fc2016.percent_forestation <= 25 THEN '0-25%'
 	  END AS percentile
 FROM fc2016
 ORDER BY 3 DESC)
 
SELECT quartiles.country_name, quartiles.region,
ROUND((fc2016.percent_forestation)::NUMERIC, 2) AS Pct_Designated_Forest 
FROM quartiles 
JOIN fc2016 
ON fc2016.country_name = quartiles.country_name
WHERE quartiles.percentile = '75%-100%'
GROUP BY 1,2,3
ORDER BY 1;
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/CL_d.png)

e) _How many countries had a percent forestation higher than the United States in 2016?_

```sql
WITH fc2016
AS
(SELECT region, country_name, ROUND(((SUM(forest_area_sqkm)/SUM(total_area_sqkm))*100)::NUMERIC, 2) AS percent_forestation
FROM forestation
WHERE year = 2016 AND region != 'World' AND forest_area_sqkm IS NOT NULL AND total_area_sqkm IS NOT NULL
GROUP BY 1,2)

SELECT COUNT(fc2016.country_name) AS no_countries
FROM fc2016
WHERE fc2016.percent_forestation > (SELECT fc2016.percent_forestation FROM fc2016
WHERE country_name = 'United States');
```
![](https://github.com/siddhu1132/DEFORESTATION-Exploration/blob/main/Images/CL_e.png)

