CREATE DATABASE TOURISM;
USE DATABASE TOURISM;
CREATE SCHEMA EUROPE;
USE SCHEMA EUROPE;
--Table & File Format Along with Copy Command
CREATE TABLE AIRBNB (
CITY VARCHAR (20),
PRICE NUMBER (10,4),
DAY VARCHAR (8),
ROOM_TYPE VARCHAR (20),
SHARED_ROOM BOOLEAN,
PRIVATE_ROOM BOOLEAN,
PERSON_CAPACITY NUMBER (3,0),
SUPERHOST BOOLEAN,
MULTIPLE_ROOMS NUMBER (1,0),
BUSINESS NUMBER (1,0),
CLEANINGNESS_RATING NUMBER (10,4),
GUEST_SATISFACTION NUMBER (3,0),
BEDROOMS NUMBER (3,0),
CITY_CENTER_KM NUMBER (8,4),
METRO_DISTANCE_KM NUMBER (8,4),
ATTRACTION_INDEX NUMBER (8,4),
NORMALSED_ATTACTION_INDEX NUMBER (8,4),
RESTAURANT_INDEX NUMBER (8,4),
NORMALISED_RESTAURANT_INDEX NUMBER (8,4)
);
SELECT * FROM AIRBNB;


-- CREATE BULK INSERT CSV FORMAT
--field optionally enclosed for double quatation in the records ""
create or replace file format csv_format
type = 'csv'
compression = 'none'
field_delimiter = ','
field_optionally_enclosed_by = '\042'
skip_header = 1;

--TRUNCATE AIRBNB;

/*Exploratory Data Analysis (EDA) */
--SNIPPET/SNAPSHOT OF THE DATASET
SELECT * FROM AIRBNB LIMIT 5;
--NUMBER OF RECORDS IN THE DATASET
SELECT COUNT(*) AS "NUMBER OF RECORDS" FROM AIRBNB;--41,714

--NUMBER OF UNIQUE/DISTINCT CITY IN THE EUROPEAN DATASET
SELECT COUNT(DISTINCT CITY)
FROM AIRBNB;
SELECT COUNT(DISTINCT CITY) AS "NUMBER OF UNIQUE CITY"
FROM AIRBNB;--9
--LISTING OF THE CITY NAMES
SELECT DISTINCT CITY AS "CITY NAMES"
FROM AIRBNB;
--NUMBER OF BOOKINGS IN EACH CITY
SELECT CITY, COUNT(CITY) AS "NUMBER OF BOOKINGS"
FROM AIRBNB
GROUP BY CITY
ORDER BY 2 DESC;

--
--Total Booking Revenue by City
SELECT CITY, ROUND(SUM(PRICE),0) AS "TOTAL BOOKING REVENUE"
FROM AIRBNB
GROUP BY CITY
ORDER BY 2 DESC;

https://github.com/RukaiyaSrabony/sql/assets/119132009/9ffde2a9-5ba9-467f-bce7-4dc1b3ef43d6

--Combining the two result
SELECT CITY, COUNT(CITY) AS "NUMBER OF BOOKINGS", ROUND(SUM(PRICE),0) AS
"TOTAL BOOKING REVENUE"
FROM AIRBNB
GROUP BY CITY
ORDER BY "TOTAL BOOKING REVENUE" DESC;

SELECT DISTINCT ROOM_TYPE
FROM AIRBNB;
--TRYING TO MAKE A CAUSAL RELATIONSHIP
SELECT CITY, COUNT(CITY) AS "NUMBER OF BOOKINGS", ROUND(SUM(PRICE),0) AS
"TOTAL BOOKING REVENUE", ROUND(AVG(GUEST_SATISFACTION),1) AS "AVERAGE
GUEST SATISFACTION SCORE"
FROM AIRBNB
--WHERE DAY ilike '%end' AND ROOM_TYPE ilike 'pri%'
GROUP BY CITY
ORDER BY 4 DESC;

https://github.com/RukaiyaSrabony/sql/assets/119132009/7cc7d898-8031-4722-ae23-6f9e4b9484ee

--NO GROUPING: SUMMARY RESULT (AGGREGATION)
SELECT
MIN(PRICE) AS "MINIMUM BOOKING PRICE",
ROUND(SUM(PRICE),0) AS "TOTAL BOOKING PRICE",
ROUND(AVG(PRICE),1) AS "AVERAGE BOOKING PRICE",
ROUND(STDDEV_POP (PRICE),1) AS STANDARD_DEVIATION_OF_PRICE,
ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY PRICE),1) AS Q1,
ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY PRICE),1) AS MEDIAN,
ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE),1) AS Q3,
ROUND(MAX(PRICE),1) AS MAXIMUM_BOOKING_PRICE
FROM AIRBNB;

https://github.com/RukaiyaSrabony/sql/assets/119132009/7e47ed48-a3d1-4d16-9804-f9760635f5f8
https://github.com/RukaiyaSrabony/sql/assets/119132009/45a8e121-6ac9-46e5-ba29-4c92bced72ff

--select * from airbnb where price = 18545.4503;
--IDENTIFYING OUTLIERS
--Q1-1.5*IQR [Lower Hinge]
--Q3+1.5*IQR [Upper Hinge]
WITH FIVE_NUMBER_SUMMARY AS
(SELECT
MIN(PRICE) AS MIN_ORDER_VALUE,
PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY PRICE) AS Q1,
PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY PRICE) AS MEDIAN,
PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE) AS Q3,
MAX(PRICE) AS MAX_ORDER_VALUE,
(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE)-PERCENTILE_CONT(0.25)
WITHIN GROUP (ORDER BY PRICE)) AS IQR
FROM AIRBNB),
HINGES AS
(SELECT (Q1-1.5*IQR) AS LOWER_HINGE, (Q3+1.5*IQR) AS UPPER_HINGE
FROM FIVE_NUMBER_SUMMARY)
SELECT
COUNT (*) AS "NUMBER OF OUTLIERS IN PRICE FIELD"
FROM AIRBNB
WHERE PRICE < (SELECT LOWER_HINGE FROM HINGES) OR PRICE > (SELECT
UPPER_HINGE FROM HINGES);--2,891
--Trying to Find Patterns in Outliers

CREATE OR REPLACE VIEW OUTLIER AS
(
WITH FIVE_NUMBER_SUMMARY AS
(SELECT
MIN(PRICE) AS MIN_ORDER_VALUE,
PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY PRICE) AS Q1,
PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY PRICE) AS MEDIAN,
PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE) AS Q3,
MAX(PRICE) AS MAX_ORDER_VALUE,
(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE)-PERCENTILE_CONT(0.25)
WITHIN GROUP (ORDER BY PRICE)) AS IQR
FROM AIRBNB),
HINGES AS
(SELECT (Q1-1.5*IQR) AS LOWER_HINGE, (Q3+1.5*IQR) AS UPPER_HINGE
FROM FIVE_NUMBER_SUMMARY)
SELECT
*
FROM AIRBNB
WHERE PRICE < (SELECT LOWER_HINGE FROM HINGES) OR PRICE > (SELECT
UPPER_HINGE FROM HINGES)
);

SELECT * FROM OUTLIER;
SELECT
ROOM_TYPE AS "ROOM TYPE",
COUNT(*) AS "NO. OF Bookings",
ROUND(MIN(PRICE),1) AS "MINIMUM OUTLIER PRICE VALUE",
ROUND(MAX(PRICE),1) AS "MAXIMUM OUTLIER PRICE VALUE",
ROUND(AVG(PRICE),1) AS "AVERAGE OUTLIER PRICE VALUE"
FROM OUTLIER
GROUP BY ROOM_TYPE;

https://github.com/RukaiyaSrabony/sql/assets/119132009/7e47ed48-a3d1-4d16-9804-f9760635f5f8
https://github.com/RukaiyaSrabony/sql/assets/119132009/45a8e121-6ac9-46e5-ba29-4c92bced72ff
https://github.com/RukaiyaSrabony/sql/assets/119132009/43691bed-6408-4467-bbc0-40140aa83c64

--COMPARE THIS WITH THE MAIN DATA
SELECT
ROOM_TYPE AS "ROOM TYPE",
COUNT(*) AS "NO. OF Bookings",
ROUND(MIN(PRICE),1) AS "MINIMUM PRICE VALUE",
ROUND(MAX(PRICE),1) AS "MAXIMUM PRICE VALUE",
ROUND(AVG(PRICE),1) AS "AVERAGE PRICE VALUE"
FROM AIRBNB
GROUP BY ROOM_TYPE;--NOTICE THE DIFFERENCE IN THE AVERAGE OUTLIER PRICEVALUE Vs. AVERAGE PRICE VALUE
--REMOVING OUTLIERS & STORING THE CLEANED DATA IN ANOTHER VIEW

CREATE VIEW CLEANED AS
(
WITH FIVE_NUMBER_SUMMARY AS
(SELECT
MIN(PRICE) AS MIN_ORDER_VALUE,
PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY PRICE) AS Q1,
PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY PRICE) AS MEDIAN,
PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE) AS Q3,
MAX(PRICE) AS MAX_ORDER_VALUE,

(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY PRICE)-PERCENTILE_CONT(0.25)
WITHIN GROUP (ORDER BY PRICE)) AS IQR
FROM AIRBNB),
HINGES AS
(SELECT (Q1-1.5*IQR) AS LOWER_HINGE, (Q3+1.5*IQR) AS UPPER_HINGE
FROM FIVE_NUMBER_SUMMARY)
SELECT * FROM AIRBNB
WHERE PRICE > (SELECT LOWER_HINGE FROM HINGES) AND PRICE < (SELECT
UPPER_HINGE FROM HINGES)
);

SELECT COUNT(*) FROM CLEANED;
SELECT
ROOM_TYPE AS "ROOM TYPE",
COUNT(*) AS "NO. OF Bookings",
ROUND(MIN(PRICE),1) AS "MINIMUM PRICE VALUE",
ROUND(MAX(PRICE),1) AS "MAXIMUM PRICE VALUE",
ROUND(AVG(PRICE),1) AS "AVERAGE PRICE VALUE"
FROM CLEANED
GROUP BY ROOM_TYPE;
SELECT
CITY,
ROOM_TYPE AS "ROOM TYPE",
COUNT(*) AS "NO. OF Bookings",
ROUND(MIN(PRICE),1) AS "MINIMUM PRICE VALUE",
ROUND(MAX(PRICE),1) AS "MAXIMUM PRICE VALUE",
ROUND(AVG(PRICE),1) AS "AVERAGE PRICE VALUE"
FROM CLEANED
GROUP BY CITY, ROOM_TYPE
ORDER BY CITY, ROOM_TYPE;

https://github.com/RukaiyaSrabony/sql/assets/119132009/e80adbb3-e232-4c14-8d45-2afab065550a

/*
CHECK OUT THE AVERAGE PRICE VALUE. This depicts the real scenario of AIRBNB Booking
Price of Europe.
That's what you can expect when you are going to book */
--SELECT * FROM AIRBNB LIMIT 5;

--ROOM_TYPE
SELECT DISTINCT ROOM_TYPE
FROM AIRBNB;--3

SELECT ROOM_TYPE,
ROUND(AVG(PRICE),0) AS AVERAGE_ROOM_TYPE_PRICE,
ROUND(MIN(PRICE),1) AS MINIMUM_BOOKING_PRICE,
ROUND(MAX(PRICE),1) AS MAXIMUM_BOOKING_PRICE,
ROUND(SUM(PRICE),0) AS "TOTAL BOOKING PRICE",
COUNT(ROOM_TYPE) AS NUMBER_OF_BOOKING
FROM CLEANED
GROUP BY ROOM_TYPE
ORDER BY AVERAGE_ROOM_TYPE_PRICE DESC;

--WEEKEND-WEEKDAY COMPARATIVE ANALYSIS
SELECT
DAY AS DAY_TYPE,
ROUND(AVG(PRICE),0) AS AVERAGE_PRICE,
ROUND(MIN(PRICE),1) AS MINIMUM_BOOKING_PRICE,
ROUND(MAX(PRICE),1) AS MAXIMUM_BOOKING_PRICE,
ROUND(SUM(PRICE),0) AS "TOTAL BOOKING PRICE",
COUNT(ROOM_TYPE) AS NUMBER_OF_BOOKING
FROM CLEANED
GROUP BY DAY
ORDER BY "TOTAL BOOKING PRICE" DESC;

https://github.com/RukaiyaSrabony/sql/assets/119132009/40fb416b-1b2d-4729-b910-645232a49194

--FACILTY(Metro Distance)
SELECT
CITY,
ROUND(avg(METRO_DISTANCE_KM), 2) AS "AVERAGE DISTANCE FROM METRO KM",
ROUND(avg(CITY_CENTER_KM), 2) AS "AVERAGE DISTANCE FROM CITY CENTRE IN KM"
FROM CLEANED
GROUP BY CITY
ORDER BY 2;

SELECT DISTINCT DAY
FROM CLEANED;--3

SELECT * FROM CLEANED LIMIT 10;
SELECT ROOM_TYPE, COUNT(*) AS NUMBER_OF_BOOKINGS
FROM CLEANED
GROUP BY ROOM_TYPE;

SELECT DAY, ROOM_TYPE, COUNT(*) AS NUMBER_OF_BOOKINGS
FROM CLEANED
GROUP BY DAY, ROOM_TYPE;

SELECT DAY,
COUNT(CASE WHEN ROOM_TYPE = 'Private room' THEN ROOM_TYPE ELSE NULL END)
AS PRIVATE_ROOM_BOOKING,
COUNT(CASE WHEN ROOM_TYPE = 'Entire home/apt' THEN ROOM_TYPE ELSE NULL
END) AS APARTMENT_ROOM_BOOKING,
COUNT(CASE WHEN ROOM_TYPE = 'Shared room' THEN ROOM_TYPE ELSE NULL END)
AS SHARED_ROOM_BOOKING
FROM CLEANED
GROUP BY DAY;

SELECT DAY,

ROUND(SUM(CASE WHEN ROOM_TYPE = 'Private room' THEN PRICE ELSE 0 END),0) AS
PRIVATE_ROOM_BOOKING_REVENUE,
ROUND(SUM(CASE WHEN ROOM_TYPE = 'Entire home/apt' THEN PRICE ELSE 0 END),0)
AS APARTMENT_ROOM_BOOKING_REVENUE,
ROUND(SUM(CASE WHEN ROOM_TYPE = 'Shared room' THEN PRICE ELSE 0 END),0) AS
SHARED_ROOM_BOOKING_REVENUE
FROM CLEANED
GROUP BY DAY;
--RE-CODING AND SUBQUERY
SELECT DISTINCT ROOM_TYPE_CLEANED
FROM
(SELECT
CASE WHEN ROOM_TYPE = 'Private room' THEN 'Private'
WHEN ROOM_TYPE = 'Entire home/apt' THEN 'APARTMENT'
ELSE 'SHARED' END AS ROOM_TYPE_CLEANED
FROM CLEANED);

--INSPECTING THE REASON BEHIND THE DIFFERENCE IN AVERAGE GUEST SATISFACTION SCORE

SELECT
CITY,
ROUND(AVG(GUEST_SATISFACTION),1) AS
AVERAGE_GUEST_SATISFACTION_SCORE,
ROUND(AVG(CLEANINGNESS_RATING),2) AS AVERAGE_CLEANLINESS_RATING,
ROUND(AVG(PRICE),0) AS AVERAGE_PRICE,
ROUND(AVG(NORMALSED_ATTACTION_INDEX),1) AS AVERAGE_ATTRACTION_INDEX,
ROUND(MAX(CITY_CENTER_KM),1) AS AVERAGE_DISTANCE_FROM_CITY_CENTER,
ROUND(MAX(METRO_DISTANCE_KM),1) AS AVERAGE_DISTANCE_FROM_METRO
FROM CLEANED
GROUP BY CITY
ORDER BY AVERAGE_GUEST_SATISFACTION_SCORE DESC;
/*
So, guests are leaving Athens, Budapest with more ratings in terms of satisfaction because
average booking price is low
compare with the other cities and those have the highest cleanlinsess rating as well.
However, the attraction index and distance from the city center or metro do not substantially
impact the guest satisfaction score.
*/
SELECT
CITY, ATTRACTION_INDEX, NORMALSED_ATTACTION_INDEX
FROM CLEANED
LIMIT 100;
https://github.com/RukaiyaSrabony/sql/assets/119132009/84474ba3-a2e2-490c-8485-67c5b8678839
https://github.com/RukaiyaSrabony/sql/assets/119132009/046d6784-1441-40dd-b00c-47829b64af0f

--correlation between GUEST SATISFACTION AND ATTRACTION INDEX
SELECT
ROUND(CORR(GUEST_SATISFACTION, NORMALSED_ATTACTION_INDEX), 2) AS
"CORRELATION BET GUEST SATISFACTION AND ATTRACTION INDEX"
FROM CLEANED;--SEE, NOT MUCH ASSOCIATION

---correlation between "GUEST SATISFACTION AND PRICE", "GUEST SATISFACTION AND CLEANINGNESS"

SELECT
ROUND(CORR(GUEST_SATISFACTION, PRICE), 2) AS "CORRELATION BET GUEST
SATISFACTION AND PRICE",
ROUND(CORR(GUEST_SATISFACTION, CLEANINGNESS_RATING), 2) AS
"CORRELATION BET GUEST SATISFACTION AND CLEANINGNESS RATING"
FROM CLEANED;--PRICE DOESN'T HAVE ANY ASSOCIATION, BUT CLEANINGNESS RATING DOES HAVE A MODERATE TO STRONG POSITIVE ASSOCIATION

-- =============== Dashboard Queries ===================== --
--KPI: AVERAGE BOOKING VALUE
SELECT ROUND(AVG(PRICE),0) AS "Average Booking Value" FROM CLEANED;
--KPI: AVERAGE Guest Satisfaction Score

SELECT ROUND(AVG(GUEST_SATISFACTION),1) AS "Average Guest Satisfaction Score"
FROM CLEANED;
--KPI: AVERAGE Cleanliness Score
SELECT ROUND(AVG(CLEANINGNESS_RATING),1) AS "Average Cleanliness Score" FROM
CLEANED;
--Association of Guest Satisfaction Score with Other Performance Metrics of the Cites
SELECT
CITY,
ROUND(AVG(GUEST_SATISFACTION),1) AS "Average Guest Satisfaction Score",
ROUND(AVG(CLEANINGNESS_RATING),2) AS "Average Cleanliness Score",
ROUND(AVG(PRICE),0) AS "Average Booking Value",
ROUND(AVG(NORMALSED_ATTACTION_INDEX),1) AS "Average Attraction Index",
ROUND(MAX(CITY_CENTER_KM),1) AS "Average Distance from City Center",
ROUND(MAX(METRO_DISTANCE_KM),1) AS "Average Distance from Metro"
FROM CLEANED
GROUP BY CITY
ORDER BY "Average Guest Satisfaction Score" DESC;

--Room Type wise Booking & Revenue
SELECT
ROOM_TYPE AS "ROOM TYPE",
COUNT(ROOM_TYPE) AS "Number of Booking",
ROUND(SUM(PRICE),0) AS "Total Revenue"
FROM CLEANED
GROUP BY "ROOM TYPE"
ORDER BY 3 DESC;
--City Ranking by Revenue
SELECT
CITY,
ROUND(SUM(PRICE),0) AS Revenue
FROM CLEANED
GROUP BY CITY
ORDER BY Revenue DESC;

SELECT
row_number() OVER(ORDER BY SUM(PRICE) DESC) AS Ranking,
CITY,
ROUND(SUM(PRICE),0) AS Revenue
FROM CLEANED
GROUP BY CITY;


---\ Normaly Distribute of data by removing outliers /---

WITH FIVE_NUMBER_SUMMARY AS
(SELECT
MIN(price) AS MIN_price,
PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price) AS Q1,
PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY price) AS MEDIAN,
PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price) AS Q3,
MAX(price) AS MAX_price,
(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price)-PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price)) AS IQR
FROM airbnb),
HINGES AS
(SELECT (Q1-1.5*IQR) AS LOWER_HINGE, (Q3+1.5*IQR) AS UPPER_HINGE
FROM FIVE_NUMBER_SUMMARY AS F)
SELECT price FROM airbnb
WHERE price > (SELECT LOWER_HINGE FROM HINGES) AND price < (SELECT UPPER_HINGE FROM HINGES);



-------------\ FREQUENCY Distribution /------------------

--Guest_Satisfaction
SELECT guest_satisfaction, count(guest_satisfaction) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC; --most of the obserbation are between 80 to 100
https://github.com/RukaiyaSrabony/sql/assets/119132009/da8a67db-3c24-453c-b5e2-1ae8250d9cf8

--PERSON_CAPACITY
SELECT person_capacity, count(person_capacity) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC;

--PRICE
SELECT price, count(price) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC; 
https://github.com/RukaiyaSrabony/sql/assets/119132009/7bb5b446-ec80-4c21-a82c-6a407090a8ac

--CLEANLINESS_RATING
SELECT CLEANINGNESS_RATING, count(CLEANINGNESS_RATING) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC;

--BEDROOMS
SELECT bedrooms, count(bedrooms) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC;
https://github.com/RukaiyaSrabony/sql/assets/119132009/d930ae5c-759f-40e0-a15d-01b517add832

--CITY_CENTER
SELECT CITY_CENTER_KM, count(CITY_CENTER_KM) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC;

--METRO_DISTANCE
SELECT METRO_DISTANCE_KM, count(METRO_DISTANCE_KM) AS Frequency
FROM airbnb
GROUP BY 1
ORDER BY 2 DESC;
https://github.com/RukaiyaSrabony/sql/assets/119132009/d930ae5c-759f-40e0-a15d-01b517add832
