# Mastering Pandas — Part 4: Data Visualization with Matplotlib & Seaborn

> **Pandas for Data Science Series — Article #4**

---

## From Clean Data to Clear Insight

In Part 3, you learned how to clean messy data and combine multiple sources into one unified DataFrame. Now that your data is ready, the next step is to communicate what's inside it — and nothing communicates data faster or more clearly than a chart.

This article covers the two most important visualization libraries in Python: **Matplotlib**, the core engine that powers all plotting, and **Seaborn**, a higher-level library built on top of it that produces beautiful statistical charts with minimal code. By the end, you'll know which tool to reach for depending on what you need to show.

We'll use this sample dataset throughout:

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

data = {
    'Country':         ['China', 'India', 'USA', 'Brazil', 'UK'],
    'Continent':       ['Asia', 'Asia', 'North America', 'South America', 'Europe'],
    '2022 Population': [1412000000, 1380000000, 331000000, 214000000, 67000000],
    'Area (km2)':      [9597000, 3287000, 9834000, 8516000, 243000]
}
df = pd.DataFrame(data)
```

---

## Part 1 — Matplotlib: The Engine

### What is Matplotlib?

Matplotlib is the foundational plotting library in Python. Every chart you create in Python — whether through Pandas, Seaborn, or directly — eventually goes through Matplotlib to render the final output. Think of it as the engine: it handles the low-level work of drawing lines, shapes, colors, axes, and text.

The submodule you'll use in practice is `matplotlib.pyplot`, which gives you a clean, high-level interface without managing every visual detail manually:

```python
import matplotlib.pyplot as plt
```

Matplotlib works across three styles, each suited to a different situation:

| Style | When to use |
|---|---|
| `df.plot(kind="bar", y="col")` | Your data is already in a DataFrame — quickest way |
| `plt.plot(xs, ys)` | Working with raw lists, arrays, or math functions |
| `fig, ax = plt.subplots()` | Multiple subplots or full layout control |

All three ultimately use Matplotlib to render the chart. `plt.show()` is always called at the end to display it.

---

### Global Settings with `plt.rcParams`

`plt.rcParams` is a global settings dictionary. Any value you set here applies to every chart created afterward in the session — so you set it once at the top of your notebook and never repeat it.

```python
plt.rcParams['figure.figsize'] = (12, 5)   # default width and height
plt.rcParams['font.size']      = 13         # default font size
plt.rcParams['figure.dpi']     = 100        # image sharpness
plt.rcParams['lines.linewidth'] = 2         # default line thickness
plt.rcParams['axes.grid']      = True       # show grid on all charts
```

---

### Setting a Visual Theme with `plt.style.use()`

One line changes the entire look of your charts — colors, background, grid style, and fonts. Call it **before** your plot, not after.

```python
plt.style.use("ggplot")
df.plot(kind="bar", y="2022 Population")
plt.show()
```

| Style | Look |
|---|---|
| `"ggplot"` | Gray background, colored lines — popular R-like style |
| `"seaborn-v0_8-whitegrid"` | Clean, modern, white with grid |
| `"fivethirtyeight"` | Bold, thick lines — news article style |
| `"dark_background"` | Black background |
| `"bmh"` | Soft colors, Bayesian style |
| `"grayscale"` | Shades of gray only |
| `"tableau-colorblind10"` | Colorblind-friendly palette |

Run `print(plt.style.available)` to see every available option.

---

### Plotting from a DataFrame with `.plot()`

Since Pandas is built on Matplotlib, every DataFrame and Series has a `.plot()` method that creates a chart in one line.

```python
df.plot(kind="bar", x="Country", y="2022 Population", figsize=(10, 5), title="Population by Country")
plt.show()
```

The `kind=` parameter selects the chart type. All common types are supported:

| `kind=` | Chart |
|---|---|
| `"bar"` | Vertical bars |
| `"barh"` | Horizontal bars |
| `"line"` | Line chart (default) |
| `"pie"` | Pie chart |
| `"hist"` | Histogram |
| `"scatter"` | Scatter plot |
| `"box"` | Box plot |

You can also call the chart type directly as a method — both styles produce identical output:

```python
# These two are exactly the same:
df.plot(kind="bar", x="Country", y="2022 Population")
df.plot.bar(x="Country", y="2022 Population")
```

The direct method style (`df.plot.bar()`) is shorter and more common in practice.

**Key parameters you'll use on almost every chart:**

| Parameter | What it does | Example |
|---|---|---|
| `kind` | Chart type | `"bar"`, `"line"`, `"scatter"` |
| `x` / `y` | Columns for axes | `x="Country"`, `y="2022 Population"` |
| `figsize` | Width and height in inches | `figsize=(10, 5)` |
| `title` | Chart title | `title="Population by Country"` |
| `color` | Bar/line color | `color="steelblue"` |
| `legend` | Show or hide legend | `legend=True` |
| `xlabel` / `ylabel` | Axis labels | `xlabel="Country"` |
| `rot` | Rotation of tick labels | `rot=45` |
| `grid` | Show background grid | `grid=True` |
| `alpha` | Transparency | `alpha=0.8` |
| `bins` | Bin count (histogram only) | `bins=20` |

```python
df.plot(
    kind="bar",
    x="Country",
    y="2022 Population",
    figsize=(10, 5),
    title="2022 World Population",
    color="steelblue",
    xlabel="Country",
    ylabel="Population",
    rot=0,
    grid=True,
    alpha=0.85
)
plt.show()
```

---

### Plotting Directly with `plt.plot()`

When your data is not in a DataFrame — raw lists, NumPy arrays, or a mathematical function — use `plt.plot()` directly.

```python
import numpy as np

xs = np.arange(-5, 5, 0.25)
ys = xs ** 2

plt.figure(figsize=(8, 5))
plt.plot(xs, ys, color="steelblue", linewidth=2, linestyle="--")
plt.title("Quadratic Function")
plt.xlabel("X")
plt.ylabel("Y")
plt.grid(True)
plt.show()
```

> Matplotlib doesn't know your data is a curve — it connects the dots you give it with straight lines. When points are closely spaced (as with `np.arange` step `0.25`), hundreds of short segments visually blend into a smooth curve.

**Customizing `plt.plot()`:**

```python
# Color
plt.plot(xs, ys, color='red')

# Line style
plt.plot(xs, ys, linestyle='--')    # dashed
plt.plot(xs, ys, linestyle=':')     # dotted
plt.plot(xs, ys, linestyle='-.')    # dash-dot

# Markers at each point
plt.plot(xs, ys, marker='o')        # circles
plt.plot(xs, ys, marker='s')        # squares
plt.plot(xs, ys, marker='^')        # triangles

# Everything combined
plt.plot(xs, ys, color='green', linestyle='--', linewidth=2, marker='o')
plt.show()
```

If you want dots without a line, use `plt.scatter()`:

```python
plt.scatter(xs, ys)
plt.show()
```

---

### Decorating Any Chart

These functions work on any chart — whether drawn through Pandas or directly:

```python
plt.title("Chart Title")
plt.xlabel("X Axis Label")
plt.ylabel("Y Axis Label")
plt.grid(True)
plt.legend()      # shows labels defined with label= on each line
plt.show()
```

When plotting multiple lines, use `label=` with each `plt.plot()` call, then `plt.legend()` to display them:

```python
ys1 = xs ** 2
ys2 = xs ** 3

plt.plot(xs, ys1, label="f(x) = x²")
plt.plot(xs, ys2, label="f(x) = x³", linestyle="--")
plt.legend()
plt.show()
```

---

### Multiple Subplots with Object-Oriented Style

When you need more than one chart in the same figure, use the object-oriented style. It gives you full control over each subplot independently.

```python
# 1 row, 2 columns — side by side
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

ax1.bar(df["Country"], df["2022 Population"], color="steelblue")
ax1.set_title("Population by Country")
ax1.set_xlabel("Country")
ax1.set_ylabel("Population")

ax2.scatter(df["Area (km2)"], df["2022 Population"], color="tomato")
ax2.set_title("Area vs Population")
ax2.set_xlabel("Area (km²)")
ax2.set_ylabel("Population")

plt.tight_layout()
plt.show()
```

`fig` is the whole figure container. `ax1` and `ax2` are the individual plotting areas. The pattern is the same as `plt.*` calls, just moved to `ax.set_*()` methods.

---

## Part 2 — Seaborn: Statistical Visualization

### What is Seaborn?

Seaborn is a library built on top of Matplotlib, designed for statistical data visualization. It produces polished, publication-quality charts with far less code than raw Matplotlib, and works natively with Pandas DataFrames.

```python
import seaborn as sns
```

Every Seaborn chart follows the same basic structure:

```python
sns.chart_type(data=df, x='column_name', y='column_name')
plt.show()
```

The `hue=` parameter is Seaborn's most powerful feature — it automatically splits and colors your data by a category column, adding a legend with no extra work:

```python
sns.scatterplot(data=df, x='Area (km2)', y='2022 Population', hue='Continent')
plt.show()
```

---

### Global Styling with `set_theme()`

Call this once at the top. Every chart afterward follows these settings automatically.

```python
sns.set_theme(style='whitegrid', palette='deep', font_scale=1.2)
```

**`style`** controls the background:

```python
sns.set_theme(style='whitegrid')   # white background with grid — most common
sns.set_theme(style='darkgrid')    # dark background with grid
sns.set_theme(style='white')       # white background, no grid
sns.set_theme(style='ticks')       # minimal, axis ticks only
```

**`palette`** controls the colors of chart elements:

```python
sns.set_theme(palette='deep')        # default, rich colors
sns.set_theme(palette='muted')       # softer, less saturated
sns.set_theme(palette='pastel')      # light pastel colors
sns.set_theme(palette='colorblind')  # accessible for colorblind readers
```

**`font_scale`** scales all text up or down:

```python
sns.set_theme(font_scale=1.5)    # larger text
sns.set_theme(font_scale=0.8)    # smaller text
```

To apply a style to one chart only without changing global settings:

```python
with sns.axes_style('whitegrid'):
    sns.barplot(data=df, x='Continent', y='2022 Population')
    plt.show()
```

To reset everything back to Matplotlib defaults: `sns.reset_defaults()`

---

### Relational Charts — Showing Relationships Between Numbers

Use these when you want to see how two numeric columns relate to each other.

**`scatterplot()`** — each row becomes one dot on the chart:

```python
# Basic
sns.scatterplot(data=df, x='Area (km2)', y='2022 Population')

# Colored by continent
sns.scatterplot(data=df, x='Area (km2)', y='2022 Population', hue='Continent')
plt.title("Area vs Population")
plt.show()
```

**`lineplot()`** — connects points with a line. Best for time series or ordered data:

```python
sns.lineplot(data=df, x='Country', y='2022 Population', hue='Continent')
plt.show()
```

**`relplot()`** — a single figure-level function for both scatter and line. Use `kind=` to switch:

```python
sns.relplot(data=df, x='Area (km2)', y='2022 Population', kind='scatter', hue='Continent')
sns.relplot(data=df, x='Country', y='2022 Population', kind='line')
plt.show()
```

---

### Distribution Charts — Showing How Values Are Spread

Use these to understand the shape and spread of a numeric column.

**`histplot()`** — bars showing how many values fall within each range:

```python
# Basic histogram
sns.histplot(data=df, x='2022 Population')

# With smooth density curve on top
sns.histplot(data=df, x='2022 Population', kde=True)

# Split by category
sns.histplot(data=df, x='2022 Population', hue='Continent')
plt.show()
```

**`kdeplot()`** — a smooth density curve, more refined than a histogram:

```python
sns.kdeplot(data=df, x='2022 Population', hue='Continent')
plt.show()
```

For a 2D density plot showing where two numeric columns overlap:

```python
sns.kdeplot(data=df, x='Area (km2)', y='2022 Population')
plt.show()
```

**`rugplot()`** — adds small tick marks along an axis showing where individual data points are. Typically layered on top of a kdeplot:

```python
sns.kdeplot(data=df, x='2022 Population')
sns.rugplot(data=df, x='2022 Population')
plt.show()
```

**`ecdfplot()`** — shows the cumulative distribution: what percentage of values fall below each point:

```python
sns.ecdfplot(data=df, x='2022 Population')
plt.show()
```

**`displot()`** — one function to rule all distribution charts. Use `kind=` to switch:

```python
sns.displot(data=df, x='2022 Population', kind='hist', hue='Continent')
sns.displot(data=df, x='2022 Population', kind='kde')
sns.displot(data=df, x='2022 Population', kind='ecdf')
plt.show()
```

---

### Categorical Charts — Comparing Groups

Use these when one axis is a category and the other is numeric.

**`barplot()`** — shows the **mean** value per category, with confidence interval bars:

```python
sns.barplot(data=df, x='Continent', y='2022 Population', hue='Continent')
plt.title("Average Population by Continent")
plt.show()
```

> Note: `barplot()` shows the *mean*, not the raw values. If you want raw counts by category, use `countplot()`.

**`countplot()`** — counts how many rows belong to each category:

```python
sns.countplot(data=df, x='Continent')
plt.show()
```

**`boxplot()`** — shows median, quartiles, and outliers. A fast way to spot unusual values:

```python
sns.boxplot(data=df, x='Continent', y='2022 Population')
plt.show()
```

**`violinplot()`** — like a boxplot, but also shows the full distribution shape on both sides. More informative for larger datasets:

```python
sns.violinplot(data=df, x='Continent', y='2022 Population')
plt.show()
```

**`stripplot()`** — shows every individual data point as a dot per category. Useful when you want to see actual values rather than a summary:

```python
sns.stripplot(data=df, x='Continent', y='2022 Population', hue='Continent')
plt.show()
```

**`swarmplot()`** — same as stripplot but repositions overlapping points so each one is visible:

```python
sns.swarmplot(data=df, x='Continent', y='2022 Population')
plt.show()
```

**`pointplot()`** — shows the mean per category as a dot, connected by lines across categories. Good for spotting trends:

```python
sns.pointplot(data=df, x='Continent', y='2022 Population', hue='Continent')
plt.show()
```

**`catplot()`** — one function for all categorical charts. Use `kind=` to switch between them:

```python
sns.catplot(data=df, x='Continent', y='2022 Population', kind='bar')
sns.catplot(data=df, x='Continent', y='2022 Population', kind='box')
sns.catplot(data=df, x='Continent', y='2022 Population', kind='violin')
sns.catplot(data=df, x='Continent', y='2022 Population', kind='strip')
sns.catplot(data=df, x='Continent', y='2022 Population', kind='swarm')
sns.catplot(data=df, x='Continent', y='2022 Population', kind='point')
plt.show()
```

---

### Matrix & Regression Charts — Correlations and Trends

**`corr()` + `heatmap()`** — the most common combination for exploring how numeric columns relate to each other:

```python
corr = df.select_dtypes(include='number').corr()
sns.heatmap(corr, annot=True, cmap='coolwarm')
plt.title("Correlation Matrix")
plt.show()
```

The values range from -1.0 (perfect negative relationship) to 1.0 (perfect positive relationship). `annot=True` displays the number inside each cell. `cmap='coolwarm'` colors high values red and low values blue.

**`clustermap()`** — same as heatmap but automatically reorders rows and columns by similarity using clustering:

```python
sns.clustermap(corr, annot=True, cmap='coolwarm')
plt.show()
```

**`pairplot()`** — creates scatter plots for every combination of numeric columns at once, with distribution plots along the diagonal. The fastest way to get a complete overview of your data:

```python
# All numeric columns
sns.pairplot(df.select_dtypes(include='number'))

# Colored by a category
sns.pairplot(df, hue='Continent')
plt.show()
```

**`jointplot()`** — shows a scatter plot of two columns, plus the distribution of each column on the margins:

```python
sns.jointplot(data=df, x='Area (km2)', y='2022 Population')

# With a regression line
sns.jointplot(data=df, x='Area (km2)', y='2022 Population', kind='reg')
plt.show()
```

**`lmplot()`** — scatter plot with a regression line fitted through the data. Supports separate lines per group:

```python
sns.lmplot(data=df, x='Area (km2)', y='2022 Population')

# Separate regression line per continent
sns.lmplot(data=df, x='Area (km2)', y='2022 Population', hue='Continent')
plt.show()
```

**`regplot()`** — the axes-level version of `lmplot()`. Does the same thing but doesn't support grouping. Use when you want to embed a regression plot inside a larger figure with subplots:

```python
sns.regplot(data=df, x='Area (km2)', y='2022 Population')
plt.show()
```

**`residplot()`** — shows the residuals of a regression: how far each actual point is from the predicted line. A flat, random scatter around zero means the regression is a good fit:

```python
sns.residplot(data=df, x='Area (km2)', y='2022 Population')
plt.show()
```

---

### Combining Seaborn with Matplotlib

Seaborn creates the chart. Matplotlib customizes it. Both work together in the same block — you never have to choose one or the other:

```python
sns.boxplot(data=df, x='Continent', y='2022 Population', palette='pastel')

plt.title('Population Distribution by Continent')
plt.xlabel('Continent')
plt.ylabel('Population')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

This is the standard workflow: Seaborn for the chart, Matplotlib for the title, labels, rotation, and layout.

---

## Choosing the Right Tool

**Use Matplotlib when:**
- Your data is in raw lists or NumPy arrays, not a DataFrame
- You need multiple subplots in a specific layout
- You want pixel-level control over every visual element

**Use Seaborn when:**
- Your data is in a DataFrame and ready to visualize
- You want a beautiful chart quickly with minimal code
- You're doing statistical analysis — distributions, correlations, regressions, group comparisons

In most real projects, you'll use both: Seaborn to draw the chart, Matplotlib to label and polish it.

---

## Complete Summary Table

| Function | Library | Category | What it shows |
|---|---|---|---|
| `plt.rcParams` | Matplotlib | Settings | Global defaults for all charts |
| `plt.style.use()` | Matplotlib | Settings | Visual theme for all charts |
| `df.plot()` | Pandas/Matplotlib | General | Any chart type from a DataFrame |
| `plt.plot()` | Matplotlib | Line/Scatter | Lines and points from raw data |
| `plt.scatter()` | Matplotlib | Scatter | Dots from raw data |
| `plt.bar()` | Matplotlib | Bar | Bars from raw data |
| `plt.hist()` | Matplotlib | Distribution | Histogram from raw data |
| `plt.subplots()` | Matplotlib | Layout | Multiple charts in one figure |
| `sns.set_theme()` | Seaborn | Settings | Global style for all Seaborn charts |
| `sns.scatterplot()` | Seaborn | Relational | Relationship between two numeric columns |
| `sns.lineplot()` | Seaborn | Relational | Trend or time series |
| `sns.relplot()` | Seaborn | Relational | Wrapper for scatter and line |
| `sns.histplot()` | Seaborn | Distribution | Histogram with optional KDE |
| `sns.kdeplot()` | Seaborn | Distribution | Smooth density curve |
| `sns.ecdfplot()` | Seaborn | Distribution | Cumulative distribution |
| `sns.rugplot()` | Seaborn | Distribution | Data point ticks on axis |
| `sns.displot()` | Seaborn | Distribution | Wrapper for all distribution charts |
| `sns.barplot()` | Seaborn | Categorical | Mean per category |
| `sns.countplot()` | Seaborn | Categorical | Row count per category |
| `sns.boxplot()` | Seaborn | Categorical | Median, quartiles, outliers |
| `sns.violinplot()` | Seaborn | Categorical | Distribution shape per category |
| `sns.stripplot()` | Seaborn | Categorical | Individual data points per category |
| `sns.swarmplot()` | Seaborn | Categorical | Non-overlapping dots per category |
| `sns.pointplot()` | Seaborn | Categorical | Mean per category with trend line |
| `sns.catplot()` | Seaborn | Categorical | Wrapper for all categorical charts |
| `sns.heatmap()` | Seaborn | Matrix | Color-coded correlation matrix |
| `sns.clustermap()` | Seaborn | Matrix | Clustered heatmap |
| `sns.pairplot()` | Seaborn | Matrix | All column pairs at once |
| `sns.jointplot()` | Seaborn | Regression | Scatter + marginal distributions |
| `sns.lmplot()` | Seaborn | Regression | Scatter with regression line (supports grouping) |
| `sns.regplot()` | Seaborn | Regression | Scatter with regression line (no grouping) |
| `sns.residplot()` | Seaborn | Regression | Regression residuals |

---

This is Part 4 of the Pandas for Data Science series. This is the **last article** in this series; we'll see you in **another series 👨‍💻**.

---

**References**

GitHub Repo: https://github.com/Hu8MA/Mastering-Pandas-Reference

Matplotlib Documentation: https://matplotlib.org/

Seaborn Documentation: https://seaborn.pydata.org/

Pandas Visualization Guide: https://pandas.pydata.org/docs/user_guide/visualization.html
