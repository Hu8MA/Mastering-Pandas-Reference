# Mastering Pandas — Part 2: GroupBy & Indexing

> **Pandas for Data Science Series — Article #2**

---

## From Loading to Understanding

In Part 1, you learned how to load data from virtually any source and get your first look at its structure. Now it's time to go deeper — to ask real questions about your data and retrieve exactly what you need from it.

This article covers two fundamental skill sets that every data analyst uses daily:

- **GroupBy** — splitting your data into groups and summarizing each one independently
- **Indexing** — selecting exactly the rows and columns you need, with precision

These two topics are deeply connected in practice. You group data to understand it at a high level, and you index into it to examine the details. Together, they form the core of exploratory data analysis.

We'll use this sample DataFrame throughout all the examples:

```python
import pandas as pd

data = {
    'country':   ['USA', 'USA', 'UK', 'UK', 'India', 'India'],
    'gender':    ['Male', 'Female', 'Male', 'Female', 'Male', 'Female'],
    'product':   ['Laptop', 'Phone', 'Laptop', 'Tablet', 'Phone', 'Laptop'],
    'price':     [1200, 800, 1100, 600, 700, 950],
    'quantity':  [3, 5, 2, 4, 6, 3],
    'rating':    [4.5, 4.2, 4.8, 3.9, 4.1, 4.6]
}
df = pd.DataFrame(data)
```

---

# Part 1 — GroupBy

`groupby()` is one of the most powerful operations in Pandas. It follows a simple but flexible pattern: **split** the data into groups based on a column's values, **apply** a function to each group independently, and **combine** the results back into a single output. This is known as the **Split → Apply → Combine** pattern.

```python
# The fundamental pattern
df.groupby("column")["target"].function()
```

---

## 1️⃣ Aggregation Functions — The Core of GroupBy

Aggregation functions collapse each group into a single summary value. These are the most commonly used GroupBy operations in day-to-day data work.

```python
df.groupby("gender")["price"].mean()     # average price per gender
df.groupby("gender")["price"].sum()      # total revenue per gender
df.groupby("gender")["price"].count()    # number of transactions per gender
df.groupby("gender")["price"].min()      # lowest price per gender
df.groupby("gender")["price"].max()      # highest price per gender
df.groupby("gender")["price"].median()   # middle value per gender
df.groupby("gender")["price"].std()      # standard deviation per gender
df.groupby("gender")["price"].var()      # variance per gender
```

---

## 2️⃣ `agg()` — Multiple Functions at Once

Instead of calling one function at a time, `agg()` lets you apply several aggregations in a single step. You can use the default names or assign your own custom labels — the second form is preferred in professional work because the output is immediately readable.

```python
# Default names
df.groupby("gender")["price"].agg(["mean", "sum", "min", "max"])

# Custom names — preferred for readability
df.groupby("gender")["price"].agg(
    Average = "mean",
    Total   = "sum",
    Lowest  = "min",
    Highest = "max"
)
```

---

## 3️⃣ `transform()` — Keep the Original Shape

Unlike aggregation functions that collapse groups into fewer rows, `transform()` returns a result with the **same shape as the original DataFrame**. This makes it ideal for adding a new column that reflects group-level statistics back onto each individual row.

```python
# Add each row's group average back to the original DataFrame
df["gender_avg_price"] = df.groupby("gender")["price"].transform("mean")
```

This is especially useful for **normalization**, **group-relative comparisons**, and **feature engineering** in machine learning pipelines.

---

## 4️⃣ `apply()` — Custom Logic Per Group

When the built-in functions aren't enough, `apply()` lets you pass any custom function — including lambdas — and run it independently on each group. This gives you full flexibility.

```python
# Range (max - min) of price per gender
df.groupby("gender")["price"].apply(lambda x: x.max() - x.min())
```

> **Note:** `apply()` is the most flexible option but also the slowest. For standard aggregations, always prefer the built-in functions or `agg()` — they are significantly faster on large datasets.

---

## 5️⃣ `filter()` — Keep or Drop Entire Groups

`filter()` evaluates a condition for each group and **keeps only the groups where the condition is True**. Unlike boolean indexing which filters individual rows, `filter()` operates at the group level — it keeps or drops entire groups together.

```python
# Keep only gender groups where the average price exceeds 200
df.groupby("gender").filter(lambda x: x["price"].mean() > 200)
```

---

## 6️⃣ `size()` vs `count()` — A Critical Distinction

These two functions look similar but behave differently, and confusing them can lead to subtle bugs.

```python
df.groupby("gender").size()    # counts ALL rows per group, including NaN values
df.groupby("gender").count()   # counts only NON-NULL values per column per group
```

| | `size()` | `count()` |
|---|---|---|
| Counts NaN | ✅ Yes | ❌ No |
| Returns | Single Series | DataFrame (one column per column) |
| Use when | You want total rows | You want non-missing values |

Use `size()` when you want to know how many rows exist in each group. Use `count()` when you want to know how much valid (non-missing) data each group has.

---

## 7️⃣ `nunique()` — Count Distinct Values Per Group

Returns the number of unique values in a column within each group. Useful for understanding variety and diversity within categories.

```python
# How many unique products does each country sell?
df.groupby("country")["product"].nunique()
```

---

## 8️⃣ `first()` & `last()` — Boundary Rows of Each Group

Returns the first or last non-null row from each group. The result preserves the original row order within each group.

```python
df.groupby("gender").first()    # first row of each gender group
df.groupby("gender").last()     # last row of each gender group
```

---

## 9️⃣ Group by Multiple Columns

You can group by more than one column at a time by passing a list. The result has a **MultiIndex** — one level per grouping column.

```python
# Average price broken down by both country and gender
df.groupby(["country", "gender"])["price"].mean()
```

---

## GroupBy Summary Table

| Function | Purpose |
|---|---|
| `mean()` | Average value per group |
| `sum()` | Total value per group |
| `count()` | Count of non-null values per group |
| `size()` | Count of all rows per group (including NaN) |
| `min()` / `max()` | Minimum and maximum per group |
| `median()` | Middle value per group |
| `std()` / `var()` | Spread of data per group |
| `agg()` | Multiple aggregations in one step |
| `apply()` | Custom function per group |
| `transform()` | Group result mapped back to original shape |
| `filter()` | Keep or drop entire groups by condition |
| `nunique()` | Count of distinct values per group |
| `first()` / `last()` | First and last row of each group |

---

# Part 2 — Indexing

Indexing is how you **reach into your DataFrame and retrieve exactly what you need** — a single value, a slice of rows, a set of columns, or a filtered subset. Pandas offers several indexing tools, each suited to a different situation.

Understanding which tool to use — and why — is one of the most important skills you can build as a data analyst.

---

## 1️⃣ `[]` — Basic Indexing

The simplest form of indexing. Use it for quick column selection or row slicing by position.

```python
df["price"]                    # select a single column → returns a Series
df[["price", "quantity"]]      # select multiple columns → returns a DataFrame
df[0:3]                        # select rows by slice (first 3 rows)
```

> **Note:** Using `[]` for row selection only works with slices (`df[0:3]`), not single integers (`df[0]` will raise an error). For precise row access, use `loc[]` or `iloc[]`.

---

## 2️⃣ `loc[]` — Label-Based Indexing

`loc[]` selects data by **label** — that is, by the actual index values and column names. It is the preferred tool when you know the names of what you're looking for.

```python
df.loc[0]                         # row with index label 0
df.loc[0:3]                       # rows with index labels 0 through 3 (inclusive)
df.loc[0, "price"]                # single value: row 0, column "price"
df.loc[0:3, "price":"rating"]     # rows 0–3, columns from "price" to "rating"
df.loc[df["gender"] == "Male"]    # all rows where gender is Male
```

> **Important:** When using `loc[]` with a slice (`0:3`), both endpoints are **inclusive**. This differs from standard Python slicing where the end is exclusive.

---

## 3️⃣ `iloc[]` — Position-Based Indexing

`iloc[]` selects data by **integer position** — the physical row and column numbers, regardless of what the index or column names are. It behaves like standard Python list indexing.

```python
df.iloc[0]           # first row (position 0)
df.iloc[-1]          # last row
df.iloc[0:3]         # first 3 rows (positions 0, 1, 2 — end is exclusive)
df.iloc[0, 2]        # row at position 0, column at position 2
df.iloc[0:3, 0:3]    # first 3 rows, first 3 columns
df.iloc[:, 0]        # all rows, first column only
```

---

## `loc[]` vs `iloc[]` — The Key Distinction

This is the most important comparison in Pandas indexing. Choosing the wrong one is a common source of bugs.

| | `loc[]` | `iloc[]` |
|---|---|---|
| Based on | Labels (names) | Integer positions |
| End of slice | **Inclusive** | **Exclusive** |
| Use when | You know column/index names | You know position numbers |
| Works after reset_index | ✅ Yes | ✅ Yes |
| Fragile after sort/filter | ⚠️ If index changes | ✅ Always positional |

```python
# These may return DIFFERENT rows if the index is not 0,1,2,3...
df.loc[2]     # row where index label == 2
df.iloc[2]    # third row regardless of its index label
```

---

## 4️⃣ `at[]` & `iat[]` — Fast Single Value Access

When you need to read or write a **single cell**, `at[]` and `iat[]` are faster than `loc[]` and `iloc[]` because they are optimized specifically for scalar access.

```python
# at[] — by label
df.at[0, "price"]              # get value at row 0, column "price"
df.at[0, "price"] = 999        # set value at row 0, column "price"

# iat[] — by position
df.iat[0, 3]                   # get value at row 0, column position 3
df.iat[0, 3] = 999             # set value at row 0, column position 3
```

Use `at[]` / `iat[]` inside loops or any time you are accessing one cell at a time — the performance difference becomes significant at scale.

---

## 5️⃣ Boolean Indexing — Filter by Condition

Boolean indexing is the standard way to filter rows based on a condition. It works by creating a True/False mask that Pandas applies to the DataFrame.

```python
df[df["price"] > 900]                                         # price above 900
df[df["gender"] == "Female"]                                  # only Female rows
df[df["country"] == "USA"]                                    # only USA rows

# Combining conditions
df[(df["price"] > 900) & (df["gender"] == "Female")]          # AND
df[(df["price"] > 900) | (df["country"] == "UK")]             # OR
df[~(df["gender"] == "Male")]                                  # NOT (negation)
```

> **Always wrap each condition in parentheses** when combining with `&`, `|`, or `~`. Python's operator precedence will cause unexpected results without them.

---

## 6️⃣ `isin()` — Match Against a List of Values

`isin()` checks whether each value in a column belongs to a provided list. It is the clean, readable alternative to chaining multiple OR conditions.

```python
df[df["country"].isin(["USA", "UK", "India"])]    # keep these countries
df[~df["country"].isin(["USA", "UK"])]            # exclude these countries
```

---

## 7️⃣ `query()` — Filter Using a String Expression

`query()` lets you filter rows using a readable string expression instead of boolean masks. It is especially useful for complex conditions — the syntax is cleaner and closer to natural language.

```python
df.query("price > 900")
df.query("gender == 'Female'")
df.query("price > 900 and gender == 'Female'")
df.query("country in ['USA', 'UK']")
```

`query()` produces the same result as boolean indexing — it is purely a readability preference. For simple one-line filters, boolean indexing is fine. For multi-condition filters, `query()` is often easier to read and maintain.

---

## 8️⃣ `where()` — Conditional Value Replacement

Unlike the filtering tools above that remove rows, `where()` **keeps all rows** but replaces values that don't meet the condition with `NaN` or a value you specify.

```python
df["price"].where(df["price"] > 900)           # keep prices > 900, rest → NaN
df["price"].where(df["price"] > 900, other=0)  # keep prices > 900, rest → 0
```

This is useful when you want to preserve the DataFrame's shape while masking out values that don't meet a criterion.

---

## 9️⃣ `set_index()` — Use a Column as the Index

By default, Pandas assigns a numeric index (0, 1, 2, ...). `set_index()` lets you replace that with a meaningful column — such as a country name or an order ID — making label-based indexing more intuitive.

```python
df.set_index("country")                        # use country as the index
```

**Understanding `inplace`:** This parameter controls whether the operation modifies the original DataFrame or returns a new one.

```python
# inplace=False (default) — original is unchanged, result is a new DataFrame
df2 = df.set_index("country")

# inplace=True — original DataFrame is modified directly
df.set_index("country", inplace=True)
```

> Most Pandas functions that modify structure accept `inplace`. The default is always `False`, meaning a new DataFrame is returned unless you explicitly ask for in-place modification.

---

## 🔟 `xs()` — Cross-Section for MultiIndex

After grouping by multiple columns, the resulting DataFrame has a **MultiIndex** — a hierarchical index with one level per grouping column. `xs()` (cross-section) lets you select a slice at a specific level of that index cleanly.

```python
# After grouping by country and gender
grouped = df.groupby(["country", "gender"])["price"].mean().reset_index()
grouped = grouped.set_index(["country", "gender"])

# Select all rows where gender == "Female" across all countries
grouped.xs("Female", level="gender")
```

---

## Indexing Summary Table

| Tool | Based On | Best For |
|---|---|---|
| `[]` | Label | Quick column select or row slice |
| `loc[]` | Label | Rows & columns by name |
| `iloc[]` | Position | Rows & columns by number |
| `at[]` | Label | Single cell — fast read/write |
| `iat[]` | Position | Single cell — fast read/write |
| Boolean `[]` | Condition | Filter rows by expression |
| `isin()` | List | Match multiple values at once |
| `query()` | String | Readable multi-condition filtering |
| `where()` | Condition | Replace non-matching values |
| `set_index()` | Column | Use meaningful column as index |
| `xs()` | Label | MultiIndex cross-section |

---

## Most Important Rules to Remember

```
loc[]    →  when you know column NAMES  (slice end is inclusive)
iloc[]   →  when you know column NUMBERS (slice end is exclusive)
query()  →  when you want clean, readable filter expressions
Boolean  →  when you want to filter rows by condition
at/iat[] →  when you need a single cell value — fastest option
```

---

## Common Patterns

### Group, then Index into the Result

The most common real-world flow: summarize by group, then retrieve a specific slice from the result.

```python
# Average price per country, then select only the top 2
df.groupby("country")["price"].mean().nlargest(2)
```

### Add a Group-Level Column, then Filter

Use `transform()` to add group statistics back to the original DataFrame, then filter using that new column.

```python
# Flag rows where price exceeds the group average for their gender
df["gender_avg"] = df.groupby("gender")["price"].transform("mean")
df[df["price"] > df["gender_avg"]]
```

### Combine GroupBy with `agg()` and `query()`

```python
# Total and average price per country, then keep only countries with avg > 800
result = df.groupby("country")["price"].agg(Total="sum", Average="mean")
result.query("Average > 800")
```

---

## Complete Summary Table

| Function | Category | Purpose |
|---|---|---|
| `mean()` / `sum()` / `count()` | GroupBy | Core aggregations per group |
| `min()` / `max()` / `median()` | GroupBy | Range and center per group |
| `std()` / `var()` | GroupBy | Spread of data per group |
| `agg()` | GroupBy | Multiple aggregations in one call |
| `apply()` | GroupBy | Custom function per group |
| `transform()` | GroupBy | Group result mapped to original shape |
| `filter()` | GroupBy | Keep or drop entire groups |
| `size()` / `nunique()` | GroupBy | Row count / unique value count per group |
| `first()` / `last()` | GroupBy | Boundary rows of each group |
| `[]` | Indexing | Quick column or slice access |
| `loc[]` | Indexing | Label-based row & column selection |
| `iloc[]` | Indexing | Position-based row & column selection |
| `at[]` / `iat[]` | Indexing | Fast single cell access |
| Boolean indexing | Indexing | Filter rows by condition |
| `isin()` | Indexing | Match a list of values |
| `query()` | Indexing | Readable string-based filtering |
| `where()` | Indexing | Replace non-matching values |
| `set_index()` | Indexing | Promote a column to the index |
| `xs()` | Indexing | MultiIndex cross-section |

---

This is Part 2 of the Pandas for Data Science series. Next up: **Part 3 — Data Cleaning & Merging/Joining.**
