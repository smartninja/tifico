# SmartNinja NoSQL

## About

SmartNinja NoSQL is a simple ODM tool which helps you **switch** between these NoSQL database systems: **TinyDB**, **Datastore**, **Firestore**, **MongoDB** and **Cosmos DB** (via MongoDB API).

TinyDB is used for localhost development. The advantage is that it saves you time configuring a Firestore, Datastore, MonogDB or Cosmos emulator on localhost.

When you deploy your web app to Google App Engine, Heroku or Azure App Service, the ODM figures out the new environment (through env variables) and switches the database accordingly.

Bear in mind that this is a simple ODM meant to be used at the [SmartNinja courses](https://www.smartninja.org) for learning purposes. So not all
features of these NoSQL databases are covered, only the basic ones.

## Installation

Add this dependency in your `requirements.txt`:

	smartninja-nosql

Make sure to install it locally using this command:

	pip install -r requirements.txt

## Other dependencies

SmartNinja NoSQL has two mandatory dependencies: `tinydb` and `tinydb_serialization`. These two help SmartNinja NoSQL use a TinyDB database for localhost development.

#### Datastore

Google Cloud Datastore dependency in `requirements.txt`:

    google-cloud-datastore

#### Firestore

Google Cloud Firestore dependency in `requirements.txt`:

    google-cloud-firestore

#### MongoDB & Cosmos DB

To use MongoDB on Heroku or Cosmos DB on Azure App Service, you'll need to add the following library in your `requirements.txt` file:

    pymongo

### Heroku

If you'll use MongoDB on Heroku, make sure to choose the **mLab MongoDB** add-on.

### Environment variables

The environment variables should already be automatically created, but still make sure they (still) have the correct names.

Heroku:

- **MONGODB_URI** (shows up when you add the mLab MongoDB add-on)
- **DYNO** (standard Heroku env var, not visible on the dashboard)

Azure:

- **APPSETTING_WEBSITE_SITE_NAME** (env vars that start with "APPSETTING_" are Azure's standard env vars. Not visible on the dashboard)
- **APPSETTING_MONGOURL** (shows up when you enable Cosmos DB with the Mongo API)

Google Cloud:

- **GAE_APPLICATION** (standard GAE env var)

#### Important: Datastore environment variable

If you'd like to use Datastore on GAE, you must add this piece of code into your `app.yaml` file:

	env_variables:
	  GAE_DATABASE: "datastore"

If you'd like to use Firestore instead, enter "firestore" or don't have this env. variable at all (Firestore is default).

## Usage

### Creating classes

This is the simplest way to create classes that use SmartNinja NoSQL:

```python3
from smartninja_nosql.odm import Model


class User(Model):
    pass

```

When you initialize a new object, you can add as many attributes as you want (no need to define them in the User model):

```python3
user = User(first_name="Matt", last_name="Ramuta", age=31, human=True)

```

But usually you'd want to specify at least some mandatory fields in a class:

```python3
class User(Model):
    def __init__(self, first_name, last_name, age, human=True, **kwargs):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age
        self.human = human
        self.created = datetime.datetime.now()

        super().__init__(**kwargs)

```

As you can see, `first_name`, `last_name` and `age` are mandatory fields, while `human` is optional and has a default value of `True`.

The `created` field is automatically assigned a value of `datetime.datetime.now()`.

**Important**: The `super().__init__(**kwargs)` line must be the last one in the `__init__` method! Also `**kwargs` must be added as a function parameter.

### Custom class methods

Your classes will inherit the following methods from the Model class:

- `create()`
- `edit()`
- `get_collection()`
- `delete()`
- `get()`
- `fetch()`
- `fetch_one()`

But you can of course create your own custom methods. Example:

```python3
class User(Model):
    def __init__(self, first_name, last_name, age, human=True, **kwargs):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age
        self.human = human
        self.created = datetime.datetime.now()

        super().__init__(**kwargs)
    
    def get_full_name(self):
        return "{0} {1}".format(self.first_name, self.last_name)

```

### Creating new objects

```python3
user = User(first_name="Matt", last_name="Ramuta", age=31)
user.create()

print(user.id)

```

As you can see, creating an object needs two things: initializing an object and saving it into a database with the `create()` method. If you don't call this method, the object will not be saved into a database.

The `create()` method returns back the **object ID**, which is **automatically** created by the database. The ID is also stored in the object itself.

### Get one object from the database

You can get an object out of the database if you know its `id`:

```python3
# Add new object to the database:
user = User(first_name="Matt", last_name="Ramuta", age=31)
user_id = user.create()

# Get the User object from the database
new_obj = User.get(obj_id=user_id)

```

You can also find an object based on some other field using the `fetch_one()` method:

```python3

# search on one field
one_user = User.fetch_one(query=["first_name", "==", "Matt"])

# search on many fields
query_age = ["age", ">", 30]
query_human = ["human", "==", True]

one_user = User.fetch_one(query_age=query_age, query_human=query_human)
```

### Edit an object

You need to pass the object id and the fields you want to edit:

```python3
User.edit(obj_id=user_id, first_name="John", age=25)

```

### Delete an object

Call the `delete()` method and pass the object id:

```python3
User.delete(obj_id=user_id)

```

### Query the database and fetch objects that match the query

You can specify **one query**:

```python3
query = ["first_name", "==", "Matt"]
users = User.fetch(query=query)

print(users[0].first_name)
print(users[0].id)

```

What you'll get is a list of objects that match the query.

You can also have **multiple queries**:

```python3
query_age = ["age", ">", 30]
query_human = ["human", "==", True]

users = User.fetch(query_age=query_age, query_human=query_human)

```

But be aware, that some databases (like Firestore) might require that you create an index for these composite queries.

**Important:**

- This query structure must be followed (as shown in examples): `["field", "operator", value]`. So the **field name** and 
the **operator** must be written in quotes.
- The "not equal" queries ("!=") are not allowed, because Firestore does not support them (although TinyDB and Cosmos 
DB do).

## Enable a TinyDB test database (for localhost testing)

Create an environment variable named `TESTING`:

	import os
	os.environ["TESTING"] = "1"

This will create a test TinyDB database: `test_db.json`.

## How the right database is determined

SmartNinja NoSQL automatically determines which database to use. If the environment has the `GAE_APPLICATION` variable, then 
the selected database is **Firestore** (unless you added the `GAE_DATABASE: "datastore"` environment variable in `app.yaml`). 

If SmartNinja NoSQL finds a `APPSETTING_WEBSITE_SITE_NAME` it assumes the environment is **Azure**, so the selected database is **Cosmos DB**. But if **none** of these two environment variables is found, the 
selected database is **TinyDB**.

## TODO

- Tests
- Continuous integration
- "order" support
- automatic composite index generator for Cloud Datastore (index.yaml)
