# Famous_Paintings_Analysis_(SQL_Analytics)

## Questions and Solutions
**Our client is a museum chain owner who wants to analyze their business. They want us to analyze the CSV files that store the data about the different museums, the paintings they showcase, the artists of these paintings, etc.** 
\
\
**Data Pipeline: EXCEL -> PYTHON -> MYSQL**
\
\
**1. Getting an overview of the business**

**1.1. Analyzing the number of museums and their locations**

````SQL
UPDATE museum
SET country = 'UK'
WHERE country = 'United Kingdom';
````

````SQL
SELECT row_number() over(order by count(*) desc) as sr_no ,country , count(*) as number_of_museums
FROM museum
GROUP BY country
ORDER BY number_of_museums desc;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/218c4830-34b6-4220-a7c0-2f3ccdf6b13c)



**Insight**
- The company operates in 16 countries and has the highest number of museums in the USA.

**1.2. Find out how many paintings are housed in each nation's museums.**

````SQL
SELECT row_number() over(order by count(*) desc) as sr_no , m.country as country , count(work_id) as number_of_paintings
FROM museum m
LEFT JOIN work w
ON m.museum_id = w.museum_id
GROUP BY m.country
ORDER BY number_of_paintings desc;
````
**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/c869ca00-809c-478d-8648-751d787c10cd)

**Insights:**
- Museums in the USA lead in the no. of paintings as well.
- Despite having fewer museums, the UK and the Netherlands house more paintings than France.

**1.3. Obtain the number of paintings that are not displayed in any museum**

````SQL
SELECT count(work_id) as number_of_paintings_that_are_not_displayed_in_museum
FROM museum m
RIGHT JOIN work w
ON m.museum_id = w.museum_id
WHERE w.museum_id is NULL;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/24f0ac3c-e237-4a24-b55e-23b64147d6ba)



Fetch the no. of paintings that are not displayed in any museums.**

````SQL
SELECT count(name) as no_of_paintings_that_are_not_displayed_in_any_museums
FROM WORK 
WHERE museum_id is null ;
````
**Output:**
| no_of_paintings_that_are_not_displayed_in_any_museums |
|-------------------------------------------------------|
| 10223                                                 |

**2. Analyzing the different types of paintings**

**2.1. What are the different styles of paintings available in these museums?**

````SQL
SELECT row_number() over(order by count(*) desc) as sr_no, w.style as styles, count(*) as no_of_paintings
FROM work w
JOIN museum m
ON w.museum_id = m.museum_id
GROUP BY w.style
ORDER BY no_of_paintings DESC ;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/511da7c3-e8eb-44ea-a278-e60a7d28289e)


**Insights:**
- There are a total of 23 different styles of paintings that are displayed in these museums.
- Impressionism and Baroque are the top 2 styles of paintings that are most displayed.
- There are a significant number of paintings that don't have any style.


**2.2. What are the different types of subjects that the paintings in these museums belong to?**
- Here we need to take care not to include those paintings that are not displayed in any museum.

````SQL
SELECT row_number() over(order by count(*) desc) as sr_no, s.subject, count(*) as no_of_paintings
FROM work w
JOIN subject s
on w.work_id = s.work_id
WHERE w.museum_id IS NOT NULL
GROUP BY s.subject
````
**Insights:**
- There are around 30 different subjects' paintings in all the museums.
- Portraits subject is the highest in number.

