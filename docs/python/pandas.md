# Notes on Pandas

Notes from the Pandas 
[Getting Started](https://pandas.pydata.org/docs/getting_started/index.html) 
guide

## Data in Pandas

A DataFrame is a 2-dimensional data structure used to store data in columns. 
To create a DataFrame manually, use a dictionary of lists:

```python
df = pd.DataFrame(
  {
    'BV Reference': ['BV22400430', 'BV22033859/1', 'BV22028098/1', 'BV2020383/1'],
    'Date of Publication': ['02/04/2023', '03/04/2023', '03/04/2023', '04/04/2023'],
    'Forename': ['Jeff', 'Dorothy', 'Sasan', 'Elizabeth'],
    'Surname': ['British', 'Irish', 'British', 'Norwegian'],
    'Date of Death': ['15/12/2021', '12/05/2002', '25/10/2023', '28/07/2012'],
    'Place of Death': ['Norfolk', 'Shropshire', 'Devon', 'Bedfordshire'],
    'Executors': ['', '', '', '' ]
  }
)
```

DataFrames can be created from CSV files using the `read_csv()` function:

```python
df = pd.read_csv("data/unclaimedestates.csv", parse_dates=True)
```

There are functions for reading data in various formats, all beginning with
a `read_` prefix: e.g. `read_json()`, `read_excel()`, `read_sql()`.
DataFrames can also be written using the corresponding `to_` functions: e.g.
`to_json()`, `to_excel`, `to_sql()`.

```python
df.to_excel("recipes.xlsx", sheet_name="flapjack")
```

Each column in a DataFrame is a Series. To select a column from a DataFrame, 
use the column label between square brackets:

```python
df["Place of Death"]
```

You can do things to the data by applying methods to the DataFrame or Series:

```python
df["Date of Death"].max()
```

Display the dataframe by calling its name. This will display the first 5 rows 
and the last 5 rows. To show the first 10 rows, use the `head()` function:

```python
df.head(10)
```

Use `tail()` to see the last *n* rows.

To check the datatypes applied to each Series in your DataFrame, use the
`dtypes` attribute. A technical summary of the data is available using the
`info()` method. Use the `describe()` method to get an aggregation summary 
of the numeric values in your data. 

Check the `shape` attribute to see the number of rows and columns in your 
DataFrame:

```python
df[["Forename", "Surname"]].shape
```

The outer brackets are used to select the data from a DataFrame, and the 
inner brackets are used to supply a list of columns. 

## Filtering Data

You can filter the output using a condition inside the selection brackets:

```python
df[df["Date of Birth"] < '01/01/1970']
```

The `isin()` function can be used to check for multiple conditions on 
a column:

```python
df[df["Surname"].isin(["Smith", "Jones"])]
```

You can also use the 'or' operator to acheve the same result: 

```python
df[(df["Surname"] == "Jones") | (df["Surname"] == "Smith")]
```

To filter out rows where a particular column has Null values, use the
`notna()` function:

```
df[df["Executors"].notna()]
```

If you need to filter both rows and columns use the `loc` or `iloc` 
operators before the selection `[]` brackets: 

```python
df.loc[df["Surname"] == "Jones", "Forename"]
```

The first parameter for `loc` specifies the row selection criteria, 
and the second parameter specifies the columns to select. The `iloc`
operator allows you to specify rows and columns by their index value. 
For example to extract the first 2 rows, and display columns 1 and 4
use:

```python
df.iloc[0:2, [0,3]]
```

Both `loc()` and `iloc()` can be used to change the values of elements in the
DataFrame. To change the value of all items in column four to 'anonymised' use: 

```python
df.iloc[:, 3] = "anonymised"
print(df["Surname"])
```

To change the values in rows 1 and 2 for column three use:

```python
df.iloc[0:2, 2] = 'unknown'
```

## Derived Data

You can add new columns to a data frame using assignments:

```python
df["Forename Lower Case"] = df["Forename"].str.lower()

print(df[["Forename", "Forename Lower Case"]])
```

Mathematical and logical operators can be used on the right-hand side
of the assignment:

```python
df["Died at Home Town"] = df["Place of Birth"] == df["Place of Death"]
```

The `rename()` function can be used to rename columns:

```python
df_renamed = df.rename(
    columns={
        "Date of Publication": "publication_date",
        "Date of Death": "death_date",
        "Date of Birth": "birth_date",
    }
)
```

## Aggregate Functions

Pandas provides access to aggregate functions that can be applied to one or more
columns:

```python
print(df[["Date of Death", "Date of Publication"]].max())
print(df[["Date of Death", "Date of Publication"]].min())
```

The `agg()` function allows you to apply multiple aggregates to a DataFrame:

```python
df_aggregations = df.agg(
    {
        "Date of Death": ["min", "max", "count", ],
        "Date of Birth": ["count"],
        "Date of Publication": ["max"]
    }
)
```

Use `group_by()` to group aggregates:

```python
religions = df[["Nationality", "Religion"]].groupby("Religion").count()
```

Data can be sorted using the `sort_values()` or `sort_index()` functions:

```python
print(df.sort_values(by=["Date of Birth", "Place of Birth"], ascending=True)[["Date of Birth", "Place of Birth"]])
```

The `pivot()` function can be used to reshape data, specifying columns and values. 
The `pivot_table()` function supports aggregation. The reverse of `pivot()` is `melt()`.

## Data Joins

The `concat()` function can be used to combine two tables with a similar structure.
To join the rows of two tables with identical column names, use:

```python
disk_data = pd.concat([disk_data_server_1, disk_data_server_2], axis=0)
```

To identify which row came from which DataFrame, you can add an additional 
(hierarchical) row index:

```python
disk_data = pd.concat([disk_data_server_1, disk_data_server_2], axis=0, keys=["srv1", "srv2"])
```

The merge function works similar to an SQL JOIN statement, producing a DataFrame 
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

Use the `to_datetime()` function to work with datetime data in your columns.:
Use the `dayfirst` parameter for dates in "%d/%m/%Y" format. 

```
df = pd.read_csv("ukgovunclaimedestates.csv", parse_dates=True)

df["Date of Birth"] = pd.to_datetime(df["Date of Birth"], dayfirst=True)
df["Date of Death"] = pd.to_datetime(df["Date of Death"], dayfirst=True)

print(df.groupby(df["Date of Birth"].dt.year)["Date of Birth"].count())

df["Age"] = df["Date of Death"].dt.year - df["Date of Birth"].dt.year
```

Converting columns to DateTime objects, provides access to utility functions
to extract 'year', 'month', 'day', 'weekday' and perform calculations via 
the `dt()` methods. If you set the index (using the `set_index()` function)
to a DateTime object, then you can use the same methods on the index. 

```python
df["date_of_birth"] = pd.to_datetime(df["Date of Birth"], dayfirst=True)
df.set_index("date_of_birth", inplace=True)

print(df.index.year)
```

## Working with String Values

Using the `str()` accessor on text data, allows us to access various 
string methods, including `lower()`, `split()`, `replace()`

`split()` returns a number of elements from a single element: use the
the `get()` method to select which element you want:

```python
df["Surname"] = df["Name"].str.split(", ").str.get(0)
df["Forename"] = df["Name"].str.split(", ").str.get(1)
```

`replace` uses a dictionary to replace values:

```python
df["Gender"] = df["Sex"].replace({"male": "m", "female": "f"})
```






## Matplotlib

To create a graph of a DataFrame use the `plot()` method. Pandas creates
a line plot by default for each of the Series in a DataFrame 
that has numeric values. All the plots created by Pandas are `Matplotlib`
objects. The `matplotlib.pyplot` library provides the `show()` function 
to display the graph: 

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("ukgovunclaimedestates.csv", parse_dates=True)

religions = df["Religion"].groupby(df["Religion"]).count()

religions.plot()

plt.show()
```

Use selection criteria on your DataFrame to choose which Series to plot. 

Pandas provides a number of graph types including line, area, bar, pie and scatter:

```python
method_name = ''
for method_name in dir(religions.plot):
    if not method_name.startswith("_"):
        print(method_name)

religions.plot.bar()

plt.show()
```

## Row Comprehensions

The `to_dict()` method of a DataFrame will produce a dictionary mapping the column names to 
an array of the column values. If you want a dictionary mapping row values to other row values
you can use the `iterrows()` function. 

The `iterrows()` function returns a list of tuples containing the index and row values for 
each row. This can be used in a comprehension to create a new dictionary:

```python
df = pd.read_csv("ukgovunclaimedestates.csv", parse_dates=True)

person_dict = {row["BV Reference"]:row["Surname"] for (index, row) in df.iterrows()}
```
