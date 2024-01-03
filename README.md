# Baby_Name_Trend_USA
A detailed analysis of baby names in USA from 1980 to 2009.
## Situation: A baby names website collects data on names that parents give their children. The baby names website is about to release their annual names report. I have to dig into the data to produce some interesting findings about baby names over the years to share in the report of my objectives.
### Objective 1 Track changes in name popularity
#### The first objective is to see how the most popular names have changed over time, and also to identify the names that have jumped the most in terms of popularity.
##### 1. Find the overall most popular girl and boy names and show how they have changed in popularity rankings over the years.
##### Query
` SELECT Name, SUM(Births) AS num_babies `  
` FROM names `  
` WHERE Gender = 'F' `  
` GROUP BY Name `  
` ORDER BY num_babies DESC `  
` LIMIT 1; `  
##### Results  
|Name|num_babies|
|:---------:|:----------:|
|Jessica|863121|
##### Query  
` SELECT Name, SUM(Births) AS num_babies `  
` FROM names `  
` WHERE Gender = 'M' `  
` GROUP BY Name `  
` ORDER BY num_babies DESC `  
` LIMIT 1; `  
##### Results
|Name|num_babies|
|:----------:|:----------:|
|Michael|1376418|
##### Query 
` SELECT * `   
` FROM `  
` (WITH girl_names AS (SELECT Year, Name, SUM(Births) AS num_babies `  
` FROM names `  
` WHERE Gender = 'F' `  
` GROUP BY Year, Name) `  
` SELECT Year, Name, `  
`		ROW_NUMBER() OVER (PARTITION BY Year ORDER BY num_babies DESC) AS popularity `  
` FROM girl_names) AS popular_girl_names `  
` WHERE Name = 'Jessica' `  
##### Results  
|Year|Name|popularity|
|:-----:|:---------:|:-----:|
|1980|Jessica|3|
|1981|Jessica|2|
|1982|Jessica|2|
|1983|Jessica|2|
|1984|Jessica|2|
##### Jessica was a popular name in 1980s and its popularity further rose in 1990s but, then it started to lose its popularity from early 2000s. 
##### Query
` SELECT * `   
` FROM `  
` (WITH boy_names AS (SELECT Year, Name, SUM(Births) AS num_babies `  
` FROM names `  
` WHERE Gender = 'M' `  
` GROUP BY Year, Name) `  
` SELECT Year, Name, `  
`		ROW_NUMBER() OVER (PARTITION BY Year ORDER BY num_babies DESC) AS popularity `  
` FROM boy_names) AS popular_boy_names `  
` WHERE Name = 'Michael' `  
##### Results
|Year|Name|popularity|
|:------:|:-------------:|:--------:|
|1980|Michael|1|
|1981|Michael|1|
|1982|	Michael|1|
|1983|	Michael|1|
|1984|Michael|1|
##### Michael had remained popular throughout this period from 1980s to 2009.

##### 2. Find the names with the biggest jumps in popularity from the first year of the data set to the last year
##### Query
` WITH names_1980 AS ( `  
` 	WITH all_names AS (SELECT Year, Name, SUM(Births) AS num_babies `  
` 	FROM names `  
` 	GROUP BY Year, Name) `  
` 	SELECT Year, Name, `  
` 			ROW_NUMBER() OVER (PARTITION BY Year ORDER BY num_babies DESC) AS popularity `  
` 	FROM all_names `  
`     WHERE Year = 1980), `  
` names_2009 AS ( `  
` 	WITH all_names AS (SELECT Year, Name, SUM(Births) AS num_babies `  
` 	FROM names `  
` 	GROUP BY Year, Name) `  
` 	SELECT Year, Name, `  
` 			ROW_NUMBER() OVER (PARTITION BY Year ORDER BY num_babies DESC) AS popularity `  
` 	FROM all_names `  
`     WHERE Year = 2009) `  
`     SELECT t1.Year, t1.Name, t1.popularity, t2.Year, t2.Name, t2.popularity, `  
`    CAST(t2.popularity AS SIGNED) - CAST(t1.popularity AS SIGNED) AS diff `  
`     FROM names_1980 t1 INNER JOIN names_2009 t2 `  
` 		ON t1.Name = t2.Name `  
` ORDER BY diff `  
` LIMIT 10; `  
##### Results
|Year|Name|popularity|Year|Name|popularity|diff|
|:------:|:--------:|:-------:|:-----:|:---------:|:----------:|:-------:|
|1980|	Colton|5789|2009|Colton|149|-5640|
|1980|	Aidan|5691|2009|Aidan|109|-5582|
|1980|Rowan|5772|2009|Rowan|445|-5327|

### OBJECTIVE 2: Compare popularity across decades
#### Your second objective is to find the top 3 girl names and top 3 boy names for each year, and also for each decade.
##### 1. For each year, return the 3 most popular girl names and 3 most popular boy names
##### Query
` SELECT * FROM `  
` (WITH babies_by_year AS (SELECT Year, Name, Gender, SUM(Births) AS num_babies `  
` FROM names `
` GROUP BY Year, Gender, Name) `
` SELECT Year, Gender, Name, num_babies, `  
`	 	ROW_NUMBER() OVER (PARTItION BY Year, Gender ORDER BY num_babies DESC) AS popularity `  
` FROM babies_by_year) AS top_three `  
` WHERE popularity < 4; `  
##### Results 
|Year|Gender|Name|num_babies|popularity|
|:-----:|:---:|:--------:|:-------:|:-------:|
|1980|	F|Jennifer|58379|1|
|1980|	F|Amanda|35819|2|
|1980|	F|Jessica|33923|3|
##### 2. For each decade, return the 3 most popular girl names and 3 most popular boy names
##### Query
` SELECT * FROM `
` (WITH babies_by_decade AS (SELECT (CASE WHEN Year BETWEEN 1980 and 1989 THEN '1980s' `  
` 		WHEN Year BETWEEN 1990 and 1999 THEN '1990s' `  
`                                 WHEN Year BETWEEN 2000 and 2009 THEN '2000s' `  
`                                ELSE 'None' END) AS decade, `  
` Name, Gender, SUM(Births) AS num_babies `  
` FROM names `  
` GROUP BY decade, Gender, Name) `  

` SELECT decade, Gender, Name, num_babies, `  
` 		ROW_NUMBER() OVER (PARTItION BY decade, Gender ORDER BY num_babies DESC) AS popularity `  
` FROM babies_by_decade) AS top_three `  
` WHERE popularity < 4; `  
##### Results
|Year|Gender|Name|num_babies|popularity|
|:------:|:-----:|:---------:|:---------:|:---------:|
|1980s	|F|Jessica|469452|1|
|1980s	|F|Jennifer|440845|2|
|1980s|F|Amanda|369705|3|

### OBJECTIVE 3: Compare popularity across regions
#### Your third objective is to find the number of babies born in each region, and also return the top 3 girl names and top 3 boy names within each region.
##### 1. Return the number of babies born in each of the six regions (NOTE: The state of MI should be in the Midwest region)
##### Query
` WITH clean_regions AS (SELECT State, `   
` 	CASE WHEN Region = 'New England' THEN 'New_England' ELSE Region END AS clean_regions `  
` FROM regions `  
` UNION `  
` SELECT 'MI' AS State, 'Midwest' AS Region) `  
` SELECT clean_regions, SUM(Births) AS num_babies `   
` FROM names n LEFT JOIN clean_regions cr `   
` 	ON n.State = cr.State `  
` GROUP BY clean_regions `  
##### Results
|Clean_regions|num_babies|
|:---------------:|:---------:|
|Mid_Atlantic|13742667|
|Midwest|22676130|
|Mountain|6282217|

##### 2. For each decade, return the 3 most popular girl names and 3 most popular boy names
##### Query
` SELECT * ` 
` FROM `  

` (WITH babies_by_region AS ( `  
` 	WITH clean_regions AS (SELECT State, `   
` 		CASE WHEN Region = 'New England' THEN 'New_England' ELSE Region END AS clean_regions `  
` 	FROM regions `  
` 	UNION `  
` 	SELECT 'MI' AS State, 'Midwest' AS Region) `  

` 	SELECT cr.clean_regions, n.Gender, n.Name, SUM(n.Births) AS num_babies `   
` 	FROM names n LEFT JOIN clean_regions cr `   
` 		ON n.State = cr.State `  
` 	GROUP BY cr.clean_regions, n.Gender, n.Name) `  
    
` SELECT clean_regions, Gender, Name, `  
`  	ROW_NUMBER() OVER (PARTITION BY clean_regions, Gender ORDER BY num_babies DESC) AS popularity `  
` FROM babies_by_region) AS region_popularity `  
` WHERE popularity <4; `  
##### Results
|Clean_regions|Gender|name|popularity|
|:----------:|:-----:|:----------:|:--------:|
|Mid_Atlantic|F|Jessica|1|
|Mid_Atlantic|F|Ashley|2|
|Mid_Atlantic|F|Jennifer|3|
### OBJECTIVE 4: Explore unique names in the dataset
#### Your final objective is to find the most popular androgynous names, the shortest and longest names, and the state with the highest percent of babies named "Chris".
##### 1. Find the 10 most popular androgynous names (names given to both females and males).
##### Query
` SELECT Name, COUNT(DISTINCT Gender) AS num_genders, SUM(Births) AS num_babies `  
` FROM names `  
` GROUP BY Name `  
` HAVING num_genders = 2 `  
` ORDER BY num_babies DESC `  
` LIMIT 10; `  
##### Results
|Name|num_genders|num_babies|
|:-----------:|:------:|:------------:|
|Michael|2|1382856|
|Christopher|2|1122213|
|Matthew|2|1034494|
|Joshua|2|960170|
##### 2. Find the length of the shortest and longest names, and identify the most popular short names (those with the fewest characters) and long names (those with the most characters)
##### Query
` SELECT Name, LENGTH(Name) AS name_length `  
` FROM names `  
` ORDER BY name_length; `  
##### Results
|Name|name_length|
|:----------:|:-------:|
|Ty|2|
##### Query
` SELECT Name, LENGTH(Name) AS name_length `  
` FROM names `  
` ORDER BY name_length DESC; `  
##### Results
|Name|name_length|
|:-----------:|:---------:|
|Franciscojavier|15|
##### Query
` WITH short_long_names AS (SELECT * `  
` FROM names `  
` WHERE LENGTH(Name) IN (2)) `  

` SELECT Name, SUM(Births) AS num_babies `  
` FROM short_long_names `  
` GROUP BY Name `  
` ORDER BY num_babies DESC; `  

##### Results
|Name|num_babies|
|:-------------:|:-------:|
|Ty|29205|
##### Query
` WITH short_long_names AS (SELECT * `  
` FROM names `  
` WHERE LENGTH(Name) IN (15)) `  

` SELECT Name, SUM(Births) AS num_babies `  
` FROM short_long_names `  
` GROUP BY Name `  
` ORDER BY num_babies DESC; `  
##### Results 
|Name|num_babies|
|:------------:|:--------:|
|Franciscojavier|52|

##### 3. The founder of Maven Analytics is named Chris. Find the state with the highest percent of babies named "Chris"
` SELECT State, num_chris/num_babies * 100 AS pct_chris `  
` FROM `  
` (WITH count_chris AS (SELECT State, SUM(Births) AS num_chris `  
` FROM names `  
` WHERE Name = 'Chris' `  
` GROUP BY State), `  

` count_all AS (SELECT State, SUM(Births) AS num_babies `  
` FROM names `  
` GROUP BY State) `  

` SELECT cc.State, cc.num_chris, ca.num_babies `  
` FROM count_chris cc INNER JOIN count_all ca `  
` 	ON cc.State = ca.State) AS state_chris_all `  
    
` ORDER BY pct_chris DESC; `  
##### Results
|State|pct_chris|
|:-----:|:--------:|
|LA|0.0335|
|HI|0.0301|
|NY|0.0296|

