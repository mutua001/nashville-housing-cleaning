#  SQL FOR  nashville-housing-cleaning
 ##This SQL script cleans the nashville_housing dataset by standardizing dates, populating missing addresses, splitting address fields, normalizing categorical values, removing duplicates, and dropping unnecessary columns.

 ```sql
-- Find records with missing  property address
SELECT * 
FROM nashville_housing
WHERE `Property Address` IS NULL
ORDER BY `Parcel ID`;

-- Identify existing addresses for same Parcel ID
SELECT a.`Parcel ID`, b.`Property Address`,
       IFNULL(a.`Property Address`, b.`Property Address`) AS `Address To Be Filled`
FROM nashville_housing a
JOIN nashville_housing b
    ON a.`Parcel ID` = b.`Parcel ID` 
    AND a.Column1 != b.Column1
WHERE a.`Property Address` IS NULL;

-- Update missing property addresses
UPDATE nashville_housing a
JOIN nashville_housing b
    ON a.`Parcel ID` = b.`Parcel ID` 
    AND a.Column1 != b.Column1
SET a.`Property Address` = IFNULL(a.`Property Address`, b.`Property Address`)
WHERE a.`Property Address` IS NULL;

# 🏠 2. Populate Missing Property Addresses

-- Find records with missing property address
SELECT * 
FROM nashville_housing
WHERE `Property Address` IS NULL
ORDER BY `Parcel ID`;

-- Identify existing addresses for same Parcel ID
SELECT a.`Parcel ID`, b.`Property Address`,
       IFNULL(a.`Property Address`, b.`Property Address`) AS `Address To Be Filled`
FROM nashville_housing a
JOIN nashville_housing b
    ON a.`Parcel ID` = b.`Parcel ID` 
    AND a.Column1 != b.Column1
WHERE a.`Property Address` IS NULL;

-- Update missing property addresses
UPDATE nashville_housing a
JOIN nashville_housing b
    ON a.`Parcel ID` = b.`Parcel ID` 
    AND a.Column1 != b.Column1
SET a.`Property Address` = IFNULL(a.`Property Address`, b.`Property Address`)
WHERE a.`Property Address` IS NULL;
# 🪓 3. Split Property Address into Address & City
 -- Extract Address and City components
ALTER TABLE nashville_housing ADD COLUMN `Property Split Address` VARCHAR(255);
ALTER TABLE nashville_housing ADD COLUMN `Property City` VARCHAR(255);

UPDATE nashville_housing 
SET `Property Split Address` = SUBSTRING(`Property Address`, 1, LOCATE(',', `Property Address`) - 1),
    `Property City` = SUBSTRING(`Property Address`, LOCATE(',', `Property Address`) + 1);
# 🏠 4. Split Owner Address into Address, City, State
-- Create split string function
CREATE FUNCTION SPLIT_STR(
  x VARCHAR(255),
  delim VARCHAR(12),
  pos INT
) RETURNS VARCHAR(255)
RETURN REPLACE(SUBSTRING(SUBSTRING_INDEX(x, delim, pos),
                         LENGTH(SUBSTRING_INDEX(x, delim, pos -1)) + 1),
               delim, '');
-- Add new columns
ALTER TABLE nashville_housing ADD COLUMN `Owner Split Address` VARCHAR(255);
ALTER TABLE nashville_housing ADD COLUMN `Owner City` VARCHAR(255);
ALTER TABLE nashville_housing ADD COLUMN `Owner State` VARCHAR(255);

-- Populate split fields
UPDATE nashville_housing 
SET `Owner Split Address` = SPLIT_STR(`Owner Address`, ',', 1),
    `Owner City` = SPLIT_STR(`Owner Address`, ',', 2),
    `Owner State` = SPLIT_STR(`Owner Address`, ',', 3);

# 🗑️ 6. Remove Duplicates
WITH RowNumCTE AS (
    SELECT *, 
           ROW_NUMBER() OVER(
               PARTITION BY
                   `Parcel ID`, `Property Address`, `Sale Price`, `Sale Date`, `Legal Reference`
               ORDER BY Column1
           ) AS row_num
    FROM nashville_housing
)
DELETE FROM nashville_housing
WHERE Column1 IN (
    SELECT Column1 FROM RowNumCTE WHERE row_num > 1
);
 # 
🧹 7. Drop Unused Columns
 ALTER TABLE nashville_housing 
DROP COLUMN `Owner Address`, 
DROP COLUMN `Tax District`, 
DROP COLUMN `Property Address`, 
DROP COLUMN `Sale Date`;

 ```
