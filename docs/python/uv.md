# UV - Python Package Manager

Notes from the [Python Developer Tooling Handbook](https://pydevtools.com/handbook/explanation/uv-complete-guide/)

uv is an all-in-one tool for Python Development: used to manage python versions, virtual environments and 
dependancies. uv provides considerable performance gains over pip and conda. 

## Install uv

Instructions to [install uv](https://docs.astral.sh/uv/getting-started/installation/#installation-methods)

For the standalone installer method, you can update UV with `uv self update`. 

Run `uv sync` to build your `venv` environment. 

## Manage Python Versions

uv can install and manage multiple versions of Python.
Use `uv python install` to install one or more Python versions: 

```bash
uv python install 3.12 3.13 3.14
```

List installed Python versions with `uv python list`.

uv downloads pre-built versions of Python, and stores these in a shared
cache, available across multiple projects. 

`uv python pin 3.12` can be used to create a `.python-version` file and 
pin the current project to this version of Python. 

## Creating and Managing Projects

Use `uv init` to initialise a new project and add a `pyproject.toml` file:

```bash
uv init my-project
cd my-project

# Add dependancies
uv add pandas numpy
uv add psycopg2 pyodbc pymssql
uv add python-dotenv

# Install dependancies
uv sync

# Run the console script created by uv
uv run my-project
```

The `uv sync` commands creates (or updates) the `.venv` directory and 
a `uv.lock` file. 

The `uv init my-project` creates a `src` directory for application or 
library style projects (using `uv init --lib my-project`). To keep the 
project layout flat use: `uv init --no-package my-project`. 


[Continue Here](https://pydevtools.com/handbook/explanation/uv-complete-guide/#creating-and-managing-projects)
