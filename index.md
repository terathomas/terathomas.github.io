---
layout: default
---
# Welcome

Hello, I am Tera Thomas. My background in academic and clinical research has launched me into the fascinating world of data analysis. I have a deep rooted passion for healthcare innovation and patient care quality. 

I welcome you to my digital portfolio, where you can explore some of my work with health care data analytics. 

The work you will see below was conducted through SQL (PgAdmin), Excel, Python (VS Code), Tableau and Power BI/Power Query. 

# Sample Patient Population Analyzing: Patients, Encounters, Conditions and Immunizations

Below you will explore a sample population retrieved from Data.CMS.gov.
The patient population was filtered using SQL, and analyzed through Excel. The visuals you will 
see were created in Tableau.

This analysis covered:

**21,670** individual patient encounters

**10** different payer groups

**7** different encounter classes

**4** different joined databases (patients, encounters, conditions, immunizations)

![SamplePop](https://terathomas.github.io/images/SamplePop.jpg)


### The Sample Population Excel File

_please note, all patient information is synthetic_

![SamplePopExcel](https://terathomas.github.io/images/SamplePopExcel.jpg)

This is a 21,670 row & 24 column data set analyzing
various population metrics.

-----------------------
----------------------- 

# Data Professional Survey 
Below you will see a demonstration using real life data provided by "Alex the Analyst", focusing on data professional survey reports. This data was transformed and cleaned through Power BI Power Query, and then displayed in a dashboard.

![PowerBI](https://terathomas.github.io/images/PowerBI.jpg)

-----------------------
-----------------------

# Tableau Dashboard for Flu Shot Statistics

Below is a Tableau dashboard for Flu Shot statistics. The sample health data was sourced from Data.CMS.gov, and all patient information is synthetic. 

<div class='tableauPlaceholder' id='viz1745968741451' style='position: relative'>
    <noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fl&#47;FluShot_17459512140430&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='FluShot_17459512140430&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fl&#47;FluShot_17459512140430&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object>
</div>                
<script type='text/javascript'>
var divElement = document.getElementById('viz1745968741451');
var vizElement = divElement.getElementsByTagName('object')[0];
if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1366px';vizElement.style.height='795px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1366px';vizElement.style.height='795px';} else { vizElement.style.width='100%';vizElement.style.height='1927px';}
var scriptElement = document.createElement('script');
scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';
vizElement.parentNode.insertBefore(scriptElement, vizElement);                
</script>

-----------------------
-----------------------

# SQL Code Samples for Preview
*Below you will see the sample SQL code used to explore a sample patient population.*

```sql
SELECT 
  e.start,
  e.stop,
  e.id AS encounter_id,
  e.patient,
  e.encounterclass,
  e.code,
  e.description,
  e.total_claim_cost,
  p.id AS patient_id,
  p.birthdate,
  p.deathdate,
  p.gender,
  p.city,
  p.state,
  p.county
FROM encounters e
JOIN patients p ON e.patient = p.id;

--count encounters by gender
SELECT patients.gender, COUNT(DISTINCT encounters.id) AS encounters_count
FROM encounters
JOIN patients ON encounters.patient = patients.id
GROUP BY patients.gender;

-- avg total_claim_cost by encounter class
SELECT encounterclass, AVG(total_claim_cost)
FROM encounters
JOIN patients ON encounters.patient = patients.id
GROUP BY encounterclass

-- count of patients with recorded death date
SELECT COUNT(patients.id)
FROM encounters
JOIN patients ON encounters.patient = patients.id
WHERE deathdate IS NOT NULL

--admission duration - exact minutes
SELECT encounters.id AS encounter_id,
       start,
       stop,
       stop - start AS admission_duration
FROM encounters
JOIN patients ON encounters.patient = patients.id

--admission duration in hours
SELECT encounters.id AS encounter_id,
       start,
       stop,
	   description,
       EXTRACT(EPOCH FROM (stop - start)) / 3600 AS admission_hours
AVG(amission_hours)
FROM encounters
JOIN patients ON encounters.patient = patients.id

--average admission hours
SELECT AVG(EXTRACT(EPOCH FROM (stop - start)) / 3600) AS avg_admission_hours
FROM encounters
JOIN patients ON encounters.patient = patients.id;

-- top 5 cities with highest total claim cost
SELECT city,
SUM(total_claim_cost) AS city_costs
FROM encounters
JOIN patients ON encounters.patient = patients.id
GROUP BY city
ORDER BY city_costs DESC
LIMIT 5

--find total unique encounters
SELECT DISTINCT encounters.encounterclass
FROM encounters
JOIN patients
  ON encounters.patient = patients.id;

--find count of each encounter
SELECT encounters.encounterclass, COUNT(*) AS encounter_count
FROM encounters
JOIN patients
  ON encounters.patient = patients.id
GROUP BY encounters.encounterclass
ORDER BY encounter_count DESC

-- which encounter class has highest avg admissions duration
SELECT encounterclass, 
	AVG(EXTRACT(EPOCH FROM (stop - start)) / 3600) AS avg_admission_hours
FROM encounters
JOIN patients ON encounters.patient = patients.id
GROUP BY encounterclass
ORDER BY avg_admission_hours DESC

-- partition total cost by patient
SELECT 
  patient,
  encounterclass,
  total_claim_cost,
  SUM(total_claim_cost) OVER (PARTITION BY patient) AS total_cost_per_patient
FROM encounters

```
-----------------------
-----------------------

# Python Code for Sample Patient Population. 
**Python** and **PANDAS** empower seamless data manipulation and intuitive visualization, turning raw healthcare data into actionable insights. By pairing flexible scripting with powerful built-in tools, I was able to uncover trends with clarity and precision...no dashboard required.

<iframe src="pythonHCprac.html" width="100%" height="600px" style="border:none;"></iframe>

# Data Cleaning and Ad Hoc SQL Results
Below you will see an example of importing, type casting, and the performance of ad hoc analysis. The data can then be visualized and shared with stakeholders for insightful business actions. 

```sql
-- Data Cleaning
-- Goals: normalize NULLs, make data types consistent, remove bad rows, trim extra space
--
DROP TABLE IF EXISTS provider_data_cleaned;

CREATE TABLE provider_data_cleaned AS
SELECT
  NULLIF(TRIM(Rndrng_Prvdr_CCN::TEXT), '')::INTEGER AS Rndrng_Prvdr_CCN,
  NULLIF(TRIM(Rndrng_Prvdr_Org_Name), '') AS Rndrng_Prvdr_Org_Name,
  NULLIF(TRIM(Rndrng_Prvdr_St), '') AS Rndrng_Prvdr_St,
  NULLIF(TRIM(Rndrng_Prvdr_City), '') AS Rndrng_Prvdr_City,
  NULLIF(TRIM(Rndrng_Prvdr_State_Abrvtn), '') AS Rndrng_Prvdr_State_Abrvtn,
  NULLIF(TRIM(Rndrng_Prvdr_State_FIPS::TEXT), '')::INTEGER AS Rndrng_Prvdr_State_FIPS,
  NULLIF(TRIM(Rndrng_Prvdr_Zip5), '') AS Rndrng_Prvdr_Zip5,
  NULLIF(TRIM(Rndrng_Prvdr_RUCA::TEXT), '')::INTEGER AS Rndrng_Prvdr_RUCA,
  NULLIF(TRIM(Rndrng_Prvdr_RUCA_Desc), '') AS Rndrng_Prvdr_RUCA_Desc,
  NULLIF(TRIM(APC_Cd::TEXT), '')::INTEGER AS APC_Cd,
  NULLIF(TRIM(APC_Desc), '') AS APC_Desc,
  NULLIF(TRIM(Bene_Cnt::TEXT), '')::INTEGER AS Bene_Cnt,
  NULLIF(TRIM(CAPC_Srvcs::TEXT), '')::INTEGER AS CAPC_Srvcs,
  NULLIF(TRIM(Avg_Tot_Sbmtd_Chrgs::TEXT), '')::NUMERIC(10,4) AS Avg_Tot_Sbmtd_Chrgs,
  NULLIF(TRIM(Avg_Mdcr_Alowd_Amt::TEXT), '')::NUMERIC(10,4) AS Avg_Mdcr_Alowd_Amt,
  NULLIF(TRIM(Avg_Mdcr_Pymt_Amt::TEXT), '')::NUMERIC(10,4) AS Avg_Mdcr_Pymt_Amt,
  NULLIF(TRIM(Outlier_Srvcs::TEXT), '')::INTEGER AS Outlier_Srvcs,
  NULLIF(TRIM(Avg_Mdcr_Outlier_Amt::TEXT), '')::NUMERIC(10,4) AS Avg_Mdcr_Outlier_Amt
FROM provider_data

-- remove rows with negative values
DELETE FROM provider_data_cleaned
WHERE Avg_Tot_Sbmtd_Chrgs < 0
   OR Avg_Mdcr_Alowd_Amt < 0
   OR Avg_Mdcr_Pymt_Amt < 0
   OR Avg_Mdcr_Outlier_Amt < 0;

-- delete any zip codes that arent 5 digits long
DELETE FROM provider_data_cleaned
WHERE Rndrng_Prvdr_Zip5 IS NOT NULL
  AND LENGTH(Rndrng_Prvdr_Zip5) != 5;

-- Returned = 0 

-- Check NULL Values
SELECT COUNT(*) AS total_rows,
       COUNT(Avg_mdcr_pymt_amt) AS non_null_payments
FROM provider_data;

SELECT DISTINCT Avg_mdcr_pymt_amt
FROM provider_data
WHERE Avg_mdcr_pymt_amt IS NOT NULL
LIMIT 10;

-- DATA CLEANING COMPLETE -- 

-- Top 10 states with most providers
SELECT rndrng_prvdr_state_abrvtn, COUNT(*) 
AS provider_count
FROM provider_data_cleaned
GROUP BY  rndrng_prvdr_state_abrvtn
ORDER BY provider_count DESC
LIMIT 10;

--Average Medicare payment amount by ZIP code - NON NULL
SELECT rndrng_prvdr_zip5, AVG(Avg_mdcr_pymt_amt) AS avg_payment
FROM provider_data_cleaned
WHERE Avg_mdcr_pymt_amt IS NOT NULL
GROUP BY rndrng_prvdr_zip5
ORDER BY avg_payment DESC
LIMIT 10;

-- Most commonly billed APC codes
SELECT APC_Cd, APC_Desc, SUM(Bene_Cnt) AS total_beneficiaries
FROM provider_data_cleaned
GROUP BY APC_Cd, APC_Desc
ORDER BY total_beneficiaries DESC
LIMIT 10;

-- Top providers by total CAPC services - NON NULL 
SELECT rndrng_prvdr_org_name, SUM(CAPC_Srvcs) AS total_capc
FROM provider_data
WHERE CAPC_Srvcs IS NOT NULL
GROUP BY rndrng_prvdr_org_name
ORDER BY total_capc DESC
LIMIT 10;

-- Avg Medicare payment by rural vs. urban
SELECT rndrng_prvdr_RUCA_desc, AVG(Avg_Mdcr_Pymt_Amt) AS avg_payment
FROM provider_data_cleaned
WHERE Avg_Mdcr_Pymt_Amt IS NOT NULL
GROUP BY Rndrng_prvdr_RUCA_desc
ORDER BY Avg_payment DESC 

-- Providers with the highest outlier payments
SELECT Rndrng_Prvdr_Org_Name, AVG(avg_mdcr_outlier_amt) AS avg_outlier_payment
FROM provider_data_cleaned
WHERE avg_mdcr_outlier_amt IS NOT NULL
GROUP BY rndrng_prvdr_org_name
ORDER BY avg_outlier_payment DESC 
LIMIT 10;
```

-----------------------
-----------------------

For other projects see below: 

[Medicare Lab Payer Project & SQL Code](./sqlcode.html)

[Link to Citations & Writing Samples](./citations.html)


