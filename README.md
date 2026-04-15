<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/bf564227-f862-4673-b0ad-df1fa7943f18" />

# SQL TOP DATA CLEANING METHOD

## 📌 Database Setup
```sql
CREATE DATABASE DataCleaning;

USE DataCleaning;

SELECT * FROM sales;
```

---

## 🔁 Handle Duplicates
```sql
WITH dupli_row AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY transaction_id ORDER BY transaction_id) AS row_num
    FROM sales
)

-- SELECT * FROM dupli_row WHERE row_num > 1;

-- DELETE dupli_row WHERE row_num > 1;

SELECT * 
FROM dupli_row
WHERE row_num > 1;

SELECT transaction_id, COUNT(*) AS duplicate_rows
FROM sales
GROUP BY transaction_id
HAVING COUNT(*) > 1;
```

---

## ⚠️ Handle NULL Values

### Check NULL (Method 1)
```sql
SELECT *
FROM sales
WHERE 
    transaction_id IS NULL OR
    customer_id IS NULL OR
    customer_name IS NULL OR
    email IS NULL OR
    purchase_date IS NULL OR
    product_id IS NULL OR
    category IS NULL OR
    price IS NULL OR
    quantity IS NULL OR
    total_amount IS NULL OR
    payment_method IS NULL OR
    delivery_status IS NULL;
```

### Check NULL (Method 2 - Dynamic)
```sql
DECLARE @SQL NVARCHAR(MAX) = '';

SELECT @SQL = STRING_AGG(
    'SELECT ''' + COLUMN_NAME + ''' AS ColumnName,
            COUNT(*) AS NullCount
     FROM ' + QUOTENAME(TABLE_SCHEMA) + '.sales
     WHERE ' + QUOTENAME(COLUMN_NAME) + ' IS NULL',
    ' UNION ALL '
)
WITHIN GROUP (ORDER BY COLUMN_NAME)
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'sales';

EXEC sp_executesql @SQL;
```

### Fix NULL Values
```sql
SELECT DISTINCT category FROM sales;

UPDATE sales 
SET category = 'NA'
WHERE category IS NULL;

UPDATE sales
SET customer_address = 'NA'
WHERE customer_address IS NULL;

UPDATE sales
SET 
    delivery_status = ISNULL(delivery_status, 'NA'),
    payment_method  = ISNULL(payment_method, 'NA'),
    price           = ISNULL(price, 0);

UPDATE sales
SET customer_name = 'User'
WHERE customer_name IS NULL;
```

**NOTE:** customer_id don't fill with random data / value

---

## ➖ Handle Negative Values
```sql
SELECT * 
FROM sales 
WHERE quantity < 0;

UPDATE sales 
SET quantity = ABS(quantity)
WHERE quantity < 0;

UPDATE sales 
SET total_amount = price * quantity
WHERE total_amount IS NULL 
   OR total_amount <> price * quantity;
```

---

## 📅 Fix Date Issues
```sql
SELECT * 
FROM sales
WHERE purchase_date = '2024-02-30';

UPDATE sales
SET purchase_date =
    CASE
        WHEN TRY_CONVERT(DATE, purchase_date, 103) IS NOT NULL
        THEN TRY_CONVERT(DATE, purchase_date, 103)
        ELSE NULL
    END;
```

---

## 📧 Fix Invalid Emails
```sql
SELECT * 
FROM sales
WHERE email NOT LIKE '%@%';

UPDATE sales 
SET email = 'NA'
WHERE email NOT LIKE '%@%';
```

---

## 🧾 Check & Fix Data Types
```sql
SELECT column_name, data_type
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = 'sales';

ALTER TABLE sales
ALTER COLUMN purchase_date DATE;
```

---

## 🚀 Conclusion
- Removed duplicates  
- Handled NULL values  
- Fixed negative values  
- Standardized date formats  
- Cleaned invalid emails  
- Corrected data types  
