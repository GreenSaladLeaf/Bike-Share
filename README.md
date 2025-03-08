# Project Background
Cyclistic is a bike-share program in Chicago, offering a fleet of over 5,800 bikes across 692 stations. Since its launch in 2016, Cyclistic has grown rapidly, providing a range of bike options, including traditional bikes, reclining bikes, hand tricycles, and cargo bikes, to meet the diverse needs of its riders. Approximately 30% of Cyclistic’s users commute to work, while others use the bikes for leisure.

Cyclistic offers three pricing plans: single-ride passes, full-day passes, and annual memberships. While annual memberships are the most profitable for the company, casual riders—those using single-ride or full-day passes—make up a large portion of the customer base. Cyclistic’s marketing strategy has traditionally focused on broad awareness and attracting casual riders. However, the company’s financial analysts have found that annual members are significantly more profitable than casual riders. As a result, Cyclistic is shifting its focus to increasing the number of annual memberships, believing that converting casual riders into members will be key to achieving long-term, sustainable growth.

Insights and recommendations are provided on the following key areas:
- Hourly, weekly and monthly trends analysis:  Evaluation of trip frequency patterns by rider type across different timeframes (hourly, weekly, and monthly)
- Bike type preference: Analysis of bike type preferences among casual riders and members, including insights on trip duration and distance
- Station comparisons: Comparison of station popularity, identifying high-traffic locations and seasonal variations.

An interactive dashboard can be found here

A comprehensive report can be found here

The SQL queries to clean, organise and prepare data for the dashboard can be found here

# Data Structure & Initial Checks
The dataset is structured as a series of trip-level records, with each record representing a single bike trip. The columns across all 12 CSV files are consistent and include:

- **ride_id**: A unique identifier for each trip, ensuring each ride is distinct in the dataset.

- **rideable_type**: The type of bike used, allowing for comparison of bike preferences between annual members and casual riders.

- **started_at** and **ended_at**: Timestamps marking the start and end of each trip, useful for calculating trip duration and analyzing peak usage times.

- **start_station_name** and **end_station_name**: The names of the stations where trips began and ended, providing insights into popular stations and bike routes.

- **start_station_id** and **end_station_id**: The unique station identifiers, used during data cleaning to check for inconsistencies and ensure correct matching between station IDs and names (e.g., identifying and correcting typos).

- **member_casual**: Indicates whether the rider is a casual rider or annual member, which is crucial for segmenting the data.

- **start_lat, start_lng, end_lat**, and **end_lng**: Geographic coordinates of trip start and end locations, useful for spatial analysis and understanding rider flow between stations.

The files are structured to facilitate merging into a single dataset without losing important details.

# Executive Summary
**1. Trip Trends by Time of Day and Season**:
  - Both casual riders and members have a peak usage time of **5 PM**, aligning with commute and leisure hours.
  - Members most active in **August**, while casual riders see their highest activity in **July**, both during the warmer months.
  - **Casual riders** predominantly take trips on **weekends** (with **Saturday** being the highest), whereas **members** take more trips on **weekdays** (with **Wednesday** being the highest), indicating a strong commuting pattern among members.

**2. Bike Type Preferences & Trip Characteristics**:
  - **casual riders** prefer **electric bikes (52.2%) over classic bikes (46.4%)**, while **members** show a slight preference for **classic bikes (51.6%) over electric bikes (48.4%)**.
  - **Casual riders** take longer trips on average (**10 mins for electric bikes, 14 mins for classic bikes**), while **members' median trip durations** are shorter (**8 mins for electric, 9 mins for classic**).
  - **Docked bikes** are used exclusively by **casual riders (1.45% of casual trips)**, indicating a strong preference for flexible, dockless options.

**3. Top Stations & Routes**:
  - Among the top 10 stations, the following stand out, with **over 55% of trips made by casual riders**:
    - **Streeter Dr & Grand Ave (74.4% casual riders)**
    - **Dusable Lake Shore Dr & Monroe St (70.7% casual riders)**
    - **Michigan Ave & Oat St (59.3% casual riders)**
    - **Dusable Lake Shore Dr & North Blvd (57.6% casual riders)**
These stations, primarily located in **downtown and lakefront areas**, reinforce the idea that **casual riders are largely tourists or leisure users**.
  - These stations experience **higher casual rider traffic in summer, while member traffic remains steady**.

Recommendations for Increasing Membership Conversions:
Target Casual Riders During Peak Season:
Promote membership heavily in summer months when casual ridership spikes.
Offer limited-time discounts or loyalty rewards to frequent casual riders to encourage membership sign-ups.
Leverage Station Data for Marketing:
Place membership promotions at high-traffic casual rider stations, particularly near tourist-heavy locations.
Add in-app membership reminders when casual riders frequently use the service.
Enhance Pricing & Experience for Casual Riders:
Introduce a short-term membership option (e.g., 1-month pass) to ease the transition from casual to full membership.
Improve bike availability at peak casual rider stations to ensure a positive experience that increases repeat usage.
