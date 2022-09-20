# References
For the Flask application we're going to be following <a href="https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world" target="#">Miguel Grinbergs Tutorial</a>
For the deployment to Azure App Services we'll be using the <a href="https://docs.microsoft.com/en-us/azure/app-service/tutorial-python-postgresql-app" target="#">Microsoft Sample</a>. 
For connecting Flask to Postgres we're using Abdel Dyouri's post on <a href="https://www.digitalocean.com/community/tutorials/how-to-use-a-postgresql-database-in-a-flask-application" target="#">Digital Ocean</a>
<a href="http://jinja.pocoo.org/" target="#">Jinja2</a>

# Setup Flask App
We're going to build a Flask Application with a Postgres backend. We'll be using Docker to build 
the various components of the application. So we'll start with a project directory:
```
mkdir flask_pg_app_service
cd flask_pg_app_service
```

Next we'll want a directory for our Flask application:
```
mkdir ear
```

To run our Flask application we will use a custom docker image based on Python 3.8.
Add a Dockerfile to the Flask application directory <code>ear/Dockerfile</code>:
```
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
```
We can also add a requirements file <code>ear/requirements.txt</code>
```
flask
flask-wtf
psycopg2-binary
flask-sqlalchemy
flask-migrate
```

We can now test this development environment by writing a simple Flask "Hello World!" 
application

Create the <i>app</i> package:
Then add the package code in <code>ear/app/__init__.py</code>
```
from flask import Flask

app=Flask(__name__)

from app import routes
```
Next create the <i>routes</i> module in <code>ear/app/routes.py</code>
```
from app import app

@app.route('/')
@app.route('/index')
def index():
  return "Hello World!"
```
Then add the Flask <i>application</i> file <code>ear/ear.py</code>
```
from app import app
```

With both the Flask app written and a Docker container defined, let's build and run the container:
```
ear$ docker image build --build-arg user_id=$(id -u) -t ear:0.1 .
ear$ docker container run --rm -p 5000:5000 --name ear_we_go ear:0.1
```

# Add a PostgreSQL Database
We'll use another container to store our Database. We'll run this container based on a 
standard Postgres image. We'll use a Docker Compose file to simplify run both containers. Create
a Compose file in the project's root directory <code>flask_pg_app_service/docker-compose.yml</code>
```
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
```
We can use the psycopg2 Python module to test that our Flask app can connect to the database, 
by updating the <code>ear/app/routes.py</code> file:
```
import psycopg2
from flask import render_template, flash, redirect, url_for
from app import app
from app.forms import LoginForm

def get_db_connection():
  conn = psycopg2.connect(
    host='db',
    database='example',
    user='example',
    password='example')
  return conn

@app.route('/')
@app.route('/index')
def index():
  conn = get_db_connection()
  cur = conn.cursor()
  cur.execute('SELECT version();')
  pg_version = cur.fetchone()
  res = ''.join(pg_version)
  servername = res[:14]
  cur.close()
  conn.close()
  return render_template('index.html', title = 'Home Page', res=servername)
```
Here we're using a simple query ('SELECT version();') to pull some data back from the 
postgres server and display this via a html template file <code>ear/app/templates/index.html</code>:
```
{% extends "base.html" %}

{% block content %}
    <h1>Hello, {{ res }}</h1>
{% endblock %}
```
The template uses the Jinja2 templating system to display variables sent in the call to 
render_template. The template itself uses another template to insert it's content into another
re-usable template that defines the structure for our HTML output 
<code>ear/app/templates/base.html</code>:
```
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
      <a href="{{ url_for('login') }}">Login</a>
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
```
We've added some additional functionality to our app to use the 'url_for' function and flash 
messaging.
We can use SQLAlchemy to add ORM functionality to the database. First we add our models 
<code>ear/app/models.py</code>:
```
from datetime import datetime
from app import db

class User(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  username = db.Column(db.String(64), index=True, unique=True)
  email = db.Column(db.String(120), index=True, unique=True)
  password_hash = db.Column(db.String(128))
  posts = db.relationship('Post', backref='author', lazy='dynamic')

  def __repr__(self):
    return '<User {}>'.format(self.username)

class Post(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  body = db.Column(db.String(149))
  timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
  user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

  def __repre__(self):
    return '<Post {}>'.format(self.body)
```
The db object also needs to be added the app file <code>ear/app/__init__.py</code>:
```
from flask import Flask
from config import Config
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app=Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```
We've also started to add configuration for the app to a config module 
<code>ear/config.py</code>:
```
import os

class Config(object):
  SQLALCHEMY_DATABASE_URI = 'postgresql+psycopg2://example:example@db/example'
  SQLALCHEMY_TRACK_MODIFICATIONS = False
  SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

# Setup Flask Shell
With the database setup, we can run interactive queries on the database using the Python shell:
```
python3
>>> from app import db
>>> from app.models import User, Post
>>> users = User.query.all()
>>> users
```
To save having to import the app, db and other objects, you can use the Flask shell after 
defining a shell context configuration <code>ear/ear.py</code>:
```
from app import app, db
from app.models import User, Post

@app.shell_context_processor
def make_shell_context():
  return {'db': db, 'User': User, 'Post': Post}
```
Now you can run 'flask shell' to open a interactive Python session with app, db, User and Post
already defined. With the 'flask shell' you can run DDL interactively:
```
>>> u1 = User(username='Mungo', email='mungo@example.com')
>>> db.session.add(u1)
>>> db.session.commit()
```

# Migration to Azure
We're now ready to migrate the application, consisting of a front-end web app and backend database, 
to Azure. We can start with the Flask application.  
