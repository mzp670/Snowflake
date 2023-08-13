## Evolving Excellence: Unveiling NYC Agencies' Service Quality Improvements

**Visualizing Progress - The Art of Data Presentation**

Imagine being able to see how different New York City agencies are getting better at handling service requests over time. It's like watching a story unfold as agencies work to improve their performance. In the world of data, we can create charts and graphs that show us this story in a visual way. These visuals help us understand complex information quickly and make informed decisions.

```sql
-- Generate yearly agency performance based on average days and plot using line graph

SELECT 
    "agency_name",
    "year", 
    round(sum("average_days"*"total_count")/sum("total_count"), 3) as "Average Days",
    sum("total_count") as "Total Count"
FROM FACT_SERVICE_QUALITY
GROUP BY "agency_name","year"
HAVING "Average Days" >=0
ORDER BY "agency_name","year";
```

![fact3a](/assets/fact3a.png)

![fact3b](/assets/fact3b.png)


**Unlocking Insights - The Power of Normalized Data**

Now, let's dive a bit deeper into the magic behind the scenes. When we work with data, it's important to make sure we're comparing things fairly. This is where normalized data comes into play. Think of it as leveling the playing field. By using techniques like min-max normalization, we can put different agencies' performance on the same scale. This makes it easier to see patterns and trends without being misled by big numbers.

```sql
-- Generate yearly agency performance based on normalized average days using min-max and plot using line graph

WITH AVE_DAYS AS (
SELECT 
    "agency_name", 
    "year", 
    ROUND(SUM("average_days"*"total_count")/ SUM("total_count"), 3) AS "Average Days",
    sum("total_count") as "Total Count"
FROM FACT_SERVICE_QUALITY
GROUP BY "agency_name", "year"
HAVING "Average Days" >=0)

SELECT
    "agency_name",
    "year",
    ("Average Days" - (SELECT MIN("Average Days") FROM AVE_DAYS)) / ((SELECT MAX("Average Days") FROM AVE_DAYS) - (SELECT MIN("Average Days") FROM AVE_DAYS)) AS "Norm Average Days",
    ("Total Count" - (SELECT MIN("Total Count") FROM AVE_DAYS)) / ((SELECT MAX("Total Count") FROM AVE_DAYS) - (SELECT MIN("Total Count") FROM AVE_DAYS)) AS "Norm Total Count"
FROM AVE_DAYS
GROUP BY "agency_name", "year", "Average Days", "Total Count"
ORDER BY "agency_name", "year";
```

![fact3c](/assets/fact3c.png)

![fact3d](/assets/fact3d.png)

![fact3e](/assets/fact3e.png)

**Celebrating Success - Spotlight on Top Performers**

Finally, let's celebrate the stars of the show! We've identified the top three agencies that are truly shining when it comes to handling service requests. These agencies are not only processing requests efficiently but also maintaining a high level of service quality. It's heartening to see their dedication making a positive impact on the city and its residents.

```sql
--- Generate top 3 agencies based on highest number of service request total count.

SELECT "agency_name", 
    round(sum("average_days"*"total_count")/sum("total_count"), 3) AS "Average Days",
    sum("total_count") AS "Total Count"
FROM FACT_SERVICE_QUALITY
GROUP BY "agency_name"
HAVING "Average Days" >=0
ORDER BY "Total Count" desc
LIMIT 3;
```

![fact3f](/assets/fact3f.png)

In the realm of data exploration, each line of code and each visualization is a step toward uncovering insights and understanding the world around us. As we navigate through this digital landscape, we discover stories of improvement, innovation, and collaboration, all brought to life by the power of data and its artful presentation.