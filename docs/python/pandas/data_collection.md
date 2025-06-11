# Notes on Pandas

Data in Pandas is managed via Series and DataFrame objects. 

## Series

A `pandas.Series` object is a data structure for an array of data of a single
type, and can be thought of as a column in a table. Create a Series by 
providing an array of values: 

```python
import numpy as np
import pandas as pd

np.random.seed(0) # set a seed so that the random numbers are always the same
pd.Series(np.random.rand(5), name='random')
```

In addition to the data we provide the Series object, pandas adds in Index
object representing the row numbers. The Index object can be used to perform 
element-wise operations on Series objects. For example, addition using two
Series objects, will add the values for the elements with matching index
values. Where the indexes do not match, the result will return a null value 
(`NaN` or `None`) for those index values. 

## DataFrames

A `pandas.DataFrame` object is a 2-dimensional data structure used to store data in columns. 

To create a DataFrame manually, use a dictionary of lists:

```python
df = pd.DataFrame(
  {
    'Forename': ['Jeff', 'James', 'Eric', 'Clark'],
    'Surname': ['Jefferson', 'Jameson', 'Ericsson', 'Clarkson'],
    'Date of Death': ['15/12/2021', '12/05/2002', '25/10/2023', '28/07/2012'],
    'Place of Death': [np.random.choice(['Norfolk', 'Shropshire', 'Devon', 'Bedfordshire']) for _ in range(4)] ,
    'Deceased': [np.random.choice(['Yes', 'No']) for _ in range(4) ],
    'Age' : np.random.rand(4) * 100
  }
)
```

Or a list of dictionaries: 

```python
df2 = pd.DataFrame([
    {'forename': 'Jeff', 'surname': 'Jefferson', 'date of death': '15/12/2021'},
    {'forename': 'James', 'surname': 'Jameson', 'date of death': '01/10/2021'},
    {'forename': 'Eric', 'surname': 'Ericsson', 'date of death': '11/05/2021'},
    {'forename': 'Clark', 'surname': 'Clarkson', 'date of death': '25/02/2021'},
])
```

Or a list of tuples:

```python
df3 = pd.DataFrame([
    ('Jeff', 'Jefferson', '15/12/2021'),
    ('James', 'Jameson', '01/10/2021'),
    ('Eric', 'Ericsson', '11/05/2021'),
    ('Clark', 'Clarkson', '25/02/2021')],
    columns=['forename', 'surname', 'date of death']
)
```

Or a list of lists:

```python
df5 = pd.DataFrame([
        ['Jeff', 'Jefferson', '15/12/2021'],
        ['James', 'Jameson', '01/10/2021'],
        ['Eric', 'Ericsson', '11/05/2021'],
        ['Clark', 'Clarkson', '25/02/2021']
    ],
    columns=['forename', 'surname', 'date of death']
)
```

Or a `numpy` array:

```python
df4 = pd.DataFrame(
    np.array([
        ['Jeff', 'Jefferson', '15/12/2021'],
        ['James', 'Jameson', '01/10/2021'],
        ['Eric', 'Ericsson', '11/05/2021'],
        ['Clark', 'Clarkson', '25/02/2021']
    ]),
    columns=['forename', 'surname', 'date of death']
)
```

## External Data

DataFrames can be created from CSV files using the pandas `read_csv` function:

```python
df = pd.read_csv("data/unclaimedestates.csv", parse_dates=True)
```

The `read_csv` function provides additional arguments to allow you to 
specifiy seperators, delimiters, etc. Use `help(pd.read_csv)` or 
`pd.read_csv?` for the full function definition. 

There are functions for reading data in various formats, all beginning with
a `read_` prefix: e.g. `read_json`, `read_excel`, `read_sql`. Try: 

```python
functions = [function for function in dir(pd) if function.startswith('read_')]
```

To read data from a database, you need to provide a both a query or table name
and a connection object: 

```python
import pandas as pd
from sqlalchemy import URL, create_engine
import psycopg2

psql_connection_string = URL.create(
    'postgresql',
    username='user_name',
    password='passwd',
    host='server_name,
    port=5432,
    database='database_name'
)
conn = create_engine(psql_connection_string)

query = '''
SELECT * FROM
"schema_name"."table_name"
WHERE collected_date > CURRENT_DATE - INTERVAL '60 days'
ORDER BY collected_date desc
'''

df = pd.read_sql_query(query,con=conn)
```

DataFrames can also be written to external files 
using the corresponding `to_` methods: e.g.
`to_json`, `to_excel`, `to_sql`. With the `to_sql` function you can provide
the table name, connection and what action to take if the table already 
exists: 

```python
df.to_sql(name='users', con=connection, if_exists='replace')
```

Pandas `to_` functions will also allow to you include or exclude
the DataFrame index in the output: 

```python
df.to_excel("recipes.xlsx", sheet_name="flapjack", index=False)
```

## Examining Data

DataFrame objects provide a number of attributes and functions that 
you can use to examine the data frame. 

| Attribute | Description | 
|---|----| 
| shape | number of rows and columns | 
| columns | names of the columns | 
| dtypes | data type for each column | 

All the output from the `shape`, `columns`, and `dtypes` attributes is 
also provided through the `info` function, along with details on the index
and memory usage: 

```python
df.info()
```

The `describe` method provides summary data (min, max, mean, count and percentiles) 
for the numeric columns, but can also be used to get some summary data 
(count, unique values, mode, frequency of mode) for non-numeric data:

```python
df.describe(include=np.object)
# or
df.describe(include='all')
```

By default `describe` provides the 0.25, 0.5, 0.75 percentiles but you can select 
these with the `percentiles` parameter:

```python
df.describe(percentiles=[0.05, 0.50, 0.95])
```

`describe` only provides summary statistics for non-null values.

Each summary statistic is also available via separate summary methods that
can be applied to either the DataFrame or Series objects, e.g `min`, `max`, 
`idxmin`, `idxmax`, `sum`, `mean`, `median`. For Series objects additional 
summary methods exist: `unique`, `value_counts` and `mode`. 

## Selecting Data

Display the dataframe by calling its name. This will display the first 5 rows 
and the last 5 rows. To show the first *n* rows, use the `head` method, or
`tail` for the last *n* rows, or `sample` for *n* random values:

```python
df.sample(10)
```

Each column in a DataFrame is a Series. To select a column from a DataFrame, 
use the column label between square brackets. For multiple columns supply a 
list:

```python
df[["Place of Death", "Date of Birth"]
```

DataFrames support `slicing` operations, which can be combined with column
selection: 

```python
df[["Place of Death", "Date of Birth"]][5:15]
# Order doesn't matter
df[5:15][["Place of Death", "Date of Birth"]]
```

For date indexes, slicing can also be done using dateparts or ranges: 

```python
df[2024]
df[2024-05]
df[2024-01:2024-04]
```

However, using slicing and selection to assign values to a DataFrame
is not the recommended approach: instead you should use the `loc` or 
`iloc` methods. `loc` and `iloc` accept two `labels`: 
a row selector and a column selector. Either selector can be 
expressed as a single value, a list or a slice. :

```python
df.loc[2, 'Surname']
df.loc[[2,3], 'Surname']
df.loc[2:3, 'Surname']

df.loc[df.Surname == 'Johnson', ['Forename','Surname']]
df.loc[df.Surname == 'Johnson', 'Forename':'Surname']

df.loc[[df.Age.idxmin(), df.Age.idxmax()], :]
```

With `loc`, slicing operations are inclusive of the end index if provided. `iloc` 
instead follows the standard python-slicing approach, and excludes the end value
of a slice if provided. `iloc` expects integer values for both row selectors and 
column selectors:

```python
# returns only the value at row 2, column 1 and 2
df.iloc[2:3, 1:3]

# returns values at rows 2 and 3, column 1, 2 and 3
df.iloc[[2,3],[1,2,3]]
```

`iloc` allows slicing for both row and column selectors. 

For selecting a single value from a DataFrame, the `at` and `iat` methods
provide better performance: 

```python
wanted = df.at[2,'Surname']
wanted = df.iat[2,1]
```


## Filtering Data

You can filter the output using a condition inside the selection brackets:

```python
df[df["Date of Birth"] < '01/01/1970']
```

The `isin()` method can be used to check for multiple conditions on 
a column:

```python
df[df["Surname"].isin(["Smith", "Jones"])]
```

You can also use the 'or' operator to acheve the same result: 

```python
df[(df["Surname"] == "Jones") | (df["Surname"] == "Smith")]
```

The **AND Operator** `(&)` can be used to specify multiple conditions
that should all return `True`. The **Negation Operator** `(~)` can 
be used to negate the return value of a condition: 

```python
df[~(df.Surname == 'Jameson')]
```

To filter out rows where a particular column has Null values, use the
`notna()` method:

```
df[df["Executors"].notna()]
```

We can also use strings and regular expressions: 

```python
# contains 'son' anywhere in the string
df[df.Surname.str.contains('son')]

## contains 'son' at the end of the string
df[df.Surname.str.contains(r'son$')]

## contains 'Jeff' or 'James' at the beginning of the string
df[df.Surname.str.contains(r'^Jeff|James')]
```

To filter numeric values with a lower and upper range use `between`:

```python
df[df.Age.between(10,32, inclusive='both')]
```

These filtering techniques can also be used for row labels in both
the `loc` or `iloc` operators.

## Derived Data

To avoid altering our original data we can use `copy` to create
a new copy of the DataFrame:

```python
df_clean = df.copy()
```

You can add new columns to a DataFrame using assignments:

```python
df["Forename Lower Case"] = df["Forename"].str.lower()

df['Retired'] = df.Age > 67
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


The `rename()` method can be used to rename columns:

```python
df_renamed = df.rename(
    columns={
        "Date of Publication": "publication_date",
        "Date of Death": "death_date",
        "Date of Birth": "birth_date",
    }
)
```

## Data Joins

The `concat()` method can be used to combine two tables with a similar structure.
You can specify an axis of '0' to add the second DataFrame as new rows, or an 
axis of '1' to add the DataFrame as new columns. 

To join the rows of two tables use:

```python
disk_data = pd.concat([disk_data_server_1, disk_data_server_2], axis=0)
```

If either DataFrame contains columns not found in the other, these will be
added as new columns in the new DataFrame, and the rows where these columns
did not exist, will be filled with `NaN`. This behaviour can be change by
specifying the join type as 'inner': which only keeps columns that are common
to the input DataFrames. 

The 'ignore_index' option can be used to create a new index for the new 
DataFrame. If you wish to keep the original indices from both input DataFrames, 
you can optionally add an additional (hierarchical) row index to identify which 
row came from which input DataFrame:

```python
disk_data = pd.concat([disk_data_server_1, disk_data_server_2], axis=0, keys=["srv1", "srv2"])
```

This produces a DataFrame with a *multi-index*. To access entries by index value, 
supply a tuple or list of tuples: 

```python
disk_data.loc[[('srv2',0),('srv2',1)],:]
```

When using `concat` in a columnwise join (`axis=1`), Pandas uses the row indexes 
to make the joins. For rows that don't share a common index, values are filled
with `NaN`. 

