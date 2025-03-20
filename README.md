## 🛠 SQL Data Cleaning Project: Layoffs Dataset  

**Dataset Source**: [Kaggle - Layoffs 2022](https://www.kaggle.com/datasets/swaptr/layoffs-2022)  

### 📌 **Project Overview**  
This project focuses on **cleaning and standardizing data** from a **tech layoffs dataset**. The primary objectives are:  

✅ Removing **duplicate records**  
✅ Standardizing **company, industry, and country names**  
✅ Handling **null values and blank fields**  
✅ Converting **date fields to the correct format**  
✅ Dropping **unnecessary columns**  

---

## ⚡ **SQL Cleaning Steps**  

### 1️⃣ **Initial Data Exploration**
We begin by inspecting the dataset structure:  
```sql
SELECT * FROM layoffs;
```

### 2️⃣ **Creating a Staging Table for Cleaning**  
To avoid modifying the raw dataset, we create a **staging table**:  
```sql
CREATE TABLE layoffs_staging LIKE layoffs;
INSERT INTO layoffs_staging SELECT * FROM layoffs;
```

### 3️⃣ **Identifying & Removing Duplicates**
We use `ROW_NUMBER()` to detect duplicates based on **company, industry, total_laid_off, percentage_laid_off, and date**:  
```sql
WITH duplicate_cte AS (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY company, location, industry, total_laid_off,
               percentage_laid_off, date, stage, country, funds_raised_millions
           ) AS row_num
    FROM layoffs_staging
)
DELETE FROM layoffs_staging2
WHERE row_num > 1;
```

---

### 4️⃣ **Standardizing Text Fields**  
#### **Trimming Extra Spaces in Company Names**  
```sql
UPDATE layoffs_staging2
SET company = TRIM(company);
```

#### **Standardizing Industry Names**  
Fix inconsistent **industry names** like "Crypto%", "Crypto Market", etc.:  
```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

#### **Fixing Country Names (Removing Periods)**  
```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

---

### 5️⃣ **Converting Dates to Proper Format**  
Many dates in the dataset are stored as **text** instead of a proper `DATE` format. We fix this using `STR_TO_DATE()`:  
```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```
Then, modify the column type to `DATE`:  
```sql
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

---

### 6️⃣ **Handling NULL and Blank Values**  
#### **Identify Missing Values in Key Columns**
```sql
SELECT * FROM layoffs_staging2
WHERE industry IS NULL OR industry = '';
```

#### **Manually Fix Missing Industry Data**
Example: Assign **"Travel"** industry to Airbnb:  
```sql
UPDATE layoffs_staging2
SET industry = 'Travel'
WHERE company = 'Airbnb'
AND (industry IS NULL OR industry = '');
```

#### **Remove Completely Empty Records**
```sql
DELETE FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

### 7️⃣ **Dropping Unnecessary Columns**  
After cleaning, we remove the `row_num` column:  
```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## 🌟 **Key Learnings**
- Used `ROW_NUMBER()` for **duplicate detection**  
- Applied `TRIM()` and `LIKE` for **text standardization**  
- Converted **string dates** into proper `DATE` format  
- Used `DELETE` and `UPDATE` to clean **null values**  
- Ensured data integrity before modifying the original dataset  

---

## 🚀 **Next Steps**
💡 Perform **exploratory data analysis (EDA)**  
💡 Visualize layoffs trends using **Power BI/Tableau**  
💡 Build **SQL queries** for key insights  

---

## 📂 **Files in This Repository**
| File Name  | Description |
|------------|------------|
| `layoffs.sql` | Full SQL script for data cleaning |
| `README.md`  | Project documentation (this file) |
| `layoffs.cvs`  | Raw data file |

---

## 👨‍💻 **Author**
🔹 **Jose Perez Guerrero**  
🔹 **https://github.com/pjose92/**  

