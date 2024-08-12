# Data Cleaning Process

In this SQL script, I worked with a dataset containing detailed information about company layoffs, including fields like company name, location, industry, total employees laid off, percentage laid off, date of the layoff, company stage, country, and funds raised. My objective was to clean and prepare this data for analysis, ensuring it was **accurate**, **consistent**, and free of **duplicates** so that insights could be dervised in future anlaysis.

## Step 1: Create a Staging Table

I began by creating a copy of the original `layoffs` table to preserve the raw data. This allowed me to work on the staging table (`layoffs_staging`) without affecting the original data. Below are the SQL commands used:

```sql
CREATE TABLE layoffs_staging LIKE layoffs;
INSERT INTO layoffs_staging SELECT * FROM layoffs;
```

## Step 2: Remove Duplicate Records

Next, I tackled duplicate records by using the ROW_NUMBER() function to identify duplicates based on a combination of key attributes such as company, location, and industry. I then removed the duplicate rows while keeping one unique entry per set of duplicates. Here is the query used to assign row numbers and delete duplicates:

```sql
WITH duplicate AS (
    SELECT *,
    ROW_NUMBER() OVER(
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num
    FROM layoffs_staging
)
DELETE FROM layoffs_staging2 WHERE row_num > 1;

```
## Step 3: Standardise Data
After ensuring I had distinct records, I then standardized fields like company names and industry labels to maintain consistency. For example, I consolidated various entries under "Crypto" by updating all similar entries using the following query:

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
I also standardized country names, consolidating variations under "United States" with these queries:
```sql
UPDATE layoffs_staging2
SET country = 'United States'
WHERE country LIKE '%United States%';

-- OR

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE '%United States%';
```
To handle the date field, which was in text format, I converted it to a date type:
```sql
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```
## Step 4: Handle Null and Blank Values
I then addressed NULL or blank values in the dataset. For example, I filled in missing industry information based on other entries with the same company details:

```sql
SELECT t1.industry, t2.industry
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
    AND t1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```
## Step 5 (Final): Clean Up
Finally, I removed any unnecessary columns created during the cleaning process and deleted rows with NULL values in both total_laid_off and percentage_laid_off, resulting in a streamlined and standardized dataset ready for further analysis. Below are the commands used:

```sql

DELETE FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

My goal throughout was to ensure that the dataset was both **reliable** and **insightful** for the analysis I planned to conduct. By meticulously cleaning and standardizing the data, I aimed to eliminate inconsistencies and inaccuracies that could have skewed the results. This thorough approach ensured that the dataset provided a solid foundation for making informed decisions and generating actionable insights, ultimately enhancing the quality and effectiveness of the analysis.




