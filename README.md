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
The dataset, consisting of 12 months of trip data in separate CSV files, was uploaded to a Google Cloud Storage bucket. A consolidated SQL table was then created by combining all 12 months of data, ensuring consistency across the files, as they shared the same column structure.

### Data Cleaning Steps
The following steps were taken to ensure the data was clean and suitable for analysis:

- **Step 1**: Review the Data
The first step involved reviewing the dataset to understand its structure and assess the quality of the data. This was done by performing an initial query to inspect a sample of rows. The goal was to identify any obvious issues such as missing values, formatting inconsistencies, or incorrect data types.

