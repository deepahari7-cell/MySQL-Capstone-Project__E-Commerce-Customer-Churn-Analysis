# MySQL-Capstone-Project__E-Commerce-Customer-Churn-Analysis

Customer Churn Data Cleaning, Transformation & Analysis

 Overview

This SQL script performs data cleaning, transformation, and exploratory data analysis (EDA) on the customer_churn dataset within the ecomm database. It also creates a new table, customer_returns, and combines both datasets for insights on churned customers with complaints.

 Steps in the Script
1. Disable Safe Updates
SET SQL_SAFE_UPDATES = 0;


This allows updating and deleting rows without specifying a key condition (useful for bulk updates).

2. Missing Value Imputation
 Impute Missing Values Using Mean (for Continuous Columns)

The following columns have missing values replaced with their average values:

WarehouseToHome

HourSpendOnApp

OrderAmountHikeFromlastYear

DaySinceLastOrder

Example:

UPDATE customer_churn
SET WarehouseToHome = (
  SELECT ROUND(AVG(WarehouseToHome)) FROM customer_churn
)
WHERE WarehouseToHome IS NULL;

🔹 Impute Missing Values Using Mode (for Categorical/Discrete Columns)

The following columns have missing values replaced with their most frequent value:

Tenure

CouponUsed

OrderCount

Example:

UPDATE customer_churn
SET Tenure = (
  SELECT Tenure
  FROM customer_churn
  GROUP BY Tenure
  ORDER BY COUNT(*) DESC
  LIMIT 1
)
WHERE Tenure IS NULL;

3. Outlier Handling

Remove rows where WarehouseToHome > 100 to handle extreme outliers.

DELETE FROM customer_churn
WHERE WarehouseToHome > 100;

4. Data Standardization and Transformation
🔹 Normalize Category Names

Convert abbreviated or inconsistent values to readable formats:

CASE 
  WHEN PreferredLoginDevice = 'Phone' THEN 'Mobile Phone'
END


Similarly done for PreferredOrderCat and PreferredPaymentMode.

🔹 Rename Columns
ALTER TABLE customer_churn RENAME COLUMN PreferedOrderCat TO PreferredOrderCat;
ALTER TABLE customer_churn RENAME COLUMN HourSpendOnApp TO HoursSpentOnApp;

🔹 Add New Columns

ComplaintReceived — derived from Complain
ChurnStatus — derived from Churn

ALTER TABLE customer_churn ADD COLUMN ComplaintReceived VARCHAR(10);
UPDATE customer_churn SET ComplaintReceived='Yes' WHERE Complain=1;
UPDATE customer_churn SET ComplaintReceived='No' WHERE Complain=0;


Drop old columns:

ALTER TABLE customer_churn DROP COLUMN Churn;
ALTER TABLE customer_churn DROP COLUMN Complain;

5. Exploratory Data Analysis (EDA)

Perform several analyses such as:

Churn distribution:

SELECT ChurnStatus, COUNT(*) FROM customer_churn GROUP BY ChurnStatus;


Average tenure and cashback of churned customers

Percentage of complaints among churned users

Complaint distribution by gender

Most churned product categories by city tier

Preferred payment mode among active users

Correlations involving marital status, order count, and satisfaction score

Cashback and coupon usage insights

Tenure-based payment preferences

Distance-based churn categorization

Each query provides business insights into customer retention and behavior.

6. Creating the customer_returns Table

Create and populate a new table to store return details:

CREATE TABLE IF NOT EXISTS customer_returns (
  ReturnID INT,
  CustomerID INT,
  ReturnDate DATE,
  RefundAmount DECIMAL(10,2)
);


Insert example data:

INSERT INTO customer_returns (ReturnID, CustomerID, ReturnDate, RefundAmount)
VALUES
(1001,50022,'2023-01-01',2130),
(1002,50316,'2023-01-23',2000), ...;

7. Join for Insights

Combine customer_churn and customer_returns to identify churned customers who made complaints and product returns:

SELECT c.*, r.*
FROM ecomm.customer_churn c
JOIN ecomm.customer_returns r
  ON c.CustomerID = r.CustomerID
WHERE c.ChurnStatus = 'Churned'
  AND c.ComplaintReceived = 'Yes';

Summary of Operations
  
Category	Description:

Data Cleaning---Missing values imputed using mean/mode
Outlier Handling----	Removed unrealistic distance values
Data Transformation	----Renamed columns, standardized labels
Feature Engineering----	Created ComplaintReceived & ChurnStatus
EDA	----Various insights about churn, satisfaction, and purchasing trends
Database Enhancement-----	Added customer_returns table and linked to churn data

 Output

The script produces:

A cleaned and transformed customer_churn table

A new customer_returns table

Analytical insights on churn, complaints, and transactions

