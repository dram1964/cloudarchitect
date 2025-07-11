# Data Cleansing

Before carrying out any data analysis tasks, the data should be prepared for 
analysis. This process will involve a number of tasks including: 

- Renaming columns
- Re-ordering columns and rows
- Data-type conversions
- Deduplication
- Handling missing or invalid data
- Filtering out unneccessary data

To avoid altering our original data we can use `copy` to create
a new copy of the DataFrame:

```python
df_clean = df.copy()
```

## Columns

The `rename` method can be used to rename columns in your DataFrame:

```python
df_renamed = df.rename(
    columns={
        "Date of Publication": "publication_date",
        "Date of Death": "death_date",
        "Date of Birth": "birth_date",
    },
)
```

You can also use lambda functions to achieve the same result:

```python
df_renamed = df.rename(lambda x: x.lower().replace(' ', '_'), axis=1)
```

Sort columns using `sort_index(axis=1)`. 

You can use pandas conversion functions to change the datatypes for 
columns: 

```python
df_renamed.date_of_birth = pd.to_datetime(df_renamed.date_of_birth)
```

Use `format='mixed'` if your date strings are in different formats:

```python
data = {
    'First Name': ['John', 'Jack', 'Jane', 'Jill', 'Jake'],
    'Family Name': ['Johnson', 'Jackson', 'Janeson', 'Jillson', 'Jakeson'],
    'Date of Birth': ['2001-05-20', '1980-12-13 12:24:00', 
        '1999-12-31 03:30:00', '1964-06-20', '2020-05-12 17:32'],
}

df = pd.DataFrame(data)
df_renamed = df.rename(lambda x: x.lower().replace(' ', '_'), axis=1)
df_renamed.date_of_birth = pd.to_datetime(
    df_renamed.date_of_birth, format='mixed'
)
```

Altering the data type of a column containing dates from 'object'
to 'datetime' means we get fuller statistics via the `describe` method
and can carry out operations associated to datetime objects:

```python
df_renamed['time_of_birth'] = df_renamed.date_of_birth.dt.time
df_renamed['day_of_birth'] = df_renamed.date_of_birth.dt.date
```

To convert strings to numeric data, you can perform string formatting
and then use the pandas `to_numeric` function: 

```python
# remove commas
df_clean['Installs'] = df_clean['Installs'].astype(str).str.replace(',',"")
# remove currency symbol
df_clean['Installs'] = df_clean['Installs'].astype(str).str.replace('£',"")
# convert to numeric
df_clean['Installs'] = pd.to_numeric(df_clean['Installs'])
```

`astype` can be used to perform some type conversions. Float64 values
can be converted to Int64 values (`astype(int))`. No rounding is performed, everything
after the decimal point is lost: 

```python
data = {
    'forename': ['John', 'Jack', 'Jane', 'Jill', 'Jake'],
    'surname': ['Johnson', 'Jackson', 'Janeson', 'Jillson', 'Jakeson'],
    'date_of_birth': ['2001-05-20', '1980-12-13 12:24:00', '1999-12-31 03:30:00', '1964-06-20', '2020-05-12 17:32'],
    'sats_score': np.random.random(5) * 100,
    'marital_status': ['divorced', 'beheaded', 'died', 'divorced', 'beheaded']
}

df1 = pd.DataFrame(data)
df1['sats_score_truncated'] = df1.sats_score.astype('int')
```

Rather than changing the precision of your Float64 objects, you can change the display format of Float64 values, to show only the first 2 decimal places: 

```python
pd.set_option('display.float_format', lambda x: '%.2f' % x)
# Alternate method
pd.options.display.float_format = "{:,.2f}".format

```

While this will change the formatting for pandas objects such as DataFrames or Series, `print` statements
and calculations will still display with scientific-precision. To alter this behaviour use `round` or
a format: 

```python
df.CostInBillingCurrency.sum().round(2)
# Alternate method
'{:,.2f}'.format(df.CostInBillingCurrency.sum())
```

The marital status column can be converted from an 'Object' type to a 'Category' type
using `astype`: 

```python
df1.marital_status = df1.marital_status.astype('category')
```

Use the `drop()` method to remove unwanted rows or columns. Set `inplace=True`
to alter the current DataFrame or use assignment to capture the resulting 
DataFrame. 

`drop` defaults to removing rows (`axis=0`), but we can also drop columns either
by specifying `axis=1` or by providing a list to the `columns` parameter: 

```python
# creates new dataframe and leaves the original unchanged
nameless_df = df.drop(columns=['Forename', 'Surname'])

# affects the original dataframe
df.drop(['Forename', 'Surname'], axis=1, inplace=True)
```

`dropna` can be used to drop columns by specifying the axis, and a 
threshold of rows that contain null values: 

```
df_clean = df.dropna(
    axis='columns',
    thresh=3 # keep if 3 or more non-null values
)

df_clean = df.dropna(
    axis=1,
    thresh=df.shape[0]*.75 # keep if 75% or more non-null values
)
```

We can also use `del df['column_name']` to remove a specific column from 
a DataFrame. 

## Indexes

If no index is specified, pandas will add an index when creating your
DataFrame. This will be a `RangeIndex`, containing integers from 0 to n-1, where
n is the number of rows in your dataset. 

You can specify an index using a specific column from your data, or generating
one manually: 

```python
data = {
    'forename': ['John', 'Jack', 'Jane', 'Jill', 'Jake'],
    'surname': ['Johnson', 'Jackson', 'Janeson', 'Jillson', 'Jakeson'],
    'date_of_birth': ['2001-05-20', '1980-12-13 12:24:00', '1999-12-31 03:30:00', '1964-06-20', '2020-05-12 17:32'],
}

df1 = pd.DataFrame(data, index=pd.date_range(start='2025-06-01', periods=5, freq='D'))

df2 = pd.DataFrame(data)
df2.date_of_birth = pd.to_datetime(df2.date_of_birth, format='mixed')
df2.index = df2.date_of_birth
```

Note that the number of elements in the `date_range` should match the number of
rows in the DataFrame. Both `df1` and `df2` now have an index of type `DatetimeIndex'. 

You can localise your DatetimeIndex (make them timezone-aware) and convert them to UTC: 

```python
# df2 = df2.tz_localize('GMT+0')
df2 = df2.tz_localize('CET')
df2 = df2.tz_convert('UTC')
```

A DatetimeIndex or a PeriodIndex can be used to slice a DataFrame by dates, but this only
works if your Datetime Index has been sorted first: 

```python
df_dated.set_index('date_of_birth', inplace=True)
df_dated.sort_index(inplace=True)

df_dated['1964':'2001']

df_dated['2001-05':'2020']
df_dated['200105':'2020']

df_dated['1980': '2025-01-01']

```

A DatetimeIndex can be converted to PeriodIndex, where the index values are
converted to the specified period (e.g. Week, Month, Year):

```python
df2 = df2.to_period('M')
```

Use `sort_index` to sort data by index value. Both `sort_index` and
`sort_values` return a new DataFrame, so you have to use assignment
to keep the sorted data, or use `inplace=True`. 

`reset_index` will copy the current index on a DataFrame to a new 
column (preserving it's current name), and create a new RangeIndex 
for the DataFrame. 

`set_index` can be used to drop the current index, and replace it
with one or more columns from the DataFrame. Rows from a MultiIndex are
accessed using tuples: `(outer_index, inner_index)`. If you set a MultiIndex, 
you can undo this later with `unstack`. The `unstack` method will move 
the innermost index column to the DataFrame columns. 

Note that `df.set_index('column_name', inplace=True)` works differently
to `df.index = df.column_name`: `df.index` will **copy** the column from the 
data and use this as the index, whereas `set_index` will **move** the column
from the data and use this as the index.

`reindex` can be used to align the DataFrame to a new index. Where the
DataFrame is missing values for the new index, you can pick a method
to fill-in the row values: 

- ffill - forward fill from nearest previous value
- bfill - back fill from nearest next value
- nearest - fill from the nearest value

## Derived Data

You can add new columns to a DataFrame using assignments:

```python
df["Forename Lower Case"] = df["Forename"].str.lower()

df['Retired'] = df.Age > 67
```

Use the `map` function to apply a dictionary lookup to add a column:

```python

workspace_names = {
    'rg1': 'project one',
    'rg2': 'project_two',
    ...
    'rgN': 'project_N',
}

df['ProjectName'] = df['ResourceGroup'].map(workspace_names)
```

The `assign` method allows us to add multiple columns at once. However,
`assign` does not change the original DataFrame, but returns a new one. 
If you want to change the original, just assign the return value back to 
the original DataFrame: 

```python
df = df.assign(
    is_retired=df.Age >= 67,
    is_working_age=(df.Age >= 18) & (df.Age < 67),
    is_child=df.Age < 18
    income=lambda x: np.where(x.salary.isnull(), x.pension, x.salary),
)
```

The same `assign` statement can be written with `lambda` functions: 

```python
df = df.assign(
    is_retired=lambda x: x.Age >= 67,
    is_working_age=lambda x: x.Age.between(18,67,inclusive='left'),
    is_child=lambda x: x.Age < 18
)
```

Using lambda functions in your assign statements offers the benefit of 
being able to reference other columns that are being assigned in the 
same statement. 

## Rows

To drop rows we need to provide a list of indices: `df.drop([1,2], inplace=True)`.

Use `isna` to find rows with null values in a specified column(s):

```python
df_null_rating_or_age = df[df.Rating.isna() | df.Age.isna()]
```

If used on the DataFrame, `isna` will return a DataFrame with the values 
replaced with TRUE or FALSE to indicate null or non-null values. 

If you use `dropna` on the DataFrame, this will return a DataFrame containing only rows 
where none of the values are null: in other words, **if any value in the row is 
null, then it will be dropped**. This behaviour can be changed using the `how` 
parameter. The default behaviour is `any`: but you can set the parameter to `all`, 
so that only rows where all values are null are dropped. Combining this with the
`subset` parameter allows you to drop only rows where all the values in the 
specified columns are null:

```python
## drop rows where the subset columns are all null
df_clean = df.dropna(how=all, subset=['home', 'var', 'lib'])

## drop rows where there is a null value in any of the subset columns
df_clean = df.dropna(how=any, subset=['home', 'var', 'lib'])
```

`fillna` can be used replace `NaN` values: 

```
df = df.assign(
    salary=lambda x: x.salary.fillna('0.00'),
    free_space=lambda x: x.free_space.ffill(),
    inplace=True,
)

df_clean.loc[:, 'pension'].fillna('0.00', inplace=True)
```

Use `inplace=True` if you don't want to create a new dataframe.

You can also use `apply` to apply the same fill strategy to all columns in the 
DataFrame or a list of columns: 

```python
df_clean = df.apply(lambda x: x.ffill(inplace=True))
```

Note that apply does not have a `inplace` option, so you have to reassign
the DataFrame or DataFrame slice: 

```python
df.loc[:, ['name', 'toy']] = df.loc[:, ['name', 'toy']].apply(
            lambda x: x.str.replace('Bat', 'Cat')
)
```

Apply lets you run vectorised operations on rows or columns, which is the 
preferred method in Pandas. Although methods like `iterrows` and `iteritems` 
exist, these should be used only as a last resort as if is less efficient to 
loop through items one-by-one in Pandas. There is an `applymap` method to run 
a function that has not been vectorised, or `np.vectorize` to vectorise your
own functions.



`duplicated` returns all duplicated rows, apart from the first occurrence of the row. 
The `subset` parameter can be used to specify which columns to use for the comparison.
Use `drop_duplicates` to remove the duplicated rows (keeping the first occurrence):

```python
df_clean = df.drop_duplicates(subset = ['App', 'Type', 'Price'])
```

You can use filtering to drop unwanted values:

```python
df_clean = df_clean[df_clean.Installs > 300]
```

## Reshaping DataFrames

The `pivot()` method can be used to reshape data from long-format to 
wide-format:

```python
import pandas as pd

data = [
    ('2025-01-06', 'home', 97),
    ('2025-01-06', 'opt', 36),
    ('2025-01-06', 'var', 83),
    ('2025-01-07', 'home', 96),
    ('2025-01-07', 'opt', 36),
    ('2025-01-07', 'var', 83),
    ('2025-01-08', 'home', 95),
    ('2025-01-08', 'opt', 45),
    ('2025-01-08', 'var', 63),
]

df = pd.DataFrame(data, columns=['date', 'mount_point', 'free_space'])
df.date = pd.to_datetime(df.date)

pivot_df = df.pivot(index='date', columns='mount_point', values='free_space')

```

If you pass in a list to the values parameter, you will have a hierarchical index
in the columns, which you will access using `[<value_column_name>][<column_name>]`: 

The reverse of `pivot()` is `melt()`, which will turn data in wide-format 
to long-format:

```python
wide_data = [
    ('2025-01-06', 97, 36, 83),
    ('2025-01-07', 96, 36, 83),
    ('2025-01-08', 95, 45, 63)
]
df2 = pd.DataFrame(wide_data, columns=['date', 'home', 'opt', 'var'])

melt_df = pivot_df.melt(
    id_vars='date',
    value_vars=['home', 'opt', 'var'],
    var_name='mount_point'
    value_name='free_space',
)
```

The `merge` method works similar to an SQL JOIN statement, producing a DataFrame 
that results from combining DataFrames using a common Series:

```python
patient_result_data = pd.merge(["patient", "pathology"], how="left", on="patient_id")
```

If the two tables to merge have the same data in different column names, use the 
`left_on` and `right_on` parameters:

```python
patient_result_data = pd.merge(["patient", "pathology"], 
    how="left", left_on="patient_id", right_on="hospital_number")
```

## Working with Datetime Values

Use the `to_datetime()` method to work with datetime data in your columns.:
Use the `dayfirst` parameter for dates in "%d/%m/%Y" format. 

```
df = pd.read_csv("ukgovunclaimedestates.csv", parse_dates=True)

df["Date of Birth"] = pd.to_datetime(df["Date of Birth"], dayfirst=True)
df["Date of Death"] = pd.to_datetime(df["Date of Death"], dayfirst=True)

print(df.groupby(df["Date of Birth"].dt.year)["Date of Birth"].count())

df["Age"] = df["Date of Death"].dt.year - df["Date of Birth"].dt.year
```

Converting columns to DateTime objects, provides access to utility methods
to extract 'year', 'month', 'day', 'weekday' and perform calculations via 
the `dt()` methods. If you set the index (using the `set_index()` method)
to a DateTime object, then you can use the same methods on the index. 

```python
df["date_of_birth"] = pd.to_datetime(df["Date of Birth"], dayfirst=True)
df.set_index("date_of_birth", inplace=True)

print(df.index.year)
```

## Working with String Values

Text data is stored in 'Object' format in a DataFrame. Using the `str()` accessor on 
text data, allows access various string methods, 
including `lower`, `split`, `replace`

`split` returns a number of elements from a single element: use the
the `get` method to select which element you want:

```python
df["Surname"] = df["Name"].str.split(", ").str.get(0)
df["Forename"] = df["Name"].str.split(", ").str.get(1)
```

`replace` uses a dictionary to replace values:

```python
df["Gender"] = df["Sex"].replace({"male": "m", "female": "f"})
```

## Row Comprehensions

The `to_dict()` method of a DataFrame will produce a dictionary mapping the column names to 
an array of the column values. If you want a dictionary mapping row values to other row values
you can use the `iterrows()` method. 

The `iterrows()` method returns a list of tuples containing the index and row values for 
each row. This can be used in a comprehension to create a new dictionary:

```python
df = pd.read_csv("ukgovunclaimedestates.csv", parse_dates=True)

person_dict = {row["BV Reference"]:row["Surname"] for (index, row) in df.iterrows()}
```
