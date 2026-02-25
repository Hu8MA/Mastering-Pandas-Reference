# Mastering Pandas — Part 1: Reading, Sorting & Displaying Data

> **Pandas for Data Science Series — Article #1**

---

## What is Pandas and Why Does It Matter?

Pandas is an open-source Python library built specifically for data manipulation and analysis. Released in 2008 and named after "Panel Data" — a term from econometrics — it has since become one of the most essential tools in the entire Python ecosystem.

At its core, Pandas gives you two powerful data structures:
- **Series** — a single column of data
- **DataFrame** — a full table with rows and columns (think of it as a turbocharged spreadsheet you control with code)

Whether you're working in **data science, machine learning, financial analysis, or business intelligence**, chances are you'll be loading, exploring, and transforming data with Pandas before you do anything else. It integrates seamlessly with libraries like NumPy, Matplotlib, Seaborn, Scikit-learn, and TensorFlow, making it the essential starting point in nearly every data workflow.

In this first article of the series, you'll cover two essential skill sets: **reading data into Pandas** from virtually any file format, and **sorting and displaying** that data in ways that let you understand it quickly.

Let's get started.

---

# Part 1 — Reading Files into Pandas

Before you can analyze anything, you need to load your data. Pandas makes this simple with a family of `read_*()` functions — one for almost every file format you'll encounter in the real world.

---

## 1️⃣ CSV File — Most Common

CSV (Comma-Separated Values) files are the bread and butter of data work. `read_csv()` is almost certainly the first Pandas function you'll use in any project.

```python
import pandas as pd

df = pd.read_csv("file.csv")                          # basic read
df = pd.read_csv("file.csv", header=0)                # first row as header
df = pd.read_csv("file.csv", index_col="order_id")    # set column as index
df = pd.read_csv("file.csv", nrows=100)               # read only first 100 rows
df = pd.read_csv("file.csv", skiprows=2)              # skip first 2 rows
df = pd.read_csv("file.csv", usecols=["price","quantity"])  # specific columns only
df = pd.read_csv("file.csv", na_values=["NA","?"])    # define missing values
df = pd.read_csv("file.csv", sep=";")                 # semicolons instead of commas
```

---

## 2️⃣ Excel File

Excel files are ubiquitous in business environments. `read_excel()` handles them cleanly, and you can target specific sheets by name or position.

```python
df = pd.read_excel("file.xlsx")                       # basic read
df = pd.read_excel("file.xlsx", sheet_name="Sheet1")  # specific sheet by name
df = pd.read_excel("file.xlsx", sheet_name=0)         # first sheet by index
df = pd.read_excel("file.xlsx", skiprows=2)           # skip first 2 rows
df = pd.read_excel("file.xlsx", usecols="A:D")        # read columns A to D
```

---

## 3️⃣ JSON File

JSON is the standard format for web APIs. `read_json()` can parse various JSON structures, including a flat list of records.

```python
df = pd.read_json("file.json")                        # basic read
df = pd.read_json("file.json", orient="records")      # list of records format
```

---

## 4️⃣ HTML File — Web Scraping Made Easy

One of Pandas' more surprising capabilities: it can extract tables directly from HTML pages, including live websites. Simple web scraping with zero extra libraries.

```python
tables = pd.read_html("file.html")                          # returns list of all tables
df     = pd.read_html("https://website.com/table")[0]       # first table from a URL
```

---

## 5️⃣ SQL Database

When your data lives in a database, Pandas can query it directly. You bring a connection object and a SQL query, and it returns a DataFrame.

```python
import sqlite3

conn = sqlite3.connect("database.db")

df = pd.read_sql("SELECT * FROM table", conn)    # full SQL query
df = pd.read_sql_table("table_name", conn)       # read entire table directly
```

---

## 6️⃣ Text File

`read_table()` is the text-file counterpart to `read_csv()`. The only real difference is its default separator — a tab instead of a comma — but it accepts all the same parameters.

```python
df = pd.read_table("file.txt")                      # tab separated (default)
df = pd.read_table("file.txt", sep=",")             # comma separated
df = pd.read_table("file.txt", sep=";")             # semicolon separated
df = pd.read_table("file.txt", sep="|")             # pipe separated
df = pd.read_table("file.txt", header=None)         # file has no header row
df = pd.read_table("file.txt", names=["col1","col2"])  # add column names manually
df = pd.read_table("file.txt", skiprows=2)          # skip first 2 rows
df = pd.read_table("file.txt", nrows=100)           # read only 100 rows
df = pd.read_fwf("file.txt")                        # fixed-width text file
```

> **`read_csv()` vs `read_table()`** — The only practical difference is the default separator: `read_csv()` uses a comma (`,`) while `read_table()` uses a tab (`\t`). Every other parameter works identically in both.

| | `read_csv()` | `read_table()` |
|---|---|---|
| Default separator | `,` comma | `\t` tab |
| File type | CSV files | TXT files |
| Speed | Same | Same |
| Parameters | Same | Same |

---

## 7️⃣ Other Formats

```python
# Clipboard — copy any table, then run this
df = pd.read_clipboard()

# Parquet — the preferred format for big data (very fast)
df = pd.read_parquet("file.parquet")

# XML
df = pd.read_xml("file.xml")
```

---

## Summary — All Read Functions at a Glance

| Function | File Type | Common Use |
|---|---|---|
| `read_csv()` | CSV | Most common — daily use |
| `read_excel()` | XLSX | Excel files from business |
| `read_json()` | JSON | Web APIs and REST data |
| `read_html()` | HTML | Web scraping tables |
| `read_sql()` | Database | SQL queries |
| `read_table()` | TXT | Tab-separated text files |
| `read_clipboard()` | Clipboard | Quick copy-paste workflow |
| `read_parquet()` | Parquet | Large-scale / big data |
| `read_xml()` | XML | Structured XML data |

---

## Most Important Parameters — Work With ALL Functions

```python
nrows      →  how many rows to read
skiprows   →  how many rows to skip at the top
usecols    →  which columns to read
index_col  →  which column to use as index
na_values  →  what to treat as missing value (NaN)
header     →  which row to use as column header
sep        →  what separator/delimiter to use
```

---

# Part 2 — Sorting and Displaying Data

Once your data is loaded, the first thing you want to do is *understand* it. The functions in this section are your primary tools for exploring structure, distributions, relationships, and ordering — before you write a single line of analysis.

We'll use this sample DataFrame throughout all the examples:

```python
import pandas as pd

data = {
    'Country':          ['China', 'India', 'USA', 'Brazil', 'UK'],
    'Continent':        ['Asia', 'Asia', 'North America', 'South America', 'Europe'],
    '2022 Population':  [1412000000, 1380000000, 331000000, 214000000, 67000000],
    'Area (km2)':       [9597000, 3287000, 9834000, 8516000, 243000]
}
df = pd.DataFrame(data)
```

---

## Sorting Functions

### 1. sort_values()

The most commonly used sorting function. Reorders the DataFrame by the values in one or more columns. By default it sorts ascending; pass `ascending=False` to flip it.

```python
# Ascending order (default)
df.sort_values('2022 Population')

# Descending order
df.sort_values('2022 Population', ascending=False)

# Sort by multiple columns: Continent A→Z, then Population largest first
df.sort_values(['Continent', '2022 Population'], ascending=[True, False])
```

---

### 2. sort_index()

Sorts by the DataFrame's index rather than column values. This becomes important after filtering, merging, or shuffling rows, which can leave the index disordered.

```python
df.sort_index()                # sort index ascending
df.sort_index(ascending=False) # sort index descending
```

---

### 3. nlargest()

Returns the top N rows with the largest values in a specified column. More efficient and more readable than combining `sort_values()` with `head()`.

```python
# Top 3 most populated countries
df.nlargest(3, '2022 Population')
```

---

### 4. nsmallest()

Returns the top N rows with the smallest values in a specified column.

```python
# 3 smallest countries by area
df.nsmallest(3, 'Area (km2)')
```

---

### 5. rank()

Assigns a rank to each row based on a column's values, without changing the row order. Useful for adding a ranking column to your existing DataFrame.

```python
# Add a population rank column (1 = largest)
df['Population Rank'] = df['2022 Population'].rank(ascending=False)
```

---

## Displaying Functions

### 1. head() & tail()

The simplest inspection tools. `head()` shows the first N rows (default 5) and `tail()` shows the last N rows. These are usually the first things you run on any new dataset.

```python
df.head()    # first 5 rows
df.head(10)  # first 10 rows
df.tail()    # last 5 rows
df.tail(3)   # last 3 rows
```

---

### 2. sample()

Returns N randomly selected rows. Ideal for getting a representative look at a large dataset where the first or last rows might not be representative.

```python
df.sample(3)  # 3 random rows
```

---

### 3. info()

Displays the DataFrame's structure: column names, data types, number of non-null values, and memory usage. This should be one of the very first functions you call on any new dataset — it immediately reveals missing data and wrong data types.

```python
df.info()
```

---

### 4. describe()

Returns a statistical summary for all numeric columns: count, mean, standard deviation, min, max, and quartiles. A quick way to understand the scale and distribution of your data.

```python
df.describe()
```

---

### 5. value_counts()

Returns the frequency of each unique value in a column. Invaluable for understanding the distribution of categorical data at a glance.

```python
# How many countries exist per continent?
df['Continent'].value_counts()
```

---

### 6. corr()

Calculates the correlation between all numeric columns. Returns values from **-1.0** (perfect negative relationship) to **1.0** (perfect positive relationship), with **0.0** meaning no relationship.

> **Important:** `corr()` requires numeric columns only. Always use `select_dtypes()` before calling it to avoid a `ValueError`.

```python
# Safe pattern — works on any DataFrame
df.select_dtypes(include='number').corr()

# Or select specific columns manually
df[['2022 Population', 'Area (km2)']].corr()
```

---

### 7. select_dtypes()

Filters and returns only the columns that match a specified data type. Useful when you need to work on a subset of columns without knowing their names in advance.

```python
df.select_dtypes(include='number')              # numeric columns only (int and float)
df.select_dtypes(include='object')              # string columns only
df.select_dtypes(include='bool')                # boolean columns only
df.select_dtypes(include=['int64', 'float64'])  # specific numeric types
df.select_dtypes(exclude='object')              # drop all string columns
df.select_dtypes(exclude='number')              # drop all numeric columns
df.select_dtypes(include='number', exclude='bool')  # numeric but not boolean
```

---

### 8. Select Specific Columns

Returns a new DataFrame containing only the columns you need. Helps reduce output to what is relevant.

```python
df[['Country', '2022 Population']]
```

---

## Matplotlib Global Settings — plt.rcParams

While not a Pandas function, `plt.rcParams` is something you'll configure right alongside your Pandas setup. It controls the default appearance of all plots in your session — set it once at the top of your notebook and every chart inherits those settings automatically.

```python
import matplotlib.pyplot as plt

plt.rcParams['figure.figsize'] = (20, 8)   # 20 wide, 8 tall (in inches)
plt.rcParams['font.size']       = 14       # default font size for all text
plt.rcParams['figure.dpi']      = 100      # resolution (higher = sharper)
plt.rcParams['lines.linewidth'] = 2        # default line width
plt.rcParams['axes.grid']       = True     # show grid on all plots by default
```

---

## Common Patterns

### Filter, then Sort

The most frequent real-world pattern: narrow the dataset to a subset, then rank within that subset.

```python
# Top 3 Asian countries by population
df[df['Continent'] == 'Asia'].nlargest(3, '2022 Population')
```

### Sort and Preview

Both lines produce the same result — the second is preferred for its clarity.

```python
df.sort_values('2022 Population', ascending=False).head(10)
df.nlargest(10, '2022 Population')  # ✅ preferred
```

### Clean Display of Numbers

Control how many decimal places are shown across the entire session.

```python
pd.set_option('display.float_format', '{:.2f}'.format)
```

---

## Complete Summary Table

| Function | Category | Purpose |
|---|---|---|
| `sort_values()` | Sorting | Sort by column values |
| `sort_index()` | Sorting | Sort by index |
| `nlargest()` | Sorting | Get top N largest values |
| `nsmallest()` | Sorting | Get top N smallest values |
| `rank()` | Sorting | Assign rank to each row |
| `head()` | Displaying | Show first N rows |
| `tail()` | Displaying | Show last N rows |
| `sample()` | Displaying | Show random N rows |
| `info()` | Displaying | Show structure and data types |
| `describe()` | Displaying | Statistical summary |
| `value_counts()` | Displaying | Count frequency of unique values |
| `corr()` | Displaying | Correlation between numeric columns |
| `select_dtypes()` | Displaying | Filter columns by data type |
| `plt.rcParams` | Matplotlib | Set global defaults for all plots |

---

*This is Part 1 of the Pandas for Data Science series. Next up: **Part 2 — Filtering, Selecting & Cleaning Data.***

---

**Tags:** `Python` `Pandas` `Data Science` `Machine Learning` `Data Analysis` `Tutorial` `Programming`
