## Polishing Insights: The Imperative of Pristine Data Cleansing

Data cleansing is a vital practice in ensuring the accuracy, reliability, and usefulness of any dataset. As data accumulates from various sources, inconsistencies, errors, and redundancies can creep in, leading to skewed insights and flawed decision-making. By meticulously cleaning and refining the data, organizations can trust that the information they rely on is consistent, complete, and valid. This not only enhances the quality of analysis but also paves the way for more accurate predictions, effective strategies, and informed business choices.


**Data Refinement Steps:**

*Complaint Type:*
Created new column to clean Complaint Type data. Valid values extracted from dimension_complaint_type, discrepancies labeled "OTHERS." Text transformed to uppercase for consistency; unwanted characters removed using "regexp_replace" function.

*Incident Zip:*
Extracted leftmost substring for Incident Zip, adhering to NYC zip code format. Non-numeric entries cleared using "regex_replace." Mismatched zip codes reclassified as "Other," dimension_zip adjusted accordingly.

*Borough:*
Null/unspecified data in Borough reclassified as “Others” using "Case When" method. Reduced data gaps, retained borough names' integrity.

*Year Week:*
Fine-tuned year week computation in dimension_year_week. Adjusted formula "datediff(WEEK, min("Created_Date"), max("Created_Date")) + 1" for complete week inclusion in fact service quality table recreation.

By applying these data refinement strategies, the irregularities within all four columns were addressed comprehensively, resulting in the substantial reduction of missing rows within the analysis.


```sql
CREATE OR REPLACE VIEW FACT_SERVICE_QUALITY_SANITIZED1 AS
    SELECT 
    "Unique_Key", 
    "Created_Date",
    TO_CHAR("Created_Date", 'YYYYMM') AS "Year Month",
    TO_CHAR("Created_Date", 'YYYY') AS "Year",
    "Closed_Date", 
    "Agency", 
    "Agency_Name", 
    "Complaint_Type",
    CASE WHEN UPPER("Complaint_Type") IN ('PAINT - PLASTER', 'PAINT/PLASTER') THEN 'PAINT - PLASTER' 
        WHEN UPPER(REGEXP_REPLACE("Complaint_Type", '[^a-zA-Z0-9]', '')) IN 
        (SELECT UPPER(REGEXP_REPLACE("type", '[^a-zA-Z0-9]', '')) 
        FROM COMPLAINT_TYPE_REFERENCE_26) 
    THEN UPPER("Complaint_Type") 
    ELSE 'OTHERS' END AS "Complaint_Type_Sanitized",
    "Descriptor", 
    "Location_Type", 
    "Incident_Zip",
    CASE WHEN LEFT(REGEXP_REPLACE("Incident_Zip", '[^0-9]', ''),5) IN 
        (SELECT ZIP FROM ZIP_NYC_BOROUGH)
        THEN LEFT(REGEXP_REPLACE("Incident_Zip", '[^0-9]', ''),5) 
        ELSE 'Other' END AS "Incident_Zip_Sanitized",
    "Incident_Address", 
    "City",
    "Status", 
    "Due_Date", 
    "Resolution_Description",
    "Resolution_Action_Updated_Date", 
    "Community_Board", 
    BBL, 
    "Borough",
    CASE WHEN "Borough" in 
        (SELECT UPPER("borough") FROM ZIP_NYC_BOROUGH)
        THEN "Borough" ELSE 'OTHERS' END AS "Borough_Sanitized",
    "X_Coordinate_(State Plane)", 
    "Y_Coordinate_(State Plane)", 
    "Open_Data_Channel_Type", 
    "Location"
FROM NYC311.SERVICE_REQUEST_32M;
```


```sql
-- CREATION AND DATA POPULATION OF A NEW TABLE FOR YEAR WEEK.

CREATE TABLE DIMENSION_YEAR_WEEK
("year_week" INTEGER PRIMARY KEY);
SET week_count = (SELECT datediff(WEEK, min("Created_Date"), max("Created_Date"))+1 AS week_count 
FROM NYC311.service_request_32m);
SELECT $week_count;
SET start_date = (SELECT min("Created_Date") AS sd FROM NYC311.service_request_32m);  	
SELECT $start_date;

INSERT INTO DIMENSION_YEAR_WEEK

WITH date_range AS
    (SELECT dateadd(WEEK, SEQ4(), $start_date) AS date_value
    FROM table (generator(rowcount => $week_count)))
    SELECT yearofweek(date_value) * 100 + week(date_value)
    FROM date_range;
```

After creating the sanitized version of the NYC311 schema and updating the “dimension_year_week” table, an updated Fact Service Quality table is generated. This updated table includes additional columns such as borough, agency name, complaint type and year, which are necessary for tasks 2 to 4.

```sql
-- Drop the old Fact Service Quality Table and recreate a new one.

CREATE OR REPLACE TABLE FACT_SERVICE_QUALITY (
    "agency_id" INTEGER, 
    "agency_name" VARCHAR(10), 
    "ZIP" VARCHAR(5), 
    "borough" VARCHAR(255), 
    "type_id" INTEGER,
    "type" VARCHAR(255), 
    "year_week" INTEGER,
    "year_month" INTEGER,
    "year" INTEGER,
    "total_count" INTEGER, 
    "average_days" FLOAT, 
    "stddev_days" FLOAT, 
    "range_days" FLOAT,
    PRIMARY KEY ("agency_id", "agency_name", "ZIP", "borough", "type_id", "type","year_week", "year_month"),
    FOREIGN KEY ("agency_id") REFERENCES DIMENSION_AGENCY ("agency_id"),
    FOREIGN KEY ("ZIP") REFERENCES DIMENSION_ZIP ("ZIP"),
    FOREIGN KEY ("type_id") REFERENCES DIMENSION_COMPLAINT_TYPE ("type_id"),
    FOREIGN KEY ("year_week") REFERENCES DIMENSION_YEAR_WEEK ("year_week"),
    FOREIGN KEY ("year_month") REFERENCES DIMENSION_YEAR_MONTH ("year_month")
);
```

Once the table is created, data is populated from the sanitized view to the Fact Service Quality table. To ensure proper joining of the modified complaint type column, the “upper” function is applied to the “type” column of the complaint type dimension table.

```sql
-- Populate the table using data from sanitized view.

INSERT INTO FACT_SERVICE_QUALITY
SELECT 
    DA."agency_id", 
    DA."agency_name",
    DB.ZIP, 
    DB."borough", 
    DC."type_id", 
    DC."type",
    DT."year_week", 
    DM."year_month",
    DM."year",
    COUNT(*) AS "total_count", 
    ROUND(AVG(DATEDIFF('second', "Created_Date", "Closed_Date")) / (60 * 60 * 24), 3) AS "average_days", 
    ROUND(STDDEV(DATEDIFF('second', "Created_Date", "Closed_Date")) / (60 * 60 * 24), 3) AS "stddev_days",
    ROUND(MAX(DATEDIFF('second', "Created_Date", "Closed_Date")) / (60 * 60 * 24), 3) AS "range_days"
FROM 
    FACT_SERVICE_QUALITY_SANITIZED SR
    JOIN DIMENSION_AGENCY DA ON SR."Agency" = DA."agency_name"
    JOIN DIMENSION_ZIP DB ON SR."Incident_Zip_Sanitized" = DB.ZIP
    JOIN DIMENSION_COMPLAINT_TYPE DC ON SR."Complaint_Type_Sanitized" = UPPER(DC."type")
    JOIN DIMENSION_YEAR_WEEK DT ON YEAROFWEEK(SR."Created_Date") * 100 + WEEK(SR."Created_Date") = DT."year_week"
    JOIN DIMENSION_YEAR_MONTH DM ON TO_CHAR(SR."Created_Date", 'YYYYMM') = DM."year_month"
GROUP BY 
    DA."agency_id",
    DA."agency_name",
    DB.ZIP, 
    DB."borough",
    DC."type_id", 
    DC."type",
    DT."year_week", 
    DM."year_month",
    DM."year";

```

With the following query, we can examine the updated fact rows which is now 3.6M and the number of covered complaints increased from 14M to 32M. There is no more missing data because of the corrections made resulting to 100% coverage of the complaint type from the overall dataset.

```sql
-- Generate and run a query for summary. 

SELECT 
    (SELECT COUNT(*) FROM FACT_SERVICE_QUALITY) AS "# Fact Rows",
    (SELECT SUM("total_count") FROM FACT_SERVICE_QUALITY) AS "# Complaints Covered",
    (SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED) AS "# Total Complaints",
    CONCAT(ROUND(((SELECT SUM("total_count") FROM FACT_SERVICE_QUALITY) / (SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED)) * 100, 1), '%') AS "% Covered",
    ((SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED) - (SELECT SUM("total_count") FROM FACT_SERVICE_QUALITY)) AS "Diff";
```

![fact1](/assets/fact1.png)


Here is the validation test used to assess the join accuracy and attempts to find missing data by counting the rows from fact service quality sanitized based on where condition to filter the specific column given the criteria “not in” the created select subquery coming from the source data tables. Result showed that there is a successful joining of the rows from source table and the main schema table.

```sql
-- VALIDATION OF EACH COLUMNS. 

SELECT
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED WHERE "Incident_Zip_Sanitized" NOT IN (SELECT "ZIP" FROM DIMENSION_ZIP)) AS "MISSING ZIP",
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED WHERE "Agency" NOT IN (SELECT "agency_name" FROM DIMENSION_AGENCY)) AS "MISSING AGENCY",
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED WHERE "Year Month" NOT IN (SELECT "year_month" FROM DIMENSION_YEAR_MONTH)) AS "MISSING YEARMONTH",
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED WHERE "Borough_Sanitized" NOT IN (SELECT UPPER("borough") FROM DIMENSION_ZIP)) AS "MISSING BOROUGH",
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED WHERE yearofweek("Created_Date") * 100 + week("Created_Date") NOT IN (SELECT "year_week" FROM DIMENSION_YEAR_WEEK)) AS "MISSING YEARWEEK",
(SELECT COUNT(*) FROM FACT_SERVICE_QUALITY_SANITIZED
    WHERE "Complaint_Type_Sanitized" NOT IN (SELECT UPPER(DIMENSION_COMPLAINT_TYPE."type") as "type" FROM DIMENSION_COMPLAINT_TYPE)) AS "MISSING COMPLAINT";
```
![fact2](/assets/fact2.png)
