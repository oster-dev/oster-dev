# Today I Learned! Daily Log

A running log of things learned on the journey to becoming an
ML Platform | Data and Feature Infrastructure Engineer.

TIL Started: April 13, 2026.

---

## April 24, 2026

**Advanced SQL | Theory Complete: Personal Reflection**

At the start, Subqueries and CTEs felt genuinely complicated. The syntax is long,
the nesting can be hard to read at first, and the mental model takes time to build.
After completing the full theory section however, the concepts feel significantly
more logical and approachable than initially expected.

The theoretical foundation is now set. What remains is consolidating this through
practice — LeetCode SQL problems and other hands-on platforms will be the focus
going forward to make SQL truly job-ready. Query performance decisions, meaning
choosing the most efficient method for a given problem to avoid unnecessary
computation, is also an area I plan to deepen over time.

The [Maven Advanced SQL](https://www.udemy.com/course/sql-advanced-queries/) course was an excellent choice for building this foundation.
It covered every relevant concept clearly, and I would fully recommend it for anyone
targeting a similar career path. The practical side will continue to develop throughout the roadmap, especially once
Spark SQL begins in Month 3.



**SQL | Data Analysis Applications — Section 8 Completed ✓**

Today I started and fully completed Section 8 — the final practical chapter of the
Maven Advanced SQL course. These are real data analysis workflow patterns, directly
transferable to Feature Engineering and data infrastructure work.

**Duplicate Detection & Removal**
- Identified and removed duplicates using `ROW_NUMBER()` — the cleanest pattern in production
- `GROUP BY` + `HAVING COUNT(*) > 1` surfaces duplicates — `ROW_NUMBER()` removes them
- Completed the assignment and solution

```sql
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id, order_date, product_id
      ORDER BY transaction_id
    ) AS rn
  FROM orders
)
SELECT * FROM ranked WHERE rn = 1;
```

**Min / Max Value Filtering**
- Filtered rows containing the min or max value per group using both subquery and window function patterns
- Window Functions are more efficient when filtering across multiple groups simultaneously
- Completed the assignment and solution

**Pivoting**
- Learned how to reshape data from rows to columns using pivot logic in SQL
- Completed the assignment and solution

**Rolling Calculations**
- Covered rolling calculations including running totals and moving averages using window frames
- Completed the full demo, assignment, and solution

**Imputing NULL Values**
- Learned how to handle and replace NULL values in analytical datasets using `COALESCE` and window-based imputation
- Completed the full demo

---

## April 23, 2026

**SQL | Functions by Data Type — Section 7 Completed ✓**
- Covered Function Basics and Numeric Functions
- Learned `CAST()` and `CONVERT()` for explicit type conversion
- Completed the Numeric Functions assignment and solution
- Covered DateTime Functions and completed the assignment and solution
- Covered String Functions and completed the assignment and solution
- Covered Pattern Matching, including a full demo, assignment, and solution
- Covered NULL Functions and completed the assignment and solution
- Reviewed Key Takeaways and passed Quiz 5: Functions by Data Type

>Numeric, DateTime, String, Pattern Matching, and NULL handling are all directly
relevant for cleaning and transforming real-world structured data in pipelines.
These are the functions that appear daily in Feature Engineering and ETL work.

**SQL | Window Functions — Section 6 Completed ✓**

**FIRST_VALUE, LAST_VALUE & NTH_VALUE**
- `FIRST_VALUE()` grabs the first value within a partition — e.g. cheapest product per category
- `LAST_VALUE()` requires an explicit frame — without it, the window only runs up to the current row, not the end of the partition
- Correct frame definition for `LAST_VALUE()`:

```sql
LAST_VALUE(unit_price) OVER (
  PARTITION BY factory
  ORDER BY unit_price
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

- `NTH_VALUE(column, n)` retrieves the nth value within a partition — e.g. second most expensive item
- Key distinction: `FIRST_VALUE` is frame-safe, `LAST_VALUE` and `NTH_VALUE` always require an explicit frame

**What I understood**
- `LAST_VALUE()` without a frame almost always returns wrong results — one of the most common Window Function bugs in production
- `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` means the entire partition, regardless of current row position

**LEAD & LAG — L5 Critical**
- `LAG(column, offset, default)` accesses the value n rows before the current row
- `LEAD(column, offset, default)` accesses the value n rows after the current row
- Default offset is 1 if not specified — `LAG(revenue)` = previous row
- The optional default value prevents `NULL` for the first or last row:

```sql
LAG(revenue, 1, 0) OVER (ORDER BY order_date)
```

- Period-over-Period is the most used Feature Engineering pattern with `LAG`:

```sql
SELECT
  user_id,
  event_date,
  session_count,
  LAG(session_count) OVER (PARTITION BY user_id ORDER BY event_date) AS prev_session_count
FROM sessions;
```

>Today I completed Section 6 in full. All Window Function types are done — from Value Functions to LEAD & LAG, NTILE, Statistical Functions, and Moving Averages. Passed Quiz 4.


**Why this matters for my L5 path**
- `LAG` and `LEAD` are Non-Negotiable for time-based Feature Engineering in production ML pipelines
- Period-over-Period comparisons are used daily in Feature Stores, event tracking, and user behaviour analysis - more Data Analysis on Section 8!
- Correct frame definitions are critical — `LAST_VALUE` without a frame is a silent bug that produces wrong output

---

## April 22, 2026

**SQL | Subqueries & CTEs — Completed ✓**
- Completed Recursive CTEs — anchor member defines the start, recursive part loops until the stop condition is met
- Completed Subqueries vs CTEs vs Temp Tables vs Views, understood when each pattern is appropriate
- Reviewed Key Takeaways and passed Quiz 3: Subqueries & CTEs

>By the end of today, the concepts behind Subqueries and CTEs are fully understood and clear.
>Writing them from scratch is still developing, but the logic and structure already make complete sense.

**SQL | Window Functions — Section 6 started**
- Covered Window Function Basics — the core rule: rows are never collapsed, every row is preserved
- Broke down the full anatomy of a Window Function with `OVER (PARTITION BY ... ORDER BY ...)`
- Completed the Window Functions assignment
- Covered the available function types for Window Functions
- Learned `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` — and the key difference between them
- Completed the Row Numbering assignment
- Learned `FIRST_VALUE()` to capture the first value in a partition
- Learned `LAST_VALUE()` to capture the last value in a partition, with careful attention to the window frame
- Learned `NTH_VALUE()` to retrieve the nth row value from within a partition

>Window Functions felt immediately more intuitive at first glance.
>`PARTITION BY` is just grouping without losing rows. `ORDER BY` inside `OVER` controls rank, not output order.

---

## April 21, 2026

**SQL | Hands-on Practice — Subqueries, CTEs & LeetCode**
- Practiced writing SQL queries independently for the first time without following a course
- Applied CTEs hands-on to solve real query problems — not just watching, but writing
- Practiced LeetCode SQL problems to validate understanding outside of course material
- Understood the concepts behind Subqueries and CTEs clearly — the logic makes sense
- Noticed that translating the concept into actual written SQL is still challenging at this stage — which is completely normal for day 2 of advanced SQL

> Writing SQL from scratch is harder than understanding it — and that gap closes only through repetition and use. This is exactly where I am right now, and it is the right place to be.

**SQL | Subqueries & CTEs**

**Subqueries**
- A subquery is a query inside another query — the inner query always runs first
- Scalar Subquery in `SELECT`: returns exactly one value, used to compare each row against a global aggregate
- Derived Table in `FROM`: treats a subquery result as a temporary table — always needs an alias
- Subquery in `WHERE` / `HAVING`: filters rows based on a dynamically calculated value
- `HAVING` cannot reference `SELECT` aliases — always repeat the aggregate function directly
- `ANY`: condition must be true for at least one value — equivalent to `> MIN()`
- `ALL`: condition must be true for every value — equivalent to `< MIN()`
- `EXISTS`: returns `TRUE` or `FALSE` based on whether a subquery returns any rows at all
- Correlated Subquery: inner query references the outer row — runs once per row, important for performance awareness

**CTEs**
- A CTE is a named temporary result set defined with `WITH name AS (...)` before the main query
- The CTE block is evaluated first — the outer query reads from it like a real table
- Direct equivalent of a Derived Table but far more readable and modular
- A single CTE can be referenced multiple times in the same query — avoids repeating logic
- Multiple CTEs can be chained with commas — each one can build on the previous

```sql
WITH step_one AS (
  SELECT factory, COUNT(*) AS product_count
  FROM products
  GROUP BY factory
),
step_two AS (
  SELECT factory, product_count
  FROM step_one
  WHERE product_count > 2
)
SELECT * FROM step_two
ORDER BY product_count DESC;
```

**Subqueries vs CTEs**
- Subqueries: inline, one-time use, harder to read when nested
- CTEs: named, reusable, easy to debug step by step
- Performance is similar — CTEs win on readability and maintainability
- Rule of thumb: subquery for simple one-liners, CTE for anything more complex

**Why this matters for my L5 path**
- CTEs are the standard query pattern in Spark SQL, dbt, Feast Feature Store, and every modern data pipeline
- Multiple CTEs map directly to pipeline stages — each CTE is one transformation step
- This is not just SQL theory — this is how production Feature Engineering queries are written

>Today I completed the most conceptually challenging chapter of the Maven Advanced SQL course. Subqueries and CTEs are the foundation of every real-world data engineering query pattern.

---

## April 20, 2026

**Advanced SQL | Month 2 Started**
- Officially started Month 2 of the L5 roadmap today
- Chose Maven Advanced SQL Querying as the main SQL course for this month
- Confirmed that Maven is the better fit for my L5 goal because it focuses on advanced querying, data analysis, subqueries, CTEs, window functions, and core SQL syntax
- The course being in MySQL is not a problem, because the concepts transfer directly to PostgreSQL and other relational databases

**SQL | Multi-Table Analysis — Review**
- Reviewed all JOIN types as a knowledge refresh: `INNER JOIN` and `LEFT JOIN` are the most practical, while `RIGHT JOIN` and `FULL JOIN` are less common in day-to-day work
- Revisited joining on multiple columns and joining more than two tables in a single query
- Revisited Self Joins and Cross Joins
- Reviewed `UNION` vs `UNION ALL`
- Completed Quiz 2 to validate the full section
>This was a deliberate review session, not new material.
Tomorrow: Subqueries and CTEs — which I already applied today to fix the `HAVING` alias issue.

**Why this matters for my L5 path**
- JOINs are Non-Negotiable for Spark SQL, Feature Stores, ETL pipelines, and any real multi-table data work
- `UNION` and `UNION ALL` are used constantly when merging data sources in pipelines
- Multi-table joining is a daily pattern in production data infrastructure

**SQL Environment Setup**
- Set up my main SQL environment with TablePlus + Docker + PostgreSQL
- Started a local PostgreSQL container successfully
- Connected to the container through TablePlus
- Imported the Maven course SQL setup into the containerized PostgreSQL database
- Verified that the `create_statements.sql` import worked correctly
- Confirmed that the `students` table exists and contains data

**SQL Learning Insight**
- Ran into an issue when trying to use a `SELECT` alias inside `HAVING`
- Learned that aliases from the `SELECT` clause are not available directly inside `HAVING`
- Fixed the problem by rewriting the query with a CTE
- Understood that the content inside the CTE parentheses is evaluated first
- That made my alias `avg_gpa` available outside the CTE for later filtering
- This made the execution order of SQL much clearer to me

**Why This Matters for My L5 Path**
- PostgreSQL, Docker, TablePlus, advanced querying, and CTEs are all part of building a strong data engineering foundation
- The goal is not just to learn SQL syntax, but to build the ability to query and work with real structured data in a clean, reproducible environment
- This setup is for now more directly aligned with my path toward Data & Feature Infrastructure / ML Platform Engineering

**Next Steps**
- Continue the Maven Advanced SQL course
- Use the Docker + PostgreSQL + TablePlus setup for all exercises
- Import course datasets step by step, only when needed for the current lesson

---

>## Month 2 TIL ⬆️

>*Entries above this line belong to Month 2 starting April 20, 2026.*

---

>## Month 1 TIL ⬇️

>*All entries below this line belong to Month 1.*

---

## April 19, 2026

**Milestone · Month 1 of the L5 Roadmap: COMPLETED ✓**
Month 1 (April 2026) — Python · SQL Basics · Git · Linux — is done.
Month 2 starts tomorrow: SQL Window Functions · Docker · AWS CLF-C02.

---

**Python | Angela Yu Bootcamp — Day 100 · Final Project**

>The final notebook of Angela Yu's 100-Days-of-Code Python Bootcamp was a complete data analysis project using real US education statistics datasets.

**Data Loading & Exploration**
- Loaded multiple CSV files with `pd.read_csv()`
- Inspected structure using `.head()`, `.shape`, `.dtypes`, and `.describe()`
- Renamed columns and reviewed them for relevance

**Data Cleaning — L5 Core Skill**
- Detected missing values with `isna().sum()`
- Applied `fillna()` and `dropna()` based on context rather than blindly
- Checked for and removed duplicates using `duplicated()` and `drop_duplicates()`
- Converted dirty numeric columns using the standard pipeline:
  `astype(str)` → `.str.replace()` → `pd.to_numeric(errors='coerce')`

**Aggregation & Ranking**
- Aggregated data by state with `groupby()` and `.mean()`
- Used `sort_values()` ascending and descending to produce rankings
- Retrieved best and worst performing states using `iloc[0]` and `iloc[-1]`
- Identified the highest and lowest high school completion rates per state

**What I understood**
- The `astype(str)` → `.str.replace()` → `pd.to_numeric()` pattern is the standard approach for messy numeric columns in real-world datasets
- `groupby().mean().sort_values()` is a clean aggregation pipeline for group-level insights
- These patterns apply directly to feature engineering in data pipelines

>**What I deliberately skipped**
>- All visualisations including barplots, scatterplots, and heatmaps — not relevant for Data & Feature Infrastructure work at L5. Time better spent on pipeline logic.

**SQL | Basics Completed ✓**
Today I finished the remaining SQL sections and closed out the full SQL basics chapter.

- Completed Views: `CREATE VIEW`, `ALTER VIEW`, `DROP VIEW`
- Completed `CASE WHEN` practice and challenge exercises
- Completed `COALESCE` — essential for NULL handling in feature pipelines
- Completed `NULLIF` — useful for safe division and NULL conversion patterns
- Completed `SUB-SELECT` / Subqueries — important for complex queries in Spark SQL
- Completed `SELF-JOIN`, `CROSS JOIN`, and `NATURAL JOIN` — full JOIN coverage done
- Reviewed SQL Best Practices section in full
  
>The gap that remains — `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`,
`PARTITION BY`, `LAG/LEAD`, and CTEs — is exactly what Month 2 is for.
That work begins tomorrow with MODE SQL.

## April 18, 2026

**Python | Auto Feature Pipeline Project Prototype**

Today I built and completed the first working version of my `auto-feature-pipeline` project. It now runs end-to-end: it fetches product data from an external API, validates selected fields against a config-driven schema, generates derived features, and exports the final output to both JSON and CSV. For now, the pipeline uses a fallback feature generation path, which still allowed me to test the full architecture successfully even without a live OpenAI API key.

The project is structured in a modular way with separate files for fetching, validation, feature engineering, registry/export, and orchestration. What matters most to me is that this is not just a random script — it is my first small step toward thinking like a Data & Feature Infrastructure / ML Platform Engineer: modular code, reproducible flow, configuration-driven logic, and outputs that can later evolve into a real feature pipeline.

At this stage, the project is intentionally simple. I am still in Month 1 of my roadmap, so my current focus remains on Python, SQL, Git, and Linux fundamentals. The more advanced infrastructure and ML platform topics are planned for later stages, not ignored. This project is simply the foundation.

>**Disclaimer:**  
>The inspiration for this project came from yesterday’s `jikan-feature-pipeline`, which effectively marked the starting point for this direction. That project has already been published on GitHub. If this `auto-feature-pipeline` continues to meet my expectations as I refine it, it will also be released publicly.

>**What this project currently demonstrates**
>- API data ingestion
>- Config-driven validation
>- Modular Python project structure
>- Fallback-based feature generation
>- JSON and CSV export
>- Basic debugging of environment, import, and request issues

>**What I want to improve next**
>- Expand the pipeline to use more input fields such as `title`, `description`, `brand`, `rating`, and `stock`
>- Improve feature generation quality so derived features reflect more than only price and category patterns
>- Add stronger logging and better exception handling
>- Add unit tests for each module
>- Improve documentation and architecture notes
>- Make the pipeline easier to extend for other APIs and schemas

>**Roadmap-aligned future improvements**
>- Docker for reproducible local execution and environment consistency
>- AWS for storage, deployment, and infrastructure thinking
>- Spark for scaling feature computation beyond small local datasets
>- Kafka and Airflow for streaming and orchestration patterns
>- Feast for feature store concepts
>- MLflow and Metaflow for experiment tracking and ML workflow management
>- CI/CD for automated testing and deployment
>- Better data quality checks, schema evolution handling, and performance optimization

**Python | Data Cleaning, Feature Engineering & Time-Series Analysis**
- Cleaned and explored a real dataset with Pandas by handling missing values, duplicates, datetime parsing, and feature creation
- Built a robust `Year` feature from mixed-format date strings and used it for time-based analysis
- Learned that mixed date formats often require more careful parsing than a single strict datetime format
- Used `groupby()`, `value_counts()`, and `size()` to answer practical analysis questions from the dataset
- Created descriptive statistics for both numeric and categorical columns to understand the data structure before visualisation
- Analysed launch activity by organisation, country, year, month, and mission status
- Compared total launches, successful launches, and failed launches over time
- Calculated failure rates over time and added a rolling average to smooth short-term fluctuations
- Built a country-level feature from messy location data and normalised historical or non-standard place names
- Converted country data into ISO-3 codes to enable choropleth mapping
- Created Plotly visualisations including line charts, bar charts, histograms, sunburst charts, pie charts, and choropleth maps
- Explored seasonal patterns in launch activity by month and identified the most and least active months
- Compared launch dominance across organisations over time, including the leading organisation in each year
- Analysed Cold War-era launch competition between the USA and the USSR
- Included former Soviet launch locations like Kazakhstan when analysing historical USSR totals
    
    **Most Relevant for My L5 Data & Feature Infrastructure ML Path**
    - Data cleaning and feature engineering from messy raw data
    - Building reliable time-based features from inconsistent date formats
    - Aggregation logic with `groupby()` for metrics, trends, and comparisons
    - Country normalisation and categorical mapping for analytical consistency
    - Time-series analysis with rolling averages and year-on-year trends
    - Interpreting data quality issues before building metrics or dashboards
    - Using domain context to define correct grouping logic, especially for historical datasets
    - Creating reusable analysis patterns that can be applied in future data and ML infrastructure work
    
    **Less Relevant for My L5 Data & Feature Infrastructure ML Path**
    - Exact Plotly syntax for every chart type
    - The specific styling details of each visualisation
    - Building a choropleth map mainly as a visualisation exercise
    - Memorising all country-code mappings by hand
    - Notebook-specific formatting details that are useful now but less important long term
    
    **What This Taught Me**
    - Data engineering thinking matters more than just plotting
    - Correct feature definitions are often more important than the chart itself
    - Historical datasets need domain-aware cleaning, not blind automation
    - Reusable analysis patterns are more valuable than one-off notebook tricks

**SQL | DROP TABLE, CHECK Constraints & Views**
- Learned how to use `DROP TABLE` to remove database tables when they are no longer needed
- Understood how `CHECK` constraints enforce data rules directly inside the database
- Practiced `CREATE TABLE` in an exercise to define structure and constraints from the start
- Reviewed import and export workflows for moving data in and out of SQL systems
- Explored the concept of views as reusable virtual tables for simplified querying
- Learned how to create a view with `CREATE VIEW`
- Learned how to update a view with `ALTER VIEW` or `REPLACE VIEW`
- Learned how to remove a view with `DROP VIEW`


---

## April 17, 2026

**Python | Requests & APIs / Jikan Feature Pipeline**
- Built a full Python data pipeline split into three clear layers
- Used `requests.get()`, `raise_for_status()`, and `response.json()` to ingest data from the Jikan REST API, which provides top anime data from MyAnimeList
- Validated the structure of the API response and cleaned individual anime records
- Normalised missing or incorrectly typed fields by converting them to `None` or fallback values
- Built derived features from the cleaned raw data:
  - `is_high_score` → boolean flag for score >= 8
  - `popularity_bucket` → categorisation into `very_popular`, `popular`, `mid_popular`, or `niche` using `if/elif/else`
  - `is_long_running` → boolean flag for episode counts greater than 24
  - `genre_count` → number of genres using `len()`
  - `has_multiple_genres` → boolean flag for more than one genre
- Added `None` protection across all feature logic because external APIs do not always return complete data
- Saved the transformed feature records as `feature_records.json` in the project folder using `Path(__file__).resolve().parent`

**First Public GitHub Project**
- Published a first public project on GitHub for the first time
- Combined API fundamentals, feature engineering, JSON output, and GitHub publishing in one day
- Finished the day with a strong sense of progress and a concrete public artifact

**Python | HTTP Requests, APIs & Environment Management**
- Built a PDF-to-Audio pipeline using `PyPDF2` to extract text and the
  ElevenLabs SDK to convert it to speech
- Managed API keys securely using `python-dotenv` and a `.env` file —
  keeping secrets out of source code
- Used `Path(__file__).parent` to load `.env` relative to the script location,
  making the path independent of the working directory
- Debugged a series of 401 API errors by distinguishing between
  `needs_authorization` (key not received) and `invalid_api_key` (wrong key)
- Resolved a mismatched virtual environment by identifying the active `.venv`
  path via `which python`
- Installed `ffmpeg` via Homebrew to enable audio playback through the
  ElevenLabs `play()` function
- Generated test PDFs programmatically using `fpdf` to validate the full
  pipeline end-to-end
- Configured VS Code workspace settings to correctly inject environment
  variables into the terminal

**NOTE | Relevance to Core Goal:**
> Proper environment isolation (`venv`), secret management (`.env`) and API
> error handling are foundational skills for any cloud or data engineering workflow.
> Understanding the difference between authentication and authorisation errors
> is directly applicable to working with cloud APIs, AWS SDKs and
> infrastructure tooling at scale.

**Python | Image Processing & Color Extraction**
- Loaded images as NumPy arrays using PIL and inspected array dimensions to understand height, width, and RGB channels
- Reshaped 3D image arrays `(height, width, 3)` into 2D pixel tables `(total_pixels, 3)` using `reshape(-1, 3)`
- Applied KMeans clustering with `n_clusters=10` to extract the top 10 dominant colors from pixel data
- Converted RGB cluster centers into HEX color codes using string formatting for web-friendly output
- Treated images as feature matrices where each pixel is a data point with 3 features: R, G, and B

**NOTE | Relevance to Core Goal:**
> This follows a useful feature engineering pattern: raw image → pixel feature extraction → clustering → dominant features.
> That pattern is directly relevant to production ML pipelines for dimensionality reduction and feature compression.

---

## April 16, 2026

**Python | Statistical Analysis & Hypothesis Testing**
- Visualised data distributions using histograms and superimposed multiple histograms
  with different sample sizes on the same chart
- Smoothed histogram data with a Kernel Density Estimate (KDE) to reveal the
  underlying distribution shape
- Improved KDE accuracy by specifying boundary constraints on the estimate
- Tested for statistical significance using `scipy` and interpreted p-values to
  determine whether an observed effect is real or due to chance
- Highlighted specific time periods on a Matplotlib time series chart to draw
  attention to key events in the data
- Added and configured a legend in Matplotlib for multi-series charts
- Used `np.where()` to conditionally process array elements without loops

**Python | Multivariable Regression**
- Used `sns.pairplot()` to quickly scan for relationships and correlations
  across all variable pairs in a dataset
- Split data into training and testing sets to evaluate model performance
  on unseen data
- Ran a multivariable regression to model a target variable using
  multiple input features simultaneously
- Evaluated the model by inspecting the sign and magnitude of its coefficients
  to confirm they match real-world logic
- Analysed residuals to detect patterns that indicate model weaknesses
  or violated assumptions
- Applied a log transformation to skewed data to improve regression
  model performance and linearity
- Made predictions by feeding custom input values into the trained model

**NOTE | Data Visualisation (lower priority for core goal):**
> Plotly, Matplotlib and Seaborn visualisation techniques 
> are not a primary focus for the path toward Data & Feature Infrastructure
> and ML Platform Engineering. Charts and visual tooling play a minor role
> in production data systems compared to pipeline engineering, distributed processing,
> and feature store architecture. Covered out of completeness and general data literacy.

**SQL | Tables & Columns**
- Reviewed the role of primary keys and foreign keys in relational database design
- Defined column-level constraints to enforce data integrity rules at the schema level
- Created new tables from scratch using `CREATE TABLE` with defined columns and types
- Practiced table creation with real examples in `CREATE TABLE — Praxis`
- Inserted new records into tables using `INSERT INTO`
- Updated existing records conditionally using `UPDATE ... SET ... WHERE`
- Deleted specific rows from a table using `DELETE FROM ... WHERE`
- Modified an existing table structure with `ALTER TABLE` — adding, renaming and dropping columns
- Practiced `ALTER TABLE` operations hands-on in `ALTER TABLE — Praxis` ✅

---

## April 15, 2026

**Python | NumPy & N-Dimensional Arrays**
- Created arrays manually with `np.array()`, as a number sequence with `np.arange()`,
  with random values using `np.random()`, and with evenly spaced values using `np.linspace()`
- Read array dimensions with `.shape` to understand structure before operations
- Sliced and subsetted multi-dimensional arrays with `arr[0:2, 1:]`
- Applied broadcasting to perform operations between arrays of different shapes
- Ran scalar operations directly on arrays — `arr * 2`, `arr + 10` — without loops
- Performed matrix multiplication with `np.matmul(a1, b1)` and the shorthand `a1 @ b1`
- Located non-zero elements by index using `np.nonzero()`
- Loaded an image with `Image.open()` from PIL and confirmed every image is a
  3D array with shape `(height, width, 3)` representing RGB channels

**Python | Seaborn & Linear Regression**
- Cleaned currency strings across multiple columns using nested loops with
  `.astype(str)` before `.str.replace()` to avoid `AttributeError`
- Ran quick data quality checks with `data.duplicated().sum()` and `data.dtypes`
- Filtered rows with `.loc[]` using multiple conditions combined with `&` and `|`
- Used `.query()` as a cleaner, more readable alternative to `.loc[]` for filtering
- Derived decade values from years using floor division — `year // 10 * 10`
- Plotted a scatter chart with an automatic regression line using `sns.regplot()`
- Applied a pre-built dark grid style with `sns.axes_style('darkgrid')`
- Added a third dimension to a scatter chart by mapping a numeric column to bubble `size`
- Fitted a linear regression model with `LinearRegression().fit(X, y)` from scikit-learn
- Interpreted the R² score as a measure of model fit — 0 means no explanatory power, 1 means perfect fit
- Read regression coefficients to quantify how much revenue increases per dollar of budget
- Concluded that higher budgets correlate with higher revenue but with significant spread

**Python | Capstone Data Analysis Project**
- Investigated and handled NaN values across a large real-world dataset
- Converted object and string columns to numeric types for analysis
- Applied `.value_counts()`, `.groupby()`, `.merge()`, `.sort_values()` and `.agg()` in a full analysis pipeline
- Created a rolling average to smooth time-series data and surface underlying trends
- Used `sns.lmplot()` with `row`, `hue`, and `lowess` parameters to show best-fit lines across multiple categories
- Visualised data distribution and descriptive statistics using a Seaborn histogram
- Compared box plots vs time-series analysis to understand how the same data tells different stories depending on the view

**NOTE | Data Visualisation (lower priority for core goal):**
> Plotly, Matplotlib and Seaborn visualisation techniques 
> are not a primary focus for the path toward Data & Feature Infrastructure
> and ML Platform Engineering. Charts and visual tooling play a minor role
> in production data systems compared to pipeline engineering, distributed processing,
> and feature store architecture. Covered out of completeness and general data literacy.

**SQL | Date & Time Functions**
- Got an overview of all available date and time functions in SQL

**SQL | Data Types**
- Learned how SQL categorises data into types: numeric, text, date/time, boolean
- Understood why correct data types matter for storage efficiency and query performance

---

## April 14, 2026

**Python | Pandas & Matplotlib**
- Used `.describe()` to get a quick statistical summary of a DataFrame
- Converted string columns to proper datetime with `pd.to_datetime()`
- Resampled two DataFrames to the same monthly frequency with `.resample('ME').sum()` and `.asfreq('ME')` to align them for comparison
- Smoothed noisy time series data with `.rolling(window=6).mean()` to reveal underlying trends
- Checked data quality with `.isna().values.sum()` before analysis
- Formatted a date-based x-axis using `YearLocator` and `DateFormatter` from `matplotlib.dates`
- Built a dual y-axis chart with `ax.twinx()` to overlay two datasets with different scales on one figure
- Differentiated two lines visually using `linestyle='--'` and `linestyle='-.'` combined with `'o'` and `'^'` markers
- Set `dpi=200` to export high-resolution charts and fine-tuned layout with `xlim`, `ylim`, `linewidth` and HEX colour codes
- Added `.grid()` to make seasonal patterns easier to read across a long time range

**Key Insight | Google Trends as a Leading Indicator**
- Noticed that search interest in *"unemployment benefits"* spiked weeks **before** the official unemployment rate appeared in FRED data
- Confirmed by the COVID spike in April 2020 where UNRATE hit ~14.7% — web searches predicted it earlier
- Concluded that search volume can be used as a real-time economic signal ahead of official reporting

**SQL | String Functions**
- Combined first and last name columns into a single field using `CONCATENATE`
- Located the position of a character inside a string with `POSITION` to split or validate data
- Extracted specific parts of a string by position and length using `SUBSTRING`
- Measured string length with `LENGTH` for data validation and cleaning
- Standardised casing across inconsistent text data with `LOWER` and `UPPER`

**SQL | Mathematical Functions & Operators**
- Applied `+`, `-`, `*`, `/` directly inside `SELECT` to compute derived columns on the fly
- Rounded query results to two decimal places with `ROUND()` for cleaner output
- Used `CEILING()` and `FLOOR()` to round values up or down to the nearest integer
- Retrieved absolute values from negative numbers with `ABS()` for distance and deviation calculations
- Calculated remainders with `MOD()` to filter even/odd rows and implement modulo-based logic

---

## April 13, 2026

**Python | Data Exploration with Pandas**
- Exploring DataFrames with `.head()`, `.tail()`, `.shape`, `.columns`, `.dtypes`
- Finding NaN values with `.isnull().sum()` and cleaning with `.dropna()` / `.fillna(0)`
- Selecting columns with `df['col']` and `df[['col1', 'col2']]`
- Accessing individual cells with `df['col'][index]` and `.loc[index]`
- Finding min/max values and their positions with `.min()`, `.max()`, `.idxmin()`, `.idxmax()`
- Sorting data with `.sort_values()` and adding new columns with `.insert()`
- Renaming columns with `.rename(columns={...})`
- Aggregating and grouping with `.groupby()` + `.sum()` / `.mean()` / `.count()`

**Python | Data Visualisation with Matplotlib**
- Cleaning date formats with `pd.to_datetime()`
- Reshaping DataFrames with `.pivot(index=, columns=, values=)`
- Replacing missing values after pivot with `.fillna(0, inplace=True)`
- Checking for NaN across entire DataFrame with `.isna().values.any()`
- Creating line charts with `plt.plot(x, y)`
- Styling charts with `.figure(figsize=)`, `.xlabel()`, `.ylabel()`, `.ylim()`, `.legend()`
- Plotting multiple lines in one chart
- Smoothing time series data with `.rolling(window=6).mean()`

**SQL | Comments & UNION**
- Inline comments with `--` and block comments with `/* */`
- `UNION` removes duplicates · `UNION ALL` keeps all rows
- Practice Test 2 completed ✅

