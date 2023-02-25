-- **Cleaning Data**

```
SELECT *
-> FROM nashville_housing_data
```

------------------------------------------------------------------------------------------

--**Standarize date format**
```
SELECT SaleDate, Sale_date-- CAST(saledate as date)
-> FROM nashville_housing_data

ALTER TABLE nashville_housing_data
-> ADD Sale_date DATE;

UPDATE nashville_housing_data
-> SET Sale_date = CAST(saledate AS DATE)
```

------------------------------------------------------------------------------------------

--**Populating PropertyAddress**
```
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
-> FROM nashville_housing_data a
-> JOIN nashville_housing_data b
-> 	 ON a.ParcelID = b.ParcelID
-> 	 AND a.UniqueID <> b.UniqueID
-> WHERE a.PropertyAddress is null

UPDATE a
-> SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
-> FROM nashville_housing_data a
-> JOIN nashville_housing_data b
->	 ON a.ParcelID = b.ParcelID
->	 AND a.UniqueID <> b.UniqueID
-> WHERE a.PropertyAddress is null
```

------------------------------------------------------------------------------------------
--**Spliting PropertyAddress**
```
SELECT ParcelID, PropertyAddress, 
->	 SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) as addr,
->	 SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) as addr
-> FROM nashville_housing_data

ALTER TABLE nashville_housing_data
-> ADD Property_Address nvarchar(255);

UPDATE nashville_housing_data
-> SET Property_Address = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)

ALTER TABLE nashville_housing_data
-> ADD Property_City nvarchar(255);

UPDATE nashville_housing_data
-> SET Property_City = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))
```
------------------------------------------------------------------------------------------

--**Spliting OwnerAddress**
```
SELECT owneraddress,
->	 PARSENAME(REPLACE(OwnerAddress, ',', '.'),3),
->	 PARSENAME(REPLACE(OwnerAddress, ',', '.'),2),
->	 PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)
-> FROM nashville_housing_data

ALTER TABLE nashville_housing_data
-> ADD Owner_Address nvarchar(255),
-> 	   Owner_City nvarchar(255),
-> 	   Owner_State nvarchar(255);

UPDATE nashville_housing_data
-> SET Owner_Address = PARSENAME(REPLACE(OwnerAddress, ',', '.'),3),
-> 	   Owner_City = PARSENAME(REPLACE(OwnerAddress, ',', '.'),2),
-> 	   Owner_State = PARSENAME(REPLACE(OwnerAddress, ',', '.'),1)
```
------------------------------------------------------------------------------------------

--**Changing Y and N to Yes and No in 'SoldAsVacant' field**
```
SELECT SoldAsVacant,
->     CASE WHEN SoldAsVacant = 'N' THEN 'No'
-> 		 WHEN SoldAsVacant = 'Y' THEN 'Yes'
-> 		 ELSE SoldAsVacant
-> 		 END
-> FROM nashville_housing_data

UPDATE nashville_housing_data
-> SET SoldAsVacant = CASE WHEN SoldAsVacant = 'N' THEN 'No'
-> 		 WHEN SoldAsVacant = 'Y' THEN 'Yes'
-> 		 ELSE SoldAsVacant
-> 		 END
```
------------------------------------------------------------------------------------------

--**Remove Duplicates**
```
SELECT ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference
-> FROM nashville_housing_data

SELECT *,
-> 	 ROW_NUMBER() OVER 
-> 	 (PARTITION BY ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference ORDER BY uniqueID) AS row_num
-> FROM nashville_housing_data
-> ORDER BY ParcelID

WITH row_numCTE AS (
-> SELECT *,
-> 	 ROW_NUMBER() OVER 
-> 	 (PARTITION BY ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference ORDER BY uniqueID) AS row_num
-> FROM nashville_housing_data
-> )

--Delete

SELECT *
-> FROM row_numCTE
-> WHERE row_num > 1
```
------------------------------------------------------------------------------------------

--**Remove Unused Columns**
```
SELECT *
-> FROM nashville_housing_data

ALTER TABLE nashville_housing_data
-> DROP COLUMN PropertyAddress, SaleDate, OwnerAddress, TaxDistrict
```
------------------------------------------------------------------------------------------
