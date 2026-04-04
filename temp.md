For the MovieLens dataset and the requirements of the CSI4142 assignment, you should perform the following technical checks on each CSV file to ensure your models in Studies 1 through 4 function correctly:

### **1. Structural Overview**
* [cite_start]**`.info()`**: This is the most critical first step[cite: 74, 187]. It reveals the data types of each column (e.g., `int64`, `float64`, or `object`). [cite_start]For this dataset, you must check if the `id` columns are consistent across files (some may be strings while others are integers), which is a common cause for merge failures[cite: 39, 41].
* [cite_start]**`.shape`**: Identify the number of rows and columns[cite: 182]. [cite_start]The full `movies_metadata.csv` has about 45,000 rows, while `ratings_small.csv` contains roughly 100,000 ratings[cite: 50, 57].
* [cite_start]**`.head(10)`**: Inspect the first 10 rows to see the actual data formatting[cite: 76]. [cite_start]This is where you will notice that columns like `genres` or `keywords` are "stringified JSON" and require parsing before use[cite: 55].

### **2. Data Integrity & Cleaning**
* [cite_start]**`.isnull().sum()`**: Identify missing values[cite: 74, 186]. [cite_start]You will need to decide whether to drop these rows or use imputation (like KNN) for numerical features such as `runtime` or `budget`[cite: 74].
* [cite_start]**`.nunique()`**: Check the cardinality of categorical columns[cite: 183]. [cite_start]For example, if a "Title" column has 45,000 unique values, it won't be useful for clustering in Study 2 but is perfect for the text-based similarity in Study 1[cite: 91, 106].
* [cite_start]**`.duplicated().sum()`**: Check for duplicate movie entries or duplicate ratings from the same user, which can skew your recommendation results in Study 4[cite: 147, 200].

### **3. Statistical & Distribution Checks**
* [cite_start]**`.describe()`**: Generate summary statistics for numerical columns like `revenue`, `budget`, and `vote_average`[cite: 183]. [cite_start]Look for "zero" values that represent missing data rather than actual zero dollars[cite: 74].
* [cite_start]**Value Counts (`.value_counts()`)**: Run this on categorical columns like `original_language` or `genres` to see the distribution[cite: 183]. [cite_start]This helps you decide which 5 attributes to pick for Study 1[cite: 91].

### **4. Assignment-Specific Visualizations**
[cite_start]The grading rubric requires at least 3 tests or graphs for the Data Preparation section[cite: 76, 187].
* **Histograms**: To see the distribution of movie runtimes or ratings.
* [cite_start]**Box Plots**: Essential for **Outlier Detection**[cite: 74]. [cite_start]High-budget outliers can heavily distort the KMeans clustering results in Study 2[cite: 114, 193].
* [cite_start]**Correlation Heatmap**: To see how features like `budget` and `popularity` relate, which helps you justify your heuristics in Study 3[cite: 123, 197].




[cite_start]By documenting these steps clearly in your notebook, you satisfy the **4 points** for Data Preparation and the **5 points** for overall report effort/reproducibility[cite: 170, 186].









--------------

Yes, you absolutely should. In fact, checking for "logical" nulls is more important for this dataset than checking for standard `NaN` values. 

If you only run `.isnull().sum()`, you will miss the vast majority of the "dirty" data because the MovieLens dataset often uses **placeholder values** instead of leaving a cell empty. 

Here are the specific "hidden" nulls you need to check for during your EDA:

### 1. The "Zero" Financials (`budget` & `revenue`)
In `movies_metadata.csv`, thousands of movies have a budget of `0`. This is a placeholder for "Data Not Available." 
* **The Problem:** If you calculate the average budget for a genre without fixing this, your average will be much lower than reality.
* **The Check:** ```python
  (df['budget'] == 0).sum() 
  ```

### 2. The "Empty List" JSONs (`genres`, `keywords`, `cast`)
These columns are stored as strings. A movie with no genres isn't `NaN`; it's often the string `"[]"`. 
* **The Problem:** Your similarity measures (like Jaccard) will treat `[]` as a valid category, which will mess up your Top 10 results.
* **The Check:** Look for empty strings or empty lists after parsing.

### 3. The "Placeholder" Runtimes
Check for movies with a runtime of `0`. Unless it's a 10-second experimental clip, a 0-minute runtime is a missing value.


### 4. The "1970-01-01" Date
Sometimes missing release dates default to the Unix epoch (January 1, 1970). If you see a massive spike of movies released on that exact day, they are likely missing data.

### 5. Vote Counts vs. Vote Averages
A movie might have a `vote_average` of 10/10, but a `vote_count` of 1. 
* **The Problem:** In **Study 1**, if you rank by "Average Rating," these 1-vote movies will fill your Top 10.
* **The EDA Step:** Check the distribution of `vote_count`. You should probably filter out movies with fewer than 10 or 50 votes before you start your similarity requests to ensure your "Top 10" are actually relevant.

### Summary Checklist for each CSV:
1.  **Count Zeros:** Especially in numeric columns that shouldn't be zero.
2.  **Identify Placeholders:** Look for "Unknown," "None," or "[]".
3.  **Cross-Reference IDs:** Check if the `id` in the metadata matches the `movieId` in the links file. If 10% of your IDs don't match, your `merge()` will silently drop those movies.

By catching these "logical nulls" now, you’ll save yourself a lot of headache when your K-Means clusters in Study 2 start looking weird because of all the $0 budget movies clumped together.