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
df[['CostInBillingCurrency', 'EffectivePrice', 'Quantity']].sum()
```

Additional aggregate functions include `mean`, `count`, `min`, `max` and `cumsum`. 

The `agg` method allows you to apply multiple aggregates to multiple 
columns in a DataFrame:

```python
df.agg({
    'CostInBillingCurrency': ['sum', 'mean'],
    'EffectivePrice': ['min', 'max', 'mean'],
    'Quantity': ['sum', 'min', 'max'],
})
```

The `groupby` method allows you to run aggregates after grouping
rows of data: 

```python
df.groupby('ResourceGroup').sum()
```

However, before applying the aggregate function, it would be sensible to 
select the columns that you wish to apply the function to, otherwise
you'll be applying the function on all columns in the DataFrame: 

```python
df[['ResourceGroup', 'CostInBillingCurrency']].groupby('ResourceGroup').sum()
```

Alternatively, the `agg` method can be combined with `groupby` to select the
columns and the specific aggregation needed: 

```python

df[
    ['MeterSubCategory', 'EffectivePrice', 'Quantity', 'CostInBillingCurrency']
].groupby(
    ['MeterSubCategory', 'EffectivePrice']
).agg(['sum', 'max'])

df.groupby(['MeterSubCategory', 'EffectivePrice']
).agg({
    'Quantity': 'sum',
    'CostInBillingCurrency': 'sum',
})
```

Note that `agg` accepts an array if you want to supply the same aggregates to the 
columns, or a dictionary if you want to specify which aggregations to apply to 
each column. 

Using multiple aggregates results in a MultiIndex in the columns: 

```python
df_agg = df[
    ['MeterSubCategory', 'EffectivePrice', 'Quantity', 'CostInBillingCurrency']
].groupby(
    ['MeterSubCategory', 'EffectivePrice']
).agg(['sum', 'max'])

df_agg.columns

MultiIndex([(             'Quantity', 'sum'),
            (             'Quantity', 'max'),
            ('CostInBillingCurrency', 'sum'),
            ('CostInBillingCurrency', 'max')],
           )
```

However, you can remove this hierarchical index, by processing the column tuples into
an array of scalars: 

```
df_agg.columns = ['_'.join(tuple_col) for tuple_col in df_agg.columns]
```

You can include a DateTimeIndex or PeriodIndex in the `groupby`, 
and sort the output by values or index: 

```python
df.groupby([df.ResourceGroup, df.index.month]).agg({
    'CostInBillingCurrency': 'sum'
}).sort_values('CostInBillingCurrency')

df.groupby([df.ResourceGroup, df.index.month]).agg({
    'CostInBillingCurrency': 'sum'
}).sort_index()

```

You can select the largest or smallest values for a group of columns using
`nlargest` or `nsmallest`: 

```python
df.groupby([df.ResourceGroup, df.index.month]).agg({
    'CostInBillingCurrency': 'sum'
}).nlargest(n=10, columns=['CostInBillingCurrency'])
```

The `groupby` method accepts `Grouper` objects: for a DataFrame with a DatetimeIndex, 
you can set a quarterly grouper: 

```python
df['CostInBillingCurrency'].groupby(pd.Grouper(freq='QE')).sum()
```

By selecting the frequency in the Grouper, you can aggregate over time
periods in a DatetimeIndex: 

```python
df.loc['2024-12-01': '2024-12-15', 'CostInBillingCurrency'].groupby(pd.Grouper(freq='D')).sum()
```

The `resample` method allows you to aggregrate data to a different granularity: 

```python
weekly_df = df.resample('1W').agg({
    'CostInBillingCurrency': 'sum',
})
```

