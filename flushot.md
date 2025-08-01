---
layout: default
title: Insurance Claims Data Cleaning & Ad Hoc
description: Cleaning Insurance Claims Data, Joining 2 Data Sources, and Ad Hoc Analysis
---

## Insurance Claims Cleaning and Joining From 2 Data Sources Example Project

```sql
CREATE TABLE health_claims2 (
    Claim_ID VARCHAR(10) PRIMARY KEY,
    Patient_ID VARCHAR(10),
    Date_of_Service DATE,
    Procedure_Code VARCHAR(10),
    Diagnosis_Code VARCHAR(10),
    Provider_Name VARCHAR(100),
    Claim_Amount DECIMAL(10, 2)
);

SELECT * 
FROM health_claims2

-- Clean data health_claims2:
-- Normalize provider names 
UPDATE health_claims2
SET provider_name = INITCAP(provider_name)

-- Trim text fields
UPDATE health_claims2
SET diagnosis_code = TRIM(diagnosis_code),
	procedure_code = TRIM(procedure_code)

-- Handle blank values
UPDATE health_claims2
SET diagnosis_code = NULL
WHERE TRIM(diagnosis_code) = '' 

-- Remove duplicates -- NONE
SELECT claim_id, COUNT(*)
FROM health_claims2
GROUP BY claim_id
HAVING COUNT(*) > 1

-- Normalize monetary values
SELECT * FROM health_claims2
WHERE claim_amount < 0 OR claim_amount IS NULL

-- Normalize ID formats
UPDATE health_claims2
SET patient_id = UPPER(patient_id)

-- Normalize dates -- don't need to change TO_DATE
SELECT * FROM health_claims2
WHERE date_of_service IS NULL

--
-- Clean data health_claims: -- 
--

-- Normalize provider names 
UPDATE health_claims
SET provider_name = INITCAP(provider_name)

-- Trim text fields
UPDATE health_claims
SET diagnosis_code = TRIM(diagnosis_code),
	procedure_code = TRIM(procedure_code)

-- Handle blank values
UPDATE health_claims
SET diagnosis_code = NULL
WHERE TRIM(diagnosis_code) = '' 

-- Remove duplicates -- NONE
SELECT claim_ID, COUNT(*)
FROM health_claims
GROUP BY claim_ID
HAVING COUNT(*) > 1

-- Normalize monetary values
SELECT * FROM health_claims
WHERE claim_amount < 0 OR claim_amount IS NULL

-- Normalize ID formats
UPDATE health_claims
SET patient_id = UPPER(patient_id)

-- Normalize dates -- don't need to change TO_DATE
SELECT * FROM health_claims
WHERE date_of_service IS NULL


-- start JOINING
-- confirm on what value to join
SELECT * FROM health_claims2
WHERE patient_id = 'P006'

SELECT * FROM health_claims
WHERE patient_id = 'P006'

SELECT * FROM health_claims2
WHERE claim_id = 'C0013'

SELECT * FROM health_claims
WHERE claim_id = 'C0013'

-- USE UNION ALL due to repeated Claim_ID values
SELECT *, 'File1' AS source FROM health_claims
UNION ALL
SELECT *, 'File2' AS source FROM health_claims2

--Rename Claim_id for unique source code
SELECT 
  CONCAT('File1_', Claim_ID) AS Unique_Claim_ID, 
  * 
FROM health_claims
UNION ALL
SELECT 
  CONCAT('File2_', Claim_ID) AS Unique_Claim_ID, 
  * 
FROM health_claims2

-- create new table with this source information saved
CREATE TABLE all_claims AS
SELECT 
    CONCAT('File1_', Claim_ID) AS Unique_Claim_ID,
    Claim_ID,
    Patient_ID,
    Date_of_Service,
    Procedure_Code,
    Diagnosis_Code,
    Provider_Name,
    Claim_Amount,
    'File1' AS Source
FROM health_claims

UNION ALL

SELECT 
    CONCAT('File2_', Claim_ID) AS Unique_Claim_ID,
    Claim_ID,
    Patient_ID,
    Date_of_Service,
    Procedure_Code,
    Diagnosis_Code,
    Provider_Name,
    Claim_Amount,
    'File2' AS Source
FROM health_claims2

SELECT * FROM
all_claims

--Validate unique claims in new table
SELECT patient_id, claim_id, date_of_service
FROM all_claims
WHERE patient_id = 'P006'

```
## Ad Hoc Analysis on Claim Volume/Frequency, Financial Analysis, 
## Procedural Trends, Patient Trends, Quality Assurance Check 

```sql
--
-- Ad hoc analysis
--

--
-- Claim volume and frequency
--

-- total number of claims: 250
SELECT COUNT(unique_claim_id)
FROM all_claims

-- claims per patient, highest claims IDs: P018, P016, P058, P061, P047
SELECT patient_id, COUNT(patient_id)
FROM all_claims
GROUP BY patient_id 
ORDER BY COUNT(patient_id) DESC

-- claims per provider, highest claim providers: Dr. Lee, Dr. Smith, Dr. Johnson
SELECT provider_name, COUNT(provider_name)
FROM all_claims
GROUP BY provider_name
ORDER BY COUNT(provider_name) DESC
LIMIT 3

-- claims per procedure: 99406, 80053, 93000, 99213, 99214
SELECT procedure_code,
COUNT(procedure_code)
FROM all_claims
GROUP BY procedure_code
ORDER BY COUNT(procedure_code) DESC
LIMIT 5

--claims per month
SELECT DATE_TRUNC('month', date_of_service) AS month, COUNT(*) AS claim_count
FROM all_claims
GROUP BY month
ORDER BY month 

--claims per quarter
SELECT 
  DATE_TRUNC('quarter', Date_of_Service) AS quarter,
  COUNT(*) AS claim_count
FROM all_claims
GROUP BY quarter
ORDER BY quarter DESC;

--
-- Financial Analysis
--

--Total claim amount
SELECT claim_amount
	,procedure_code,
	SUM(claim_amount) AS total_claim_amount
FROM all_claims
WHERE claim_amount IS NOT NULL AND procedure_code IS NOT NULL
GROUP BY claim_amount, procedure_code
ORDER BY total_claim_amount DESC

--Claim amount by provider: 
-- Dr. Lee 49 claims 9664.75, Dr. Smith 48 claims 9366.25, Dr. Patel 46 claims 8067.00
SELECT provider_name,
	COUNT(*) AS claims_by_provider,
	SUM(claim_amount) AS total_claim_amount
FROM all_claims
WHERE claim_amount IS NOT NULL AND provider_name IS NOT NULL
GROUP BY provider_name
ORDER BY claims_by_provider DESC
LIMIT 3

--Claim amount by procedure code
--
SELECT procedure_code,
	COUNT(*) AS claims_by_code,
	SUM(claim_amount) AS total_claim_amount_proced
FROM all_claims
WHERE claim_amount IS NOT NULL AND procedure_code IS NOT NULL
GROUP BY procedure_code
ORDER BY total_claim_amount_proced DESC

--Claim amount by diagnosis code
SELECT diagnosis_code,
	COUNT(*) AS claims_by_code,
	SUM(claim_amount) AS total_claim_amount_diag
FROM all_claims
WHERE claim_amount IS NOT NULL AND diagnosis_code IS NOT NULL
GROUP BY diagnosis_code
ORDER BY total_claim_amount_diag DESC

--Patients with highest total spend
--TOP 5: P061, P018, P058, P016, P025
SELECT patient_id,
	COUNT(*) AS claims_by_patient,
	SUM(claim_amount) AS spend_by_patient
FROM all_claims
WHERE patient_id IS NOT NULL AND claim_amount IS NOT NULL
GROUP BY patient_id
ORDER BY spend_by_patient DESC
LIMIT 5

--
-- Procedure/Diagnosis Trends
--

--Most common procedure codes
SELECT procedure_code,
	COUNT(*) AS amt_procedure
FROM all_claims
WHERE procedure_code IS NOT NULL
GROUP BY procedure_code
ORDER BY amt_procedure DESC

--Most common diagnosis codes
SELECT diagnosis_code,
	COUNT(*) AS amt_diagnosis
FROM all_claims
WHERE diagnosis_code IS NOT NULL
GROUP BY diagnosis_code
ORDER BY amt_diagnosis DESC

--
--Patient Level Insights
--

-- First and last visit per patient
SELECT patient_id
	,MIN(date_of_service) AS first_visit
	,MAX(date_of_service) AS last_visit
FROM all_claims
GROUP BY patient_id
ORDER BY first_visit ASC

-- Time between visits
SELECT patient_id
	,MIN(date_of_service) AS first_visit
	,MAX(date_of_service) AS last_visit,
	AGE(MAX(date_of_service), MIN(date_of_service)) AS time_between_visits
FROM all_claims
GROUP BY patient_id
ORDER BY time_between_visits

--
-- Quality Check
--

--Claims with missing or null values
-- Total: 29 claims 
SELECT * 
FROM all_claims
WHERE claim_amount IS NULL OR claim_amount <= 0 

-- Duplicate claim IDs 
-- Result: NONE
SELECT claim_id
	,patient_id
	,date_of_service
	,
COUNT(*) AS duplicate_count
FROM all_claims
GROUP BY claim_id, patient_id, date_of_service
HAVING COUNT(*) > 1

--Claims with negative amounts
-- Result: NONE
SELECT claim_amount
FROM all_claims
WHERE claim_amount IS NOT NULL
GROUP BY claim_amount
HAVING claim_amount < 0
```

[back](./)