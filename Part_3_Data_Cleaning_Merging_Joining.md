# Mastering Pandas — Part 3: Data Cleaning, Merging & Joining

> **Pandas for Data Science Series — Article #3**

---

## Real Data Is Never Clean

In Part 2, you learned how to group and slice a DataFrame with precision. But in real projects, the data you receive rarely arrives ready to analyze. Column names have spaces. Ages are negative. Phone numbers mix letters with digits. Duplicate rows sneak in. And your data often lives across multiple files that need to be combined before you can do anything useful.

This article covers the two skills that bridge raw data and real analysis:

- **Data Cleaning** — detecting and fixing everything wrong with your data before it corrupts your results
- **Merging & Joining** — combining separate DataFrames into a single, unified dataset

These topics flow naturally into each other. You clean each source first, then you combine them. That's the order every data analyst follows in practice — and it's the order we'll follow here.

We'll use this sample dataset throughout the cleaning section:

```python
import pandas as pd
import numpy as np

data = {
    "Name":             ["Alice", "bob", "Carol", "Alice", "  dave  ", "Eve"],
    "Age":              [25, -5, 200, 25, 30, None],
    "Phone":            ["055-123-4567", None, "55123x", "055-123-4567", "055-987-0000", "055abc"],
    "Email":            ["alice@mail.com", "bob@mail.com", None, "alice@mail.com", "dave@mail.com", "not-an-email"],
    "Paying_Customer":  ["Y", "N", "Yes", "Y", "N", "y"],
    "Score":            [85, 110, 72, 85, -10, 90],
}
df = pd.DataFrame(data)
```

---

## Step 1 — Inspect Before You Touch Anything

The most important rule in data cleaning: **look before you act**. Jumping straight into fixes without understanding the full picture leads to mistakes you won't catch until much later.

These are the inspection tools you should run on every new dataset:

```python
df.shape          # (rows, columns) — how big is this?
df.info()         # column names, data types, non-null counts
df.head()         # first 5 rows
df.tail()         # last 5 rows
df.describe()     # min, max, mean, std — spot impossible values here
```

For missing values specifically:

```python
df.isna().sum()           # count NaN per column
df.isnull().sum()         # same as isna() — both work
df[df["Phone"].notna()]   # rows that actually have a phone number
```

For duplicates and inconsistent categories:

```python
df.duplicated()                         # True/False per row
df[df.duplicated()]                     # show only the duplicate rows
df["Paying_Customer"].value_counts()    # reveals: Y, N, Yes, y — all meaning the same thing
```

> **Always run `describe()` on numeric columns** — the min and max alone will tell you immediately if something is wrong. A minimum age of -5 or a score of 110 out of 100 is a red flag you can't miss.

---

## Step 2 — Fix Column Names First

Messy column names make everything harder. Clean them before anything else, so every subsequent operation uses predictable, consistent names.

**Rename specific columns:**

```python
df.rename(columns={"Old Name": "New_Name", "phone number": "Phone"})
```

**Clean all column names at once — the standard one-liner:**

```python
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```

This strips surrounding spaces, converts to lowercase, and replaces spaces with underscores in a single chain. After this, `"First Name"` becomes `"first_name"`, `"  Age "` becomes `"age"`, and so on.

```
Before:              After:
"First Name"    →    "first_name"
"  Age "        →    "age"
"Phone Number"  →    "phone_number"
"ZIP Code"      →    "zip_code"
```

---

## Step 3 — Handle Missing Values

Missing values appear as `NaN` in Pandas. Before deciding what to do with them, you need to understand *why* they're missing — because the right fix depends on the cause.

**Fill missing values with a fixed value:**

```python
df["Phone"].fillna("000-000-0000")    # placeholder for missing phone
df["Age"].fillna(0)                   # fill numeric with 0
df["Name"].fillna("Unknown")          # fill text with label
df["Score"].fillna(df["Score"].mean()) # fill with column average
```

**Fill missing numeric values by estimating from neighbors:**

```python
df["Sales"] = df["Sales"].interpolate()
```

`interpolate()` fills NaN by calculating the midpoint between the values above and below it. This is ideal for time series or sensor data where values change gradually — it would be wrong to use for random datasets.

```
Before:     After interpolate():
100         100
NaN    →    150   ← estimated midpoint
200         200
NaN    →    225   ← estimated midpoint
250         250
```

**Remove rows with missing values:**

```python
df.dropna()                            # drop any row with at least one NaN
df.dropna(subset=["Phone"])            # drop only if Phone is NaN
df.dropna(thresh=3)                    # drop rows that have fewer than 3 non-NaN values
```

> **`inplace` rule:** use either `inplace=True` or variable assignment — never both. They do the same thing, and combining them produces a bug where `df` becomes `None`.
> ```python
> df.dropna(subset=["Phone"], inplace=True)   # ✅ modifies df directly
> df = df.dropna(subset=["Phone"])            # ✅ returns new df
> df = df.dropna(subset=["Phone"], inplace=True)  # ❌ df becomes None
> ```

---

## Step 4 — Remove Duplicates

Duplicate rows silently inflate counts, averages, and totals. Always check for them.

```python
df.drop_duplicates()                         # remove fully identical rows
df.drop_duplicates(subset=["Name"])          # remove rows with duplicate Name only
df.drop_duplicates(subset=["Name", "Email"]) # remove rows where both columns match
df.drop_duplicates(keep="last")              # keep the last occurrence instead of the first
```

```
Before:                        After drop_duplicates():
Name   Email                   Name   Email
Alice  alice@mail.com          Alice  alice@mail.com
Bob    bob@mail.com      →     Bob    bob@mail.com
Alice  alice@mail.com ← dup    Carol  carol@mail.com
Carol  carol@mail.com
```

---

## Step 5 — Fix Data Types

Pandas sometimes reads numeric columns as strings, or dates as plain text. Operations on wrong types either crash or silently produce wrong results.

**Change column type directly:**

```python
df["Zip_Code"] = df["Zip_Code"].astype(str)     # treat as text, not a number
df["Age"] = df["Age"].astype(int)               # convert to integer
df["Price"] = df["Price"].astype(float)         # convert to decimal
```

**Handle messy numbers that contain symbols or text:**

```python
df["Price"] = pd.to_numeric(df["Price"], errors="coerce")
```

`errors="coerce"` converts anything that can't be parsed into `NaN` instead of crashing. This is the safe way to clean columns like `"$1,200"` or `"N/A"` that can't be directly cast with `astype()`.

```
Before:           After pd.to_numeric(..., errors="coerce"):
"1200"    →       1200.0
"$850"    →       NaN     ← couldn't parse
"700.5"   →       700.5
"N/A"     →       NaN     ← couldn't parse
```

**Convert to datetime:**

```python
df["Date"] = pd.to_datetime(df["Date"])
df["Date"] = pd.to_datetime(df["Date"], format="%d/%m/%Y")  # specify format if needed
```

Once a column is proper datetime, you unlock sorting by date, filtering by year/month, calculating time differences, and much more.

---

## Step 6 — Clean Strings

String columns are almost always the messiest part of a real dataset. Pandas provides a full `.str` accessor with everything you need.

**Strip unwanted characters from edges:**

```python
df["Name"] = df["Name"].str.strip()           # remove spaces from both sides
df["Name"] = df["Name"].str.strip(" ./-")     # remove spaces, dots, slashes, dashes
df["Name"] = df["Name"].str.lstrip()          # left side only
df["Name"] = df["Name"].str.rstrip()          # right side only
```

**Replace characters anywhere in the string:**

```python
df["Phone"] = df["Phone"].str.replace(r"[a-zA-Z]", "", regex=True)   # remove all letters
df["Phone"] = df["Phone"].str.replace(r"\D", "", regex=True)          # keep digits only
df["Name"]  = df["Name"].str.replace(r"[./]", "", regex=True)         # remove dots and slashes
```

**Standardize text case:**

```python
df["Name"] = df["Name"].str.lower()     # all lowercase → "alice"
df["Name"] = df["Name"].str.upper()     # all uppercase → "ALICE"
df["Name"] = df["Name"].str.title()     # title case    → "Alice"
```

**Check text content:**

```python
df[df["Email"].str.contains("@")]           # filter valid-looking emails
df[df["Phone"].str.len() != 12]             # find rows where phone length is wrong
df[df["Name"].str.startswith("A")]          # names starting with A
df[df["Name"].str.endswith("e")]            # names ending with e
```

**Extract a pattern using regex:**

```python
df["Zip"] = df["Address"].str.extract(r"(\d{5})")        # pull 5-digit zip code
df["Area_Code"] = df["Phone"].str.extract(r"^(\d{3})")   # pull first 3 digits
```

**Split one column into multiple columns:**

```python
df[["Street", "State", "Zip"]] = (
    df["Address"]
    .str.split(",", n=2, expand=True)
)
```

`n=2` limits the split to 2 times, producing exactly 3 parts regardless of how many commas the string contains. `expand=True` spreads those parts into separate columns.

```
Before:                              After:
"123 Main St, Texas, 75001"   →   Street="123 Main St"  State="Texas"  Zip="75001"
```

---

## Step 7 — Replace and Standardize Values

Raw data often has the same value written multiple ways: `"Y"`, `"Yes"`, `"yes"`, `"y"` — all meaning the same thing. Standardize them.

**Replace whole cell values using a dictionary:**

```python
df["Status"] = df["Status"].replace({"Y": "Yes", "N": "No", "y": "Yes"})

# Apply to multiple columns at once
df[["Paying_Customer", "Do_Not_Contact"]] = (
    df[["Paying_Customer", "Do_Not_Contact"]]
    .replace({"Y": "Yes", "N": "No", "y": "Yes", "n": "No"})
)
```

**Overwrite invalid values using `loc[]`:**

```python
df.loc[df["Age"] < 0,   "Age"] = None     # negative age → invalid
df.loc[df["Age"] > 120, "Age"] = None     # age over 120 → invalid
df.loc[df["Price"] < 0, "Price"] = None   # negative price → invalid
```

**Keep valid values, replace everything else with NaN:**

```python
df["Salary"] = df["Salary"].where(df["Salary"] > 0)
# Rows where Salary <= 0 become NaN; the rest are untouched
```

---

## Step 8 — Handle Outliers

An outlier is a value that exists but shouldn't — a score of 110/100, an age of 200, a price of -50. `clip()` is the clean way to enforce valid ranges.

**Cap values within a valid range:**

```python
df["Score"] = df["Score"].clip(0, 100)      # 110 → 100, -10 → 0
df["Age"]   = df["Age"].clip(0, 120)        # 200 → 120, -5  → 0
df["Age"]   = np.clip(df["Age"], 0, 120)    # NumPy version — same result
```

```
Before:   After clip(0, 100):
85        85
110   →   100   ← capped at max
-10   →   0     ← capped at min
72        72
```

> `clip()` is gentler than dropping rows — it preserves the row but brings the outlier within the valid boundary. Use it when the row has useful data in other columns that you don't want to lose.

---

## Step 9 — Drop Rows and Columns

Sometimes the cleanest fix is simply to remove what you don't need.

**Drop rows by index:**

```python
df.drop(0)              # remove row at index 0
df.drop([0, 2, 5])      # remove multiple rows by index
```

**Drop columns:**

```python
df.drop(columns=["Age", "Phone"])      # preferred style
df.drop("Age", axis=1)                 # axis=1 means "columns"
df.drop(["Age", "Phone"], axis=1)      # multiple columns
```

---

## Step 10 — Create New Columns

Cleaning isn't only about removing bad data — it's also about building the right structure for analysis.

**Fixed value:**

```python
df["Source"] = "CSV Import"    # tag every row with its data source
```

**Calculated from another column:**

```python
df["Price_With_Tax"] = df["Price"] * 1.15
df["Name_Length"]    = df["Name"].str.len()
```

**Based on a condition:**

```python
df["Status"] = np.where(df["Age"] >= 18, "Adult", "Minor")
```

**Based on multiple conditions:**

```python
conditions = [df["Score"] >= 90, df["Score"] >= 70, df["Score"] >= 50]
choices    = ["A", "B", "C"]
df["Grade"] = np.select(conditions, choices, default="F")
```

**Custom logic with `apply()`:**

```python
df["Name"] = df["Name"].apply(lambda x: x.title() if isinstance(x, str) else x)

def categorize_price(price):
    if price > 1000: return "Expensive"
    elif price > 500: return "Medium"
    else:            return "Cheap"

df["Category"] = df["Price"].apply(categorize_price)
```

---

## Step 11 — Add New Rows

**Add a single row using `concat()`:**

```python
new_row = pd.DataFrame([{"Name": "Frank", "Age": 28, "Phone": "055-111-2222"}])
df = pd.concat([df, new_row], ignore_index=True)
```

**Add multiple rows at once:**

```python
new_rows = pd.DataFrame([
    {"Name": "Grace", "Age": 31},
    {"Name": "Hank",  "Age": 24},
])
df = pd.concat([df, new_rows], ignore_index=True)
```

`ignore_index=True` resets the index after concatenation so there are no gaps or repeated numbers:

```
Without ignore_index:    With ignore_index=True:
0  Alice                 0  Alice
1  Bob          →        1  Bob
0  Grace                 2  Grace   ← clean
1  Hank                  3  Hank    ← clean
```

---

## Step 12 — Reset the Index

After dropping rows, the index has gaps. Reset it before analysis or before passing data to another function.

```python
df = df.reset_index(drop=True)
```

`drop=True` discards the old index numbers entirely. `drop=False` keeps the old index as a new column — useful if you need to trace where a row originally came from.

---

## Putting It All Together — A Real Cleaning Pipeline

Here's what a complete cleaning workflow looks like from start to finish:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data.csv")

# 1. Inspect
print(df.info())
print(df.isna().sum())
print(df.describe())

# 2. Fix column names
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")

# 3. Drop useless columns and duplicates
df.drop(columns=["not_useful"], inplace=True)
df.drop_duplicates(inplace=True)

# 4. Handle missing values
df.dropna(subset=["phone_number"], inplace=True)
df["email"].fillna("unknown@unknown.com", inplace=True)
df["sales"] = df["sales"].interpolate()

# 5. Fix data types
df["date"]     = pd.to_datetime(df["date"])
df["price"]    = pd.to_numeric(df["price"], errors="coerce")
df["zip_code"] = df["zip_code"].astype(str)

# 6. Clean strings
df["name"] = df["name"].str.strip().str.title()
df["phone_number"] = (
    df["phone_number"]
    .str.replace(r"\D", "", regex=True)
    .str.replace(r"(\d{3})(\d{3})(\d{4})", r"\1-\2-\3", regex=True)
    .replace("", "000-000-0000")
)

# 7. Fix invalid values and outliers
df.loc[df["age"] < 0, "age"] = None
df["score"] = df["score"].clip(0, 100)

# 8. Standardize categories
df["status"] = df["status"].replace({"Y": "Yes", "N": "No", "y": "Yes"})

# 9. Filter rows
df = df[df["do_not_contact"] != "Yes"]

# 10. Reset index
df = df.reset_index(drop=True)

print(df.info())
print(df.head())
```

---

## Now Combine — Merging & Joining DataFrames

Once each source is clean, the next challenge is combining them. In the real world, data almost never lives in a single file. You might have orders in one table, customers in another, and products in a third. Merging is how you bring those pieces together.

Pandas provides three main tools for this:

| Tool | What it does |
|---|---|
| `merge()` | Joins two DataFrames by matching on a shared column |
| `concat()` | Stacks DataFrames vertically or side by side |
| `join()` | Joins on the index instead of a column |

We'll use these two DataFrames for all the examples:

```python
customers = pd.DataFrame({
    "ID":        [1, 2, 3, 4],
    "Name":      ["Alice", "Bob", "Carol", "Dave"],
    "Country":   ["USA", "UK", "India", "USA"],
})

orders = pd.DataFrame({
    "ID":        [1, 2, 3, 5],
    "Product":   ["Laptop", "Phone", "Tablet", "Monitor"],
    "Price":     [1200, 800, 600, 400],
})
```

Notice that `ID=4` (Dave) exists in `customers` but not in `orders`, and `ID=5` exists in `orders` but not in `customers`. This is exactly the kind of mismatch you encounter in practice — and why the `how=` parameter matters so much.

---

## 1️⃣ `merge()` — The Main Tool

`merge()` works like a SQL JOIN. It aligns two DataFrames by finding rows where a shared column has the same value.

**Basic syntax:**

```python
customers.merge(orders, how="inner", on="ID")
```

### The `how=` Parameter — All Four Types

```python
# INNER → only rows that match in both DataFrames (default)
customers.merge(orders, how="inner", on="ID")
# Result: IDs 1, 2, 3 only — Dave (4) and Monitor (5) are excluded

# LEFT → all rows from the left + matched rows from the right
customers.merge(orders, how="left", on="ID")
# Result: all 4 customers — Dave gets NaN for Product and Price

# RIGHT → all rows from the right + matched rows from the left
customers.merge(orders, how="right", on="ID")
# Result: all 4 orders — Monitor (ID=5) gets NaN for Name and Country

# OUTER → all rows from both, NaN where no match
customers.merge(orders, how="outer", on="ID")
# Result: all 5 unique IDs — gaps filled with NaN on both sides
```

| `how=` | Keeps from Left | Keeps from Right | NaN Possible |
|---|---|---|---|
| `inner` | Matched only | Matched only | No |
| `left` | All rows | Matched only | Right side |
| `right` | Matched only | All rows | Left side |
| `outer` | All rows | All rows | Both sides |

**CROSS merge — every row with every row:**

```python
customers.merge(orders, how="cross")
# Produces 4 × 4 = 16 rows — every customer paired with every order
# No on= parameter — it will raise an error if you include it
```

---

### The `on=` Parameter — What to Match On

```python
# Match on a single column
customers.merge(orders, how="inner", on="ID")

# Match on multiple columns — both must match simultaneously
customers.merge(orders, how="inner", on=["ID", "Country"])

# Auto-detect — Pandas finds all shared column names automatically
customers.merge(orders)
```

> **Be careful with auto-detect.** If both DataFrames happen to share a column that you don't intend to join on (like a generic `"Name"` column), Pandas will include it in the match condition and produce unexpected results. Explicit is safer.

---

### Handling Shared Column Names — `_x` and `_y`

If both DataFrames have a column with the same name, but you only join on `ID`, Pandas keeps both copies and renames them automatically:

```python
# Both DataFrames have a "Name" column, but you only merge on "ID"
customers.merge(orders, how="inner", on="ID")
# Result: Name_x (from customers), Name_y (from orders)
```

Three ways to fix this:

```python
# Option 1 — merge on all shared columns so no duplicates appear
customers.merge(orders, how="inner", on=["ID", "Name"])

# Option 2 — use custom suffixes instead of _x and _y
customers.merge(orders, how="inner", on="ID", suffixes=("_customer", "_order"))

# Option 3 — drop the shared column from one DataFrame before merging
orders_clean = orders.drop(columns=["Name"])
customers.merge(orders_clean, how="inner", on="ID")
```

---

### Other Useful Parameters

```python
# left_on / right_on — when the key column has different names in each DataFrame
customers.merge(orders, how="inner", left_on="CustomerID", right_on="OrderID")

# indicator — adds a column showing where each row came from
customers.merge(orders, how="outer", on="ID", indicator=True)
# _merge column values: "left_only", "right_only", "both"
```

The `indicator=True` parameter is especially useful for auditing: after a merge, you can immediately see which rows matched, which came only from the left, and which came only from the right.

---

## 2️⃣ `concat()` — Stacking DataFrames

While `merge()` combines DataFrames horizontally by matching values, `concat()` simply stacks them — either rows on top of rows, or columns side by side.

```python
pd.concat([df1, df2])        # stack vertically (default)
pd.concat([df1, df2], axis=1)  # stack horizontally
```

### `axis=` Parameter

```python
# axis=0 → stack rows vertically (append one DataFrame below another)
monthly_data = pd.concat([january, february, march], axis=0)

# axis=1 → stack columns side by side
combined = pd.concat([names_df, scores_df], axis=1)
```

### `join=` Parameter

```python
# outer (default) → keep all columns, NaN where a column doesn't exist in one source
pd.concat([df1, df2], join="outer")

# inner → keep only columns that exist in all DataFrames
pd.concat([df1, df2], join="inner")
```

### `ignore_index=` Parameter

```python
# Without ignore_index — original indices are preserved, which can repeat
pd.concat([df1, df2])
# Index: 0, 1, 2, 0, 1, 2  ← 0, 1, 2 repeats

# With ignore_index=True — fresh sequential index
pd.concat([df1, df2], ignore_index=True)
# Index: 0, 1, 2, 3, 4, 5  ← clean
```

Always use `ignore_index=True` after stacking unless you specifically need the original indices.

### `keys=` Parameter

```python
# Label each source so you can identify where each row came from
combined = pd.concat([df1, df2], keys=["source_a", "source_b"])
```

This creates a MultiIndex where the first level is your label and the second is the original row index — useful when you need to trace data back to its source after combining.

### `verify_integrity=` Parameter

```python
# Raises an error if any duplicate index values are found after concatenation
pd.concat([df1, df2], verify_integrity=True)
```

---

### `merge()` vs `concat()` — When to Use Which

| | `merge()` | `concat()` |
|---|---|---|
| Direction | Horizontal (by matching) | Vertical or horizontal (by stacking) |
| Needs a shared key column | Yes | No |
| Best for | Joining related tables | Combining same-structure data |
| Equivalent to | SQL JOIN | SQL UNION |

A simple rule: if you're **combining columns from different tables**, use `merge()`. If you're **stacking more rows of the same structure**, use `concat()`.

---

## 3️⃣ `join()` — Index-Based Joining

`join()` works like `merge()`, but it joins on the **index** instead of a regular column. You need to set the index first.

```python
customers_indexed = customers.set_index("ID")
orders_indexed    = orders.set_index("ID")

customers_indexed.join(orders_indexed, how="inner")
```

The `how=` options are the same as `merge()`: `inner`, `left`, `right`, `outer`.

### Why `merge()` Is Almost Always Better

| | `merge()` | `join()` |
|---|---|---|
| Join on any column | ✅ Yes | ❌ No — index only |
| Join on index | ✅ Yes | ✅ Yes |
| Requires `set_index()` first | ❌ No | ✅ Yes |
| Simpler syntax | ✅ Yes | ⚠️ Extra steps |

> Use `join()` only when your data is already indexed — for example, after a `groupby()` that produced a meaningful index. Otherwise, `merge()` is cleaner and more flexible.

---

## Common Patterns

### Clean first, then merge

The most important workflow: clean each source independently before combining them.

```python
# Clean customers
customers["Name"] = customers["Name"].str.strip().str.title()
customers = customers.drop_duplicates(subset=["ID"])

# Clean orders
orders["Price"] = pd.to_numeric(orders["Price"], errors="coerce")
orders = orders.dropna(subset=["Price"])

# Now merge
result = customers.merge(orders, how="left", on="ID")
```

### Merge and immediately filter

```python
# Keep only rows that matched in both DataFrames (inner), then filter by price
result = customers.merge(orders, how="inner", on="ID")
result = result[result["Price"] > 500]
```

### Combine monthly files with concat, then analyze

```python
import glob

all_files = glob.glob("sales_*.csv")
monthly_dfs = [pd.read_csv(f) for f in all_files]
full_year = pd.concat(monthly_dfs, ignore_index=True)

# Now clean and analyze the full year
full_year["Date"] = pd.to_datetime(full_year["Date"])
full_year.drop_duplicates(inplace=True)
```

---

## Complete Summary Table

| Function | Category | Purpose |
|---|---|---|
| `df.info()` | Inspection | Column types and null counts |
| `df.describe()` | Inspection | Statistical summary |
| `df.isna().sum()` | Inspection | Count missing values per column |
| `df.value_counts()` | Inspection | Frequency of each unique value |
| `df.duplicated()` | Inspection | Identify duplicate rows |
| `df.fillna()` | Missing Values | Fill NaN with a value |
| `df.dropna()` | Missing Values | Remove rows with NaN |
| `df.interpolate()` | Missing Values | Estimate NaN from neighbors |
| `df.drop_duplicates()` | Duplicates | Remove duplicate rows |
| `df.drop()` | Structure | Remove rows or columns |
| `df.rename()` | Structure | Rename specific columns |
| `df.columns.str.*` | Structure | Clean all column names at once |
| `df.astype()` | Types | Change column data type |
| `pd.to_numeric()` | Types | Convert to number, coerce errors |
| `pd.to_datetime()` | Types | Convert to datetime |
| `str.strip()` | Strings | Remove characters from edges |
| `str.replace()` | Strings | Replace characters in text |
| `str.split()` | Strings | Split into multiple columns |
| `str.extract()` | Strings | Extract regex pattern |
| `str.contains()` | Strings | Filter rows by text match |
| `str.len()` | Strings | Get text length per row |
| `df.replace()` | Values | Replace whole cell values |
| `df.loc[cond, col]` | Values | Overwrite values matching condition |
| `df.where()` | Values | Keep valid values, NaN the rest |
| `df.clip()` | Outliers | Cap values within a valid range |
| `np.where()` | New Columns | Column based on one condition |
| `np.select()` | New Columns | Column based on multiple conditions |
| `df.apply()` | New Columns | Custom function per row or column |
| `df.reset_index()` | Structure | Reset index after dropping rows |
| `merge()` | Combining | Join two DataFrames by matching column |
| `concat()` | Combining | Stack DataFrames vertically or horizontally |
| `join()` | Combining | Join two DataFrames by index |

---

This is Part 3 of the Pandas for Data Science series. Next up: **Part 4 — Data Visualization with Matplotlib & Seaborn**

---
Refrences

GitHub Repo : https://github.com/Hu8MA/Mastering-Pandas-Reference

Website : https://pandas.pydata.org/

GitHub Library : https://github.com/pandas-dev/pandas

Course : https://youtu.be/Mdq1WWSdUtw


