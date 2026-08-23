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

## Migrate an Existing venv project to uv

First generate a `requirements.txt` from the existing project: 

```bash
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip freeze > requirements.txt
deactivate
```

Initialise the project with uv and re-create the virtual environment: 

```bash
uv init --bare
rm -rf .venv
uv venv
```

> [!TIP]
> The `uv init` command by default creates an **application** project 
> layout, with a `src` directory. Use the `--bare` option to just add
> a `pyproject.toml` file to your project. The `--no-package` option 
> can also be used to create a flat project structure with both a 
> `pyproject.toml` and a `main.py`. 

Add your dependancies to to uv: 

```bash
uv add -r requirements.txt
```

Once complete, uv will manage the project dependancies using `pyproject.toml`
and `uv.lock`, so the original `requirements.txt` can be safely deleted. 

## Run Jupyter with uv

From [How to run a Jupyter Notebook with uv](https://pydevtools.com/handbook/how-to/jupyter-notebook-with-uv/)

Setup the environment: 

```bash
# Create a new project directory
mkdir jupyter-project
cd jupyter-project

# Initialize a project
uv init --bare

# Add Jupyter and other dependencies
uv add --dev jupyter matplotlib pandas numpy
```

Launch Jupyter: 

```bash
uv run jupyter lab
```

[Continue Here](https://pydevtools.com/handbook/explanation/uv-complete-guide/#creating-and-managing-projects)
