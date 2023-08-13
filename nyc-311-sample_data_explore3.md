## Unveiling Borough Excellence: A Journey Through NYC's Service Performance

Join us as we embark on a data-driven adventure through New York City's service landscape. This journey takes us to the boroughs of NYC, revealing a story of efficiency, progress, and the power of diverse viewpoints. Through data exploration, we uncover which boroughs are excelling and improving over time, shedding light on their journey toward better service quality.

```sql
-- Generate normalized borough improvement and the average days per year.

WITH AVE_DAYS AS (
SELECT 
    "borough", "year", 
    ROUND(SUM("average_days"*"total_count")/ SUM("total_count"), 3) AS "Average Days",sum("total_count") as "Total Count"
FROM FACT_SERVICE_QUALITY
GROUP BY "borough", "year"
HAVING "Average Days" >=0)

SELECT
    "borough", "year",
    "Average Days" - (SELECT MIN("Average Days") FROM AVE_DAYS)) / ((SELECT MAX("Average Days") FROM AVE_DAYS) - (SELECT MIN("Average Days") FROM AVE_DAYS)) AS "Norm Average Days",
    ("Total Count" - (SELECT MIN("Total Count") FROM AVE_DAYS)) / ((SELECT MAX("Total Count") FROM AVE_DAYS) - (SELECT MIN("Total Count") FROM AVE_DAYS)) AS "Norm Total Count"
FROM AVE_DAYS
GROUP BY "borough", "year", "Average Days", "Total Count"
ORDER BY "borough", "year"; ORDER BY "borough", "year";
```

![fact4a](/assets/fact4a.png)


**Seeing from Different Angles -  The Magic of Data Exploration**

Imagine data exploration as putting on various lenses, each revealing a unique view of reality. By diving into borough performance data, we uncover hidden patterns and trends. Our first stop introduces a horizontal bar graph showcasing the normalized improvement and average response times for each borough. This graph offers an easy-to-understand snapshot of each borough's progress.

```sql
-- Generate list of boroughs and the average days per year.

SELECT 
    "borough",
    "year", 
    round(sum("average_days"*"total_count")/sum("total_count"), 3) "Average Days",sum("total_count") as "Total Count"
FROM FACT_SERVICE_QUALITY
GROUP BY "borough","year"
HAVING "Average Days" >= 0
ORDER BY "borough","year";
```

![fact4b](/assets/fact4b.png)

![fact4c](/assets/fact4c.png)

**Taking in the Wider View - Insights from Bar Graphs**

Continuing our exploration, we move to another perspective – a horizontal bar graph displaying boroughs alongside their average response times per year. This bird's-eye view allows direct comparisons, helping us understand how each borough performs based on count.

```sql
-- Generate contribution of top 3 agencies and average days per borough for the total duration.

SELECT
    "agency_name", "borough", 
    round(sum("average_days"*"total_count")/sum("total_count"), 3) AS "Average Days",
    sum("total_count") AS "Total Count"
FROM FACT_SERVICE_QUALITY
WHERE "agency_name" in ('NYPD','HPD','DOT')
GROUP BY "agency_name","borough"
HAVING "Average Days" >=0
ORDER BY "agency_name","borough";
```

![fact4d](/assets/fact4d.png)

![fact4e](/assets/fact4e.png)

**Celebrating Unsung Heroes - Top 3 Agencies and Remarkable Performance**

But the journey doesn't end there. Within the data, we find three agencies deserving special recognition. These agencies stand out for their exceptional response times, even with fewer transactions. Their contribution to borough efficiency underscores their dedication and hard work, proving that numbers can tell a compelling story of achievement.

![fact4f](/assets/fact4f.png)

So, join us as we navigate this visual journey through borough performance. Witness insights emerge, patterns take shape, and progress come to life. Through the magic of data visualization, we unravel NYC's borough story – a narrative that resonates with every resident, every service request, and every commitment to making things better.