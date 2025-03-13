# Case Study: Bike_share 
**Author**: Siow Hui Hui

**Date created**: 22 November 2024

## Project Background
Cyclistic is a bike-share program in Chicago, offering a fleet of over 5,800 bikes across 692 stations. Since its launch in 2016, Cyclistic has grown rapidly, providing a range of bike options, including traditional bikes, reclining bikes, hand tricycles, and cargo bikes, to meet the diverse needs of its riders. Approximately 30% of Cyclistic’s users commute to work, while others use the bikes for leisure.

Cyclistic offers three pricing plans: single-ride passes, full-day passes, and annual memberships. While annual memberships are the most profitable for the company, casual riders—those using single-ride or full-day passes—make up a large portion of the customer base. Cyclistic’s marketing strategy has traditionally focused on broad awareness and attracting casual riders. However, the company’s financial analysts have found that annual members are significantly more profitable than casual riders. As a result, Cyclistic is shifting its focus to increasing the number of annual memberships, believing that converting casual riders into members will be key to achieving long-term, sustainable growth.

Insights and recommendations are provided on the following key areas:
- Hourly, weekly and monthly trends analysis:  Evaluation of trip frequency patterns by rider type across different timeframes (hourly, weekly, and monthly)
- Bike type preference: Analysis of bike type preferences among casual riders and members, including insights on trip duration and distance
- Station comparisons: Comparison of station popularity, identifying high-traffic locations and seasonal variations.

An interactive dashboard can be found [here](https://public.tableau.com/app/profile/rae.siow/viz/bike_share_17343442737480/Dashboard2)

A comprehensive report with detailed SQL queries to clean, organise and prepare data for the dashboard can be found [here](https://github.com/GreenSaladLeaf/bike_share/blob/main/README_comprehensive.md))

## Data Structure & Initial Checks
The dataset is structured as a series of trip-level records, with each record representing a single bike trip. The columns across all 12 CSV files are consistent and include:

- **ride_id**: A unique identifier for each trip, ensuring each ride is distinct in the dataset.

- **rideable_type**: The type of bike used, allowing for comparison of bike preferences between annual members and casual riders.

- **started_at** and **ended_at**: Timestamps marking the start and end of each trip, useful for calculating trip duration and analyzing peak usage times.

- **start_station_name** and **end_station_name**: The names of the stations where trips began and ended, providing insights into popular stations and bike routes.

- **start_station_id** and **end_station_id**: The unique station identifiers, used during data cleaning to check for inconsistencies and ensure correct matching between station IDs and names (e.g., identifying and correcting typos).

- **member_casual**: Indicates whether the rider is a casual rider or annual member, which is crucial for segmenting the data.

- **start_lat, start_lng, end_lat**, and **end_lng**: Geographic coordinates of trip start and end locations, useful for spatial analysis and understanding rider flow between stations.

The files are structured to facilitate merging into a single dataset without losing important details.

## Executive Summary
**1. Trip Trends by Time of Day and Season**:
  - Both casual riders and members have a peak usage time of **5 PM**, aligning with commute and leisure hours.
  - Members most active in **August**, while casual riders see their highest activity in **July**, both during the warmer months.
  - **Casual riders** predominantly take trips on **weekends** (with **Saturday** being the highest), whereas **members** take more trips on **weekdays** (with **Wednesday** being the highest), indicating a strong commuting pattern among members.

![image](https://github.com/user-attachments/assets/7468a3fd-ba0e-47f7-8857-36f4b47d9239)

![image](https://github.com/user-attachments/assets/d45b51fa-c2eb-4cae-b462-40833c105b68)

![image](https://github.com/user-attachments/assets/9cc4646c-5476-46c8-bdff-5a497aef7295)

**2. Bike Type Preferences & Trip Characteristics**:
  - **Casual riders** prefer **electric bikes (52.2%) over classic bikes (46.4%)**, while **members** show a slight preference for **classic bikes (51.6%) over electric bikes (48.4%)**.
  - **Casual riders** take longer trips on average (**10 mins for electric bikes, 14 mins for classic bikes**), while **members' median trip durations** are shorter (**8 mins for electric, 9 mins for classic**).
  - **Docked bikes** are used exclusively by **casual riders (1.45% of casual trips)**, indicating a strong preference for flexible, dockless options.

![image](https://github.com/user-attachments/assets/7c6979c7-5005-4c1c-aab2-7e3fde1b163b)

![image](https://github.com/user-attachments/assets/d1b30bb5-99c4-4467-9951-19af683a56e8)

**3. Popular Stations**:
  - Among the top 10 stations, the following stand out, with **over 55% of trips made by casual riders**:
    - **Streeter Dr & Grand Ave (74.4% casual riders)**
    - **Dusable Lake Shore Dr & Monroe St (70.7% casual riders)**
    - **Michigan Ave & Oat St (59.3% casual riders)**
    - **Dusable Lake Shore Dr & North Blvd (57.6% casual riders)**
These stations, primarily located in **downtown and lakefront areas**, reinforce the idea that **casual riders are largely tourists or leisure users**.
  - These stations experience **higher casual rider traffic in summer, while member traffic remains steady**.

![image](https://github.com/user-attachments/assets/5e0c2f1e-8b57-4ac5-8e17-fda82d353944)

![image](https://github.com/user-attachments/assets/cc4d8c51-91ac-4e91-b5e6-f2bd04c80b75)

## Recommendations for Increasing Membership Conversions:
#### 1. Time-Based Membership Promotions: Align with Peak Usage Patterns
Casual riders show distinct time-based usage patterns:
- **Peak Hour: 5 PM**, indicating high evening commuter and tourist activity.
- **Peak Days: Weekends**, suggesting leisure-based ridership.
- **Peak Month: July**, when casual ridership is at its highest.

Based on these patterns, **targeted membership campaigns** could be more effective if launched during these peak periods.

Potential strategies include:
- **Evening commuter membership promotions** during the 5 PM peak to encourage frequent riders to subscribe.
- **Weekend-based trial memberships** to convert leisure riders into long-term users.
- **Seasonal summer campaigns** when casual ridership spikes, reinforcing the value of membership for repeat use.

#### 2. Bike Type Preferences: Casual Riders Favor Electric Bikes
Analyzing bike type usage reveals:
- **Electric bikes are slightly more popular among casual riders (52.2%)**, compared to members.
- **Classic bike trips tend to be longer in duration**, especially for casual users.

To leverage these insights:
- Membership incentives could highlight **priority access to electric bikes**, addressing casual riders’ preferences.
- **Pricing adjustments or promotions on longer bike trips** may encourage casual users to convert to members.

#### 3. Location-Based Membership Targeting: Focus on High-Casual-Rider Stations
Casual riders make up a significant share of trips at key downtown and lakefront stations. The following locations have the highest percentage of casual riders:
- **Streeter Dr & Grand Ave (74.4% casual riders)**
- **Dusable Lake Shore Dr & Monroe St (70.7% casual riders)**
- **Michigan Ave & Oak St (59.3% casual riders)**
- **Dusable Lake Shore Dr & North Blvd (57.6% casual riders)**

Given these trends, **targeted membership promotions** at these stations could be effective in converting casual riders. Potential strategies Cyclistic could explore include:
- **In-app membership promotions** triggered when riders start trips at these locations.
- **Partnership discounts with nearby hotels or attractions** to encourage tourists to sign up for memberships.
- **On-site signage and QR codes** at high-casual-traffic stations to promote the benefits of membership.
