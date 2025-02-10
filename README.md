# Case Study: Bike_share  
**Author**: Siow Hui Hui

**Date created**: 22 November 2024

## Company Background
Cyclistic is a bike-share program in Chicago, offering a fleet of over 5,800 bikes across 692 stations. Since its launch in 2016, Cyclistic has grown rapidly, providing a range of bike options, including traditional bikes, reclining bikes, hand tricycles, and cargo bikes, to meet the diverse needs of its riders. Approximately 30% of Cyclistic’s users commute to work, while others use the bikes for leisure.

Cyclistic offers three pricing plans: single-ride passes, full-day passes, and annual memberships. While annual memberships are the most profitable for the company, casual riders—those using single-ride or full-day passes—make up a large portion of the customer base. Cyclistic’s marketing strategy has traditionally focused on broad awareness and attracting casual riders. However, the company’s financial analysts have found that annual members are significantly more profitable than casual riders. As a result, Cyclistic is shifting its focus to increasing the number of annual memberships, believing that converting casual riders into members will be key to achieving long-term, sustainable growth.

## Business Task Statement
This analysis will identify key differences in how annual members and casual riders use Cyclistic bikes, focusing on trip frequency, duration, bike type preferences, and popular routes and stations. The insights will help the marketing team develop targeted strategies to convert casual riders into annual members, boosting revenue and improving customer retention.

## Key Stakeholders
The following stakeholders are critical to this analysis and will be involved in reviewing and acting upon the findings:

- **Lily Moreno (Director of Marketing)**: Lily is responsible for the development of Cyclistic’s marketing strategies. She has tasked the analytics team with providing insights into how casual riders and annual members use the bikes differently, with the goal of converting more casual riders to annual memberships.

- **Cyclistic Marketing Analytics Team**: The team, including myself, is responsible for collecting, analyzing, and interpreting the data. We will use this analysis to support marketing strategies aimed at increasing annual memberships.

- **Cyclistic Executive Team**: This group of senior executives will be reviewing the final recommendations to determine whether the proposed marketing strategies will be approved. Their primary focus is on maximizing the company’s profitability by increasing the number of annual memberships.

## Description of Data Sources Used
For this analysis, Cyclistic's publicly available historical trip data from July 2023 to June 2024 was used. The data was provided by Motivate International Inc. under this [license](https://divvybikes.com/data-license-agreement). It includes detailed information about each bike trip, such as ride timestamps, user type, bike type, and station information. This dataset allows for an in-depth analysis of bike usage patterns, including trip duration, bike preferences, station usage, and geographic trends.

### Key Data Columns:
The dataset is structured as a series of trip-level records, with each record representing a single bike trip. The columns across all 12 CSV files are consistent and include:

- **ride_id**: A unique identifier for each trip, ensuring each ride is distinct in the dataset.

- **rideable_type**: The type of bike used, allowing for comparison of bike preferences between annual members and casual riders.

- **started_at** and **ended_at**: Timestamps marking the start and end of each trip, useful for calculating trip duration and analyzing peak usage times.

- **start_station_name** and **end_station_name**: The names of the stations where trips began and ended, providing insights into popular stations and bike routes.

- **start_station_id** and **end_station_id**: The unique station identifiers, used during data cleaning to check for inconsistencies and ensure correct matching between station IDs and names (e.g., identifying and correcting typos).

- **member_casual**: Indicates whether the rider is a casual rider or annual member, which is crucial for segmenting the data.

- **start_lat, start_lng, end_lat**, and **end_lng**: Geographic coordinates of trip start and end locations, useful for spatial analysis and understanding rider flow between stations.

The files are structured to facilitate merging into a single dataset without losing important details.

### Data Quality:
- **Complete Data**: The following columns have no null values: ride_id, rideable_type, member_casual, started_at, ended_at, start_lat, start_lng. These columns were fully populated, ensuring the integrity of ride-specific and timestamp-based analyses.

- **Duplicate Ride IDs**: A total of 211 duplicate ride IDs were identified and removed to avoid skewing trip counts and other derived metrics.

- **Station Data Issues**:
  - **Missing Station Names and IDs**: A total of 933,003 rows have missing start station names and IDs, while 980,556 rows have missing end station names and IDs. These gaps are primarily due to dockless bike trips, where bikes are picked up and dropped off at locations not associated with fixed stations. Missing station names were replaced using the COALESCE() function, substituting with corresponding latitude and longitude values to enable location-based analysis.
  - **Duplicate or Inconsistent Station Data**: Some station names and IDs had inconsistencies or duplicates, which could cause issues in station-based analysis. These inconsistencies are being addressed in the data cleaning section to ensure consistency across the dataset.

- **Spatial Data Quality**:
  - **Missing End Latitude and Longitude**: Around 0.1% of trips had missing destination data, which were excluded from spatial analyses.
  - **Negative or Zero Distances**: Some trips had negative or zero distances calculated using the ST_DISTANCE() function between the start and end coordinates (start_lat, start_lng, end_lat, end_lng). These values were likely caused by data entry errors. Additionally, trips with a distance traveled of less than 10 meters were excluded, as they were considered too short to be valid for analysis.
  - **Geographic Filtering**: Rows with latitude and longitude coordinates outside the expected service area of Chicago and its surroundings were filtered out to ensure all data points were geographically relevant.

- **Invalid Trip Durations**: Negative or zero trip durations (where ended_at was earlier than or equal to started_at) were removed from the dataset.
  
### Bias in the Data
- **Lack of Demographic Information**: The dataset does not contain demographic information (e.g., age, gender, income), which limits analysis on how specific groups use Cyclistic bikes and could skew marketing strategies.
- **No Context on Rider Motivation**: Without data on why people use the bikes (e.g., commuting vs. leisure), it’s difficult to analyze how motivations might influence membership conversions.

### Data Scope Limitations
- **No Payment Data**: The dataset does not include information on the payment type or pricing plan (e.g., single rides vs. annual memberships), except for the member_casual indicator. As a result, we cannot investigate the impact of pricing models or specific promotions on the conversion of casual riders into annual members.
- **Geographic and Timeframe Limitations**: The dataset includes 12 months of trip data from Cyclistic’s service area in Chicago, from July 2023 to June 2024. As a result, the analysis is limited to trends and patterns within this specific geographic area and time frame. This may not fully capture seasonal, long-term, or external factors (such as unusual events or weather conditions) that could affect bike usage beyond this period.

### Security, Privacy, and Licensing
- **Licensing**: The dataset is publicly available under this [license](https://divvybikes.com/data-license-agreement), allowing unrestricted use for analysis and research purposes. No explicit usage restrictions were identified.
- **Privacy and Security**: The data is anonymized to protect PII, ensuring no individual rider can be identified. It is stored securely in Google Cloud Storage and is accessible to authorized users for analysis.

## Data Cleaning and Manipulation
### Tools and Techniques Used
The following tools were selected for their efficiency and suitability in handling large datasets and performing complex data manipulations:
- **SQL**: Given the size of the dataset (12 months of trip data across multiple CSV files), SQL was chosen for its ability to handle large datasets, perform complex queries, and execute data manipulation tasks like filtering, joining, and aggregating data efficiently.  
- **Tableau**: Tableau was used for visualizing the results once the data was cleaned and prepared. Its intuitive interface facilitated quick creation of visualizations, allowing for a deeper understanding of the patterns and trends in the data.

### Handling Multiple Large CSV Files 
The dataset consists of 12 months of trip data across separate CSV files, uploaded to a Google Cloud Storage bucket. These files were consolidated into a single SQL table to ensure consistency, as all files shared the same column structure.

### Data Cleaning Steps

#### Step 1: Review the Data
An initial review was performed to understand the dataset’s structure and assess the quality of the data. A sample query was run to inspect a few rows and identify any potential issues such as missing values or formatting inconsistencies.

```sql

SELECT  *
FROM `bike-share-case-study-430704.Bike_share.bike_share_12months`
LIMIT 10;

```
Some inconsistencies were found in the started_at and ended_at columns, along with null values in key columns such as start_station_name, start_station_id, end_station_name, and end_station_id. Below is a snapshot of the data:

|ride_id|started_at|ended_at|start_station_name|start_station_id|end_station_name|end_station_id|
|---|---|--- |---|---|---|---|
|1825FC51729D24C7|2024-06-15 13:22:02.900000 UTC|2024-06-15 13:24:41.489000 UTC|null|null|null|null|
|7F08386D8DAB72FB|2024-04-24 13:50:55 UTC|2024-04-24 13:53:49 UTC|null|null|null|null|

#### Step 2: Count the number of trips
A query was run to count the total number of trips in the dataset:

```sql
SELECT
  COUNT(*) AS total_trips
FROM `bike-share-case-study-430704.Bike_share.bike_share_12months`
```
|Row| total_trips|
|---|---|
|1|5734381|

#### Step 3: Standardize, Filter, and Remove Duplicates
Ensure consistency in the dataset by standardizing timestamp precision, normalizing textual data, rounding geographic coordinates, and removing any duplicate rows.

This step involves:

- **Standardizing Timestamps and Text Fields**:
  - Timestamps (`started_at` and `ended_at`) were truncated to the second to ensure uniform precision, removing any unnecessary fractional seconds.
  - All text fields (e.g., `rideable_type`, `station names`, and member status) were trimmed of extra spaces and converted to lowercase for consistency.
  - Latitude and longitude values were rounded to four decimal places (about 10 meters of precision) to standardize the dataset while maintaining accuracy.

- **Filtering Invalid Latitude and Longitude**:
  - The dataset was filtered to exclude any rows where the latitude and longitude of either the start or end points fell outside the defined geographic boundaries of central Chicago (latitude between 41.6 and 42.1, longitude between -88 and -87.5).
  - This ensured that only trips within Chicago were considered for analysis, excluding any trips that crossed into neighboring suburbs or had invalid geographic data.
 
- **Removing Duplicates**:
The `DISTINCT` keyword is applied to ensure that any exact duplicate rows (where all column values are identical) are removed from the dataset.

***Throughout the data transformation process, Common Table Expressions (CTEs) were used extensively. CTEs allowed for a flexible and organized approach to data cleaning and manipulation, without altering the original dataset.*** 
```sql
WITH formatted_data AS (
SELECT DISTINCT       -- remove duplicate
    TRIM(ride_id) AS ride_id,                      -- remove extra space
    TRIM(LOWER(rideable_type)) AS rideable_type,   -- remove extra space and standardize case
    TIMESTAMP_TRUNC(started_at, SECOND) AS cleaned_started_at,  -- Truncate timestamps to second precision
    TIMESTAMP_TRUNC(ended_at, SECOND) AS cleaned_ended_at,      -- Truncate timestamps to second precision
    TRIM(LOWER(start_station_name)) AS start_station_name,  -- remove extra space and standardize case
    TRIM(LOWER(start_station_id)) AS start_station_id,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_name)) AS end_station_name,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_id)) AS end_station_id,          -- remove extra space and standardize case
    TRIM(LOWER(member_casual)) AS member_casual,            -- remove extra space and standardize case
    ROUND(start_lat,4) AS start_lat,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(start_lng,4) AS start_lng,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lat,4) AS end_lat,         -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lng,4) AS end_lng         -- rounding to 4 decimal places (precision of about 10 meters)
FROM 
    `bike-share-case-study-430704.Bike_share.bike_share_12months`
)
SELECT  *
FROM formatted_data 
WHERE
    start_lat BETWEEN 41.6 AND 42.1  -- Chicago-only latitude range
    AND start_lng BETWEEN -88 AND -87.5  -- Chicago-only longitude range
    AND end_lat BETWEEN 41.6 AND 42.1    -- Chicago-only latitude range
    AND end_lng BETWEEN -88 AND -87.5  -- Chicago-only longitude range
    AND start_lat IS NOT NULL   -- exclude null value
    AND start_lng IS NOT NULL  
    AND end_lat IS NOT NULL
    AND end_lng IS NOT NULL
```

By standardizing the dataset, duplicates are removed, and inconsistencies in timestamps, text fields, and geographic coordinates are addressed. This ensures that any analysis, whether it be trip durations or station usage, is based on clean, uniform data

#### Step 4 : Investigate data that have been filtered out 
Investigates any rows that are outside this region, to understand the nature of the data that is being excluded. 

```sql
WITH formatted_data AS (
-- Refer to step 3
)
SELECT  *
FROM formatted_data 
WHERE
    (
        (start_lat < 41.6 OR start_lat > 42.1)  -- Latitude outside Chicago range
        OR (start_lng < -88 OR start_lng > -87.5)  -- Longitude outside Chicago range
        OR (end_lat < 41.6 OR end_lat > 42.1)    -- End latitude outside Chicago range
        OR (end_lng < -88 OR end_lng > -87.5)  -- End longitude outside Chicago range
    )
    AND (start_lat IS NOT NULL AND start_lng IS NOT NULL)  -- Ensure start coordinates are not null
    AND (end_lat IS NOT NULL AND end_lng IS NOT NULL);  -- Ensure end coordinates are not null
```
There are **27 rows** of data fell outside this range:
- 26 rows are slightly outside the Chicago boundaries, likely representing trips that crossed into neighboring suburbs. These trips are valid but don't align with the analysis focus on central Chicago and should be excluded.
  
- The 1 row with 0,0 coordinates and station information ('Stony Island Ave & 63rd St') represents an outlier due to incorrect geographic data, and no other matching records exist for the station. This row should also be excluded, as it does not contribute meaningful or reliable data.
  
These 27 rows represent a small fraction of the total dataset (less than 0.0005% of the 5,734,381 trips), excluding them helps ensure the analysis is focused on high-quality, relevant data that accurately reflects trips within central Chicago.

#### Step 5: Calculate the percentage of null value for lattitude and longtitude
To understand the extent of missing geographic data, a query was run to calculate the percentage of rows with NULL values for the start_lat, start_lng, end_lat, or end_lng fields.
```sql
WITH formatted_data AS (
    -- Refer to step 3
)

SELECT 
  ROUND(
    (COUNT(
      CASE WHEN start_lat IS NULL
                OR start_lng IS NULL
                OR end_lat IS NULL
                OR end_lng IS NULL
                THEN 1 
                END
            ) / COUNT(*) * 100)
  , 2) AS percentage
FROM
    formatted_data
```

|Row|percentage|
|---|---|
|1|0.14|

This shows that only 0.14% of the dataset has missing geographic data, which is a small percentage. The rest of the dataset is complete with respect to geographic coordinates, ensuring the analysis is based on robust data.

#### Step 6: Check for Other Null Values in the Dataset
##### 1. Count of Null Values in Essential Columns

```sql
SELECT
  COUNT(CASE WHEN ride_id IS NULL THEN 1 END) AS null_ride_id,
  COUNT(CASE WHEN rideable_type IS NULL THEN 1 END) AS null_rideable_type,
  COUNT(CASE WHEN started_at IS NULL THEN 1 END) AS null_started_at,
  COUNT(CASE WHEN ended_at IS NULL THEN 1 END) AS null_ended_at,
  COUNT(CASE WHEN start_station_name IS NULL THEN 1 END) AS null_start_station_name,
  COUNT(CASE WHEN start_station_id IS NULL THEN 1 END) AS null_start_station_id,
  COUNT(CASE WHEN end_station_name IS NULL THEN 1 END) AS null_end_station_name,
  COUNT(CASE WHEN end_station_id IS NULL THEN 1 END) AS null_end_station_id,
  COUNT(CASE WHEN member_casual IS NULL THEN 1 END) AS null_member_casual
FROM `bike-share-case-study-430704.Bike_share.bike_share_12months`
```

|null_ride_id|null_rideable_type|null_started_at|null_ended_at|null_start_station_name|null_start_station_id|	null_end_station_name|null_end_station_id|null_member_casual|
|---|---|---|---|---|---|---|---|---|
|0|0|0|0|933003|933003|980556|980556|0|

- **No Missing Data for Essential Columns**:
`ride_id`, `rideable_type`, `started_at`, `ended_at`, and `member_casual` have no null values, which is great for ensuring data consistency in terms of ride identification, timestamps, and user status.
- **Missing Station Data**:
  - `start_station_name` and `start_station_id` have 933,003 missing values.
  - `end_station_name` and `end_station_id` have 980,556 missing values.
  - indicating that some trips (likely dockless) do not have station-related information.

##### 2. Calculate the Percentage of Null Values for Station Information
```sql
SELECT
  ROUND((COUNT(CASE WHEN start_station_name IS NULL THEN 1 END) / COUNT(*)) * 100, 2) AS null_percentage_start_name,
  ROUND((COUNT(CASE WHEN start_station_id IS NULL THEN 1 END) / COUNT(*)) * 100, 2) AS null_percentage_start_id,
  ROUND((COUNT(CASE WHEN end_station_name IS NULL THEN 1 END) / COUNT(*)) * 100, 2) AS null_percentage_end_name,
  ROUND((COUNT(CASE WHEN end_station_id IS NULL THEN 1 END) / COUNT(*)) * 100, 2) AS null_percentage_end_id
FROM `bike-share-case-study-430704.Bike_share.bike_share_12months`
```
|null_percentage_start_name| null_percentage_start_id|null_percentage_end_name|null_percentage_end_id|
|---|---|---|---|
|16.27|16.27|17.1|17.1|

- 16.27% of the dataset has missing start_station_name and start_station_id.
- 17.1% of the dataset has missing end_station_name and end_station_id.
- These percentages suggest that a significant portion of trips is dockless, which is common in modern bike-sharing systems.
- While missing station data is expected for dockless bikes, it's important to note that the absence of station information could potentially impact analyses that rely on station-based metrics, such as the most used stations, station-to-station trip patterns, or station-related demand analysis.

##### 3. Checking for Mismatched Station Name and ID Pairs
```sql
SELECT
  -- Mismatched cases for start station
  COUNT(CASE WHEN start_station_name IS NOT NULL AND start_station_id IS NULL THEN 1 END) AS num_mismatch_start_1,
  COUNT(CASE WHEN start_station_id IS NOT NULL AND start_station_name IS NULL THEN 1 END) AS num_mismatch_start_2,
  
  -- Mismatched cases for end station
  COUNT(CASE WHEN end_station_name IS NOT NULL AND end_station_id IS NULL THEN 1 END) AS num_mismatch_end_1,
  COUNT(CASE WHEN end_station_id IS NOT NULL AND end_station_name IS NULL THEN 1 END) AS num_mismatch_end_2
FROM `bike-share-case-study-430704.Bike_share.bike_share_12months`
```
|num_mismatch_start_1|num_mismatch_start_2|num_mismatch_end_1|num_mismatch_end_2|
|---|---|---|---|
|0|0|0|0|

- There are **no mismatched cases** where the station name is present but the ID is missing, or the reverse. This suggests that the missing data for station names and IDs is likely due to the fact that some rides are dockless and thus do not require station information.

#### Step 7: Checking for Multiple Names for One Station ID
This step investigates cases where a single 'start_station_id' or 'end_station_id' is linked to multiple 'start_station_name' or 'end_station_name' values. The goal is to identify potential inconsistencies in station data caused by factors such as temporary relocations, naming variations, shared IDs, or historical name changes.

##### 7.1. Identifying Station IDs with Multiple Names
The start and end station IDs were combined into a single 'station_id' column, and the corresponding names into a 'station_name' column to identifies station IDs with more than one distinct name.
```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
SELECT  *
FROM  formatted_data
WHERE
    start_lat BETWEEN 41.6 AND 42.1  -- Chicago-only latitude range
    AND start_lng BETWEEN -88 AND -87.5  -- Chicago-only longitude range
    AND end_lat BETWEEN 41.6 AND 42.1    -- Chicago-only latitude range
    AND end_lng BETWEEN -88 AND -87.5  -- Chicago-only longitude range 
    AND start_lat IS NOT NULL
    AND start_lng IS NOT NULL
    AND end_lat IS NOT NULL
    AND end_lng IS NOT NULL
)

,station_data AS (
SELECT DISTINCT
  start_station_id AS station_id, 
  start_station_name AS station_name
FROM filtered_data
WHERE start_station_name IS NOT NULL AND start_station_id IS NOT NULL
UNION ALL
SELECT DISTINCT 
  end_station_id AS station_id, 
  end_station_name AS station_name
FROM filtered_data 
WHERE end_station_name IS NOT NULL AND end_station_id IS NOT NULL 
)

SELECT 
  station_id,
  ARRAY_AGG(DISTINCT station_name) AS station_names,
  COUNT(DISTINCT station_name) AS name_count
FROM station_data
GROUP BY station_id
HAVING name_count > 1
ORDER BY name_count DESC 
```
- **90 rows** were found where a station id has more than one station name.

##### 7.2. Analyzing Station Name Variations
To better understand these discrepancies, we examined the distinct station names and their corresponding coordinates for each identified station_id.
```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7
)

SELECT DISTINCT
  station_id,
  station_name,
  latitude,
  longtitude
FROM (
  SELECT DISTINCT start_station_id AS station_id, start_station_name AS station_name, start_lat AS latitude, start_lng AS longtitude
  FROM filtered_data
  UNION ALL
  SELECT DISTINCT end_station_id AS station_id, end_station_name AS station_name, end_lat AS latitude, end_lng AS longtitude
  FROM filtered_data
) AS station_info
WHERE station_id IN (
  SELECT station_id
  FROM station_data
  GROUP BY station_id
  HAVING COUNT(DISTINCT station_name) > 1
)
ORDER BY station_id
```
**Observations**

- **Temporary Relocations**:
  - Example: start_station_id = '13290'
  - Names: 'noble st & milwaukee ave' and 'noble st & milwaukee ave (temp)'
  - Coordinates: Slightly different but close, suggesting a temporary relocation.
  - Resolution: Could be standardized to one name.

- **Minor Naming Variations**:
  - Example: start_station_id = '21322'
  - Names: 'grace st & cicero ave' and 'grace & cicero'
  - Coordinates: Identical, indicating a formatting inconsistency.
  - Resolution: Could be standardized to one format.
    
- **Distinct Locations and Shared IDs**:
  - Example: start_station_id = '523'
  - Names: 'eastlake ter & howard st' and 'public rack - pulaski rd & roosevelt rd'
  - Coordinates: Far apart geographically.
  - Observation: Likely represents a shared ID for distinct station types (public rack vs. standard station).
    
- **Name Changes Over Time**:
  - Example: start_station_id = 'ta1305000030'
  - Names: 'clark st & randolph st' and 'wells st & randolph st'
  - Coordinates: Identical.
  - Observation: Likely a historical name change. Can be standardized to the most recent name.

##### 7.3. Time-Based Investigation
To validate name changes over time, the query below was used to identify the first and last occurrences of each station name:
```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)
  
SELECT
  station_id,
  station_name,
  MIN(cleaned_started_at) AS first_occurrence,
  MAX(cleaned_started_at) AS last_occurrence,
FROM (
  SELECT DISTINCT start_station_id AS station_id, start_station_name AS station_name, cleaned_started_at
  FROM filtered_data
  UNION ALL
  SELECT DISTINCT end_station_id AS station_id, end_station_name AS station_name, cleaned_started_at
  FROM filtered_data
) AS station_time
WHERE
  station_id IN (
    SELECT station_id
    FROM station_data
    GROUP BY station_id
    HAVING COUNT(DISTINCT station_name) > 1
  ) 
GROUP BY 
  station_id, station_name
ORDER BY
  station_id
```
**Example Result**:

|Row|start_station_id|start_station_name|first_occurrence|last_occurrence|
|---|---|---|---|---|
|160|ta1305000030|clark st & randolph st|2023-07-01 00:48:15 UTC|2024-01-24 08:28:31 UTC|
|161|ta1305000030|wells st & randolph st|2024-01-25 17:21:15 UTC|2024-06-30 21:12:05 UTC|

In this case:

- "clark st & randolph st" was the station name from July 2023 to January 24, 2024.
- "wells st & randolph st" became the station name starting January 25, 2024, with no overlap in usage.

This shows a transition in station naming, likely reflecting updates or corrections. Standardizing to the newer name enhances data consistency.


##### 7.4. Address Shared IDs
To resolve shared IDs, station IDs were updated based on specific naming patterns, such as appending a suffix for station names starting with "public rack -".

```sql
WITH formatted_data AS (
  -- refer to step 3
) 

,filtered_data AS (
  -- refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
SELECT DISTINCT
    station_id,
    station_name,
FROM station_data
WHERE station_id IN (
                      SELECT station_id
                      FROM station_data
                      GROUP BY station_id
                      HAVING COUNT(DISTINCT station_name) > 1
                      )
)

SELECT DISTINCT
  CASE -- assign new id to station name like 'public rack -%'
    WHEN station_name IN (
        SELECT station_name
        FROM sn2
        WHERE sn2.station_name LIKE 'public rack -%'  -- Check if any name starts with 'public rack -'
    ) THEN CONCAT(station_id, 'a')  -- Append 'a' if the condition is met
    ELSE station_id  -- Keep the original station_id otherwise
  END AS station_id,
  
  station_name  -- Selecting the original station_name for reference
FROM station_data 
```
- to verify:
```sql
WITH formatted_data AS (
  -- refer to step 3
) 

,filtered_data AS (
  -- refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
SELECT DISTINCT
    station_id,
    station_name,
FROM station_data
WHERE station_id IN (
                      SELECT station_id
                      FROM station_data
                      GROUP BY station_id
                      HAVING COUNT(DISTINCT station_name) > 1
                      )
)

,new_station_id AS (
SELECT DISTINCT
  CASE -- assign new id to station name like 'public rack -%'
    WHEN station_name IN (
        SELECT station_name
        FROM sn2
        WHERE sn2.station_name LIKE 'public rack -%'  -- Check if any name starts with 'public rack -'
    ) THEN CONCAT(station_id, 'a')  -- Append 'a' if the condition is met
    ELSE station_id  -- Keep the original station_id otherwise
  END AS station_id,
  
  station_name  -- Selecting the original station_name for reference
FROM station_data
)
SELECT 
  station_id,
  ARRAY_AGG(DISTINCT station_name) AS station_names,
  COUNT(DISTINCT station_name) AS name_count
FROM new_station_id
GROUP BY station_id
HAVING name_count > 1
ORDER BY name_count DESC 
```
After applying this update, the number of station IDs with multiple names was reduced to 15 rows, effectively filtering out shared ID cases.

  
##### 7.5. Addressing Inconsistencies in Naming Variations
For inconsistencies caused by temporary relocations, minor naming variations, or historical name changes, standardized station names were assigned to each station ID.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

SELECT DISTINCT
    station_id,

    -- ### Assign New Station Names ###
    CASE
        -- Temporary relocations 
        WHEN station_id = '13290' THEN 'noble st & milwaukee ave'
        WHEN station_id = '15541' OR station_id = '15541.1.1' THEN 'buckingham fountain'
        WHEN station_id = 'hubbard bike-checking (lbs-wh-test)' THEN 'base - 2132 w hubbard' 
        ---The names 'scooters - 2132 w hubbard st' and 'scooters classic - 2132 w hubbard st' were recorded only briefly, each existing for a single day with very few data entries. These instances suggest a temporary relocation or test setup. For consistency, all occurrences were standardized to 'base - 2132 w hubbard'.---
        -- Minor Naming Variations
        WHEN station_id = '21322' THEN 'grace st & cicero ave'
        WHEN station_id = '21366' THEN 'spaulding ave & 16th st'
        WHEN station_id = '21371' THEN 'kildare ave & chicago ave'
        WHEN station_id = '21393' THEN 'oketo ave & addison st'
        WHEN station_id = '23187' THEN 'lockwood ave & grand ave'
        WHEN station_id = '23215' THEN 'lexington st & california ave'
        -- Name changes Over Time
        WHEN station_id = '24156' THEN 'granville ave & pulaski rd'
        WHEN station_id = 'ka1503000074' THEN 'museum of science and industry'
        WHEN station_id = 'ta1305000030' THEN 'wells st & randolph st'
        WHEN station_id = 'ta1309000042' THEN 'lincoln ave & melrose st'
        WHEN station_id = '647' THEN 'racine ave & 57th st'
        ELSE station_name
    END AS station_name
FROM new_station_id
ORDER BY station_id
```

- to verify :
```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
SELECT DISTINCT
    station_id,

    -- ### Assign New Station Names ###
    CASE
        -- Temporary relocations 
        WHEN station_id = '13290' THEN 'noble st & milwaukee ave'
        WHEN station_id = '15541' OR station_id = '15541.1.1' THEN 'buckingham fountain'
        WHEN station_id = 'hubbard bike-checking (lbs-wh-test)' THEN 'base - 2132 w hubbard' 
        ---The names 'scooters - 2132 w hubbard st' and 'scooters classic - 2132 w hubbard st' were recorded only briefly, each existing for a single day with very few data entries. These instances suggest a temporary relocation or test setup. For consistency, all occurrences were standardized to 'base - 2132 w hubbard'.---
        -- Minor Naming Variations
        WHEN station_id = '21322' THEN 'grace st & cicero ave'
        WHEN station_id = '21366' THEN 'spaulding ave & 16th st'
        WHEN station_id = '21371' THEN 'kildare ave & chicago ave'
        WHEN station_id = '21393' THEN 'oketo ave & addison st'
        WHEN station_id = '23187' THEN 'lockwood ave & grand ave'
        WHEN station_id = '23215' THEN 'lexington st & california ave'
        -- Name changes Over Time
        WHEN station_id = '24156' THEN 'granville ave & pulaski rd'
        WHEN station_id = 'ka1503000074' THEN 'museum of science and industry'
        WHEN station_id = 'ta1305000030' THEN 'wells st & randolph st'
        WHEN station_id = 'ta1309000042' THEN 'lincoln ave & melrose st'
        WHEN station_id = '647' THEN 'racine ave & 57th st'
        ELSE station_name
    END AS station_name
FROM new_station_id
ORDER BY station_id
)

SELECT 
  station_id,
  ARRAY_AGG(DISTINCT station_name) AS station_names,
  COUNT(DISTINCT station_name) AS name_count
FROM standardized_name
GROUP BY station_id
HAVING name_count > 1
ORDER BY name_count DESC
```
- there is no data to display.
  
Verification confirmed that all station IDs now have unique and consistent station names, with no remaining cases of multiple names for a single ID.

#### Step 8: Identifying and Resolving Station Names with Multiple IDs
This step identifies station names associated with multiple station IDs and standardizes them for consistency in analysis. The process involves identifying patterns, resolving duplicates, and validating results.

##### 8.1. Identifying Station Names with Multiple IDs
To identify station names with multiple station IDs, the query aggregates station IDs for each station name and counts distinct IDs.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
  -- Refer to step 7
)

SELECT 
  station_name,
  ARRAY_AGG(DISTINCT station_id) AS station_id,
  COUNT(DISTINCT station_id) AS id_count
FROM standardized_name
GROUP BY station_name
HAVING id_count > 1
ORDER BY id_count DESC
```
**Findings:**
- **49** station names are associated with more than one station ID.
- **47** of these station names follow a pattern where an additional '21-' prefix is added to the station ID.
- One station has an ID that is exactly the same as its station name.
- Another station name has two IDs: '15541' and '15541.1.1', indicating a potential versioning issue or an inconsistency.
  
**Example Results**:
|Row|station_name|station_id|id_count|
|---|---|---|---|
|1|campbell ave & 51st st|383|2|
| | |21383| |
|4|california ave & 36th st|21338|2|
| | |338| |
|9|buckingham fountain|15541.1.1|2|
| | |15541| |
|48|cicero ave & wellington ave|24174|2|
| | |cicero ave & wellington ave| |

##### 8.2. Resolving Station Name Equal to Station ID and Variations
For instances where a station name matches a station ID, or where variations in IDs exist for the same name:

- Cicero ave & wellington ave: Assigned a single station ID ('24174').
- Buckingham fountain: Resolved by using the base station ID ('15541').
  
```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
  -- Refer to step 7
)

SELECT DISTINCT
    -- ### Assign New Station IDs ###
    CASE
        -- handling entry error station id equal to station name and station id variations--
        WHEN station_name = 'cicero ave & wellington ave' THEN '24174'
        WHEN station_name = 'buckingham fountain' THEN '15541'
        ELSE station_id
    END AS station_id,
    station_name
FROM standardized_name
ORDER BY station_id
```

##### 8.3. Validating Time-Based Changes
To confirm whether duplicate station IDs represent changes over time, the query examines the first and last occurrences for each combination of station ID and station name.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
  -- Refer to step 7
)

,corrected_id AS (
SELECT DISTINCT
    -- ### Assign New Station IDs ###
    CASE
        -- handling entry error station id equal to station name and station id variations--
        WHEN station_name = 'cicero ave & wellington ave' THEN '24174'
        WHEN station_name = 'buckingham fountain' THEN '15541'
        ELSE station_id
    END AS station_id,
    station_name
FROM standardized_name
ORDER BY station_id
)

SELECT
  station_id,
  station_name,
  MIN(cleaned_started_at) AS first_occurrence,
  MAX(cleaned_started_at) AS last_occurrence,
FROM (
  SELECT DISTINCT start_station_id AS station_id, start_station_name AS station_name, cleaned_started_at
  FROM filtered_data
  UNION ALL
  SELECT DISTINCT end_station_id AS station_id, end_station_name AS station_name, cleaned_started_at
  FROM filtered_data
) AS station_time
WHERE
  station_name IN (
    SELECT station_name
    FROM corrected_id
    GROUP BY station_name
    HAVING COUNT(DISTINCT station_id) > 1
  ) 
GROUP BY 
  station_id, station_name
ORDER BY
  station_name
```
**Results**:
- IDs with a '21-' prefix are confirmed as newer entries without overlapping timeframes.
  
**Example Results**:

|Row|station_id|station_name|first_occurrence|last_occurrence|
|---|---|---|---|---|
|1|21345|artesian ave & 55th st|2024-04-27 16:42:36 UTC|2024-06-30 20:38:58 UTC|
|2|345|artesian ave & 55th st|2023-07-03 14:47:22 UTC|2024-04-14 11:22:36 UTC|
|3|338|california ave & 36th st|2023-07-01 20:56:28 UTC|2024-04-09 19:54:34 UTC|
|4|21338|california ave & 36th st|2024-04-17 11:18:50 UTC|2024-06-25 20:17:40 UTC|

##### 8.4. Assigning Standardized Station IDs
The longest station ID, preferring those starting with '21-', is assigned to each station name.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
  -- Refer to step 7
)

,corrected_id AS (
  -- Refer to above
)

SELECT DISTINCT
  station_id,
  station_name 
FROM (
        SELECT 
            station_name,
            station_id,
            LENGTH(station_id) AS id_length,
            ROW_NUMBER() OVER (
              PARTITION BY station_name 
              ORDER BY LENGTH(station_id) DESC,
                        station_id NOT LIKE '%.0'DESC,
                        station_id LIKE '21%' DESC) AS rn
        FROM corrected_id
    ) AS subquery
WHERE rn = 1 -- Pick the longest station_id and station id that start with '21-' for each station_name
```

##### 8.5. Verifying and Creating a Mapping Table
The verification ensures no station name is associated with multiple IDs.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,station_data AS (
  -- Refer to step 7  
)

,sn2 AS (
  -- Refer to step 7  
)

,new_station_id AS (
  -- Refer to step 7  
)

,standardized_name AS (
  -- Refer to step 7
)

,corrected_id AS (
  -- Refer to above
)

,mapping_station AS (
SELECT DISTINCT
  station_id,
  station_name 
FROM (
        SELECT 
            station_name,
            station_id,
            LENGTH(station_id) AS id_length,
            ROW_NUMBER() OVER (
              PARTITION BY station_name 
              ORDER BY LENGTH(station_id) DESC,
                        station_id NOT LIKE '%.0'DESC,
                        station_id LIKE '21%' DESC) AS rn
        FROM corrected_id
    ) AS subquery
WHERE rn = 1 -- Pick the longest station_id for each station_name
)

SELECT 
  station_name,
  ARRAY_AGG(DISTINCT station_id) AS station_id,
  COUNT(DISTINCT station_id) AS id_count
FROM mapping_station
GROUP BY station_name
HAVING id_count > 1
ORDER BY id_count DESC
```
**Result**:
- No station names with multiple IDs remain.

The final mapping table is exported to bike-share-case-study-430704.Bike_share.mapping_station.

#### Step 9: Applying the Standardized Station Names and IDs
This step integrates the standardized station names and IDs across the dataset by joining it with the mapping_station table. This ensures consistency in station information throughout the data.

##### 9.1 Query to Apply Standardized Names and IDs
The mapping_station table is used to update both start_station and end_station names and IDs. This is achieved through two levels of joins, prioritizing station IDs and names for mapping.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

-- Apply standardized names for station IDs --
,cleaned_name AS (
SELECT DISTINCT
    ride_id, 
    rideable_type,
    cleaned_started_at,
    cleaned_ended_at,
    f.start_station_id,
    COALESCE(m_start.station_name, f.start_station_name) AS start_station_name,
    f.end_station_id,
    COALESCE(m_end.station_name, f.end_station_name) AS end_station_name,
    member_casual,
    start_lat,
    start_lng,
    end_lat,
    end_lng
FROM 
    filtered_data AS f
LEFT JOIN `bike-share-case-study-430704.Bike_share.mapping_station` AS m_start
    ON f.start_station_id = m_start.station_id
LEFT JOIN `bike-share-case-study-430704.Bike_share.mapping_station` AS m_end
    ON f.end_station_id = m_end.station_id
)

-- Apply standardized IDs for station names --
,cleaned_name2 AS (
SELECT DISTINCT
    ride_id, 
    rideable_type,
    cleaned_started_at,
    cleaned_ended_at,
    COALESCE(m_start.station_id, n.start_station_id) AS start_station_id,
    n.start_station_name,
    COALESCE(m_end.station_id, n.end_station_id) AS end_station_id,
    n.end_station_name,
    member_casual,
    start_lat,
    start_lng,
    end_lat,
    end_lng
FROM 
    cleaned_name AS n
LEFT JOIN `bike-share-case-study-430704.Bike_share.mapping_station` AS m_start
    ON n.start_station_name = m_start.station_name 
LEFT JOIN `bike-share-case-study-430704.Bike_share.mapping_station` AS m_end
    ON n.end_station_name = m_end.station_name
)

```
##### 9.2. Validation Queries
To confirm that all station names and IDs are standardized, validation queries check for multiple IDs per name and multiple names per ID.

1. Validate Start Station Name to IDs Mapping

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 3
) 

,cleaned_name AS (
  -- Refer to above
)

,cleaned_name2 AS (
  -- Refer to above
)
  
SELECT 
  start_station_name,
  ARRAY_AGG(DISTINCT start_station_id) AS station_ids,
  COUNT(DISTINCT start_station_id) AS id_count
FROM cleaned_name2
GROUP BY start_station_name
HAVING id_count > 1
ORDER BY id_count DESC
```
Result: No data to display.

2. Validate End Station Name to IDs Mapping
```sql
SELECT 
  end_station_name,
  ARRAY_AGG(DISTINCT end_station_id) AS station_ids,
  COUNT(DISTINCT end_station_id) AS id_count
FROM cleaned_name2
GROUP BY end_station_name
HAVING id_count > 1
ORDER BY id_count DESC
```
Result: No data to display.

3. Validate Start Station ID to Names Mapping
```sql
SELECT 
  start_station_id,
  ARRAY_AGG(DISTINCT start_station_name) AS station_names,
  COUNT(DISTINCT start_station_name) AS name_count
FROM cleaned_name2
GROUP BY start_station_id
HAVING name_count > 1
ORDER BY name_count DESC
```
Result: No data to display.

4. Validate End Station ID to Names Mapping
```sql
SELECT 
  end_station_id,
  ARRAY_AGG(DISTINCT end_station_name) AS station_names,
  COUNT(DISTINCT end_station_name) AS name_count
FROM cleaned_name2
GROUP BY end_station_id
HAVING name_count > 1
ORDER BY name_count DESC
```
Result: No data to display.

**Outcome**
The validation confirms that:

- Each station name is associated with a single station ID.
- Each station ID is associated with a single station name.
  
The standardized data is now ready for further analysis, ensuring consistency and accuracy in station information.

#### Step 10: Handling Null value 
- This step addresses missing station information by filling start_station_name, start_station_id, end_station_name, and end_station_id with their respective latitude and longitude coordinates where necessary.
- This ensures the dataset remains complete with geographic information, even when station data is missing (e.g., dockless bike trips).

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 7
) 

,cleaned_name AS (
  -- Refer to step 9
)

,cleaned_name2 AS (
  -- Refer to step 9
)

SELECT
  ride_id,
  rideable_type,
  cleaned_started_at,
  cleaned_ended_at,

  -- Use COALESCE to fill missing station names with lat/lng values
  COALESCE(start_station_name, CONCAT('Lat: ', CAST(start_lat AS STRING), ', Long: ', CAST(start_lng AS STRING))) AS start_station_name,  
  COALESCE(start_station_id, CONCAT('Lat: ', CAST(start_lat AS STRING), ', Long: ', CAST(start_lng AS STRING))) AS start_station_id,
  COALESCE(end_station_name, CONCAT('Lat: ', CAST(end_lat AS STRING), ', Long: ', CAST(end_lng AS STRING))) AS end_station_name,  
  COALESCE(end_station_id, CONCAT('Lat: ', CAST(end_lat AS STRING), ', Long: ', CAST(end_lng AS STRING))) AS end_station_id,
  
  member_casual,
  start_lat,
  start_lng,
  end_lat,
  end_lng
  
FROM 
  cleaned_name2 
```
- The **COALESCE()** function is used to populate missing station names and IDs with the corresponding latitude and longitude values. The **CONCAT()** function combine these latitude and longitude values into a string, indicating the approximate location of the trip's start or end point.
  
#### Step 11: Checking for misspelling 
Consistency checks were performed on the rideable_type and member_casual columns to identify any misspellings in categorical values.

##### 11.1. Query to Check rideable_type

```sql
SELECT DISTINCT
  rideable_type
FROM
  `bike-share-case-study-430704.Bike_share.bike_share_12months`
```
|Row|rideable_type|
|---|---|
|1|electric_bike|
|2|classic_bike|
|3|docked_bike|

No misspellings were found in the rideable_type column.

##### 11.2. Query to Check member_casual
```sql
SELECT DISTINCT
  member_casual
FROM
  `bike-share-case-study-430704.Bike_share.bike_share_12months`
```
|Row|member_casual|
|---|---|
|1|member|
|2|casual|

No misspellings were found in the member_casual column.

#### Step 12: Calculate trip length and trip distance
This step calculates the trip duration (in minutes) and the distance (in meters) for each trip, ensuring accurate measurement of ride characteristics.

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 7
) 

,cleaned_name AS (
  -- Refer to step 9
)

,cleaned_name2 AS (
  -- Refer to step 9
)

,filled_name AS (
SELECT
  ride_id,
  rideable_type,
  cleaned_started_at,
  cleaned_ended_at,

  -- Use COALESCE to fill missing station names with lat/lng values
  COALESCE(start_station_name, CONCAT('Lat: ', CAST(start_lat AS STRING), ', Long: ', CAST(start_lng AS STRING))) AS start_station_name,  
  COALESCE(start_station_id, CONCAT('Lat: ', CAST(start_lat AS STRING), ', Long: ', CAST(start_lng AS STRING))) AS start_station_id,
  COALESCE(end_station_name, CONCAT('Lat: ', CAST(end_lat AS STRING), ', Long: ', CAST(end_lng AS STRING))) AS end_station_name,  
  COALESCE(end_station_id, CONCAT('Lat: ', CAST(end_lat AS STRING), ', Long: ', CAST(end_lng AS STRING))) AS end_station_id,
  
  member_casual,
  start_lat,
  start_lng,
  end_lat,
  end_lng
  
FROM 
  cleaned_name2
)

SELECT 
    *,
    ROUND(ST_DISTANCE(
        ST_GEOGPOINT(cleaned_start_lng, cleaned_start_lat),  -- Start point (longitude, latitude)
        ST_GEOGPOINT(cleaned_end_lng, cleaned_end_lat)       -- End point (longitude, latitude)
    ),2) AS distance_meters,   -- The result is in meters
    TIMESTAMP_DIFF(cleaned_ended_at, cleaned_started_at, MINUTE) AS trip_duration
FROM 
    filled_name
```

##### 12.1. Filter Invalid Records
Rows are filtered to exclude trips where:

- The calculated distance is less than or equal to 10 meters (likely errors or very short trips).
- The trip duration is less than or equal to 0 (invalid or duplicate records).

```sql
WITH formatted_data AS (
  -- Refer to step 3
)

,filtered_data AS (
  -- Refer to step 7
) 

,cleaned_name AS (
  -- Refer to step 9
)

,cleaned_name2 AS (
  -- Refer to step 9
)

,filled_name AS (
  -- Refer to step 10
)

,cleaned AS (
SELECT 
    *,
    ROUND(ST_DISTANCE(
        ST_GEOGPOINT(start_lng, start_lat),  -- Start point (longitude, latitude)
        ST_GEOGPOINT(end_lng, end_lat)       -- End point (longitude, latitude)
    ),2) AS distance_meters,   -- The result is in meters
    TIMESTAMP_DIFF(cleaned_ended_at, cleaned_started_at, MINUTE) AS trip_duration
FROM 
    filled_name
)

SELECT DISTINCT
    *
FROM
    cleaned
WHERE 
    distance_meters > 10   -- Filtering out very short trips
    AND trip_duration > 0  -- Filtering out negative and zero value
ORDER BY
    trip_duration DESC
```

After processing, the final cleaned table was exported to the dataset bike-share-case-study-430704.Bike_share.cleaned_table.

- to verify there is no duplicate for ride_id:
```sql
SELECT
    ride_id,
    COUNT(*) AS count
FROM
    `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY
    ride_id
HAVING
    COUNT(*) > 1
```
Result: No data to display.

## Analysis
With the dataset cleaned and prepared, the next step focuses on analyzing the data to derive meaningful insights. The goal of the analysis is to uncover trends and patterns related to bike usage, user behavior, and station performance. This includes examining trip durations, ride types, popular routes, and the impact of seasonality on bike usage.

### 1. Total Number Of Member vs. Casual Rider Trips
- Members account for **65.48%** of all trips, while casual riders account for 34.52%.
- Casual riders make up **34.52%** offer a strong opportunity for conversion.
![image](https://github.com/user-attachments/assets/ef7e3504-49e9-4bfe-8e93-6b03a784284c)

```sql
SELECT 
  member_casual,
  COUNT(*) AS ride_count,
  ROUND((COUNT(*) * 100.0) / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY member_casual
```
|Row |member_casual |ride_count |percentage|
|--- |--- |--- |--- |
|1 |casual |1,850,211|34.52|
|2 |member |3,508,986|65.48|


### 2. Trip Frequency and Patterns by User Type: Daily, Weekly, and Seasonal Trends

```sql
SELECT 
    member_casual,
    EXTRACT(HOUR FROM cleaned_started_at) AS hour_of_day,
    EXTRACT(DAYOFWEEK FROM cleaned_started_at) AS day_of_week,
    EXTRACT(MONTH FROM cleaned_started_at) AS month_of_year,
    ROUND(AVG(distance_meters),2) AS avg_distance_meters,
    ROUND(AVG(trip_duration),2) AS avg_trip_duration_minutes,
    COUNT(*) AS trips
FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY 
    member_casual, hour_of_day, day_of_week, month_of_year
ORDER BY 
    hour_of_day, avg_distance_meters DESC\
```
#### Monthly Trip Trends
- **Peak Usage Periods**:
  - The number of trips by members peaks in August, with approximately 437,000 trips recorded.
  - Casual usage peaks slightly earlier, in July, with about 297,000 trips.
The peak summer months (June to August) account for the highest share of trips for both user groups.

- **Seasonal Variation**:
  - Both members and casual users show significant seasonal variation:
  - Members display a relatively steady rise and fall in trip numbers, indicating regular, year-round use.
  - Casual riders exhibit a more pronounced increase in trips during the warmer months, suggesting greater sensitivity to weather and seasonal conditions.
  - Winter months (December to February) have the lowest usage for both groups, but casual ridership drops more dramatically during this period.
 
- **Member vs. Casual Trends**:
  - Members consistently take more trips than casual riders across all months of the year.
  - Both groups show a similar pattern of rising trip counts in spring, peaking in summer, and dropping in winter, indicating that both casual and member users are influenced by seasonal factors.
    
![image](https://github.com/user-attachments/assets/eeb85501-0d3e-4277-83bc-a401f7cefedd)

#### Weekly Trip Trends
- **Distinct Usage Patterns**:
  - Members dominate usage on weekdays, with consistent trips from Monday to Friday, peaking on Wednesday with around 565,000 trips. This trend indicates that members primarily use Cyclistic for weekday commuting, likely for work or school.
  - Casual users have a different pattern, with fewer trips during the weekdays and a significant spike in trips on Saturday (374,000 trips) and Sunday. This suggests that casual users mainly use the service for leisure and recreational purposes over the weekend.
    
![image](https://github.com/user-attachments/assets/284b87c4-9549-48f5-965f-f9f37dd18446) 

#### Daily Trip Trends
- **Member Usage Patterns**:
Members exhibit distinct peaks in trip volume during typical commuting hours:
  - Morning peak: Around 8 AM.
  - Evening peak: Around 5 PM.
This pattern suggests that members primarily use the service for work-related trips during weekdays.

- **Casual User Usage Patterns**:
  - Casual users display a steady increase in trips throughout the morning, with a pronounced peak around midday (5 PM).
  - This suggests that casual users tend to use Cyclistic more for leisurely or occasional trips, often in the late afternoon and evening.

- **Comparison of Member vs. Casual User Trends**:
  - Peak usage times: Members dominate during peak commuting hours (8 AM and 5 PM), while casual users have their peak mainly around 5 PM.
  - Low-activity period: Both user types experience a significant drop in activity late at night, with the lowest volumes occurring between 12 AM and 5 AM.

![image](https://github.com/user-attachments/assets/6230e3f6-2a3c-4d85-a0ca-bc796c9fd677)


### 3. Preferred Bike Types by Rider Type
The following query explores bike preferences among members and casual riders:
```sql
SELECT 
  rideable_type AS bike_type,
  member_casual AS rider_type,
  COUNT(*) AS ride_count,
  ROUND((COUNT(*) * 100.0) / SUM(COUNT(*)) OVER (PARTITION BY member_casual), 2) AS ride_percentage
FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY rideable_type, member_casual
ORDER BY ride_count DESC, member_casual
```
|Row	|bike_type |rider_type |ride_count|ride_percentage|
|--- |---|---|---|---|
|1	|classic_bike |member |1,809,287|51.56|
|2	|electric_bike |member| 1,699,699|48.44|
|3	|electric_bike |casual |965,443|52.18|
|4	|classic_bike |casual |857,927|46.37|
|5	|docked_bike |casual |26,841|1.45|

![image](https://github.com/user-attachments/assets/e7ac1780-7ad3-4274-99ee-3c6c76f61e79)

- This breakdown shows that casual riders prefer electric bikes (52.18%), while members prefer classic bikes (51.56%) over electric bikes (48.44%) . The preference for docked bikes is minimal with no member riders uses it.

### 4. Top 10 Routes
```sql
WITH route_user_type AS (
  -- Top 10 popular routes and counts for member and casual user trips
  SELECT
    LEAST(start_station_name, end_station_name) AS start_station,
    GREATEST(start_station_name, end_station_name) AS end_station,
    COUNT(DISTINCT ride_id) AS num_of_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'member' THEN ride_id END) AS num_of_member_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'casual' THEN ride_id END) AS num_of_casual_trips
  FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
  GROUP BY 
    start_station, end_station
  ORDER BY 
    num_of_trips DESC
  LIMIT 10
)

-- Add percentage calculations for member and casual users
SELECT
  CONCAT(start_station, ' - ', end_station) AS route,
  num_of_trips,
  num_of_member_trips,
  num_of_casual_trips,
  ROUND((num_of_member_trips / num_of_trips) * 100, 2) AS member_percentage,
  ROUND((num_of_casual_trips / num_of_trips) * 100, 2) AS casual_percentage
FROM 
  route_user_type
```
- The most frequently used route is "Calumet Ave & 33rd St - State St & 33rd St" with 11,823 trips.
- Ellis Ave & 60th St - University Ave & 57th St is another highly traveled route, indicating a potential hub near educational institutions.
- Routes near Dusable Lake Shore Dr appear multiple times, suggesting high tourist or recreational traffic.
- The top routes are clustered around South Chicago, suggesting areas of high commuter or leisure activity.

![image](https://github.com/user-attachments/assets/99899821-7bc0-4b7d-9d28-a7ccde8d0a8d)

#### Top 10 Routes: Member vs. Casual Usage Breakdown
- **High Member Rider Usage Routes**:
  - Calumet Ave & 33rd St - State St & 33rd St has the highest member usage (96.03%), indicating a strong reliance on this route for commuting or regular travel.
  - Other heavily member-dominated routes include MLK Jr Dr & 29th St - State St & 33rd St (96.16%) and Loomis St & Lexington St - Morgan St & Polk St (93.73%). These routes likely serve commuters and frequent users.

- **High Casual Rider Usage Routes**:
  - Dusable Lake Shore Dr & Monroe St - Streeter Dr & Grand Ave is dominated by casual riders (88.49%), suggesting that tourists or leisure riders heavily use this route.
  - Dusable Lake Shore Dr & Monroe St - Shedd Aquarium (88.64%) further confirms that casual riders prefer lakefront and recreational destinations.
  - Marketing Implication: Target casual riders with short-term promotions, flexible rental options, and discounts at these locations to increase engagement.

![image](https://github.com/user-attachments/assets/570e5245-1498-40f6-9bc6-4b2bb41aa1de)

#### Top 10 Routes: Seasonal Usage

```sql
WITH SeasonalData AS (
  SELECT
    LEAST(start_station_name, end_station_name) AS start_station,
    GREATEST(start_station_name, end_station_name) AS end_station,
    ride_id,
    EXTRACT(MONTH FROM cleaned_started_at) AS month_of_year,
    
    -- Seasonal classification based on Chicago's weather
    CASE
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 3 AND 5 THEN 'Spring'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 6 AND 8 THEN 'Summer'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 9 AND 11 THEN 'Fall'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) IN (12, 1, 2) THEN 'Winter'
    END AS season
  FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
)

-- Aggregate trips by route and season, and then calculate the percentages
SELECT 
  CONCAT(start_station, ' - ',end_station) AS route,
  COUNT(DISTINCT ride_id) AS num_of_trips,
  
  -- Calculate season percentages for each route
  ROUND(
    (SUM(CASE WHEN season = 'Spring' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS spring_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Summer' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS summer_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Fall' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS fall_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Winter' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS winter_percentage

FROM 
  SeasonalData
GROUP BY 
  route
ORDER BY 
  num_of_trips DESC
LIMIT 10
```

- **1. Casual-Dominated Routes: Strong Summer Peaks, Sharp Winter Decline**

  - Routes such as:
     - "Dusable Lake Shore Dr & Monroe St - Streeter Dr & Grand Ave"
     - "Dusable Lake Shore Dr & North Blvd - Streeter Dr & Grand Ave"
     - "Dusable Lake Shore Dr & Monroe St - Shedd Aquarium"

    These routes show a high percentage of trips in summer (above 40%) but drop significantly in winter. They are also heavily used by casual riders, reinforcing that these routes are likely tourist or leisure-friendly.

    These routes align with locations that are popular attractions in Chicago.
    
- **2. Member-Dominated Routes: Fall Peaks and More Even Seasonal Distribution**

  - Routes such as:
    - "Mik Jr Dr & 29th St - State St & 33rd St"
    - "Calumet Ave & 33rd St - State St & 33rd St"
    - "Loomis St & Lexington St - Morgan St & Polk St"

    These routes show a higher percentage of trips in fall (above 40%) and a more evenly distributed seasonal pattern compared to casual-heavy routes. While winter remains the lowest period, these routes retain higher winter usage than tourist-dominated routes.

    Member percentage remains consistently high, suggesting reliance on these routes for transportation rather than leisure.
    
- **3. General Seasonal Trends: Winter Usage Drops Across All Routes**

Regardless of membership type, winter consistently has the lowest trip volume.
Member-dominant routes retain some winter usage, but casual-heavy routes see near-total drop-off.
This suggests that weather plays a critical role in ridership behavior, especially among casual users.
![image](https://github.com/user-attachments/assets/671c1bea-fafa-4393-b299-3947ced7877c)


### Trip Duration Analysis
This section explores trip duration patterns to understand user engagement.
- Overall Trends: What is the typical trip length?
- Rider and Bike Type Breakdown: How do durations vary across rider and bike types?

#### Average and Median Trip Durations by Rider Type
```sql
WITH median_duration AS (
  SELECT 
    member_casual AS rider_type,
    PERCENTILE_CONT(trip_duration, 0.5) OVER (PARTITION BY member_casual) AS median_trip_duration_minutes
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
)

,average_duration AS (
  SELECT 
    member_casual AS rider_type,
    ROUND(AVG(trip_duration)) AS average_trip_duration_minutes
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
  GROUP BY member_casual
)

SELECT DISTINCT 
  m.rider_type, 
  m.median_trip_duration_minutes, 
  a.average_trip_duration_minutes
FROM median_duration AS m
JOIN average_duration AS a
ON m.rider_type = a.rider_type
```
|Row	 |rider_type |median_trip_duration_minutes|average_trip_duration_minutes|
|---|---|---|---|
|1	|casual |12.0|20.0|
|2 	|member |8.0 |12.0|

- Casual riders tend to take longer trips on average (20 minutes) compared to members (12 minutes).
- The median trip durations show a smaller difference: 8 minutes for members and 12 minutes for casual riders.

![image](https://github.com/user-attachments/assets/b80c23e9-42ce-4494-8de0-c322d7bcb6b3)

#### Trip Durations by Bike Type
Trip durations are further analyzed by bike type for both rider categories:

```sql
SELECT
  member_casual,
  rideable_type,
  ROUND(AVG(trip_duration)) AS average_trip_duration_minutes
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table` 
GROUP BY
  member_casual,
  rideable_type
```

|Row	|member_casual |rideable_type |average_trip_duration_minutes|
|---|---|---|---|
|1	|member |electric_bike |11.0|
|2	|member |classic_bike |13.0|
|3	|casual |electric_bike |15.0|
|4	|casual |classic_bike |25.0|
|5	|casual |docked_bike |49.0|

- Electric Bikes have the shortest trip durations:
  - Members average 11 minutes, while casual riders take 15 minutes—a 4-minute difference.
  - This suggests efficiency and ease of use might attract members to electric bikes.
- Classic Bikes Show a Large Gap:
  - Members average 13 minutes per trip, while casual riders take nearly twice as long.
  - Possible reasons: route familiarity, confidence, or pricing differences between members and casual users.
- Docked Bikes Have the Longest Trips (Casual Only):
  - 49 minutes on average—far exceeding other bike types.
  - Indicates casual users may be leisure riders, tourists, or struggling with docking locations.
    
**Business Strategies**:
- Promote Membership for Casual Riders Using Docked & Classic Bikes
Since casual riders take longer trips, offering discounts or incentives for long rides with membership could convert them.
- Expand Electric Bike Usage Among Casual Riders
Casual riders already use electric bikes for quicker trips, so targeted promotions, trial offers, or lower e-bike rates could encourage membership.
- Optimize Docking Station Locations & Pricing
If docked bike trips are long due to limited station availability, improving station placement could reduce trip times and enhance rider experience.

![image](https://github.com/user-attachments/assets/97e48bcb-b4b0-4851-a71a-c621a968add6)

### Trip Distance Analysis
This analysis explores trip distances to assess Cyclistic's geographical reach and user behavior.
- Overall Trends: Are trips typically short or long?
- Rider and Bike Type Breakdown: How do distances vary by rider type (casual vs. member) and bike type (classic, docked, electric)?

#### Overall Trip Distance
The median and average distances for all trips provide an overview of the typical distances traveled.
- **Median Trip Distance**: 

```sql
SELECT DISTINCT
  ROUND(PERCENTILE_CONT(distance_meters,0.5) OVER()) AS median_distance_meters
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table`
```

|Row	|median_distance_meters|
|---|---|
|1	|1658.0|

The median trip distance is 1658 meters (or approximately 1.65 kilometers), indicating that half of the trips are shorter than this distance.

- **Average Trip Distance**:

```sql
SELECT 
  ROUND(AVG(distance_meters)) AS average_distance_meters
FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
```

|Row	|average_distance_meters|
|---|---|
|1	|2269.0|

The average trip distance is 2269 meters (or approximately 2.27 kilometers), which is slightly higher than the median, reflecting the impact of longer trips in the dataset.

#### Trip Distance by Rider Type
The average and median distances differ between member and casual riders, suggesting variations in how frequently they use the service and the nature of their trips.

- **Median Distance by Rider Type**:

```sql
SELECT DISTINCT
  member_casual AS rider_type,
  ROUND(PERCENTILE_CONT(distance_meters,0.5) OVER(PARTITION BY member_casual))  AS median_distance
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table`
```

|Row	|rider_type|median_distance|
|---|---|---|
|1	|member|1620.0|
|2	|casual|1756.0|

Members have a median distance of 1620 meters (1.62 km), while casual riders have a median distance of 1756 meters (1.76 km). Casual riders, on average, tend to take slightly longer trips than members.

- **Average Distance by Rider Type**:

```sql
SELECT
  member_casual AS rider_type,
  ROUND(AVG(distance_meters)) AS avg_distance
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table` 
GROUP BY
  rider_type
```

|Row|	rider_type|avg_distance|
|---|---|---|
|1	|casual|2336.0|
|2	|member|2234.0|

- Casual riders have an average trip distance of 2336 meters (2.34 km), while members have an average of 2234 meters (2.23 km).
- This indicates that casual riders, who might use the service for leisure or irregular trips, tend to travel slightly further than members, who may use bikes more frequently for short, routine trips.

#### Trip Distance by Bike Type
The trip distance analysis by bike type highlights preferences and patterns in the use of different bike types by rider type.

- **Median Distance by Bike Type**:
```sql
SELECT DISTINCT
  member_casual AS rider_type,
  rideable_type AS bike_type,
  ROUND(PERCENTILE_CONT(distance_meters,0.5) OVER(PARTITION BY member_casual, rideable_type))  AS median_distance
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table`
ORDER BY 
  rider_type DESC
```
|Row	|rider_type|bike_type|median_distance|
|---|---|---|---|
|1	|member|electric_bike|1882.0|
|2	|member|classic_bike|1431.0|
|3	|casual|docked_bike|2009.0|
|4	|casual|electric_bike|1791.0|
|5	|casual|classic_bike|1719.0|

![image](https://github.com/user-attachments/assets/669df180-9f70-4812-8672-cb30c5976ae9)


- **Average Distance by Bike Type**:

```sql
SELECT
  member_casual AS rider_type,
  rideable_type AS bike_type,
  ROUND(AVG(distance_meters)) AS avg_distance
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table` 
GROUP BY
  rider_type,
  bike_type
ORDER BY
  rider_type DESC,
  avg_distance DESC
```

|Row	|rider_type|bike_type|avg_distance|
|---|---|---|---|
|1	|member|electric_bike|2487.0|
|2	|member|classic_bike|1996.0|
|3	|casual|docked_bike|2691.0|
|4	|casual|electric_bike|2355.0|
|5	|casual|classic_bike|2303.0|

![image](https://github.com/user-attachments/assets/d6b471bc-34a5-4c7f-b776-6cc81fcfb27f)


- **Electric bikes**: Members traveling on electric bikes have the longer average trip distance of 2487 meters (2.49 km), which could be due to the comfort and efficiency of electric bikes for longer trips. Casual riders using electric bikes have an average distance of 2355 meters (2.36 km).
- **Classic bikes**: Members have an average trip distance of 1996 meters (1.99 km), while casual riders have a slightly higher average at 2303 meters (2.30 km).
- **Docked bikes**: Casual riders using docked bikes travel the farthest, with an average distance of 2691 meters (2.69 km), indicating that docked bikes are used for relatively longer, planned trips.

- Members and casual riders both seems to prefer electric bikes over classic bike for longer distance trips.

#### Distance Distribution
The distance analysis highlights the patterns in trip lengths within the bike-sharing system, segmented into distinct categories:

**Trip Categories**:
Trips were categorized by their lengths into four groups:

```sql
SELECT
  CASE 
    WHEN distance_meters BETWEEN 10 AND 2000 THEN 'Short (10m-2km)'
    WHEN distance_meters BETWEEN 2001 AND 5000 THEN 'Medium (2km-5km)'
    WHEN distance_meters BETWEEN 5001 AND 10000 THEN 'Long (5km-10km)'
    ELSE 'Very Long (10km+)' 
  END AS distance_category,
  COUNT(*) AS trip_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY distance_category
ORDER BY trip_count DESC
```

|Row	|distance_category |trip_count|percentage|
|---|---|---|---|
|1	|Short (10m-2km)|3,147,188|58.72|
|2	|Medium (2km-5km)|1,754,597|32.74|
|3	|Long (5km-10km)|416,437|7.77|
|4	|Very Long (10km+)|40,975|0.76|

![image](https://github.com/user-attachments/assets/c38c999f-ed61-437b-9bb8-1fda42e80098)


Trip Distribution Across Categories:
- The majority of trips fall under the "Short" category (10m-2km), making up 58.72% of all rides. "Medium" trips (2km-5km) are the second most common at 32.74%, while "Long" (7.77%) and "Very Long" (0.76%) trips are rare. This indicates that the bike-sharing system is primarily used for short, local trips.

**Distance Breakdown by Rider Type**:
```sql
SELECT
  distance_category,
  member_casual AS rider_type,
  COUNT(*) AS trip_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY member_casual), 2) AS percentage_within_rider_type
FROM (
  SELECT
    CASE 
      WHEN distance_meters BETWEEN 10 AND 2000 THEN 'Short (10m-2km)'
       WHEN distance_meters BETWEEN 2001 AND 5000 THEN 'Medium (2km-5km)'
      WHEN distance_meters BETWEEN 5001 AND 10000 THEN 'Long (5km-10km)'
    ELSE 'Very Long (10km+)'
    END AS distance_category,
    member_casual,
    distance_meters
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
) AS categorized
GROUP BY distance_category, member_casual
ORDER BY member_casual, percentage_within_rider_type DESC
```
|Row	|distance_category|rider_type|trip_count|percentage_within_rider_type|
|---|---|---|---|---|
|1	|Short (10m-2km)|casual|1,045,942|56.53|
|2	|Medium (2km-5km)|casual|648,190|35.03|
|3	|Long (5km-10km)|casual|138,516|7.49|
|4	|Very Long (10km+)|casual|17,563|0.95|
|5	|Short (10m-2km)|member|2,101,246|59.88|
|6	|Medium (2km-5km)|member|1,106,407|31.53|
|7	|Long (5km-10km)|member|277,921|7.92|
|8	|Very Long (10km+)|member|23,412|0.67|

![image](https://github.com/user-attachments/assets/d6a90eb8-0666-4a78-b1dd-d3782866a07f)


**Rider Type Behavior**:
- Both casual riders and members follow similar patterns, with most trips being short, followed by medium. However, members account for a larger volume of rides overall, particularly in the "Short" and "Medium" categories. This suggests that members use the system more frequently, likely for commuting or routine tasks, whereas casual riders engage with it for occasional, leisurely activities.


**Distance Breakdown by Bike Type**:
```sql
SELECT
  distance_category,
  bike_type,
  COUNT(*) AS trip_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (PARTITION BY distance_category), 2) AS percentage_within_bike_type
FROM (
  SELECT
    CASE 
      WHEN distance_meters BETWEEN 10 AND 2000 THEN 'Short (10m-2km)'
      WHEN distance_meters BETWEEN 2001 AND 5000 THEN 'Medium (2km-5km)'
      WHEN distance_meters BETWEEN 5001 AND 10000 THEN 'Long (5km-10km)'
      ELSE 'Very Long (10km+)' 
    END AS distance_category,
    rideable_type AS bike_type,
    distance_meters
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
) AS categorized
GROUP BY distance_category, bike_type
ORDER BY distance_category, percentage_within_bike_type DESC
```
|Row	|distance_category|bike_type|trip_count|percentage_bike_type|
|---|---|---|---|---|
|1	|Long (5km-10km)|electric_bike|239,309|57.47|
|2	|Long (5km-10km)|classic_bike|174,204|41.83|
|3	|Long (5km-10km)|docked_bike|2924|0.7|
|4	|Medium (2km-5km)|electric_bike|959356|54.68|
|5	|Medium (2km-5km)|classic_bike|785155|44.75|
|6	|Medium (2km-5km)|docked_bike|10086|0.57|
|7	|Short (10m-2km)|classic_bike|1691499|53.75|
|8	|Short (10m-2km)|electric_bike|1442329|45.83|
|9	|Short (10m-2km)|docked_bike|13360|0.42|
|10	|Very Long (10km+)|electric_bike|24148|58.93|
|11	|Very Long (10km+)|classic_bike|16356|39.92|
|12	|Very Long (10km+)|docked_bike|471|1.15|

![image](https://github.com/user-attachments/assets/c87c5b92-5d88-4356-ba40-f4d45f9de578)

- Electric Bikes are preferred for medium (54.68%), long (57.47%), and very long trips (58.93%) due to their efficiency over greater distances.
- Classic Bikes dominate short trips (53.75%) but also contribute significantly to medium (44.75%) and long trips (41.83%), reflecting their versatility.
- Docked Bikes account for a minimal share across all categories, with their highest usage in very long trips(1.15%).

___
Behavior Differences:
- Casual riders may use the system more for leisure or one-off longer trips, while members lean towards practical, short-distance rides for daily routines.
---
---
Electric Bike Popularity:
- The preference for electric bikes in longer trips highlights their role in expanding the system's reach, making them crucial for encouraging higher usage and reducing physical effort.
---
---
Optimization Opportunities:
- Promote membership plans targeting casual users with medium or long-distance usage patterns.
- Increase the availability of electric bikes, especially in areas with higher demand for medium and long trips.
---

**Cyclistic Trip Frequency and Patterns by User Type: Hourly, Daily, Weekly, and Seasonal Trends**

```sql
SELECT 
    member_casual,
    EXTRACT(HOUR FROM cleaned_started_at) AS hour_of_day,
    EXTRACT(DAYOFWEEK FROM cleaned_started_at) AS day_of_week,
    EXTRACT(MONTH FROM cleaned_started_at) AS month_of_year,
    ROUND(AVG(distance_meters),2) AS avg_distance_meters,
    ROUND(AVG(trip_duration),2) AS avg_trip_duration_minutes,
    COUNT(*) AS trips
FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY 
    member_casual, hour_of_day, day_of_week, month_of_year
ORDER BY 
    hour_of_day, avg_distance_meters DESC\
```

**Hourly Trip Volume Trends: Member vs. Casual Users**
![image](https://github.com/user-attachments/assets/6230e3f6-2a3c-4d85-a0ca-bc796c9fd677)

- **Member Usage Patterns**:
Members exhibit distinct peaks in trip volume during typical commuting hours:
  - Morning peak: Around 8 AM.
  - Evening peak: Around 5 PM.
This pattern suggests that members primarily use the service for work-related trips during weekdays.

- **Casual User Usage Patterns**:
  - Casual users display a steady increase in trips throughout the morning, with a pronounced peak around midday (5 PM).
  - This suggests that casual users tend to use Cyclistic more for leisurely or occasional trips, often in the late afternoon and evening.

- **Comparison of Member vs. Casual User Trends**:
  - Peak usage times: Members dominate during peak commuting hours (8 AM and 5 PM), while casual users have their peak mainly around 5 PM.
  - Low-activity period: Both user types experience a significant drop in activity late at night, with the lowest volumes occurring between 12 AM and 5 AM.


**Cyclistic Trips by Day of the Week**
![image](https://github.com/user-attachments/assets/284b87c4-9549-48f5-965f-f9f37dd18446)

- **Distinct Usage Patterns**:
  - Members dominate usage on weekdays, with consistent trips from Monday to Friday, peaking on Wednesday with around 565,000 trips. This trend indicates that members primarily use Cyclistic for weekday commuting, likely for work or school.
  - Casual users have a different pattern, with fewer trips during the weekdays and a significant spike in trips on Saturday (374,000 trips) and Sunday. This suggests that casual users mainly use the service for leisure and recreational purposes over the weekend.
 

**Cyclistic Monthly Trip Trends**
![image](https://github.com/user-attachments/assets/eeb85501-0d3e-4277-83bc-a401f7cefedd)

This analysis highlights seasonal patterns in trip usage by members and casual riders throughout the year.

- **Peak Usage Periods**:
  - Members: The number of trips by members peaks in August, with approximately 437,000 trips recorded.
  - Casual: Casual usage peaks slightly earlier, in July, with about 297,000 trips.
The peak summer months (June to August) account for the highest share of trips for both user groups.

- **Seasonal Variation**:
  - Both members and casual users show significant seasonal variation:
  - Members display a relatively steady rise and fall in trip numbers, indicating regular, year-round use.
  - Casual riders exhibit a more pronounced increase in trips during the warmer months, suggesting greater sensitivity to weather and seasonal conditions.
  - Winter months (December to February) have the lowest usage for both groups, but casual ridership drops more dramatically during this period.
 
- **Member vs. Casual Trends**:
  - Members consistently take more trips than casual riders across all months of the year.
  - Both groups show a similar pattern of rising trip counts in spring, peaking in summer, and dropping in winter, indicating that both casual and member users are influenced by seasonal factors.

**Cyclistic Riders' Average Trip Distance and Duration by Hour of Day**
![image](https://github.com/user-attachments/assets/c398ee04-b7b9-4c1d-9270-b0d5264c36a7)

**Top Chart (Avg. Distance in Meters by Hour of Day)**:
  - Members generally cover slightly longer distances during most hours compared to casual users.
  - Casual users exhibit more variability in average distance throughout the day.
  - There's a noticeable peak in distance covered by member users around early morning.

**Bottom Chart (Average Trip Duration in Minutes by Hour of Day)**:
  - Casual users have significantly longer trip durations, especially during midday (around noon).
  - Members maintain more consistent trip durations throughout the day, averaging about 10–15 minutes.
  - Casual users’ average trip duration spikes sharply around 10 a.m. to 2 p.m. and decreases in the evening.

**Cyclistic Riders' Average Trip Distance and Duration by Day of Week**
![image](https://github.com/user-attachments/assets/56d9b81b-f17d-43f6-b284-ceab263f8cd1)

Casual Riders:
- Take longer trips (both in distance and duration) than members, especially on weekends.
Average trip duration and distance peak on Sundays, indicating leisure or recreational rides.

Members:
- Consistent trip distance and duration throughout the week, aligning with commuter or short errand trips.
Smaller fluctuations suggest trips are more utilitarian in nature.

Comparison:
- The difference in trip behavior between casual and member users supports the idea that casual riders use Cyclistic bikes primarily for recreational purposes, while members use them more for practical purposes.

**Cyclistic Riders' Average Trip Distance and Duration by Month of Year**
![image](https://github.com/user-attachments/assets/70e9e13d-3677-4996-9b17-f6791fab02d3)

Casual Riders Take Longer Trips
- Casual riders consistently have longer average trip durations than members.
- The difference is most pronounced in the summer months (May–August) when casual riders’ trips peak at around 20 minutes, while members’ trips remain around 12–13 minutes.
- This suggests that casual riders may be using bikes more for leisure and sightseeing, rather than for commuting or quick trips.

Trip Distances Are Similar, but Casual Riders Travel Slightly Less Overall
- Both groups have their longest average trip distances in June and July (~2.4 km).
- However, casual riders’ trip distances decline more steeply after the summer, whereas members maintain a relatively stable distance year-round.
- This implies that casual riders are more seasonal users, while members have a more consistent and practical biking habit.

Casual Riders’ Usage Drops in the Off-Season
- As temperatures drop (September–December), casual riders' trip duration and distance decrease significantly, while members maintain more stable behavior.
- This seasonal drop suggests that casual riders are likely tourists or occasional users, and marketing efforts should focus on converting them into year-round users.

Marketing Implications & Strategies:
Target Leisure Riders with Membership Benefits
Since casual riders take longer trips in summer, Cyclistic could promote seasonal membership discounts or "unlimited ride" options during peak months to encourage commitment.
Highlight the Convenience of Membership for Frequent Trips
Members' trips are shorter but consistent, indicating they likely use the service for daily commutes or errands.
Cyclistic could advertise membership benefits like cost savings for frequent riders, exclusive docking privileges, or priority access to bikes to encourage casual users to switch.
Retain Casual Riders Beyond Summer
Since casual riders significantly reduce their trips in the off-season, Cyclistic could launch fall/winter promotions (e.g., discounted first-month membership, loyalty rewards for repeated rides).
Providing comfortable, weather-resistant bikes and partnering with local businesses for incentives (e.g., discounts at coffee shops for members) might also help.

![image](https://github.com/user-attachments/assets/d711a5bc-694b-4881-85e5-2db658548428)

Trip Volume Trends Summary

Hourly Trends:

Members show a gradual increase in trip volume during commuting hours (6–9 AM and 4–7 PM), with peaks corresponding to typical work hours.

Casual users have a pronounced peak in trip volume around midday, indicating higher recreational usage during non-commute hours.

Weekly Trends:

Members have a more consistent trip volume throughout the week, with slightly higher activity on weekdays, suggesting regular commuting patterns.

Casual users, on the other hand, experience a noticeable increase in trip volume on weekends, reflecting recreational or leisure-based activity.

Monthly Trends:

Both user groups see the highest trip volumes during the summer months (June, July, and August), aligning with warm weather and longer daylight hours.

There is a sharp decline in trip volume during the winter months (December, January, and February) due to colder weather.

Members maintain relatively steady activity during the colder months compared to casual users, who exhibit highly seasonal behavior.


![image](https://github.com/user-attachments/assets/85e338f3-5747-4b9e-ba2b-8b022dca8ff1)

Trip Distance and Duration Trends Summary

Hourly Trends:

Trip Distance: Members generally cover slightly longer distances during most hours of the day compared to casual users.

There is a noticeable peak in the distance covered by members during early morning hours, likely corresponding to commute times.

The lowest average distance for members occurs around midday, whereas casual users tend to cover longer distances during this period, reflecting recreational usage.

Trip Duration: Casual users consistently have significantly longer trip durations than members, with a prominent spike around midday (approximately noon).

Members maintain more consistent trip durations throughout the day, averaging around 10 to 15 minutes, reflecting routine and predictable travel patterns.

Weekly Trends:

Trip Distance: Both members and casual users cover longer distances on the weekends compared to weekdays, indicating increased recreational and leisure activities during weekends.

During weekdays, members maintain a steady average trip distance, while casual users cover relatively lower distances.

Trip Duration: Casual users show a significant increase in average trip duration on weekends, indicating more leisurely and recreational rides.

Members, on the other hand, maintain steady and predictable trip durations throughout the week, reinforcing their usage pattern for commuting and routine travel.

Monthly Trends:

Trip Volume: Both members and casual users exhibit the highest trip volumes during the summer months (June, July, and August), which aligns with favorable weather conditions for biking.

Member vs. Casual Patterns: Members maintain a relatively higher trip volume during colder months compared to casual users, suggesting year-round commuting behavior.

Casual users' trips are highly seasonal, with a significant increase during warmer months, reflecting recreational or tourist-based usage patterns.



**Casual Users vs. Members: Behavioral Differences**
- Casual users consistently take longer trips in terms of duration compared to members, with peak trip durations occurring on weekends and during summer months. This indicates recreational use, especially during leisure periods.
- Members have shorter and more consistent trip durations throughout the week and across seasons, aligning with routine commuting behavior.

**Hourly Trends Summary**:
1. Trip Distance:
  - There is a noticeable peak in the distance covered by members during the early morning hours, likely corresponding to their commute times.
  - The lowest average distance for members occurs around midday, whereas casual users tend to cover longer distances during this period, reflecting recreational usage.

2. Trip Duration:
  - Casual users consistently have significantly longer trip durations than members, with a prominent spike around midday (approximately noon).
  - Members maintain more consistent trip durations throughout the day, averaging around 10 to 13 minutes, reflecting routine and predictable travel patterns.

**Weekly Trends**:
1. Trip Distance:
  - Both members and casual users cover longer distances on the weekends compared to weekdays, indicating increased recreational and leisure activities during weekends.
  - During weekdays, members maintain a steady average trip distance, while casual users cover relatively lower distances.

3. Trip Duration:
  - Casual users show a significant increase in average trip duration on weekends, indicating more leisurely and recreational rides.
  - Members, on the other hand, maintain steady and predictable trip durations throughout the week, reinforcing their usage pattern for commuting and routine travel.

**Seasonal Trends (Monthly Patterns)**:
  - Both casual users and members take longer trips (in terms of distance) during summer months (June to August).
  - Casual users' trip durations peak in June, suggesting increased recreational cycling during warm weather.
  - Members maintain relatively stable trip durations throughout the year(10 to 13 minutes), with a slight increase during summer.





```sql
  -- Top 10 popular routes and counts for member and casual user trips
SELECT
    LEAST(start_station_name, end_station_name) AS start_station,
    GREATEST(start_station_name, end_station_name) AS end_station,
    COUNT(DISTINCT ride_id) AS num_of_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'member' THEN ride_id END) AS num_of_member_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'casual' THEN ride_id END) AS num_of_casual_trips
FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY 
    start_station, end_station
ORDER BY 
  num_of_trips DESC
LIMIT 10
```
|Row	|start_station|end_station|num_of_trips|num_of_member_trips|num_of_casual_trips|
|---|---|---|---|---|---|
|1	|calumet ave & 33rd st|state st & 33rd st|11823|11354|469|
|2	|ellis ave & 55th st|ellis ave & 60th st|10985|7405|3580|
|3	|ellis ave & 60th st|university ave & 57th st|10609|8141|2468|
|4	|dusable lake shore dr & monroe st|streeter dr & grand ave|8243|949|7294|
|5	|loomis st & lexington st|morgan st & polk st|6177|5790|387|
|6	|kimbark ave & 53rd st|university ave & 57th st|5107|3789|1318|
|7	|mlk jr dr & 29th st|state st & 33rd st|4844|4658|186|
|8	|lake park ave & 56th st|university ave & 57th st|4667|3520|1147|
|9	|dusable lake shore dr & monroe st|shedd aquarium|4165|473|3692|
|10	|dusable lake shore dr & north blvd|streeter dr & grand ave|4090|830|3260|

```sql
WITH route_user_type AS (
  -- Top 10 popular routes and counts for member and casual user trips
  SELECT
    LEAST(start_station_name, end_station_name) AS start_station,
    GREATEST(start_station_name, end_station_name) AS end_station,
    COUNT(DISTINCT ride_id) AS num_of_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'member' THEN ride_id END) AS num_of_member_trips,
    COUNT(DISTINCT CASE WHEN member_casual = 'casual' THEN ride_id END) AS num_of_casual_trips
  FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
  GROUP BY 
    start_station, end_station
ORDER BY 
  num_of_trips DESC
LIMIT 10
)

-- Calculate percentage of member and casual trips for each route
SELECT
  CONCAT(start_station, ' - ',end_station) AS route,
  num_of_trips,
  ROUND((num_of_member_trips / num_of_trips) * 100, 2) AS member_percentage,
  ROUND((num_of_casual_trips / num_of_trips) * 100, 2) AS casual_percentage
FROM 
  route_user_type

```

|Row	|start_station|end_station|num_of_trips|member_percentage|casual_percentage|
|---|---|---|---|---|---|
|1	|calumet ave & 33rd st|state st & 33rd st|11823|96.03|3.97|
|2	|ellis ave & 55th st|ellis ave & 60th st|10985|67.41|32.59|
|3	|ellis ave & 60th st|university ave & 57th st|10609|76.74|23.26|
|4	|dusable lake shore dr & monroe st|streeter dr & grand ave|8243|11.51|88.49|
|5	|loomis st & lexington st|morgan st & polk st|6177|93.73|6.27|
|6	|kimbark ave & 53rd st|university ave & 57th st|5107|74.19|25.81|
|7	|mlk jr dr & 29th st|state st & 33rd st|4844|96.16|3.84|
|8	|lake park ave & 56th st|university ave & 57th st|4667|75.42|24.58|
|9	|dusable lake shore dr & monroe st|shedd aquarium|4165|11.36|88.64|
|10	|dusable lake shore dr & north blvd|streeter dr & grand ave|4090|20.29|79.71|

![image](https://github.com/user-attachments/assets/a889e63d-6e7a-4341-89bb-82f45b21383a)



#### Cyclistic's Top 10 Routes: Seasonal Usage

```sql
WITH SeasonalData AS (
  SELECT
    LEAST(start_station_name, end_station_name) AS start_station,
    GREATEST(start_station_name, end_station_name) AS end_station,
    ride_id,
    EXTRACT(MONTH FROM cleaned_started_at) AS month_of_year,
    
    -- Seasonal classification based on Chicago's weather
    CASE
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 3 AND 5 THEN 'Spring'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 6 AND 8 THEN 'Summer'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) BETWEEN 9 AND 11 THEN 'Fall'
      WHEN EXTRACT(MONTH FROM cleaned_started_at) IN (12, 1, 2) THEN 'Winter'
    END AS season
  FROM 
    `bike-share-case-study-430704.Bike_share.cleaned_table`
)

-- Aggregate trips by route and season, and then calculate the percentages
SELECT 
  CONCAT(start_station, ' - ',end_station) AS route,
  COUNT(DISTINCT ride_id) AS num_of_trips,
  
  -- Calculate season percentages for each route
  ROUND(
    (SUM(CASE WHEN season = 'Spring' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS spring_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Summer' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS summer_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Fall' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS fall_percentage,

  ROUND(
    (SUM(CASE WHEN season = 'Winter' THEN 1 ELSE 0 END) / COUNT(DISTINCT ride_id)) * 100, 
    2
  ) AS winter_percentage

FROM 
  SeasonalData
GROUP BY 
  route
ORDER BY 
  num_of_trips DESC
LIMIT 10
```

![image](https://github.com/user-attachments/assets/36862796-889f-4ce8-bc79-c0b33ca6eca5)




