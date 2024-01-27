# Famous_Paintings_Analysis_(SQL_Analytics)

## Questions and Solutions

**1. Fetch the no. of paintings that are not displayed in any museums.**

````SQL
SELECT count(name) as no_of_paintings_that_are_not_displayed_in_any_museums
FROM WORK 
WHERE museum_id is null ;
````
**Output:**
| no_of_paintings_that_are_not_displayed_in_any_museums |
|-------------------------------------------------------|
| 10223                                                 |

**2. Are there museums without any paintings?**

````SQL
SELECT m.name as "museum_that_don't_have_paintings"
FROM museum m
LEFT JOIN work w
on m.museum_id = w.museum_id
WHERE work_id is null ; 
````
**Output:**
| museum_that_don't_have_paintings |
|----------------------------------|
|                                  |

**It shows that there are no museums that don't have any paintings.**


