# Famous_Paintings_Analysis_(SQL_Analytics)

**Python Script to load the dataset(CSV files) into MySQL database**

````PYTHON
import pandas as pd
from sqlalchemy import create_engine
import config

# MySQL database configuration
db_config = {
    'user': config.username,
    'password': config.password,
    'host': config.host,
    'database': config.database,
    'raise_on_warnings': True
}

csv_files = ['artist','canvas_size', 'museum', 'museum_hours', 'product_size', 'subject', 'work' ]

for i in csv_files :
    
    # CSV file path
    csv_file_path = f'{i}.csv'

    # Table name in MySQL
    table_name = f'{i}'

    # Create a SQLAlchemy engine
    engine = create_engine(f"mysql+mysqlconnector://{db_config['user']}:{db_config['password']}@{db_config['host']}/{db_config['database']}")

    # Read CSV into a Pandas DataFrame
    df = pd.read_csv(csv_file_path)

    # Insert DataFrame records into MySQL using SQLAlchemy
    df.to_sql(name=table_name, con=engine, if_exists='replace', index=False)

    print("Data imported successfully.")
````

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

\
\
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

**2.3. What are the different canvas sizes in which the paintings are available in the museums?**

````SQL
SELECT row_number() over(order by count(*) desc) as sr_no, label, width, height, count(w.work_id) as no_of_paintings
FROM work w
JOIN product_size p on w.work_id = p.work_id
JOIN canvas_size c on p.size_id = c.size_id
WHERE w.museum_id IS NOT NULL
GROUP BY label, width, height
ORDER BY no_of_paintings DESC;
````

**Insights:**
- There are more than 60 different canvas sizes among which 30" long edge and 24" long edge are the widely used ones.

\
\
**3. Analyzing the popularity of museums, paintings, and artists.**
\
\
**3.1. Top 3 most popular museums**
- The popularity of museums is based on the number of paintings they house

````SQL
with museum_popularity as
(SELECT m.name as name, m.country, count(work_id) as no_of_paintings, DENSE_RANK() OVER(ORDER BY COUNT(work_id) DESC) as rnk
FROM museum m
JOIN work w
ON m.museum_id = w.museum_id
GROUP BY m.name)

SELECT name, country, no_of_paintings
FROM museum_popularity
WHERE rnk <= 3 ;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/807ae4f9-4d4f-45bd-b673-052ef57a882f)

**3.2. Top 3 most popular painting subjects**

````SQL
with popular_painting_subjects as 
(
SELECT s.subject as subject, COUNT(s.work_id) as no_of_paintings , DENSE_RANK() over( order by count(s.work_id) DESC ) as rnk
FROM work w 
join subject s
ON w.work_id = s.work_id
WHERE w.museum_id IS NOT NULL
GROUP BY s.subject 
 )

SELECT subject, no_of_paintings 
FROM popular_painting_subjects
WHERE rnk <=3;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/461a0853-428b-447a-88bd-5fab5ca0d735)

**3.3 Top 3 popular artists**

````SQL
with popular_painters as 
(
SELECT a.full_name as artist_name, COUNT(w.work_id) as no_of_paintings , DENSE_RANK() over( order by count(w.work_id) DESC ) as rnk
FROM work w 
join artist a
ON w.artist_id = a.artist_id
WHERE w.museum_id IS NOT NULL
GROUP BY a.full_name 
 )

SELECT artist_name, no_of_paintings 
FROM popular_painters
WHERE rnk <=3;
````

**Output:**

![image](https://github.com/Mangeshgp14/Famous_Paintings_Analysis_-SQL_Analytics-/assets/107695842/f28e5973-47c3-4c94-9a46-4ea5655bab54)
