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
With the dataset cleaned and prepared, the next step focuses on analyzing the data to derive meaningful insights. This analysis uncovers trends in trip frequency, bike type preferences, popular routes, and how usage patterns differ between casual riders and members.

### 1. Total Number Of Member vs. Casual Rider Trips
- **Members account for 65.48%** of all trips, while **casual riders make up 34.52%**, presenting a strong opportunity for conversion.
  
```sql
SELECT 
  member_casual AS rider_type,
  COUNT(*) AS ride_count,
  ROUND((COUNT(*) * 100.0) / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
GROUP BY member_casual
```
|Row |rider_type |ride_count |percentage|
|--- |--- |--- |--- |
|1 |casual |1,850,211|34.52|
|2 |member |3,508,986|65.48|

![image](https://github.com/user-attachments/assets/ef7e3504-49e9-4bfe-8e93-6b03a784284c)

### 2. Trip Frequency and Patterns by User Type
#### 2.1 Monthly Trip Trends

- **Members** peaks in **August(~437,000 trips)**.
- **Casual** riders peak in **July (~297,000 trips)**.
- Members consistently take more trips than casual riders across all months of the year.
- Both groups show a similar seasonal pattern, with trip counts rising in spring, peaking in summer, and dropping in winter. This indicates that both casual and member users are influenced by seasonal factors.
 
#### 2.2 Weekly Trip Trends
- **Members** dominate usage on weekdays, peaking on **Wednesday (~565,000 trips)**. This trend indicates that members primarily use Cyclistic for weekday commuting, likely for work or school.

- **Casual users** show a different pattern, with fewer trips during the weekdays and a significant spike in trips on **Saturday (~374,000 trips)** and Sunday. This suggests that casual users mainly use the service for leisure and recreational purposes over the weekend.

#### 2.3 Daily Trip Trends
- **Members** exhibit distinct peaks in trip volume during typical commuting hours:
  - **Morning peak**: Around 8 AM.
  - **Evening peak**: Around 5 PM.

- **Casual users** display a steady increase in trips throughout the morning, peak around early evening(5 PM), suggesting more flexible, leisure-based trips.

- Both user types experience a significant drop in activity late at night, with the lowest volumes occurring between 12 AM and 5 AM.

```sql
SELECT 
    member_casual AS rider_type,
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
    hour_of_day, avg_distance_meters DESC
```

![image](https://github.com/user-attachments/assets/9cc4646c-5476-46c8-bdff-5a497aef7295)

![image](https://github.com/user-attachments/assets/d45b51fa-c2eb-4cae-b462-40833c105b68)

![image](https://github.com/user-attachments/assets/7468a3fd-ba0e-47f7-8857-36f4b47d9239)


### 3. Preferred Bike Types by Rider Type
- **Casual riders** prefer **electric bikes (52.18%)** over **classic bikes (46.37%)**.
- **Members** prefer **classic bikes (51.56%)** over **electric bikes (48.44%)**.
- **Docked bikes** have minimal preference, with **casual riders (1.45%)** being the only group to use them.
  
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

![image](https://github.com/user-attachments/assets/7c6979c7-5005-4c1c-aab2-7e3fde1b163b)


### 4. Top 10 Popular Routes
#### 4.1 Overview of the Top 10 Routes
- The most frequently used route is **Calumet Ave & 33rd St - State St & 33rd St** with **11,823 trips**.
- Many top routes are clustered around **South Chicago**, highlighting areas of concentrated commuter and leisure activity.

![image](https://github.com/user-attachments/assets/99899821-7bc0-4b7d-9d28-a7ccde8d0a8d)

#### 4.2 Rider Type Breakdown: Member vs. Casual Users

- **High Member Rider Usage Routes**:
  - Calumet Ave & 33rd St - State St & 33rd St (96.03% member trips)
  - MLK Jr Dr & 29th St - State St & 33rd St (96.16%)
  - Loomis St & Lexington St - Morgan St & Polk St (93.73%)
  - Likely used for daily commuting by regular users.

- **High Casual Rider Usage Routes**:
  - Dusable Lake Shore Dr & Monroe St - Shedd Aquarium (88.64%)
  - Dusable Lake Shore Dr & Monroe St - Streeter Dr & Grand Ave (88.49% casual trips)
  - Dusable Lake Shore Dr & North Blvd - Streeter Dr & Grand Ave (79.71%)
  - Indicates heavy tourist & leisure activity near lakefront attractions.

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
![image](https://github.com/user-attachments/assets/dcce08a5-7c09-4b39-95ec-49d5098e4f9c)


#### 4.3 Top 10 Routes: Seasonal Usage Patterns
- **1. Casual-Dominated Routes: Strong Summer Peaks, Drop in Winter**

  - Routes such as:
     - "Dusable Lake Shore Dr & Monroe St - Streeter Dr & Grand Ave"
     - "Dusable Lake Shore Dr & North Blvd - Streeter Dr & Grand Ave"
     - "Dusable Lake Shore Dr & Monroe St - Shedd Aquarium"
show a high percentage of trips in summer (above 40%) but drop significantly in winter. They are heavily used by casual riders, reinforcing that these routes are likely tourist or leisure-friendly.

  - These routes align with locations that are popular attractions in Chicago.
    
- **2. Member-Dominated Routes: Fall Peaks and More Even Seasonal Distribution**

  - Routes such as:
    - "Mik Jr Dr & 29th St - State St & 33rd St"
    - "Calumet Ave & 33rd St - State St & 33rd St"
    - "Loomis St & Lexington St - Morgan St & Polk St"

    These routes show a higher percentage of trips in fall (above 40%) and a more evenly distributed seasonal pattern compared to casual-heavy routes. While winter remains the lowest period, these routes retain higher winter usage than tourist-dominated routes.

  - Member percentage remains consistently high, suggesting reliance on these routes for transportation rather than leisure.
    
- **3. General Seasonal Trends: Winter Usage Drops Across All Routes**

  - Regardless of membership type, winter consistently has the lowest trip volume.
Member-dominant routes retain some winter usage, but casual-heavy routes see near-total drop-off.
  - This suggests that weather plays a critical role in ridership behavior, especially among casual users.


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

![image](https://github.com/user-attachments/assets/671c1bea-fafa-4393-b299-3947ced7877c)


### 5. Trip Duration Analysis: Member vs. Casual Riders
To understand trip engagement levels, we analyzed trip duration by rider type and bike type.

#### Median Vs. Average Trip Durations by Rider Type
- Casual riders take longer trips than members on both average and median measures.
- On average, casual trips last **20 minutes**, which is **8 minutes longer** than member trips (12 minutes).
- The median trip duration for casual riders is **12 minutes**, compared to **8 minutes** for members, reinforcing a trend of longer rides among casual users.
- Members, on the other hand, show a more consistent pattern with shorter trips.

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

![image](https://github.com/user-attachments/assets/b80c23e9-42ce-4494-8de0-c322d7bcb6b3)

#### Trip Durations by Bike Type
Trip durations are further analyzed by bike type for both rider categories:

```sql
SELECT DISTINCT
  member_casual AS rider_type,
  rideable_type AS bike_type,
  PERCENTILE_CONT(trip_duration, 0.5) OVER (PARTITION BY member_casual, rideable_type) AS median_trip_duration_minutes
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
ORDER BY 
  rider_type DESC
```

|Row	|rider_type |bike_type |median_trip_duration_minutes|
|---|---|---|---|
|1	|member |electric_bike |8.0|
|2	|member |classic_bike |9.0|
|3	|casual |electric_bike |10.0|
|4	|casual |classic_bike |14.0|
|5	|casual |docked_bike |26.0|

**Casual Riders Take Longer Trips Across All Bike Types**
- Casual riders have longer median trip durations compared to members, suggesting more leisure-oriented or exploratory rides.
- The biggest gap is seen in docked bikes, where casual riders have a median duration of 26 minutes, much longer than any other bike type.

**Members Have More Consistent and Shorter Rides**
- Members have shorter and more predictable trip durations, reinforcing the idea that they primarily use Cyclistic for commuting or frequent short trips.
- The difference between classic and electric bikes is minimal for members (8 min vs. 9 min), showing they choose bikes based on availability rather than trip length.

**Bike Type Preferences and Usage Patterns**
- Electric bikes: Both members (8 min) and casuals (10 min) have the shortest trip durations, likely due to the convenience and speed of e-bikes.
- Classic bikes: Casual riders (14 min) use classic bikes for longer rides than members (9 min), possibly for sightseeing or recreation.
- Docked bikes (casual riders only): With the longest median trip duration (26 min), these are likely used for leisure or necessity when no other bikes are available.
    
**Business Strategies**:
- Encouraging membership for casual riders: Since casual users take longer rides, offering membership perks for extended rides (e.g., discounts on longer rentals) could appeal to them.
- Targeted pricing strategies: Cyclistic could introduce a pricing structure that makes shorter casual trips less expensive with membership, nudging casuals to convert.
- Bike availability and placement: Since casual riders take the longest trips on docked bikes, increasing availability in key leisure areas may improve overall ridership.
  
![image](https://github.com/user-attachments/assets/d1b30bb5-99c4-4467-9951-19af683a56e8)


### 6. Trip Distance Analysis: Member vs. Casual Riders
This analysis examines how far Cyclistic users typically travel, comparing median and average trip distances for casual riders and members.

#### Average and Median Trip Durations by Rider Type
```sql
WITH median_distance AS (
  SELECT 
    member_casual AS rider_type,
    ROUND(PERCENTILE_CONT(distance_meters, 0.5) OVER(PARTITION BY member_casual) / 1000, 2) AS median_distance_km
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
)

,average_distance AS (
  SELECT 
    member_casual AS rider_type,
    ROUND(AVG(distance_meters) / 1000, 2) AS average_distance_km
  FROM `bike-share-case-study-430704.Bike_share.cleaned_table`
  GROUP BY member_casual
)

SELECT DISTINCT 
  m.rider_type,
  m.median_distance_km, 
  a.average_distance_km
FROM median_distance AS m
JOIN average_distance AS a 
ON m.rider_type = a.rider_type

```

|Row	|rider_type|median_distance_km|average_distance_km|
|---|---|---|---|
|1	|casual|1.76|2.34|
|2	|member|1.62|2.23|

- Casual riders travel slightly longer distances than members, with a median trip of 1.76 km vs. 1.62 km.
- The average distance is higher than the median for both groups, indicating that some longer trips are skewing the average.
- The difference between members and casual riders is not drastic, but casual riders tend to have more variability in their trip lengths.

**Business Strategies**:
- Since casual riders already take slightly longer trips, offering membership perks for extended rides (e.g., reduced rates after a certain distance) may encourage sign-ups.
- Improving Member Experience:
  - Members have slightly shorter and more consistent trip distances, indicating a focus on commuting and routine travel.
  - Enhancing services like better bike availability at key commuter hubs could increase member retention.

![image](https://github.com/user-attachments/assets/54f3dec5-31f8-4cc5-81f4-aed1f02a38b1)

#### Trip Distance by Bike Type
The trip distance analysis by bike type highlights preferences and patterns in the use of different bike types by rider type.

```sql
SELECT DISTINCT
  member_casual AS rider_type,
  rideable_type AS bike_type,
  ROUND(PERCENTILE_CONT(distance_meters,0.5) OVER(PARTITION BY member_casual, rideable_type)/1000,2) AS median_distance_km
FROM
  `bike-share-case-study-430704.Bike_share.cleaned_table`
ORDER BY 
  rider_type DESC
```
|Row	|rider_type|bike_type|median_distance_km|
|---|---|---|---|
|1	|member|electric_bike|1.88|
|2	|member|classic_bike|1.43|
|3	|casual|electric_bike|1.79|
|4	|casual|classic_bike|1.72|
|5	|casual|docked_bike|2.01|

**Casual vs. Member Riding Behavior**:
- Casual riders tend to take longer trips than members across all bike types, suggesting more leisure-oriented usage.
- Members have shorter, more consistent trip distances, indicating routine commuting patterns.

**Bike Type Preferences & Usage Patterns**:
- Electric bikes: Members travel the farthest on electric bikes (1.88 km median distance), likely due to their efficiency and speed for commuting. Casual riders take slightly shorter trips (1.79 km), possibly for convenience.
- Classic bikes: Members travel shorter distances (1.43 km) compared to casual riders (1.72 km), suggesting that casual users may take longer scenic or recreational rides.
- Docked bikes (casual riders only): Casual users take the longest trips on docked bikes (2.01 km), potentially due to station-based limitations requiring them to return the bike to a specific location.

**Business Strategy**:
- To convert casual riders into members, Cyclistic could promote membership perks for longer rides on docked and classic bikes.
- For electric bikes, emphasizing time savings and convenience could attract both casual users and retain members.
- Since casual riders take longer trips, targeted discounts on multi-hour rides or bundled trip passes could encourage sign-ups.

![image](https://github.com/user-attachments/assets/c78fda88-cd84-4256-8c72-3139616c5e8e)


#### Distance Distribution
The distance analysis highlights the patterns in trip lengths within the bike-sharing system, segmented into distinct categories:

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

**Rider Type Behavior**:
-Both casual and member riders display a similar pattern in their trip distances. For both groups, the vast majority of trips are short, followed by medium, long, and extra long trips. 
  
- Notably,for casual riders, short trips (56.53%) combined with medium trips (35.03%) make up over 91% of their total trips, highlighting that they predominantly use the service for shorter journeys.
  
**Business Strategy**:
- Bundle Rides at a Discount:
Create bundled packages specifically tailored for short and medium trips (e.g., “Buy 10 rides at a special price”). This package can demonstrate immediate cost savings compared to individual rides, making the membership upgrade more appealing to casual users.
- Dynamic Pricing Incentives:
Offer targeted discounts or loyalty rewards for riders who frequently take short and medium trips. For example, after a set number of rides, provide a discount or bonus credits if they convert to a membership. This approach leverages their typical usage pattern to deliver tangible value.

![image](https://github.com/user-attachments/assets/40fc9f58-08fe-44d7-aaba-b4fc48014721)

![image](https://github.com/user-attachments/assets/c22733ba-e522-4cfa-b25c-3284bbe76206)



---

**Business Strategies**
- Casual riders take longer trips, membership perks for extended rides could attract sign-ups.
- Members' predictable, shorter trips suggest commuter-focused incentives might work best for retention.
