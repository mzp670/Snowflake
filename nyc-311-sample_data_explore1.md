## Unveiling Agency Diversity: A Tale of Multi-Agency Handlings

In the bustling city of Metropolis, the NYC311 service orchestrates a symphony of citizen requests, each note echoing the diversity of concerns that make up urban life. Among the harmonies and cacophonies, an intriguing question emerges: Which complaint types, rather than being under the jurisdiction of a single agency, find themselves tended to by multiple entities?

Delving into the data, we embark on a journey to decipher the complex web of multi-agency involvements. Armed with a query, we navigate through the labyrinthine corridors of the database, revealing the intricate threads that tie these requests to more than one agency's care.

In the world of structured SQL, our journey unfolds:

```sql
-- Generate list of complaint types handled by more than one agency. 

SELECT "Complaint_Type_Sanitized", COUNT("Agency") as Agency_Count,
    CASE WHEN "Complaint_Type_Sanitized" = 'OTHERS' THEN 'Various' 
    ELSE LISTAGG("Agency", ', ') WITHIN GROUP (ORDER BY "Agency") END AS Agency_List
FROM (SELECT DISTINCT "Complaint_Type_Sanitized", "Agency" FROM FACT_SERVICE_QUALITY_SANITIZED) AS subquery
GROUP BY "Complaint_Type_Sanitized"
HAVING Agency_Count > 1
ORDER BY Agency_Count Desc,"Complaint_Type_Sanitized";

```

![fact3](/assets/fact3.png)

Imagine you're exploring a treasure trove of information hidden in a computer. With a special spell called SQL, we're able to uncover a list of problems that aren't just handled by one group, but by several teams working together. Each problem tells a unique story, revealing how different groups join forces to tackle challenges in the city.

Think of it like a big puzzle, where each piece is a different problem and each agency is a person helping to solve it. When we put the pieces together, we see a picture of teamwork and collaboration that makes the city better. Even though things might seem complicated, this teamwork shows that when people come together, they can make a positive impact on the community.