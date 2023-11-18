##Introduction
The way people experience cities around the world has been completely transformed by Airbnb in the fields of contemporary travel and hospitality. Airbnb provides a variety of lodging options, ranging from comfortable private rooms to roomy entire apartments, as an online marketplace that connects hosts with travelers. The European Booking Dataset, a thorough compilation of information from nine renowned cities—Amsterdam, Athens, Barcelona, Berlin, Budapest, Lisbon, Paris, Rome, and Vienna—is the subject of this project. The dataset has undergone meticulous curation and cleaning, making it a useful tool for analysis and insight.

This project's main goals are to perform exploratory data analysis (EDA), identify outliers in the dataset, and determine any possible causal links between outliers and guest satisfaction. In order to identify patterns, trends, and elements that affect visitors' overall experiences, we will combine data from various cities and Airbnb stays. We use SQL queries within the Snowflake environment to accomplish this, allowing us to quickly process and analyze large amounts of data.

##Dataset Overview
The dataset comprises several key variables that shed light on different dimensions of Airbnb stays. These variables include:

City: The name of the city in which the Airbnb stay is located.
Price: The price of the Airbnb stay.
Day: Indicates whether the stay falls on a weekday or weekend.
Room Type: The type of Airbnb accommodation, such as an entire apartment, private room, or shared room.
Shared Room: Indicates whether the room is shared by multiple guests.
Private Room: Indicates the availability of a private room within the accommodation.
Person Capacity: The maximum number of individuals the accommodation can host.
Superhost: A binary indicator of whether the host is classified as a superhost.
Multiple Rooms: Indicates if the Airbnb offers multiple rooms (2-4 rooms).
Business: A binary indicator of whether the accommodation offers more than four listings, potentially suggesting a business-oriented host.
Cleaningness Rating: A rating reflecting the cleanliness of the place, as provided by guests.
Guest Satisfaction: The satisfaction score left by guests after their stay.
Bedrooms: The number of bedrooms available in the facility.
City Center (km): The distance from the accommodation to the city center.
Metro Distance (km): The distance from the accommodation to the nearest metro service.
Attraction Index: An index measuring the proximity of attractions to the accommodation.
Normalized Attraction Index: A normalized version of the attraction index.
Restaurant Index: An index measuring the proximity of restaurants to the accommodation.
Normalized Restaurant Index: A normalized version of the restaurant index.

##Project Goals
Exploratory Data Analysis (EDA): Before delving into outlier detection and causal relationship establishment, it's crucial to understand the data. Through visualization and statistical analysis, we'll uncover insights such as price distributions, room type preferences, and location-based trends.

Outlier Detection: Identifying outliers can provide valuable insights into unusual occurrences or exceptional cases within the dataset. By applying appropriate techniques, we can pinpoint instances that deviate significantly from the norm, potentially shedding light on hidden factors affecting guest satisfaction.

Causal Relationship with Guest Satisfaction: Establishing causal relationships involves examining factors that might influence guest satisfaction scores. Through rigorous analysis, we'll explore correlations and potentially infer causal links between variables such as cleanliness ratings, distance to city center, and room types.

##Task:
1.Perform exploratory data analysis using Snowflake SQL and tell a story you'd like to tell with this dataset.
2.Create a Database in Snowflake named "TOURISM"
3.Create a Schema under that Database named "EUROPE"
4.Create a Table under that Database and Schema named "AIRBNB" [Be careful with the Field Names and Field Types]
5.Create an Appropriate File Format to Bulk insert the CSV file (Airbnb Europe Dataset.csv)
6.Insert the dataset using the created file format [K]
7.If you see any error, then work on that. You must try until you see no error.
8.Proof Check: The number of Records should be 41,714
9.Perform Descriptive Analysis and Frequency Distribution [Build Charts as well in your result if you find the output important enough]
10.Check if there is any outlier in the "PRICE" Field; If so, then report the observations and then remove the outliers
11.Apart from the descriptive analysis, try to Explore which Fields/Variables have a Causal Relationship with "Guest Satisfaction".
12.Build a Simple Dashboard using Snowflake [Keep one or two filters on top; whatever field you find most important ones to look at]
13.Write a Short Report based on your findings [Report should be written in Microsoft Word File and the Analyses, Charts should be included in the report; Add a Screenshot of your dashboard as well]
