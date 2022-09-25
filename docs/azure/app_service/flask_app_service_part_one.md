# Deploy a Flask App to Azure: Part 1

##  Intro

In this series of articles, we're going to build a Flask App connected to a backend PostgreSQL 
database. We'll develop the application and database locally using Docker, 
then we'll install a production version on Azure using Azure App Service and 
Azure Database for PostgreSQL

On this page we'll focus on developing the Flask application locally. 

## References

For the Flask application we're going to be following <a href="https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world" target="#">Miguel Grinbergs Tutorial</a>
For the deployment to Azure App Services we'll be using the <a href="https://docs.microsoft.com/en-us/azure/app-service/tutorial-python-postgresql-app" target="#">Microsoft Sample</a>. 
For connecting Flask to Postgres we're using Abdel Dyouri's post on <a href="https://www.digitalocean.com/community/tutorials/how-to-use-a-postgresql-database-in-a-flask-application" target="#">Digital Ocean</a>
<a href="http://jinja.pocoo.org/" target="#">Jinja2</a>

## Setup Local Development Environment

For our local development in Docker, we'll be using to containers:

1. A customised Python image to run the Flask Application and
2. A standard PostgreSQL image

Begin by creating a project directory *(flask_pg_app_service)* and create a subfolder to hold the 
application *(ear)*

    mkdir flask_pg_app_service
    cd flask_pg_app_service
    mkdir ear

To run our Flask application we will use a custom docker image based on Python 3.8.
Add a Dockerfile to the Flask application directory `flask_pg_app_service/ear/Dockerfile`:

    FROM python:3.8-slim-buster

    ARG user_id=1000
    ARG user_name=ear

    EXPOSE 5000

    RUN groupadd -g $user_id $user_name && \
    useradd -u $user_id -g $user_id -d /home/$user_name -m $user_name

    WORKDIR /opt

    COPY requirements.txt requirements.txt

    RUN pip install -r requirements.txt

    COPY . .

    USER $user_id

    ENV FLASK_APP=/opt/ear.py

    ENV FLASK_DEBUG=1

    CMD ["flask", "run", "--host=0.0.0.0"]

We'll build and run this image using Docker Compose. The docker-compose.yml file will be stored 
in the root of the project directory `flask_pg_app_service/docker-compose.yml`

    version: '3.8'

    services:
      app:
        build: 
          context: ./ear
          args: 
            user_id: 1000
        image: ear:1.0
        ports: 
          - 5000:5000
        volumes:
          - ./ear:/opt
        depends_on: [db]
      db:
        image: postgres:9.0
        restart: always
        environment:
          POSTGRES_PASSWORD: example
          POSTGRES_USER: example
        volumes:
          - pgdata:/var/lib/postgresql/data

    volumes:
      pgdata:

Before we can build and run the containers, we'll need to add some code for the Flask App.

## Setup a Minimal Flask Application

We can start with the requirements file for the application `flask_pg_app_service/ear/requirements.txt`

    flask
    flask-wtf
    psycopg2-binary
    flask-sqlalchemy
    flask-migrate


Then add the package code in `flask_pg_app_service/ear/app/__init__.py`

    from flask import Flask

    app=Flask(__name__)

    from app import routes

Next create the <i>routes</i> module in `flask_pg_app_service/ear/app/routes.py`

    from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        return "Hello World!"

Then add the Flask <i>application</i> file `flask_pg_app_service/ear/ear.py`

    from app import app


With both the Flask app written and a Docker container defined, let's build and run the container:

    flask_pb_app_service/ear$ docker image build --build-arg user_id=$(id -u) -t ear:0.1 .
    flask_pg_app_service/ear$ docker container run --rm -p 5000:5000 --name ear_we_go ear:0.1

We can test that this works by visiting <http://localhost:5000>. If all is working well, we can stop 
this container and remove the image, since we'll build the image from the docker-compose file later. 

## Connect the Flask App to the PostgreSQL Database

We can now expand our application to use a backend database. The application will start simple, and
will display a list of books. The book list will be stored in a single table on our PostgreSQL 
database. We'll be using SQLAlchemy to connect to the database. Lets add some configuration 
data in `flask_pg_app_service/ear/config.py

    class Config(object):
        SQLALCHEMY_DATABASE_URI = 'postgresql+psycopg2://example:example@db/example'
        SQLALCHEMY_TRACK_MODIFICATIONS = False


Now we can use this configuration in our app module to define a *db* and *migrate* objects for use
by the application `flask_pg_app_service/ear/app/__init__.py`


    from flask import Flask
    from config import Config
    from flask_sqlalchemy import SQLAlchemy
    from flask_migrate import Migrate

    app=Flask(__name__)
    app.config.from_object(Config)
    db = SQLAlchemy(app)
    migrate = Migrate(app, db)

    from app import routes, models

Now we add the Models to the app with a single table to hold the book list `flask_pg_app_service/ear/app/models.py`

    from datetime import datetime
    from app import db

    class Book(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        title = db.Column(db.String(120), index=True, unique=True)
        author = db.Column(db.String(120), index=True)
        status = db.Column(db.String(30))

        def __repr__(self):
        return '{} by {}'.format(self.title, self.author)

We want our index page to display the list of books, so we need to update the Routes module: `flask_pg_app_service/ear/app/routes.py`

    from flask import render_template, flash, redirect, url_for
    from app import app, db
    from app.models import Book

    @app.route('/')
    @app.route('/index')
    def index():
        books = Book.query.all()
        return render_template('index.html', title = 'Home Page', books=books)

We're using `render_template` to display handle our views, so we need to add a template for the 
index page: `flask_pg_app_service/ear/app/templates/index.html`


    {% extends "base.html" %}

    {% block content %} 
        <h1>Reading List</h1>
        {% for book in books %}
        <div><p>{{ book.title }} by {{ book.author }} ( {{ book.status }} )</b></p></div>
        {% endfor %}
    {% endblock %}

We're using a base template to hold the overall layout for pages in our site, so we need to create
this also `flask_pg_app_service/ear/app/templates/base.html`

    <!doctype html>
    <html>
    <head>
        {% if title %}
        <title>{{ title }} - EAR</title>
        {% else %}
        <title>Welcome to EAR</title>
        {% endif %}
    </head>
    <body>
        <div>Ear: 
        <a href="{{ url_for('index') }}">Home</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
        {% for message in messages %}
        <li>{{ message }}</li>
        {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
    </html>

## Create the Database Tables

We now have all the code in place to run our Flask application but the books table does not exist 
yet. We can use the app container to create the table. First we need to build the app container:

    flask_pg_app_service/ear$ docker compose build --build-arg user_id=$(id -u) app

Once built we can start both the database and the Flask app with

    flask_pg_app_service/ear$ docker compose up

If we try to connect to the index page on the application, we will receive an error that the books 
table does not exist. We can use flask-migrate to create the books table. First, connect to the 
app container:

    docker compose exec app bash

From here we can run the flask-migrate commands. First time around you will run:

    flask db init

This will setup the repository to hold the database migrations. Once complete you can 
create the first migration using: 

    flask db migrate -m "Add book table to database"

This will create python scripts to install the Model to the database. To install the Model
you will need to run: 

    flask db upgrade

Now with the book table created, we can connect to the index page <http://localhost:5000/>, which 
will diplay the page header ('Reading List'), but there will be no books displayed. We need to add
some data into the books table. 

## Add Data to the Books Table

You can manually connect to the db container and add some rows to the book table using the psql 
command. Alternatively, you can use Python from the app container to do the same. If you use 
Python, you will need to import the db and book objects. An alternative method is to use 
the Flask shell. To use Flask shell you will need to setup a shell context configuration in 
`ear/ear.py`:
```
    from app import app, db
    from app.models import Book

    @app.shell_context_processor
    def make_shell_context():
        return {'db': db,  'Book': Book}
```
Now you can run `flask shell` on the app container to open a interactive Python session 
with app, db, and Book already defined. With the 'flask shell' you can run DDL interactively:
```
>>> b1 = Book(title='East of Eden', author='John Steinbeck' status='In Progress')
>>> db.session.add(b1)
>>> db.session.commit()
>>> quit()

Once we add some data to the book table we can refresh the index page on the app and the book
should display. 
```
We can also add a page to the app for adding books to the book table. We'll start by adding a route
to the `flask_pg_app_service/ear/app/routes.py`

    from flask import render_template, flash, redirect, url_for
    from app import app, db
    from app.forms import BookForm
    from app.models import Book

    @app.route('/')
    @app.route('/index')
    def index():
        books = Book.query.all()
        return render_template('index.html', title = 'Home Page', books=books)

    @app.route('/add_book', methods=['GET', 'POST'])
    def add_book():
        form = BookForm()
        if form.validate_on_submit():
            book = Book(
                author=form.author.data,
                title=form.title.data, 
                status=form.status.data
            )
            db.session.add(book)
            db.session.commit()
            flash('Your changes have been saved')
            return redirect(url_for('index'))
        return render_template('add_book.html', title='Add Book', form=form)

The new route references a form object and a new template which we'll need to create. First 
there's the BookForm object. We'll create this in the `flask_pg_app_service/ear/app/forms.py:

    from flask_wtf import FlaskForm
    from wtforms import StringField, PasswordField, BooleanField, SubmitField
    from wtforms.validators import DataRequired

    class BookForm(FlaskForm):
        author = StringField('Author', validators=[DataRequired()])
        title = StringField('Title', validators=[DataRequired()])
        status = StringField('Status', validators=[DataRequired()])
        submit = SubmitField('Add Book')

Now we can create the page to display this form `flask_pg_app_service/ear/app/templates/add_book.html`


    {% extends "base.html" %}

    {% block content %}
        <h1>Add New Book</h1>
        <form action="" method="Post" novalidate>
            {{ form.hidden_tag() }}
            <p>
                {{ form.author.label }}<br>
                {{ form.author(size=120) }}
            </p>
            <p>
                {{ form.title.label }}<br>
                {{ form.title(size=120) }}
            </p>
            <p>
                {{ form.status.label }}<br>
                {{ form.status(size=30) }}
            </p>
            <p>{{ form.submit() }}</p>
        <form>
    {% endblock %}

The `form.hidden_tag()` in the page, allows us to use CSRF, but we need to add a SECRET_KEY to the 
application configuration `flask_pg_app_service/ear/config.py`

    import os

    class Config(object):
        SQLALCHEMY_DATABASE_URI = 'postgresql+psycopg2://example:example@db/example'
        SQLALCHEMY_TRACK_MODIFICATIONS = False
        SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'

Now we can add books by navigating to <http://localhost:5000/add_book>. To make life easier, we
can add this link to either the navigation or to the index page. Let's add it to the index
page `flask_pg_app_service/ear/app/templates/index.html`


    {% extends "base.html" %}

    {% block content %} 
        <h1>Reading List</h1>
        {% for book in books %}
        <div><p>{{ book.title }} by {{ book.author }} ( {{ book.status }} )</b></p></div>
        {% endfor %}
        <p><a href="{{ url_for('add_book') }}">Add a New Book</a></p>
    {% endblock %}

## Summary

We now have a working Flask application that displays data from a database. The application still
has a lot of features missing:

1. Authentication and Authorisation have not been implemented
2. Additional CRUD operations are missing
3. Data can be added without validation
4. Site does not yet run with SSL

In a production environment, you would want to fix these issues. We can do this as we develop the 
app further, but we have enough code now to try to deploy the app to Azure. 
