🧹 SQL Data Cleaning Project – housing_data

This project demonstrates a step-by-step cleaning process of a real estate dataset using SQL. The goal is to transform raw housing data into a clean, structured format for better analysis and reporting.
📁 Dataset: housing_data

A raw dataset containing information on property sales, owner and property addresses, sale dates, and other real estate-related fields.
🔧 Cleaning Steps
1. ⏳ Format the SaleDate Column

Converted SaleDate from VARCHAR to DATE type for consistency.

UPDATE housing_data
SET saledate = CONVERT(DATE, saledate)

Also stored it in a new column:

ALTER TABLE housing_data ADD date_new DATE;
UPDATE housing_data SET date_new = CONVERT(DATE, saledate);

2. 🏡 Fix Missing PropertyAddress Data

Used JOIN on parcelid to fill in missing property addresses based on matching records.

UPDATE a
SET a.propertyaddress = ISNULL(a.propertyaddress, b.propertyaddress)
FROM housing_data a
JOIN housing_data b ON a.parcelid = b.parcelid AND a.uniqueid <> b.uniqueid
WHERE a.propertyaddress IS NULL;

3. 📬 Split PropertyAddress into StreetName and City

Used SUBSTRING and CHARINDEX to split the address.

ALTER TABLE housing_data ADD streetname NVARCHAR(255), city NVARCHAR(255);

UPDATE housing_data
SET streetname = SUBSTRING(propertyaddress, 1, CHARINDEX(',', propertyaddress) - 1),
    city = SUBSTRING(propertyaddress, CHARINDEX(',', propertyaddress) + 1, LEN(propertyaddress));

4. 👤 Split OwnerAddress into Street, City, State

Used PARSENAME to extract structured parts from owneraddress.

ALTER TABLE housing_data ADD ownerstreet NVARCHAR(255), ownercity NVARCHAR(255), ownerstate NVARCHAR(255);

UPDATE housing_data
SET ownerstreet = PARSENAME(REPLACE(owneraddress, ',', '.'), 3),
    ownercity   = PARSENAME(REPLACE(owneraddress, ',', '.'), 2),
    ownerstate  = PARSENAME(REPLACE(owneraddress, ',', '.'), 1);

5. ✅ Normalize SoldAsVacant Column

Standardized the values to Yes and No.

UPDATE housing_data
SET soldasvacant = CASE
                     WHEN soldasvacant = 'Y' THEN 'Yes'
                     WHEN soldasvacant = 'N' THEN 'No'
                     ELSE soldasvacant
                   END;

6. 🗑 Remove Duplicate Records

Used ROW_NUMBER() to identify and delete duplicate rows.

WITH rownumcte AS (
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY parcelid, propertyaddress, saleprice, saledate, legalreference ORDER BY uniqueid) AS row_num
    FROM housing_data
)
DELETE FROM rownumcte WHERE row_num > 1;

7. 🧹 Drop Unnecessary Columns

Dropped unused or redundant columns to reduce clutter.

ALTER TABLE housing_data
DROP COLUMN propertyaddress, owneraddress, saledate;

✅ Final Output

A cleaned and structured dataset ready for further analysis or integration with visualization tools such as Power BI or Tableau.
🛠 Tools Used

    SQL Server Management Studio (SSMS)

    T-SQL

📌 Author

Made with 💻 by a data analyst in training — transitioning from sales/marketing to data analysis.

    Connect on LinkedIn (Add your link here)
