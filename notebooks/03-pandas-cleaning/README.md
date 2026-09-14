# 03 — Pandas and Cleaning

Pandas gives Python a table. Once your data is a DataFrame, you stop writing loops and start describing the result you want, which is the same shift you already made when you learned SQL.

Most of this unit is a small set of moves repeated on messier and messier data: look at it, pick rows and columns, fix the types, group it, join it, and check that you did not break anything.

This is not every pandas feature. It is the stuff you will actually use.

| Notebook | Day | What it covers |
|---|---|---|
| `2026-09-14 — SQL to Python to DataFrames — Lecture.ipynb` | Mon | What a DataFrame is, selecting, grouping, merging |
| `2026-09-16 — Pandas Core Ops — Studio.ipynb` | Wed | The same moves on your own, plus the duplication trap |
| `2026-09-18 — Pandas Challenge — Lab.ipynb` | Fri | Full pass on unfamiliar data — **graded** |
| `2026-09-21 — Data Cleaning (types, missing, duplicates) — Lecture.ipynb` | Mon | The five fixes, and the order to do them in |
| `2026-09-23 — Cleaning Clinic — Studio.ipynb` | Wed | Diagnosing a dirty file before touching it |
| `2026-09-25 — Cleaning Gauntlet — Lab.ipynb` | Fri | Clean it, prove it, report what changed — **graded** |

---

## Two objects, and that is most of it

- A **DataFrame** is the whole table: rows, named columns, one data type per column.
- A **Series** is one column, with its row labels attached.

One bracket gives you a Series. Two brackets give you a DataFrame.

```python
orders['item']            # Series
orders[['item', 'qty']]   # DataFrame
```

That distinction causes real confusion later, because some methods only exist on one of the two.

---

## Start every session the same way

```python
import pandas as pd
import numpy as np
```

`pd` and `np` are conventions, not requirements, but everyone uses them and so should you.

---

## Load data

From a file:

```python
orders = pd.read_csv('orders.csv')
```

From a URL:

```python
orders = pd.read_csv('https://example.com/orders.csv')
```

From a string, which is how the notebooks build small examples:

```python
from io import StringIO

orders = pd.read_csv(StringIO('''order_id,item,qty
1001,Cheeseburger,3
1002,Foam Finger,1'''))
```

Arguments worth knowing:

```python
pd.read_csv('f.csv', dtype={'zip': str})        # keep leading zeros
pd.read_csv('f.csv', parse_dates=['order_date'])
pd.read_csv('f.csv', na_values=['', 'NA', 'n/a', 'missing'])
pd.read_csv('f.csv', nrows=100)                 # peek at something huge
```

Excel and JSON work the same way:

```python
pd.read_excel('f.xlsx', sheet_name='Orders')
pd.read_json('f.json')
```

Writing back out:

```python
clean.to_csv('orders_clean.csv', index=False)
```

Use `index=False` unless you actually want the row numbers as a column. You almost never do.

---

## Look around before you work

```python
orders.shape        # (rows, columns)
orders.head()       # first 5 rows
orders.tail(3)      # last 3
orders.info()       # types and non-null counts
orders.describe()   # numeric summary
orders.columns      # column names
orders.dtypes       # type of each column
```

`.info()` is the equivalent of reading the schema. It tells you the row count, the column types, and how many non-null values each column has, which explains most errors you are about to hit.

For a single column:

```python
orders['category'].unique()        # what values exist
orders['category'].nunique()       # how many distinct ones
orders['category'].value_counts()  # how many of each
```

`value_counts()` ignores missing values by default. To see them:

```python
orders['category'].value_counts(dropna=False)
```

---

## Pick columns

```python
orders['item']                        # one column, as a Series
orders[['item', 'qty', 'price']]      # several, as a DataFrame
orders.drop(columns=['notes'])        # everything except
```

Rename:

```python
orders = orders.rename(columns={'ord_id': 'order_id', 'amt': 'revenue'})
```

---

## Filter rows

A filter is a boolean Series used as a mask:

```python
orders[orders['qty'] > 2]
orders[orders['category'] == 'Food']
orders[orders['category'] != 'Food']
```

Combine with `&` and `|`, never `and` and `or`, and parenthesize every condition:

```python
orders[(orders['qty'] >= 3) & (orders['price'] < 10)]
orders[(orders['category'] == 'Food') | (orders['category'] == 'Drink')]
orders[~(orders['category'] == 'Food')]            # ~ means not
```

Forgetting the parentheses produces one of the least helpful error messages in Python. If you see something about "truth value of a Series is ambiguous," this is why.

Sets and ranges:

```python
orders[orders['category'].isin(['Food', 'Drink'])]
orders[orders['qty'].between(2, 5)]                # inclusive on both ends
```

Text matching:

```python
orders[orders['item'].str.contains('Burger', case=False, na=False)]
orders[orders['item'].str.startswith('UVA')]
```

Use `na=False` on `.str.contains` or missing values raise instead of filtering.

Label and position based selection:

```python
orders.loc[orders['qty'] > 2, ['item', 'qty']]   # rows by condition, columns by name
orders.iloc[0]                                    # first row, by position
orders.iloc[0:5, 0:3]                             # first 5 rows, first 3 columns
```

`.loc` uses labels, `.iloc` uses integer positions. When you need to filter rows *and* pick columns in one step, `.loc` is the clean way to do it.

---

## The `.copy()` rule

This is the one that bites everybody:

```python
food = orders[orders['category'] == 'Food']
food['discounted'] = food['revenue'] * 0.9      # SettingWithCopyWarning
```

The slice may be a view onto the original, so pandas cannot promise where your assignment lands. The fix is one word:

```python
food = orders[orders['category'] == 'Food'].copy()
food['discounted'] = (food['revenue'] * 0.9).round(2)
```

Make it a reflex: **if you filter and then modify, add `.copy()`.**

---

## Add and change columns

Arithmetic applies to the whole column at once:

```python
orders['revenue'] = orders['qty'] * orders['price']
orders['revenue'] = orders['revenue'].round(2)
```

Categories from a condition, which is `CASE WHEN` in SQL:

```python
orders['size'] = np.where(orders['qty'] >= 5, 'bulk', 'single')
```

More than two buckets:

```python
orders['band'] = pd.cut(orders['revenue'],
                        bins=[0, 10, 50, np.inf],
                        labels=['small', 'medium', 'large'])
```

Map values one to one:

```python
orders['zone'] = orders['vendor_id'].map({'V-01': 'A', 'V-05': 'B'})
```

---

## Sort and take the top

```python
orders.sort_values('revenue')
orders.sort_values('revenue', ascending=False)
orders.sort_values(['category', 'revenue'], ascending=[True, False])
orders.sort_values('revenue', ascending=False).head(5)
```

`.head()` after a sort is how you do `ORDER BY ... LIMIT`. `nlargest` is shorter when that is all you want:

```python
orders.nlargest(5, 'revenue')
```

---

## Group and summarize

`groupby` splits rows into buckets, applies a function, and puts the answers back together. That is `GROUP BY`, with more freedom about what comes next.

One number per group:

```python
orders.groupby('category')['revenue'].sum()
orders.groupby('category')['revenue'].mean()
orders.groupby('category').size()              # rows per group
```

Sorted, which is usually what you want to read:

```python
orders.groupby('category')['revenue'].sum().sort_values(ascending=False)
```

Several summaries at once, with names you choose:

```python
orders.groupby('category').agg(
    orders=('order_id', 'count'),
    units=('qty', 'sum'),
    revenue=('revenue', 'sum'),
    avg_ticket=('revenue', 'mean'),
).round(2)
```

Group by more than one column:

```python
orders.groupby(['category', 'vendor_id'])['revenue'].sum()
```

Keep the group labels as columns instead of as an index:

```python
orders.groupby('category', as_index=False)['revenue'].sum()
```

`HAVING` is just a filter after the grouping:

```python
totals = orders.groupby('category')['revenue'].sum()
totals[totals > 100]
```

---

## Join tables

```python
joined = orders.merge(vendors, on='vendor_id', how='left')
```

The `how` argument is the whole decision:

| `how` | Keeps |
|---|---|
| `'inner'` | only rows that matched in both — the default |
| `'left'` | every row of the left table, `NaN` where no match |
| `'right'` | every row of the right table |
| `'outer'` | everything from both sides |

Different key names on each side:

```python
orders.merge(vendors, left_on='vendor_id', right_on='id', how='left')
```

### Always check the join

Add `indicator=True` and count the rows. This is the difference between catching a data problem and shipping it.

```python
joined = orders.merge(vendors, on='vendor_id', how='left', indicator=True)

print('rows before:', len(orders), '| rows after:', len(joined))
print(joined['_merge'].value_counts())
```

Two things to look for:

1. **`left_only` rows** mean keys in your data have no match in the lookup table. Those columns are `NaN` now. Find them:

   ```python
   joined[joined['_merge'] == 'left_only']
   ```

2. **More rows after than before** means the duplication trap. If the right table has two rows for the same key, every matching left row is duplicated, and every total you compute afterward is inflated. Check the lookup table first:

   ```python
   vendors['vendor_id'].duplicated().sum()    # should be 0 for a lookup table
   ```

Drop the indicator when you are done with it:

```python
joined = joined.drop(columns=['_merge'])
```

### Stacking instead of joining

`merge` adds columns. `concat` adds rows.

```python
all_weeks = pd.concat([week1, week2], ignore_index=True)
```

Use `ignore_index=True` unless you have a reason to keep the original row labels.

---

## Cleaning

### The order that works

Doing these out of order makes each one harder:

1. **Duplicates first.** Cheaper to fix five rows than to fix six and then delete one.
2. **Types next.** You cannot filter on `qty > 0` while `qty` is text.
3. **Then missing and out-of-range values**, now that comparisons work.
4. **Then text normalization**, so grouping puts the right rows together.
5. **Then dates**, so anything time-based becomes possible.

Take inventory before you change anything:

```python
df.info()
df.isna().sum()                  # missing values per column
df.duplicated().sum()            # fully identical rows
for col in df.columns:
    print(col, df[col].unique()[:10])
```

### Duplicates

Decide which kind of duplicate you mean:

```python
df.duplicated().sum()                          # every column identical
df.duplicated(subset=['order_id']).sum()       # same order_id, other columns may differ
```

The second is usually the real question. Drop them:

```python
df = df.drop_duplicates()
df = df.drop_duplicates(subset=['order_id'], keep='first')
```

`keep='first'`, `keep='last'`, and `keep=False` (drop every copy) do different things. Look at the rows before you choose:

```python
df[df.duplicated(subset=['order_id'], keep=False)].sort_values('order_id')
```

### Numbers that arrived as text

One bad value makes the whole column text, so `sum()` concatenates instead of adding.

```python
df['qty'].dtype     # object (or str on newer pandas) means text, not numbers
```

Strip the junk, then convert:

```python
df['price'] = df['price'].str.replace('$', '', regex=False)
df['price'] = df['price'].str.replace(',', '', regex=False)
df['price'] = pd.to_numeric(df['price'], errors='coerce')
```

`errors='coerce'` turns anything unconvertible into `NaN` instead of raising. That forces a decision, so look at what you just destroyed:

```python
df[df['price'].isna()]
```

`astype` is fine when you know the column is already clean, and raises when it is not:

```python
df['qty'] = df['qty'].astype(int)
```

Convert to a nullable integer when the column has missing values, since plain `int` cannot hold `NaN`:

```python
df['qty'] = df['qty'].astype('Int64')     # capital I
```

### Missing values

```python
df.isna().sum()                    # count per column
df['qty'].isna().sum()             # one column
df.isna().sum().sum()              # total
df[df['qty'].isna()]               # look at the actual rows
```

`isnull` and `isna` are the same method with two names. `notna` is the opposite.

Filling is a claim about the data, so do not do it reflexively:

```python
df['notes'] = df['notes'].fillna('none given')
df['qty'] = df['qty'].fillna(0)              # only if missing really means zero
df['price'] = df['price'].fillna(df['price'].median())
```

Dropping:

```python
df = df.dropna()                             # any row with any missing value
df = df.dropna(subset=['order_id'])          # only rows missing a key
df = df.dropna(axis=1)                       # drop columns instead
```

`dropna()` with no arguments is a blunt instrument. Name the column you care about.

### Text that means the same thing

`Food`, `food`, and `  Food ` are three different groups until you fix them, which quietly splits your totals.

```python
df['category'] = df['category'].str.strip()
df['category'] = df['category'].str.lower()
df['category'] = df['category'].str.title()
```

The `.str` accessor gives you Python string methods on a whole column:

```python
df['item'].str.len()
df['item'].str.upper()
df['item'].str.replace('  ', ' ', regex=False)
df['vendor'].str.split(' - ').str[0]         # take the first piece
```

Collapse variants that normalizing cannot catch:

```python
df['category'] = df['category'].replace({'Bev': 'Drink', 'Beverage': 'Drink'})
```

Confirm it worked by counting groups again:

```python
df['category'].value_counts()
```

### Dates

```python
df['order_date'] = pd.to_datetime(df['order_date'], errors='coerce')
df['order_date'].isna().sum()                # what failed to parse
```

When the format is known and consistent, say so — it is faster and it will not guess wrong:

```python
pd.to_datetime(df['order_date'], format='%m/%d/%Y')
```

Once it is a real datetime, the `.dt` accessor opens up:

```python
df['order_date'].dt.year
df['order_date'].dt.month
df['order_date'].dt.day_name()
df['order_date'].dt.date
```

Then time comparisons work like you expect:

```python
df[df['order_date'] >= '2026-09-01']
```

### Prove it

Every cleaning pass should end with checks that would fail loudly if you got it wrong:

```python
assert df['order_id'].duplicated().sum() == 0
assert df['qty'].notna().all()
assert (df['qty'] > 0).all()
assert pd.api.types.is_datetime64_any_dtype(df['order_date'])
print('rows in:', len(raw), '| rows out:', len(df))
```

Report what changed, not just that you cleaned it. "Dropped 4 duplicate order lines and coerced 3 unparseable prices to missing" is a finding. "Cleaned the data" is not.

---

## SQL to Pandas

| SQL | Pandas |
|---|---|
| `SELECT a, b FROM t` | `df[['a', 'b']]` |
| `WHERE qty > 2` | `df[df['qty'] > 2]` |
| `WHERE a > 2 AND b = 'x'` | `df[(df['a'] > 2) & (df['b'] == 'x')]` |
| `WHERE a IN ('x', 'y')` | `df[df['a'].isin(['x', 'y'])]` |
| `WHERE a IS NULL` | `df[df['a'].isna()]` |
| `ORDER BY qty DESC` | `df.sort_values('qty', ascending=False)` |
| `LIMIT 5` | `df.head(5)` |
| `SELECT DISTINCT a` | `df['a'].unique()` |
| `COUNT(DISTINCT a)` | `df['a'].nunique()` |
| `GROUP BY cat` + `SUM(x)` | `df.groupby('cat')['x'].sum()` |
| `HAVING SUM(x) > 100` | filter the result of the groupby |
| `JOIN ... ON a.id = b.id` | `a.merge(b, on='id')` |
| `LEFT JOIN` | `a.merge(b, on='id', how='left')` |
| `CASE WHEN` | `np.where(...)` or `pd.cut(...)` |
| `UNION ALL` | `pd.concat([a, b])` |

---

## Mistakes you will make once

- Using `and` / `or` instead of `&` / `|` in a filter
- Leaving out the parentheses around each condition
- Filtering, then assigning, without `.copy()`
- Expecting a method to change the DataFrame in place — most return a new one, so assign the result
- Calling `sum()` on a column that is secretly text, and getting concatenation
- `dropna()` with no arguments, which deletes rows you needed
- Merging without checking the row count, and reporting inflated totals
- Grouping on text that has not been normalized, so one category appears three times
- Forgetting `index=False` in `to_csv` and growing a mystery column
- Comparing dates while they are still strings

If a result looks wrong, check `.shape` and `.dtypes` first. Those two catch most of it.

---

## One good workflow

1. Load the file and look at `.info()` and `.head()`.
2. Count what is broken: missing values, duplicates, wrong types.
3. Clean in the order above, one fix per cell, checking after each.
4. Assert the things that must be true.
5. Only then group, join, and answer the question.
6. Print the row count in and the row count out, and say what you changed.

If a chain of operations stops making sense, break it into separate cells and look at the result of each one. Pandas is easiest to debug in small steps.
