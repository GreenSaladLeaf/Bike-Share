# Case Study: Bike_share  
Author: Siow Hui Hui
Date created: 22 November 2024

## Company Background
Cyclistic is a bike-share program in Chicago, launched in 2016 with a mission to provide a sustainable, accessible, and flexible transportation option for the city’s residents and visitors. The company operates a fleet of over 5,800 bikes across 692 docking stations, offering a variety of bikes including traditional bikes, reclining bikes, hand tricycles, and cargo bikes. These options make the program inclusive, catering to people with disabilities and riders who need a non-traditional bike.
Cyclistic’s growth has been driven by its flexible pricing plans, including single-ride passes, full-day passes, and annual memberships. The company’s marketing efforts have focused on building broad awareness and attracting casual riders. However, Cyclistic’s financial analysts have identified that annual members are significantly more profitable than casual riders, making it a priority for the company to find ways to convert casual riders into members for sustainable growth.

## Project Overview
This analysis aims to identify how annual members and casual riders use Cyclistic bikes differently, focusing on trip frequency, duration, bike type preferences, and the most popular routes and stations. The insights will help the marketing team develop targeted strategies to convert casual riders into annual members, driving revenue and improving customer retention.

## Key Stakeholders
The following stakeholders are critical to this analysis and will be involved in reviewing and acting upon the findings:

Lily Moreno (Director of Marketing): Lily is responsible for the development of Cyclistic’s marketing strategies. She has tasked the analytics team with providing insights into how casual riders and annual members use the bikes differently, with the goal of converting more casual riders to annual memberships.

Cyclistic Marketing Analytics Team: The team, including myself, is responsible for collecting, analyzing, and interpreting the data. We will use this analysis to support marketing strategies aimed at increasing annual memberships.

Cyclistic Executive Team: This group of senior executives will be reviewing the final recommendations to determine whether the proposed marketing strategies will be approved. Their primary focus is on maximizing the company’s profitability by increasing the number of annual memberships.

Cyclistic Finance Team: The finance team will assess the potential revenue and profitability impacts of converting casual riders to annual members. While they aren’t directly involved in the analysis, their insights may help shape the financial viability of any marketing recommendations.

or

Lily Moreno (Director of Marketing): Oversees the marketing strategy and aims to increase annual memberships by converting casual riders.

Cyclistic Marketing Analytics Team: Responsible for analyzing the data and providing insights to guide the marketing strategy.

Cyclistic Executive Team: Reviews the analysis and makes decisions on whether to approve marketing recommendations.

Cyclistic Finance Team: Assesses the financial implications of converting casual riders to annual members.


## Description of Data Sources Used
For this analysis, we used Cyclistic's historical trip data from the previous 12 months, which is publicly available and provided by Motivate International Inc. under a specific license. The dataset contains detailed information about each bike trip, including ride timestamps, trip duration, bike type, and station information.

### Key Data Columns:
ride_id: A unique identifier for each trip, ensuring each ride is distinct in the dataset.

rideable_type: The type of bike used, allowing us to compare bike preferences between annual members and casual riders.

started_at and ended_at: Timestamps marking the start and end of each trip, used to calculate trip duration and analyze peak usage times.

start_station_name and end_station_name: The names of the stations where trips began and ended, providing insights into popular stations and bike routes.

start_station_id and end_station_id: The unique station identifiers, used during data cleaning to check for inconsistencies and ensure each station ID corresponds to the correct name (e.g., identifying and correcting typos in station names).

member_casual: Indicates whether the rider is a casual rider or annual member, which is essential for segmenting the data.

start_lat, start_lng, end_lat, and end_lng: Geographic coordinates of trip start and end locations, useful for spatial analysis and understanding rider flow between stations.

## Data Limitations and Constraints
Data Privacy and Anonymization
The dataset has been anonymized to protect user privacy, meaning personally identifiable information (PII), such as names, credit card numbers, and addresses, is excluded. As a result, it is impossible to link specific bike trips to individual riders or track their behavior across multiple trips. This limitation restricts our ability to analyze certain aspects of rider behavior, such as whether casual riders are frequent users or if they live in the service area.

No Direct Demographic Information
The dataset does not include demographic information (e.g., age, gender, income) for riders. Without these details, we are unable to explore how different demographic groups might use Cyclistic bikes differently or target specific groups with personalized marketing campaigns.

Data Accuracy of Station Names and IDs
During the data cleaning process, inconsistencies were found between station names and station IDs. While this was addressed by matching station names to the correct IDs, small discrepancies may still exist in the dataset due to errors in the original data, especially when stations undergo renaming or reorganization. Although we corrected for obvious errors, minor inconsistencies may still impact analyses involving station names.

No Information on Rider Motivation
The dataset does not provide information about why riders choose to use Cyclistic bikes (e.g., for commuting, exercise, or leisure). Without this context, we are unable to directly address the question of why casual riders might purchase a membership or what factors influence their decision to switch from casual to annual membership.

Geographic Limitations
The geographic data, including start_lat, start_lng, end_lat, and end_lng, is useful for mapping trips and analyzing usage patterns. However, these coordinates are limited to the area covered by Cyclistic’s service area. We are not able to assess riders’ behavior outside of this area, and geographic trends are confined to Chicago.

Potential Data Gaps or Missing Entries
While we performed basic data cleaning, the dataset may still contain missing values or gaps. For example, trips with missing timestamps, station names, or coordinates could affect the completeness of the analysis. Although filtering out incomplete records reduces these gaps, any missing or inconsistent data might still slightly skew some results.

Limited Time Frame
The dataset covers only the past 12 months, meaning any insights gained from this time period may not be fully representative of long-term trends or seasonal variations. For example, if there were unusual weather conditions, special events, or other external factors affecting bike usage during this specific period, these anomalies may not reflect typical bike usage patterns.

No Payment Data
The dataset does not include information on the payment type or pricing plan (e.g., single rides vs. annual memberships), except for the member_casual indicator. As a result, we cannot investigate the impact of pricing models or specific promotions on the conversion of casual riders into annual members.

Summary:
These limitations are important to consider when interpreting the findings. While the dataset provides valuable insights into trip behaviors, the absence of certain contextual and demographic information, the anonymization of user data, and other data quality concerns may limit the depth of analysis we can perform in certain areas. Despite these constraints, the analysis still provides valuable insights into the general trends in how casual riders and annual members use Cyclistic bikes.
