# Today I Learned! Daily Log

A running log of things learned on the journey to becoming an
ML Platform | Data and Feature Infrastructure Engineer.

TIL Started: April 13, 2026.

---

## April 18, 2026

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

