# Intermediate Python

## Generators

Generators are useful for working with large datasets that might
be too large to fit into memory. Instead of loading a large set
into memory by assigning to an array, you can create an iterable 
object that 'yields' each object one by one. For example: 

```python
books = [
    {"title": "Book One", "author": "Author A"},
    {"title": "Book Two", "author": "Author B"},
    {"title": "Book Three", "author": None},
    {"title": "Book Four", "author": "Author A"},
    {"title": "Book Five"},
]

def get_unique_authors(books):
    yield from {book.get("author") for book in books if book.get("author")}

# Using the generator function
for author in get_unique_authors(books):
    print(author)
```

The generator uses a set comprehension to return unique values from the 
book object. The set is not saved into a variable, but each value is accessed
one by one in the `for` loop.

## Zip Iterator

Zip can be used to iterate over corresponding values from two or more lists 
to avoid indexing issues:

```python
authors = ['Charles Dickens', 'J.D. Salinger', 'Ernest Hemmingway']
books = ['A Tale of Two Cities', 'Catcher in the Rye', 'The Grapes of Wrath']
years = ['1859', '1940', '1938']

for author, book, year in zip(authors, books, years):
    print(f"{author} wrote '{book}' in {year}")
```
