---
layout: default
title: Cleaning & Joining Practice 
description: Insurance Claims Data From Various Sources
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

[back](./)