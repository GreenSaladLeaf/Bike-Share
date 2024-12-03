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

```sql
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
    ROUND(end_lng,4) AS end_lng,         -- rounding to 4 decimal places (precision of about 10 meters)
FROM 
    `bike-share-case-study-430704.Bike_share.bike_share_12months`
WHERE
    start_lat BETWEEN 41.6 AND 42.1  -- Chicago-only latitude range
    AND start_lng BETWEEN -88 AND -87.5  -- Chicago-only longitude range
    AND end_lat BETWEEN 41.6 AND 42.1    -- Chicago-only latitude range
    AND end_lng BETWEEN -88 AND -87.5;  -- Chicago-only longitude range  
```

By standardizing the dataset, duplicates are removed, and inconsistencies in timestamps, text fields, and geographic coordinates are addressed. This ensures that any analysis, whether it be trip durations or station usage, is based on clean, uniform data

#### Step 4 : Investigate data that have been filtered out 
Investigates any rows that are outside this region, to understand the nature of the data that is being excluded. 

```sql
SELECT DISTINCT
    TRIM(ride_id) AS ride_id,                      -- remove extra space
    TRIM(LOWER(rideable_type)) AS rideable_type,   -- remove extra space and standardize case
    TIMESTAMP_TRUNC(started_at, SECOND) AS cleaned_started_at,
    TIMESTAMP_TRUNC(ended_at, SECOND) AS cleaned_ended_at,
    TRIM(LOWER(start_station_name)) AS start_station_name,  -- remove extra space and standardize case
    TRIM(LOWER(start_station_id)) AS start_station_id,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_name)) AS end_station_name,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_id)) AS end_station_id,          -- remove extra space and standardize case
    TRIM(LOWER(member_casual)) AS member_casual,            -- remove extra space and standardize case
    ROUND(start_lat,4) AS start_lat,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(start_lng,4) AS start_lng,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lat,4) AS end_lat,         -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lng,4) AS end_lng,         -- rounding to 4 decimal places (precision of about 10 meters)
FROM 
    `bike-share-case-study-430704.Bike_share.bike_share_12months`
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
There are 27 rows of data fell outside this range:
- 26 rows are slightly outside the Chicago boundaries, likely representing trips that crossed into neighboring suburbs. These trips are valid but don't align with the analysis focus on central Chicago and should be excluded.
  
- The 1 row with 0,0 coordinates and station information ('Stony Island Ave & 63rd St') represents an outlier due to incorrect geographic data, and no other matching records exist for the station. This row should also be excluded, as it does not contribute meaningful or reliable data.
  
These 27 rows represent a small fraction of the total dataset (less than 0.0005% of the 5,734,381 trips), excluding them helps ensure the analysis is focused on high-quality, relevant data that accurately reflects trips within central Chicago.

#### Step 5: Calculate the percentage of null value for lattitude and longtitude
To understand the extent of missing geographic data, a query was run to calculate the percentage of rows with NULL values for the start_lat, start_lng, end_lat, or end_lng fields.
```sql
WITH formatted_data AS (
    SELECT DISTINCT
    TRIM(ride_id) AS ride_id,                      -- remove extra space
    TRIM(LOWER(rideable_type)) AS rideable_type,   -- remove extra space and standardize case
    TIMESTAMP_TRUNC(started_at, SECOND) AS cleaned_started_at,  -- standardise timestamp
    TIMESTAMP_TRUNC(ended_at, SECOND) AS cleaned_ended_at,      -- standardise timestamp
    TRIM(LOWER(start_station_name)) AS start_station_name,  -- remove extra space and standardize case
    TRIM(LOWER(start_station_id)) AS start_station_id,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_name)) AS end_station_name,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_id)) AS end_station_id,          -- remove extra space and standardize case
    TRIM(LOWER(member_casual)) AS member_casual,            -- remove extra space and standardize case
    ROUND(start_lat,4) AS start_lat,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(start_lng,4) AS start_lng,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lat,4) AS end_lat,         -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lng,4) AS end_lng,         -- rounding to 4 decimal places (precision of about 10 meters)
FROM 
    `bike-share-case-study-430704.Bike_share.bike_share_12months`
)

SELECT 
  ROUND(((SELECT COUNT(ride_id) as missing_end_coordinate
    FROM formatted_data
      WHERE 
        start_lat is NULL
        or start_lng is null
        or end_lat is null
        or end_lng is null) / count(distinct ride_id)*100),2) As percentage

  FROM
    formatted_data
```

|Row|percentage|
|---|---|
|1|0.14|

This shows that only 0.14% of the dataset has missing geographic data, which is a small percentage. The rest of the dataset is complete with respect to geographic coordinates, ensuring the analysis is based on robust data.

#### Step 6: Check for other null values in the dataset

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

#### Step x: Check if all duplicate has been removed
```sql
WITH duplicate_removed AS (
SELECT DISTINCT
    TRIM(ride_id) AS ride_id,                      -- remove extra space
    TRIM(LOWER(rideable_type)) AS rideable_type,   -- remove extra space and standardize case
    TIMESTAMP_TRUNC(started_at, SECOND) AS cleaned_started_at,
    TIMESTAMP_TRUNC(ended_at, SECOND) AS cleaned_ended_at,
    TRIM(LOWER(start_station_name)) AS start_station_name,  -- remove extra space and standardize case
    TRIM(LOWER(start_station_id)) AS start_station_id,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_name)) AS end_station_name,      -- remove extra space and standardize case
    TRIM(LOWER(end_station_id)) AS end_station_id,          -- remove extra space and standardize case
    TRIM(LOWER(member_casual)) AS member_casual,            -- remove extra space and standardize case
    ROUND(start_lat,4) AS start_lat,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(start_lng,4) AS start_lng,       -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lat,4) AS start_lat,         -- rounding to 4 decimal places (precision of about 10 meters)
    ROUND(end_lng,4) AS start_lng,         -- rounding to 4 decimal places (precision of about 10 meters)
FROM 
    `bike-share-case-study-430704.Bike_share.bike_share_12months`
)
SELECT
    ride_id,
    COUNT(*) AS count,
FROM
    duplicate_removed 
GROUP BY
    ride_id
HAVING
    COUNT(*) > 1;
```

