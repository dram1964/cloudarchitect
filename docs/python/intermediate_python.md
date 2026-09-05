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

If the lists have different lengths, `zip` will stop at the shortest list:
this is known as **short circuiting**. You can use `itertools` to handle
lists of unequal lengths (missing values are filled with `None`:

```python
import itertools

for element in itertools.zip_longest(authors, books, years):
    print(f"{element[0]} wrote {element[1]} in {element[2]}")
```

## Gradio

Gradio is a useful data science tool to generate Web UIs from a 
Jupyter Notebook. Gradio expects a function that takes inputs and
produces outputs. To create a Web UI try: 

```python
import gradio as gr

def greeting(name):
    return f"Hello, {name}"

gr.Interface(
    fn=greeting, 
    inputs="textbox", 
    outputs="textbox", 
    flagging_mode="never"
).launch(inbrowser=True)
```

Gradio also comes with a `ChatInterface` which expects a `chat function`. The
chat function should take two inputs: message and history. The message is the
current prompt, and history is a list of openai-style dictionaries with 
'role' and 'content' keys. This makes it easy to create a UI to query an LLM
and retain chat history:

```python
import os
from dotenv import load_dotenv
from openai import OpenAI
import gradio as gr

MODEL_LLAMA = 'llama3.2'
OLLAMA_BASE_URL = "http://localhost:11434/v1"
ollama = OpenAI(base_url=OLLAMA_BASE_URL, api_key='ollama')

system_message = "You are a helpful assistant"

# chat function
def chat(message, history):
    history = [{"role":h["role"], "content":h["content"]} for h in history]
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]
    stream = ollama.chat.completions.create(model=MODEL_LLAMA, messages=messages, stream=True)
    response = ""
    for chunk in stream:
        response += chunk.choices[0].delta.content or ''
        yield response

# Gradio ChatInterface
gr.ChatInterface(fn=chat).launch(inbrowser=True)
```

See [Creating a Chatbot Fast](https://gradio.app/guides/creating-a-chatbot-fast) for additional options. 
