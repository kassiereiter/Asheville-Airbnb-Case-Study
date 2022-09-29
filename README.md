# Asheville Airbnb Case Study
Author: Kassie Reiter  
Date: September 29, 2022  

This case study is an analysis I performed as my capstone project for the Google Data Analytics Course. It is intended to show my knowledge of the data analysis process, as well as my skills in SQL and Tableau.

This case study follows the six step data analysis process:

[Ask](#1-ask)   
[Prepare](#2-prepare)  
[Process](#3-process)  
[Analyze](#4-analyze)  
[Share](#5-share)  
[Act](#6-act)  

## 1. ASK  

Business Task: Find the best neighborhood in Asheville, NC to open a new Airbnb.

Primary stakeholders: People looking to open an Airbnb in Asheville.  

Questions I asked:  
- What makes a neighborhood the best? (most profitable)  
- What defines the neighborhoods? (zip code)

## 2. PREPARE  

Data Source: http://insideairbnb.com/get-the-data/

The dataset has 6 CSV files. A detailed account of listings, calendar, and reviews. A summary of listings and reviews. And a list of neighborhoods.  

Reliability: Data isn't coming directly from the source, but after comparing some of the data to Airbnb's website the data seems accurate.  
Original: Data is third party since I got the data from Inside Airbnb and they got it from Airbnb.  
Comprehensive: Data has a lot of vauable information. It is missing the number of days each listing was booked last year and the names of the neighborhoods.  
Current: Data is as recent as 09-14-2022. I filtered data to only go back to 09-14-2021. So all data is from the last year.  
Cited: I gathered the data from http://insideairbnb.com/get-the-data/. Their website says data is complied from the Airbnb website.  

Dataset limitations:  
The calendar dataset only has availability for the future. This means I am unable to see how many days each listing was booked. Therefore, I will have to estimate revenue based on the number of reviews, minimum stay required, and price.  

## 3. PROCESS  

First I load the data into BiqQuery and preview the data. The listings_summary and review_summary are just summarized versions of the listings and reviews data, so I decide I will just use the original sets of data. 
The calendar data has availability for the future which I will not be using for my analysis. The neighboorhoods data is a list of zip codes and neighborhood names that are all null.
I decide to Google those zip codes and find the neighborhood names. Since I'm working in BigQuery Sandbox (which doesn't allow me to update tables), I update the data in Google Sheets and reupload to BigQuery.  


## 4. ANALYZE  

I joined my neighborhood file with the listings file to create a table that has the listings with what neighborhood they are in.
```
SELECT id, name, zip_code, neighbourhoods.neighbourhood AS neighborhood, host_id, room_type, bedrooms, accommodates, price, minimum_nights, review_scores_rating, review_scores_location,number_of_reviews, calculated_host_listings_count,longitude, latitude,
FROM `cosmic-mariner-362014.Airbnb.neighbourhoods` AS neighbourhoods
JOIN `cosmic-mariner-362014.Airbnb.listings` AS listings
ON zip_code = neighbourhood_cleansed
```
The reviews dataset was the only data with dates, so I performed a query to count the amount of reviews each listing received from 09-14-2021 to 09-14-2022.
```
SELECT listings.id, name, count(listings.id) AS number_of_reviews,
FROM `cosmic-mariner-362014.Airbnb.reviews_summary` AS reviews
JOIN `cosmic-mariner-362014.Airbnb.listings` AS listings
ON reviews.listing_id = listings.id
WHERE date BETWEEN '2021-09-14' AND '2022-09-14'
GROUP BY listings.id, name;
```
I saved those results as a table in BigQuery called 'reviews_from_last_year'. I joined this table with the original listings data so I could calculate an estimated revenue.
I did this by multiplying the number of reviews by the minimum nights stay required by the price per night and dividing by 12 to get an estimated revenue per month.
```
SELECT listings.id, listings.name, reviews.number_of_reviews, price, minimum_nights, (reviews.number_of_reviews*minimum_nights)/12 AS stays_per_month, (reviews.number_of_reviews*minimum_nights*price)/12 AS revenue_per_month, longitude, latitude,
FROM `cosmic-mariner-362014.Airbnb.reviews_from_last_year` AS reviews
RIGHT JOIN `cosmic-mariner-362014.Airbnb.listings` AS listings
ON reviews.id = listings.id
WHERE minimum_nights < 30;
```
I uploaded my two new tables listings with neighborhoods and listings with estimated revenue to Tableau. I joined them by listing id and made the dashboard below.  

## 5. SHARE  

[Tableau Dashboard](https://public.tableau.com/app/profile/kassie2664/viz/AshevilleAirbnb_16644789825510/Dashboard1#1)

## 6. ACT  

Conclusions from my analysis:  
- Downtown Asheville and East Asheville have the highest average revenue per month which means these areas are likely the most profitable  
- Downtown Asheville and West Asheville have the largest sum of revenue per month which indicates that there are a lot of Airbnbs already there  
- The most popular type of Airbnb is renting an entire house or apartment  

My Recommendation:  
I would recommand that people looking to open a new Airbnb in Asheville, NC look for properties that are in Downtown or East Asheville. 
I particularly recommend East Asheville because that market is less saturated than Downtown. The Biltmore and North Asheville areas are the next areas I would recommend.
