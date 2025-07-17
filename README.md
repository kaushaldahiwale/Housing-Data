# 🧹 SQL Data Cleaning Project – `housing_data`

This project showcases the step-by-step data cleaning process on a real estate dataset using SQL. The primary goal is to transform raw, inconsistent housing data into a clean and structured format suitable for analysis.

---

## 📁 Dataset: `housing_data`

A raw dataset containing:
- Property sales information
- Owner and property addresses
- Sale dates
- Other real estate-related fields

---

## 🔧 Cleaning Steps

### 1. ⏳ Format the `SaleDate` Column

Converted the `saledate` field from `VARCHAR` to `DATE` for consistency and ease of use.

```sql
UPDATE housing_data
SET saledate = CONVERT(DATE, saledate);

ALTER TABLE housing_data ADD date_new DATE;
UPDATE housing_data SET date_new = CONVERT(DATE, saledate);
```

---

### 2. 🏡 Fix Missing `PropertyAddress` Values

Used `JOIN` on `parcelid` to fill missing `propertyaddress` fields based on matching records.

```sql
UPDATE a
SET a.propertyaddress = ISNULL(a.propertyaddress, b.propertyaddress)
FROM housing_data a
JOIN housing_data b ON a.parcelid = b.parcelid AND a.uniqueid <> b.uniqueid
WHERE a.propertyaddress IS NULL;
```

---

### 3. 📬 Split `PropertyAddress` into `streetname` and `city`

Used string functions to extract street name and city from `propertyaddress`.

```sql
ALTER TABLE housing_data ADD streetname NVARCHAR(255), city NVARCHAR(255);

UPDATE housing_data
SET streetname = SUBSTRING(propertyaddress, 1, CHARINDEX(',', propertyaddress) - 1),
    city = SUBSTRING(propertyaddress, CHARINDEX(',', propertyaddress) + 1, LEN(propertyaddress));
```

---

### 4. 👤 Break Down `OwnerAddress` into Components

Split `owneraddress` into `ownerstreet`, `ownercity`, and `ownerstate` using `PARSENAME`.

```sql
ALTER TABLE housing_data
ADD ownerstreet NVARCHAR(255), ownercity NVARCHAR(255), ownerstate NVARCHAR(255);

UPDATE housing_data
SET ownerstreet = PARSENAME(REPLACE(owneraddress, ',', '.'), 3),
    ownercity   = PARSENAME(REPLACE(owneraddress, ',', '.'), 2),
    ownerstate  = PARSENAME(REPLACE(owneraddress, ',', '.'), 1);
```

---

### 5. ✅ Normalize `SoldAsVacant` Values

Standardized values in the `soldasvacant` column to `Yes` and `No`.

```sql
UPDATE housing_data
SET soldasvacant = CASE
                     WHEN soldasvacant = 'Y' THEN 'Yes'
                     WHEN soldasvacant = 'N' THEN 'No'
                     ELSE soldasvacant
                   END;
```

---

### 6. 🗑 Remove Duplicate Records

Identified and removed duplicate rows based on key fields.

```sql
WITH rownumcte AS (
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY parcelid, propertyaddress, saleprice, saledate, legalreference ORDER BY uniqueid) AS row_num
    FROM housing_data
)
DELETE FROM rownumcte WHERE row_num > 1;
```

---

### 7. 🧹 Drop Unused Columns

Dropped redundant columns to streamline the dataset.

```sql
ALTER TABLE housing_data
DROP COLUMN propertyaddress, owneraddress, saledate;
```

---

## ✅ Final Result

A cleaned and structured dataset ready for:
- Analytical processing
- Dashboard building (Power BI, Tableau)
- Reporting and insights

---

## 🛠 Tools Used

- **SQL Server Management Studio (SSMS)**
- **T-SQL**

---

## 📌 Author

Made with 💻 by a data analyst in training — transitioning from a sales & marketing background into data analytics.  
Feel free to connect on [LinkedIn](#) _(insert your link here)_

---
