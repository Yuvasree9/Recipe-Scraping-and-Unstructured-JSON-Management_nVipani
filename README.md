# Recipe Scraping and Unstructured JSON Management

## Overview

This project assessment automates the process of scraping, cleaning, and normalizing recipe data from various online sources.  
The extracted data is then enriched with nutritional information, stored as structured JSON files, and finally uploaded into a **Neon PostgreSQL** database for further analysis or application development.

---

## Data Sources Used

1. **Google Custom Search API**

   - Used to discover recipe URLs dynamically based on cuisine type and recipe name.
   - Example search queries: _“Indian dosa recipe”_, _“Chocolate cake recipe”_.

2. **Individual Recipe Websites**

   - Actual recipe details (title, description, ingredients, cooking steps, etc.) were scraped from recipe pages retrieved through Google Search.
   - Scraping was performed using the `requests` and `BeautifulSoup` libraries.

3. **Nutritionix API**
   - Provides detailed nutritional data (calories, protein, fats, carbohydrates, etc.) for the scraped ingredients.
   - Integrated to enrich the recipe dataset with accurate nutritional values.

---

Each recipe JSON file follows a normalized structure with:

- Metadata (`title`, `source_url`, `description`)
- Ingredients list
- Step-by-step instructions
- Nutrition data
- Grocery mapping (for retail integration)

---

## Database Integration

All normalized recipe files were uploaded into the **`recipes_normalized`** table in **Neon PostgreSQL** using SQLAlchemy.  
A simplified version of the schema is as follows:

```sql
CREATE TABLE recipes_normalized (
    id SERIAL PRIMARY KEY,
    filename TEXT,
    title TEXT,
    description TEXT,
    cuisine_type TEXT,
    course_type TEXT,
    ingredients JSONB,
    nutrition JSONB,
    instructions JSONB,
    grocery_mapping JSONB,
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

After successful upload, the table data was **exported as an Excel file** for offline analysis and sharing.

---

## How to Run the Script?

### 1. Prerequisites

Install dependencies:

```bash
pip install requests beautifulsoup4 pandas sqlalchemy psycopg2-binary
```

### 2. Configure API Keys

Replace your keys inside the notebook or script:

```python
GOOGLE_API_KEY = "your_google_api_key"
CX = "your_custom_search_engine_id"
NUTRITIONIX_APP_ID = "your_app_id"
NUTRITIONIX_APP_KEY = "your_app_key"
```

**Note:** Details of getting API Keys are neatly mentioned in the jupyter notebook `Assessment_nVipani.ipynb` file.

### 3. Run the Notebook or Script

```bash
python name_of_the_file.py
```

or open `Assessment_nVipani.ipynb` in Jupyter Notebook and execute the cells sequentially.

### 4. Verify Database Upload

```python
from sqlalchemy import create_engine, text
engine = create_engine(DATABASE_URL)
with engine.connect() as conn:
    count = conn.execute(text("SELECT COUNT(*) FROM recipes_normalized;")).scalar()
    print("Total recipes in DB:", count)
```

---

## Challenges Faced & How They Were Handled?

During this project, several challenges were encountered while scraping, processing, and uploading the recipe data.

#### 1. Inconsistent HTML Structures Across Websites

Since every recipe website follows a different HTML layout, identifying titles, ingredients, and cooking steps was difficult. To solve this, the scraping logic was made dynamic using multiple fallback conditions in BeautifulSoup. The parser checked for various possible tags and attributes to ensure that information was captured correctly even when the site structure varied.

#### 2. Missing or Irregular Data Fields

Some recipes lacked certain details such as nutrition facts, cooking steps, or descriptions. Instead of discarding those recipes, a normalization layer was implemented. This layer filled missing values with None or suitable defaults, ensuring all JSON files followed a consistent structure and were still valid for database insertion.

#### 3. API Rate Limits and Connection Failures

The Google Custom Search and Nutritionix APIs have request limits and sometimes failed due to network timeouts. To handle this, retry logic and timed delays (time.sleep) were added between requests. This approach prevented API throttling and ensured that all recipes were eventually retrieved without interruptions.

#### 4. File Saving and Path Issues on Windows

Initially, the saved JSON files weren’t visible in the expected folder because of incorrect or relative file paths. The issue was fixed by switching to absolute Windows paths using Python’s pathlib.Path and verifying file existence with Path.exists() before saving. This ensured every JSON file was properly written into the “Normalized Recipes” folder.

#### 5. Database Upload and JSON Serialization Errors

While inserting data into PostgreSQL, errors occurred due to null values, complex JSON structures, and encoding mismatches. This was handled by using json.dumps() to safely serialize JSON fields and by wrapping insert statements in try-except blocks with SQLAlchemy’s exception handling. This made the upload process stable and prevented incomplete data inserts.

---

## Current Dataset

| Recipe               | Status   |
| -------------------- | -------- |
| Dosa                 | Uploaded |
| Biryani              | Uploaded |
| Chocolate Milkshake  | Uploaded |
| Chocolate Cake       | Uploaded |
| Chai                 | Uploaded |
| Chicken Pizza        | Uploaded |
| Noodles              | Uploaded |
| Ramen                | Uploaded |
| Paneer Butter Masala | Uploaded |
| Sambar Rice          | Uploaded |

All recipes have been successfully uploaded to the **`recipes_normalized`** table and exported as **Excel** for review.

---

**Author:** Yuvasree  
**Database:** Neon PostgreSQL  
**Language:** Python 3  
**Libraries:** Requests, BeautifulSoup4, Pandas, SQLAlchemy, Psycopg2
