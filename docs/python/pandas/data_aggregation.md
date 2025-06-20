# Data Aggregation

## Merging DataFrames

The pandas `concat` function works similar to the SQL `UNION ALL`: 
by appending one dataset to another. The pandas `merge` method
is equivalent to an SQL `JOIN`, and support inner, outer and left 
and right joins.

```python
class_lookup = pd.DataFrame(
    [ ( 1, 'First'), (2, 'Second'), (3, 'Third') ],
    columns = ['id', 'description']
)

inner_join = df.merge(class_lookup, left_on='Pclass', right_on='id')
```

The `left_on` and `right_on` parameters are necessary because the two
dataframes use different names for the 'passenger class' field. This 
join also means that our resulting `inner_join` DataFrame contains two 
columns containing the same data: 'Pclass' and 'id'. We can rename the 
column in one DataFrame to match the other, resulting in only one 
column for the 'passenger class': 

```python
inner_join = df.merge(
    class_lookup.rename(dict(id='Pclass'), axis=1),
    on='Pclass'
)
```

Left, Right and Outer Joins are made using the `how` parameter: 


```python
left_join = df.merge(
    class_lookup.rename(dict(id='Pclass'), axis=1),
    on='Pclass', how='left'
)
```

Outer joins also allow an `indicator` parameter, that adds a column
to indicate if the row resulted from a match on both tables, or from 
no match on either the left or right side of the join: 


```python
outer_join = df.merge(
    class_lookup.rename(dict(id='Pclass'), axis=1),
    on='Pclass', how='outer', indicator=True
)
```

`merge` operations can also be done using the DataFrame indexes: 

```python
inner_join = df.merge(
    class_df, left_index=True, right_index=True
)
```

If both DataFrames have columns with the same names, the columns
from the left DataFrame will be suffixed with `_x` and the 
columns from the right DataFrame with `_y`. You can alter this 
behaviour with the `suffixes` parameter, providing a tuple of 
the required suffixes. 

If joining DataFrames using indexes, then you can use `join` instead
of `merge`. The left table must use the index, but for the right table
you can specify a column using the `on` parameter: 

```python
inner_join = df.join(
    class_df, lsuffix='_l', rsuffix='_r'
    on='passenger_class_id'
)
```

Use **set** operations to test the effects of joins before they're made. 
An Inner Join is represented as an intersection: 

```python
patient.set_index('patient_id', inplace=True)
pathology.set_index('patient_id', inplace=True)

patient.index.intersection(pathology.index)
```

The difference shows what will be lost in the join, either where the join
type is inner, left or right: 

```python
# index values in patient but not in pathology
patient.index.difference(pathology.index)

# index values in pathology but not in patient
pathology.index.difference(patient.index)
```

## Aggregate Functions

Pandas provides access to aggregate methods that can be applied to one or more
columns:

```python
print(df[["Date of Death", "Date of Publication"]].max())
print(df[["Date of Death", "Date of Publication"]].min())
```

The `agg()` method allows you to apply multiple aggregates to a DataFrame:

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

Data can be sorted using the `sort_values()` or `sort_index()` methods:

```python
print(df.sort_values(by=["Date of Birth", "Place of Birth"], ascending=True)[["Date of Birth", "Place of Birth"]])
```

You can select the largest or smallest values for a group of columns using
`nlargest` or `nsmallest`: 

```python
df.nlargest(n=10, columns=['Nationality', 'Religion']
```

## Matplotlib

To create a graph of a DataFrame use the `plot()` method. Pandas creates
a line plot by default for each of the Series in a DataFrame 
that has numeric values. All the plots created by Pandas are `Matplotlib`
objects. The `matplotlib.pyplot` library provides the `show()` method 
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
In JupyterLab, you can specify the x and y axes for your plot:

```python
plt.plot(df.index, df['java'])
```

You can use slicing to drop elements from the plot:

```python
plt.plot(df.index[:-1], df['java'][:-1])
```

You can also plot mulitple columns on the same graph, and add some graph
formating:

```python
plt.figure(figsize=(16,10))
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.xlabel('Date', fontsize=14)
plt.ylabel('No of Posts', fontsize=14)
plt.ylim(0,35000)

plt.plot(df.index, df['java'])
plt.plot(df.index, df['python'])
```

To plot multiple columns use a `for` loop:

```python
plt.figure(figsize=(16,10))
plt.xticks(fontsize=14)
plt.yticks(fontsize=14)
plt.xlabel('Date', fontsize=14)
plt.ylabel('No of Posts', fontsize=14)
plt.ylim(0,35000)

for column in df.columns:
    plt.plot(df.index, df[column], linewidth=3, label=df[column].name)

plt.legend(fontsize=14)
```

For time series data you can specify a rolling mean to smooth out the data:
instead of plotting the value of each data point, you can specify a window 
to calculate an average value for each data point based on the values either
side of the data point:

```python
df['rolling_mem'] = df.mem_used_percent.rolling('3D').mean()
```

To calculate a rolling average, your time-series data must be monotonic: 
that is the index must change in equal steps. Missing dates are not allowed.
Check for monotonicity using either `df.index.is_monotonic_increasing` or
`df.index.is_monotonic_decreasing`.

Plotting two columns with varying value-ranges can look untidy and make
trend-spotting difficult. Instead you can specify two different y-axes
for each plot to make comparison easier:

```python
ax1 = plt.gca() # gets the current axis
ax2 = ax1.twinx() # create another axis that shares the same x-axis

ax1.plot(sets_by_year.index[:-2], sets_by_year.set_num[:-2], color='g')
ax2.plot(themes_by_year.index[:-2], themes_by_year.nr_themes[:-2], color='b')

ax1.set_xlabel('Year')
ax1.set_ylabel('Number of Sets', color='g')
ax2.set_ylabel('Number of Themes', color='b')
```

If your xticks overlap, you can specify a rotation to make them readable:

```python
plt.xticks(fontsize=14, rotation=45)
```

You can also use locator functions for marking the x- and y- axes:

```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

df_unemployment = pd.read_csv('UE Benefits Search vs UE Rate 2004-19.csv')
df_unemployment['MONTH'] = pd.to_datetime(df_unemployment['MONTH'])

roll_df = df_unemployment[['UE_BENEFITS_WEB_SEARCH', 'UNRATE']].rolling(window=6).mean()
roll_df['month'] = df_unemployment['MONTH']


plt.figure(figsize=(16,10))
plt.title('Rolling Web Searches vs Unemployment Rate', fontsize=20)
plt.xticks(fontsize=14, rotation=45)
plt.yticks(fontsize=14)

ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.grid(color='gray', linestyle='--')

ax1.set_ylabel('Web Searches', fontsize=16, color='indianred')
ax2.set_ylabel('Unemployment Rate', fontsize=16, color='cadetblue')

ax1.set_xlim(roll_df.month[5:].min(), roll_df.month[5:].max())

years = mdates.YearLocator()
months = mdates.MonthLocator()
years_fmt = mdates.DateFormatter('%Y')
ax1.xaxis.set_major_locator(years)
ax1.xaxis.set_major_formatter(years_fmt)
ax1.xaxis.set_minor_locator(months)

ax1.plot(roll_df.month[5:], roll_df.UE_BENEFITS_WEB_SEARCH[5:], color='indianred', linewidth=3, marker='o')
ax2.plot(roll_df.month[5:], df_unemployment.UNRATE[5:], color='cadetblue', linewidth=3)
```



Pandas provides a number of graph types including line, area, bar, pie and scatter:

- `plt.plot` for a line chart
- `plt.scatter` for a scatter plot
- `plt.bar` for a bar chart


## Plotly

Bar charts are a good way to visualise 'categorical' data. You can use 
`value_counts()` to quickly create categorical data:

```python
ratings = df.value_counts('Content_Rating')
```

Given a dataframe that contains the following data:

```
Content_Rating
Everyone           6621
Teen                912
Mature 17+          357
Everyone 10+        305
Adults only 18+       3
Unrated               1
Name: count, dtype: int64
```

we can present this as a pie chart using plotly:

```python
fig = px.pie(
    labels=ratings.index,
    values=ratings.values,
    names=ratings.index,
    title="Content Rating"
)
fig.show()
```

We can change the location of the text on the pie chart using `update_traces()`:

```python
fig = px.pie(
    labels=ratings.index,
    values=ratings.values,
    title='Content Rating',
    names=ratings.index,
)
fig.update_traces(
    textposition='outside', 
    textinfo='percent+label'
)
fig.show()
```

To format the pie chart as a 'doughnut chart' add the 'hole' parameter:

```python
fig = px.pie(
    labels=ratings.index,
    values=ratings.values,
    title='Content Rating',
    names=ratings.index,
    hole=0.4
)
fig.update_traces(
    textposition='inside',
    textfont_size=15,
    textinfo='percent'
)
fig.show()
```

Which produces:<br>

![doughnut](../images/doughnut.png)

