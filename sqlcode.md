---
layout: default
title: Medicare Lab Project Presentation & SQL Code
description: Medicare Presentation & SQL Code Sample
---
# Medicare Clinical Laboratory Fee Schedule Private Payer Rates and Volumes 

Below you will explore a sample analysis using data provided by Data.CMS.gov.

My goal when analyzing this data was to find insightful trends and patterns
that would be relevant for payer groups or billing offices within a
hospital system. 

The images provided were taken from a presentation I created to represent the 
data findings (see the full presentation [here](https://terathomas.github.io/downloads/MedicareSample.pdf)). A similar presentation could be used for hospital administrators or health care providers within the clinic environment.

Here are excerpts from the presentation that I found to be the most insightful:

![TopOrdered](https://terathomas.github.io/images/TopOrdered.jpg)

This chart represents the top 15 most frequently ordered laboratory codes used within this sample. 

![TopRevenue](https://terathomas.github.io/images/TopRevenue.jpg)

This chart represents the top 15 highest earning codes within this sample.
Note, some of the most frequently ordered codes are *not* the highest earning (as shown by the green and orange highlighting).

![Var](https://terathomas.github.io/images/Var.jpg)
 
 This plot demonstrates wide variability demonstrated for the most
 frequently ordered code, G0483. 

 This result demonstrates an urgent need to examine the reason behind the
 extreme outliers within the sample. 

![VarExcludeOutlier](https://terathomas.github.io/images/VarExcludeOutlier.jpg)

This is the same plot as previous, but with the outlier removed. 

Though the new price variability still seems extreme, there were _several_ orders near this new maximal value of $12,098.

You can see how it reduced the average price and standard deviation for this order. 

# SQL Code Samples for Preview
*Below you will see the sample SQL code used to explore the Medicare Lab Payer Project.*

```sql
-- 
-- Finding top orders used:
--
SELECT hcpcs_cd, SUM(vol_txt) AS total_orders
FROM public.medicarelabs
GROUP BY hcpcs_cd
ORDER BY total_orders DESC
--
-- Finding proportion from total orders in table: 
--
SELECT 
    hcpcs_cd, 
    SUM(Vol_txt) AS total_orders, 
    SUM(Vol_txt) * 1.0 / (SELECT SUM(Vol_txt) FROM public.medicarelabs) AS proportion
FROM public.medicarelabs
GROUP BY hcpcs_cd
ORDER BY total_orders DESC;
--
-- Amount billed for the top orders, casting vol_txt and price_amt as NUMERIC values:
--
SELECT 
    hcpcs_cd, 
    SUM(CAST(vol_txt AS NUMERIC)) AS total_orders, 
    SUM(CAST(vol_txt AS NUMERIC) * CAST(price_amt AS NUMERIC)) AS total_revenue
FROM public.medicarelabs
GROUP BY hcpcs_cd
ORDER BY total_revenue DESC

SELECT *
FROM PUBLIC.MEDICARELABS
--
-- Count unique orders, average orders per code, and avg price: 
--
SELECT 
    COUNT(DISTINCT hcpcs_cd) AS unique_codes,
    AVG(CAST(vol_txt AS NUMERIC)) AS avg_orders_per_code,
    AVG(CAST(price_amt AS NUMERIC)) AS avg_price
FROM public.medicarelabs;
--
-- Find avg price per order:
--
SELECT 
    hcpcs_cd, 
    SUM(CAST(price_amt AS NUMERIC)) / NULLIF(SUM(CAST(vol_txt AS NUMERIC)), 0) AS avg_price_per_order
FROM public.medicarelabs
GROUP BY hcpcs_cd
ORDER BY avg_price_per_order DESC
--
-- Validate solution: 
SELECT hcpcs_cd,
	price_amt,
	vol_txt
FROM public.medicarelabs
WHERE hcpcs_cd = '81280'
--
-- price variability for 'G0483' --> have to cast as NUMERIC Value!!
--
SELECT hcpcs_cd,
	COUNT(*) AS record_count,
	MIN(price_amt::NUMERIC) AS min_price,
	MAX(price_amt::NUMERIC) AS max_price,
	AVG(price_amt::NUMERIC) AS avg_price,
	PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS q1_price,
	PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS median_price,
	PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS q3_price,
	STDDEV(price_amt::NUMERIC) AS stddev_price
FROM public.medicarelabs
WHERE hcpcs_cd = 'G0483'
GROUP BY hcpcs_cd
--
-- validate
--
SELECT hcpcs_cd,
	vol_txt,
	price_amt
FROM public.medicarelabs
WHERE hcpcs_cd = 'G0483'
ORDER BY price_amt DESC
--
-- excluding the outlier
--
WITH filtered_data AS (
    SELECT *
    FROM public.medicarelabs
    WHERE hcpcs_cd = 'G0483'
    AND price_amt::NUMERIC < (SELECT MAX(price_amt::NUMERIC) FROM public.medicarelabs WHERE hcpcs_cd = 'G0483')
)
SELECT 
    hcpcs_cd,
    COUNT(*) AS record_count,
    MIN(price_amt::NUMERIC) AS min_price,
    MAX(price_amt::NUMERIC) AS max_price,  -- New max after removing the outlier
    AVG(price_amt::NUMERIC) AS avg_price,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS q1_price,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS median_price,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price_amt::NUMERIC) AS q3_price,
    STDDEV(price_amt::NUMERIC) AS stddev_price
FROM filtered_data
GROUP BY hcpcs_cd;
--
-- cost efficiency (inc vol, dec price)
--
SELECT 
    hcpcs_cd,
    SUM(CAST(price_amt AS NUMERIC)) AS total_price_amt,
    SUM(CAST(vol_txt AS NUMERIC)) AS total_vol_txt,
    CASE 
        WHEN SUM(CAST(vol_txt AS NUMERIC)) = 0 THEN NULL 
        ELSE SUM(CAST(price_amt AS NUMERIC)) / SUM(CAST(vol_txt AS NUMERIC)) 
    END AS cost_efficiency
FROM public.medicarelabs
GROUP BY hcpcs_cd
ORDER BY cost_efficiency ASC;
```

[back](./)