# Case Study: Bike_share  
Author: Siow Hui Hui
Date created: 22 November 2024

## Company Background
Cyclistic is a bike-share program in Chicago, offering a fleet of over 5,800 bikes across 692 stations. Since its launch in 2016, Cyclistic has grown rapidly, providing a range of bike options, including traditional bikes, reclining bikes, hand tricycles, and cargo bikes, to meet the diverse needs of its riders. Approximately 30% of Cyclistic’s users commute to work, while others use the bikes for leisure.

Cyclistic offers three pricing plans: single-ride passes, full-day passes, and annual memberships. While annual memberships are the most profitable for the company, casual riders—those using single-ride or full-day passes—make up a large portion of the customer base. Cyclistic’s marketing strategy has traditionally focused on broad awareness and attracting casual riders. However, the company’s financial analysts have found that annual members are significantly more profitable than casual riders. As a result, Cyclistic is shifting its focus to increasing the number of annual memberships, believing that converting casual riders into members will be key to achieving long-term, sustainable growth.

## Business Task Statement
This analysis will identify key differences in how annual members and casual riders use Cyclistic bikes, focusing on trip frequency, duration, bike type preferences, and popular routes and stations. The insights will help the marketing team develop targeted strategies to convert casual riders into annual members, boosting revenue and improving customer retention.

## Key Stakeholders
The following stakeholders are critical to this analysis and will be involved in reviewing and acting upon the findings:

Lily Moreno (Director of Marketing): Lily is responsible for the development of Cyclistic’s marketing strategies. She has tasked the analytics team with providing insights into how casual riders and annual members use the bikes differently, with the goal of converting more casual riders to annual memberships.

Cyclistic Marketing Analytics Team: The team, including myself, is responsible for collecting, analyzing, and interpreting the data. We will use this analysis to support marketing strategies aimed at increasing annual memberships.

Cyclistic Executive Team: This group of senior executives will be reviewing the final recommendations to determine whether the proposed marketing strategies will be approved. Their primary focus is on maximizing the company’s profitability by increasing the number of annual memberships.

## Description of Data Sources Used
For this analysis, Cyclistic's publicly available historical trip data from July 2023 to June 2024 was used. The data was provided by Motivate International Inc. under a specific license. It includes detailed information about each bike trip, such as ride timestamps, user type, bike type, and station information. This dataset allows for an in-depth analysis of bike usage patterns, including trip duration, bike preferences, station usage, and geographic trends.

### Key Data Columns:
The dataset is structured as a series of trip-level records, with each record representing a single bike trip. The columns across all 12 CSV files are consistent and include:

ride_id: A unique identifier for each trip, ensuring each ride is distinct in the dataset.

rideable_type: The type of bike used, allowing for comparison of bike preferences between annual members and casual riders.

started_at and ended_at: Timestamps marking the start and end of each trip, useful for calculating trip duration and analyzing peak usage times.

start_station_name and end_station_name: The names of the stations where trips began and ended, providing insights into popular stations and bike routes.

start_station_id and end_station_id: The unique station identifiers, used during data cleaning to check for inconsistencies and ensure correct matching between station IDs and names (e.g., identifying and correcting typos).

member_casual: Indicates whether the rider is a casual rider or annual member, which is crucial for segmenting the data.

start_lat, start_lng, end_lat, and end_lng: Geographic coordinates of trip start and end locations, useful for spatial analysis and understanding rider flow between stations.

The files are structured to facilitate merging into a single dataset without losing important details.

## Data Quality
### Null Station Names and IDs
The dataset contains null values in station names and station IDs, likely due to dockless rides, where bikes are picked up and dropped off at locations not tied to fixed stations. While this reflects the flexibility of the system, it limits station-level analysis, such as identifying popular stations or routes.

### Duplicate or Inconsistent Station Data
Some station IDs are associated with multiple names due to factors such as typos, station renaming, relocations, or cases where different locations share the same ID. While these inconsistencies are relatively minor, they do not significantly affect the overall analysis of station popularity and rider patterns. These issues were addressed through data cleaning procedures to ensure the integrity of the results.

### Missing End Latitude and Longitude
Around 0.1% of the trips in the dataset have missing end_lat or end_lng values. These trips also have null values for end_station_name and end_station_id. While this is a small fraction of the data, it does represent rides where the destination location is not available. These records were excluded from the analysis to ensure that only trips with complete geographic data were included, preserving the integrity of spatial analyses.

## Bias or credibility 
### Bias: 
The dataset includes data on both casual riders and annual members, so it is representative of users of the Cyclistic bike-share program. However, bias could be introduced if, for example, certain types of users (e.g., seasonal riders or specific demographic groups) are underrepresented. This data does not include demographic details (such as age, income, or geography), which could limit some aspects of analysis related to these groups.

#### Lack of Demographic Information
The dataset does not include demographic data (e.g., age, gender, or income). Without this information, we are unable to analyze how different demographic segments use Cyclistic bikes or to tailor marketing strategies based on specific rider characteristics.

#### No Context on Rider Motivation
The dataset does not include any information about why riders use Cyclistic bikes (e.g., commuting, exercise, leisure). This lack of rider motivation data makes it difficult to assess how motivations may influence a rider's decision to purchase an annual membership.

### Credibility: 
The data comes directly from Cyclistic’s bike-share program, making it credible for the purpose of analyzing rider behavior. The data is comprehensive in terms of trip details but lacks some demographic or motivational context.

### licensing, privacy, security, and accessibility
Licensing: The dataset is publicly available, so there are no restrictions on its use for analysis.

Privacy: The dataset has been anonymized to protect personally identifiable information (PII), such as names, credit card numbers, and addresses. As a result, it is impossible to link specific bike trips to individual riders or track their behavior across multiple trips. 

Security: The data is stored in a Google Cloud Storage bucket, which provides secure, encrypted storage to protect the data from unauthorized access.

Accessibility: The data is easily accessible within the Google Cloud Storage bucket, and proper permissions are set to allow for authorized access for analysis.

## Data Limitations 
### No Payment Data
The dataset does not include information on the payment type or pricing plan (e.g., single rides vs. annual memberships), except for the member_casual indicator. As a result, we cannot investigate the impact of pricing models or specific promotions on the conversion of casual riders into annual members.

### Geographic and Timeframe Limitations
The dataset includes 12 months of trip data from Cyclistic’s service area in Chicago, from July 2023 to June 2024. As a result, the analysis is limited to trends and patterns within this specific geographic area and time frame. This may not fully capture seasonal, long-term, or external factors (such as unusual events or weather conditions) that could affect bike usage beyond this period.



OR this?:

## Data Limitations and Constraints
### Data Privacy and Anonymization
The dataset has been anonymized to protect personally identifiable information (PII), such as names, credit card numbers, and addresses. As a result, it is impossible to link specific bike trips to individual riders or track their behavior across multiple trips. This limitation prevents us from analyzing certain aspects of rider behavior, such as identifying whether casual riders are frequent users or whether they live within the service area.

### Lack of Demographic Information
The dataset does not include demographic data (e.g., age, gender, or income). Without this information, we are unable to analyze how different demographic segments use Cyclistic bikes or to tailor marketing strategies based on specific rider characteristics.

### Null Station Names and IDs
The dataset contains null values in station names and station IDs, likely due to dockless rides, where bikes are picked up and dropped off at locations not tied to fixed stations. While this reflects the flexibility of the system, it limits station-level analysis, such as identifying popular stations or routes.

### Duplicate or Inconsistent Station Data
Some station IDs are associated with multiple names due to factors such as typos, station renaming, relocations, or cases where different locations share the same ID. While these inconsistencies are relatively minor, they do not significantly affect the overall analysis of station popularity and rider patterns. These issues were addressed through data cleaning procedures to ensure the integrity of the results.

### Missing End Latitude and Longitude
Around 0.1% of the trips in the dataset have missing end_lat or end_lng values. These trips also have null values for end_station_name and end_station_id. While this is a small fraction of the data, it does represent rides where the destination location is not available. These records were excluded from the analysis to ensure that only trips with complete geographic data were included, preserving the integrity of spatial analyses.

### No Context on Rider Motivation
The dataset does not include any information about why riders use Cyclistic bikes (e.g., commuting, exercise, leisure). This lack of rider motivation data makes it difficult to assess how motivations may influence a rider's decision to purchase an annual membership.

### No Payment Data
The dataset does not include information on the payment type or pricing plan (e.g., single rides vs. annual memberships), except for the member_casual indicator. As a result, we cannot investigate the impact of pricing models or specific promotions on the conversion of casual riders into annual members.

### Geographic and Timeframe Limitations
The dataset includes 12 months of trip data from Cyclistic’s service area in Chicago, from July 2023 to June 2024. As a result, the analysis is limited to trends and patterns within this specific geographic area and time frame. This may not fully capture seasonal, long-term, or external factors (such as unusual events or weather conditions) that could affect bike usage beyond this period.

## Data Limitations and Constraints

### Data Quality Issues
- **Null Station Names and IDs**: Some trips have missing station data due to dockless bike usage. This limits station-level analysis.
- **Duplicate or Inconsistent Station Data**: Inconsistencies in station names/IDs were addressed through data cleaning.
- **Missing End Latitude and Longitude**: About 0.1% of trips have missing destination data, and these were excluded from spatial analyses.

### Bias in the Data
- **Lack of Demographic Information**: The dataset does not contain demographic information (e.g., age, gender, income), which limits analysis on how specific groups use Cyclistic bikes and could skew marketing strategies.
- **No Context on Rider Motivation**: Without data on why people use the bikes (e.g., commuting vs. leisure), it’s difficult to analyze how motivations might influence membership conversions.

### Data Scope Limitations
- **No Payment Data**: The dataset lacks information on payment types or pricing models, preventing an analysis of how pricing impacts conversion from casual riders to members.
- **Geographic and Timeframe Limitations**: The data is limited to Cyclistic’s service area in Chicago from July 2023 to June 2024, which may not fully capture seasonal trends or external factors that affect bike usage.

### Licensing, Privacy, and Security
- **Privacy and Security**: The data is anonymized to protect PII, ensuring no individual rider can be identified. It is stored securely in Google Cloud Storage and is accessible to authorized users for analysis.

## Data Cleaning and Manipulation
### Tools Used
The dataset was cleaned and processed using SQL for efficient handling of large datasets and complex queries. Tableau was used for visualizing the results and gaining insights from the cleaned data.

### Handling Multiple Large CSV Files 
The dataset, consisting of 12 months of trip data in separate CSV files, was uploaded to a Google Cloud Storage bucket. A consolidated SQL table was then created by combining all 12 months of data, ensuring consistency across the files, as they shared the same column structure.
### ROCCC:
Reliable: The data comes directly from Cyclistic’s system, which is a trusted source.

Original: The data is original, containing raw trip-level records from the bike-share system.

Comprehensive: The dataset is comprehensive in terms of trip-level data, but lacks demographic and motivational details.

Current: The dataset covers 12 months of recent data, providing insights into current usage patterns.

Cited: The dataset is sourced from Cyclistic’s bike-share program, and its origin is noted in the documentation.

### licensing, privacy, security, and accessibility?
Licensing: The dataset is publicly available, so there are no restrictions on its use for analysis.

Privacy: The dataset has been anonymized to remove personally identifiable information (PII). This ensures that no personal details about riders are exposed, preserving privacy.

Security: The data is stored in a Google Cloud Storage bucket, which provides secure, encrypted storage to protect the data from unauthorized access.

Accessibility: The data is easily accessible within the Google Cloud Storage bucket, and proper permissions are set to allow for authorized access for analysis.
### 

