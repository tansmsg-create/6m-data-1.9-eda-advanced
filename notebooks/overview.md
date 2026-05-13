# Lesson 1.9: Advanced Data Wrangling & Analysis — Overview

---

## Part 1: Financial Time Series & Window Functions

**Big picture:** Handle data where *order* matters. Learn to work with dates, fill gaps, smooth noisy data, and measure relationships between stocks.

### Key Concepts

**DatetimeIndex**
- When you use `datetime` objects as a Series index, pandas auto-converts them to a `DatetimeIndex`
- Think of it as a **smart row label** that understands time — it knows that Jan 5 comes after Jan 2, and that "2011" means all dates within that year
- This intelligence enables powerful date-based indexing and slicing (`ts["2011"]`, `ts["2011-05":"2011-08"]`) that a plain integer index can't do

**`datetime` vs `Timestamp`**
- Both represent a single point in time, but from different worlds
- `datetime` — Python standard library, microsecond precision, not pandas-aware
- `Timestamp` — pandas native, nanosecond precision, has extra attributes (`.day_of_week`, `.quarter`, `.is_month_end`)
- Think of `Timestamp` as `datetime` with a pandas upgrade — more tools, more precision
- Largely interchangeable for indexing — pandas silently converts `datetime` to `Timestamp` behind the scenes

**`.dt` accessor**
- A "key" that unlocks datetime-specific tools on a **column** containing dates
- Without `.dt`, pandas doesn't know you want datetime operations — the column could hold any data type
- When dates are in a **column**: `df["Date"].dt.month`
- When dates are the **index**: `df.index.month` (no `.dt` needed)
- Similarly, `.str` unlocks string methods on text columns: `df["name"].str.upper()`

**Resampling**
- Your data's time frequency doesn't always match what you need to analyze — resampling lets you **change the lens** you view your time series through
- Going **lower frequency** (daily → monthly): zoom out to see trends, requires aggregation (`.sum()`, `.mean()`)
- Going **higher frequency** (business days → calendar days): zoom in to fill gaps, creates `NaN` for missing periods, use `.ffill()` to fill
- Rule of thumb: lower frequency = aggregate, higher frequency = fill gaps
- Common frequencies: `'D'` daily, `'B'` business day, `'MS'` month start, `'QS'` quarter start, `'YS'` year start

**Rolling / Window Functions**
- Raw time series data is often noisy — a single bad day can distort the picture
- Rolling functions smooth this out by computing a calculation over a **sliding window** of N periods
- Think of it like a moving spotlight — it looks at the last 30 days, moves forward one day, looks at the last 30 days again, and so on
- `price.rolling(30).mean()` — 30-day moving average
- Use `min_periods` to handle NaN at the start of the series before the window is fully filled

**Returns vs Raw Prices**
- Raw prices trend upward over time — this makes unrelated stocks look correlated just because the whole market rises together
- Returns (`pct_change()`) measure day-to-day % change — they remove the upward trend and reveal the *true* relationship between stocks
- Returns also normalize across different price levels — a 5% move means the same thing whether the stock is $10 or $500
- Analogy: comparing people's heights over 10 years looks correlated (everyone grows) — comparing *growth spurts* is more meaningful

**Covariance vs Correlation**
- Both answer: *"When stock A goes up, does stock B also go up?"*
- Covariance — measures direction of relationship, but the number is hard to interpret (is `0.00023` strong or weak?)
- Correlation — same idea but normalized to always be between -1 and 1, making it easy to compare
- Think of covariance as raw temperature in Kelvin vs correlation as Celsius — same information, but one is far more intuitive
- `1` = perfect positive, `-1` = perfect negative, `0` = no relationship

---

## Part 2: Data Wrangling (Merge & Reshape)

**Big picture:** Your data rarely comes in one perfect table. In the real world, data lives in separate sources — you need to combine tables and reshape them before you can analyze anything.

### Key Concepts

**Merging (Joins)**
- Combines two DataFrames **horizontally** (adding columns) by matching rows based on a common key
- Equivalent to SQL JOIN operations — if you've used databases, this is the same concept
- The key question is: *"What happens to rows that don't have a match in the other table?"* — that's what the join type controls

| Join Type | Analogy | Result |
|---|---|---|
| `inner` (default) | Only keep rows both tables agree on | Rows where key exists in **both** tables |
| `outer` | Keep everything from both tables | All rows from **both** tables, NaN where no match |
| `left` | The left table is the boss | All rows from **left** table, NaN where no match in right |
| `right` | The right table is the boss | All rows from **right** table, NaN where no match in left |

- Use `on=` for same column name, `left_on=` / `right_on=` for different names
- Use `left_index=True` / `right_index=True` to merge on index
- Use `suffixes=` to customize overlapping column names (default: `_x`, `_y`)

**Reshaping**
- Sometimes the *structure* of your table is the problem, not the data itself
- **Wide format** — Excel style, each variable has its own column (good for humans to read)
- **Long format** — Database style, one row per observation (good for analysis and plotting)
- `melt` — **Wide to Long**: collapses many columns into two columns (variable name + value)
- `pivot` — **Long to Wide**: expands one column into many columns
- Think of it like folding and unfolding a table — same data, different shape

---

## Part 3: Aggregation & Reporting

**Big picture:** You have clean, combined data. Now you need to summarize it to answer business questions — totals, averages, breakdowns by category.

### Key Concepts

**GroupBy (Split → Apply → Combine)**
- The most fundamental aggregation tool in pandas — it answers questions like *"What is the average tip by day of week?"* or *"Total sales by region?"*
- **Split** — divide the data into groups based on a column (e.g. group by `day`)
- **Apply** — run a calculation on each group independently (e.g. compute mean)
- **Combine** — stitch the results back into a single table
- Think of it like sorting a deck of cards by suit, counting each pile, then reporting the counts
- `df.groupby("key").mean()` — average per group
- `df.groupby(["key1", "key2"]).size()` — count per group combination
- Select columns *before* aggregation for efficiency: `df.groupby("key")[["col1"]].mean()`

**Custom Aggregation**
- Standard functions (mean, sum, std) don't always answer your specific question
- `.agg()` lets you plug in **any custom function** — anything that takes an array and returns a single value
- Pass a single function: `grouped.agg(peak_to_peak)`
- Pass a list for multiple functions at once: `grouped.agg(["mean", "std", peak_to_peak])`
- This is powerful for domain-specific metrics (e.g. range, percentiles, business-defined KPIs)

**Pivot Tables**
- GroupBy is powerful but the output is a flat list — pivot tables give you a **2D summary** (rows AND columns as categories)
- Same as Excel pivot tables — drag fields into rows, columns, values, and aggregation
- Great for quick reporting: *"Show me average tip % broken down by day (rows) and smoker status (columns)"*
- `df.pivot_table(index=, columns=, values=, aggfunc=)`
- Default aggregation is `mean`
- `margins=True` adds row/column totals

**Cross-Tabulation**
- A simplified pivot table that only **counts how many times** combinations occur
- No aggregation needed — just frequencies
- Great for survey data: *"How many smokers vs non-smokers showed up each day?"*
- `pd.crosstab(index=, columns=, margins=True)`

| Tool | Best for | Output |
|---|---|---|
| `groupby` | Flexible, programmatic aggregation | Flat table |
| `pivot_table` | Quick 2D summary reports | 2D table |
| `crosstab` | Counting occurrences | 2D frequency table |

---

## Part 4: Advanced Toolkit (Optional / Deep Dive)

**Big picture:** Power tools for complex data structures and transformations. Parts 1-3 cover 80% of daily work — Part 4 is what separates a good data scientist from a great one. Reach for these when standard tools aren't enough.

### Key Concepts

**Hierarchical Indexing / MultiIndex**
- Standard indexes have one level (one label per row) — MultiIndex has **multiple levels** of labels, like folders within folders
- Think of it as: Region → City → Product, where each row belongs to all three levels simultaneously
- This lets you represent higher-dimensional data (3D, 4D) in a flat 2D table without duplicating columns
- Partial indexing: `data["b"]` selects all rows under the "b" group, `data.loc[:, 2]` selects inner level 2
- `swaplevel()` — reorder which level is "outer" vs "inner"
- `set_index()` / `reset_index()` — move columns to index and back

**Concatenation**
- Merging combines tables *horizontally* (by matching keys) — concatenation stacks them *vertically* (just pile them on top of each other)
- Think of it like stacking monthly sales reports into one annual table — same columns, just more rows
- Like SQL UNION operations
- `pd.concat([df1, df2])` — along rows (default)
- `pd.concat([df1, df2], axis="columns")` — along columns
- `join="inner"` to intersect indexes instead of union
- `ignore_index=True` when the original row indexes don't carry meaning

**Stack / Unstack**
- Alternative to Melt/Pivot but designed to work specifically on **index levels** rather than regular columns
- `stack()` — rotates column labels down into row index → produces a MultiIndex Series (wide → long)
- `unstack()` — rotates row index up into column labels → produces a DataFrame (long → wide)
- Use when your data already has a MultiIndex and you want to reorganize its shape

**GroupBy Apply**
- Standard GroupBy (`.mean()`, `.sum()`) only returns one value per group — `apply` removes that constraint
- You write **any custom function** that receives a full DataFrame chunk for each group and can return anything — a scalar, a Series, or even a transformed DataFrame
- Think of it as: *"For each group, do whatever you want with it"*
- `tips.groupby("smoker").apply(top)` — get top 5 tippers per smoker group
- More flexible than `.agg()` — the function doesn't have to return a single summary value

**GroupBy Transform**
- Like Apply, but with one important constraint: the output **must be the same shape as the input**
- Instead of collapsing each group into a summary, it broadcasts the result back to every original row
- This makes it perfect for **adding a new column** to the original DataFrame based on group statistics
- Classic use case: z-score normalization within groups — each row gets replaced by how many standard deviations it is from its group's mean
- `g.transform(normalize)` — every row in group "a" gets normalized against group "a"'s mean and std

| Tool | Returns | Best for |
|---|---|---|
| `.agg()` | One value per group | Standard summary stats |
| `apply` | Anything — can change shape | Custom logic per group |
| `transform` | Same shape as input | Adding group-based columns |

---

## Quick Reference: When to Use What

| Task | Tool |
|---|---|
| Parse date strings | `pd.to_datetime()` |
| Change time frequency | `resample()` |
| Fill gaps in time series | `.ffill()` / `.bfill()` |
| Smooth noisy data | `.rolling().mean()` |
| Measure stock relationship | `.corr()` / `.cov()` on returns |
| Combine two tables by key | `pd.merge()` |
| Stack tables vertically | `pd.concat()` |
| Wide → Long format | `pd.melt()` |
| Long → Wide format | `.pivot()` |
| Group & summarize | `.groupby().agg()` |
| 2D summary report | `.pivot_table()` |
| Count frequencies | `pd.crosstab()` |
| Custom group function | `.groupby().apply()` |
| Add group-based column | `.groupby().transform()` |
