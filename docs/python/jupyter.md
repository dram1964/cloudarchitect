# Jupyter

The IPython shells in Jupyter Notebooks allow us to run 
shell commands by prefixing them with an exclamation
mark. Typically you can use this to run `pip install` 
or `conda install` directly from a notebook. But you can
also use this to capture shell output to a python variable: 

```python
files = !ls -lh data_dir
```

We can also use this to see the number of columns in a CSV file: 

```python
!awk -F',' '{print NF; exit}' data_dir/text_data.csv
```
