# Statistics

## Z-Score

A z-score is a statistical measure that quantifies how many standard
deviations a data point is from the mean. The formula for calculating
a z-score is: 

```
z = (x - μ) / σ
```

Or z equals x - mean / standard deviation. 

To add a z-score to a dataframe try: 

```python
score_df = df.assign(
    z_score=lambda x: x.Age.sub(x.Age.mean()).div(x.Age.std()),
    z_score_absolute=lambda x: x.z_score.abs(),
)
```

## Rate of Change

In a DataFrame with a Datetime Index, you can calculate the change in a column 
from one day to the next using `diff` or `pct_change` and also use `rank` to 
rank the size of the change: 

```python
change_df = df.assign(
    actual_change=lambda x: x.docker_volume_usage_percent.diff().fillna(0).abs(),
    percent_change=lambda x: x.docker_volume_usage_percent.pct_change().fillna(0).abs(),
    ranked_change=lambda x: x.actual_change.rank()
)

change_df.sort_values('ranked_change', ascending=False).head()
```

## Binning Data

When dealing with discrete measurements, it can be useful to categorise
the data. For instance, if you have a series of disk usage measurements, 
ranging from 73% to 99% you could categorise the measurements into 3 bins 
spread equally between the min and max values and label them 
'normal', 'high', 'critical': 

```python

df['volume_usage_level'] = pd.cut(
    df.docker_volume_usage_percent, 
    bins=3, labels=['normal', 'high', 'critical']
)
```

Now you can search the DataFrame by `volume_usage_level` or run
`df.volume_usage_level.value_counts()` to see how often the volume
is at 'normal', 'high' or 'critical' levels.

`qcut` can categorise your data into bins containing 
equal number of measurements.
