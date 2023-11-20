# SQL, Models, and Migrations

## SQL and Data

SQLite Types
- TEXT
- NUMERIC (date, time, etc)
- INTEGER
- REAL
- BLOB (binary data(files, audio, etc))

Other database languages have more types!

### CREATE TABLE

Creating a table:
```SQL
CREATE TABLE flights (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    origin TEXT NOT NULL,
    desitnation TEXT NOT NULL,
    duration INTEGER NOT NULL
);
```
- table name: flights
- id: easy way to uniquely identify the row
- the rest are attributes of our row in our database.

Inserting data into our table:
```sql
INSERT INTO flights
    (origin, destination, duration)
    VALUES ("New York", "London", 415);
```

### SELECT

Retrieve data from a table:
```sql
SELECT * FROM flights;
```
This query (*) will select the entire database.

```sql
SELECT origin, destination FROM flights;
```
This query selects all rows but limits the columns to origin and destination.

```
SELECT * FROM flights WHERE id = 3;
```
This will only select one row with id=3 but display all columns. It will only be one row since all id's are unique with how we created the table.

```sql
SELECT * FROM flights WHERE origin = "New York";
```
This will display all columns but restrict the rows to only those will an origin of New York.

```sql
SELECT * FROM flights WHERE duration > 500 OR destination = "Paris";
```
Additional contraints we can use for WHERE:
- boolean (OR, AND)
- arithemtic (<, >, =)
- ___ IN (element1, element2, ...)
- LIKE ___ (using regular expressions.)

### Update

Updating data once inside database:
```sql
UPDATE flights
    SET duration = 430
    WHERE origin = "New York"
    AND destination = "London";
```

other clauses
- LIMIT
- ORDER BY
- GROUP BY
- HAVING
- much more...

### JOIN
Question: How can we relate tables to eachother?<br>
Example: We have a table "people" and "flights" and we want to relate them together.<br>
Solution: Create a table 
"passengers" to relate these two tables together:

| people_id | flight_id | 
|:-------------|:--------------|
| Integer         | Integer         | 

```sql
SELECT first, origin, destination
FROM flights JOIN passengers
ON passengers.flight_id = flights.id
```
This query will tell us which people are traveling from what origin to what destination.

### Security and SQL
Suppose we have user and password for accounts in our database. We always want to make sure we are looking out for ways the database can be attacked. For example -- means comment in SQL. So in the username "hacker --" will bypass the password check if we are not careful. We can fix this by making sure we treat -- as literal when entered into our database.<br>
Another solution is to use abstraction with a secure framework that will handle these issues for us like Django!

## Django Models
Recall to make a new project:
```
$ django-admin startproject airline
```

Create an app:
```
$ python manage.py startapp flights
```

In settings.py add app as INSTALLED_APP. <br>
In urls.py import include and add path
```python
path("flights/", include("flights.urls"))
```
Create urls.py in our new app as it does not exist by default:
```python
from django.urls import path

from . import views

urlpatterns = [

]
```

### models.py
Creating a table in Django!
```python
class Flight(models.Model):
    origin = models.CharField(max_length=64)
    destination = models.CharField(max_length=64)
    duration = models.IntegerField()
```

We can modify our model to be more descripitive when running queries:
```python
class Flight(models.Model):
    origin = models.CharField(max_length=64)
    destination = models.CharField(max_length=64)
    duration = models.IntegerField()

    def __str__(self):
        return f"{self.id}: {self.origin} to {self.destination}"
```

Adding a new model Airport and having Flight relate to it.
```python
class Airport(models.Model):
    code = models.CharField(max_length=3)
    city = models.CharField(max_length=64)

        def __str__(self):
            return f"{self.city} ({self.code})"

class Flight(models.Model):
    origin = models.ForeignKey(Airport, on_delete=models.CASCADE, related_name="departures")
    destination = models.ForeignKey(Airport, on_delete=models.CASCADE, related_name="arrivals")
    duration = models.IntegerField()

    def __str__(self):
        return f"{self.id}: {self.origin} to {self.destination}"
```

### Migrations
Updates the changes made in code to the database.<br>
To create the new models:
```
$ python manage.py makemigrations
```
For our Flight model it will give instructions to Django to create the model in a SQL database.<br>
To apply the migrations to the database:
```
$ python manage.py migrate
```
Notice a flights table will be created and db.sqlite3 (database).

### Shell
We can use a shell to write python commands that will be executed on our application.
```
$ python manage.py shell
```
import our flight model:
```python
from flights.models import Flight
```
Create a flight:
```python
f = Flight(origin="New York", destination="London", duration=415)
f.save()
```
Created a flight from New York to London and inserted into the database.<br>
Query the flight:
```python
Flight.objects.all()
<QuerySet [<Flight>: Flight object (1)]>
<QuerySet [<Flight>: 1: New York to London]>
```
The two QuerySet is just to demonstrate the difference between adding an description and not.<br>
With shell we can access all the properties of a flight.

### Using models with views.py
To use are model one example is:
```python
def index(request):
    return render(request, "flights/index.html", {
        "flights": Flight.objects.all()
    })
```
passes the model to our HTML render.

### admin.py

Can create a super user to manage the database.
```
$ python manage.py createsuperuser
```

In order to edit our models as a admin we need to add them to admin.py
```python
from .models import Flight, Airport

admin.site.register(Airport)
admin.site.register(Flight)
```

### many-to-many relationship
When creating a model we can use many-to-many to identify that the row may have many relationships.
```python
class Passenger(models.Model):
    ...
    flights = models.ManyToManyField(Flight, blank=True, related_name="passengers")

    def __str__(self):
        return f"{self.first} {self.last}"
```

### Users
Lets create an app to represent users!
```
$ python manage.py startapp users
```
```python
#settings.py
INSTALLED_APPS = [
    'users',
    ...
]
```
```python
#urls.py (project)
urlpatterns = [
    path("users/", include("users.urls")),
    ...
]
```
```python
#urls.py (users)
urlpatterns = [
    path("", views.index, name="index"),
    path("login", views.login_view, name="login"),
    path("logout", views.logout_view, name="logout")
]
```
```python
# views.py
def index(request):
    if not request.user.is_authenticated:
        return HttpResponseRedirect(reverse("login"))

def login_request(request):
    if request.method == "POST":
        username = request.POST["username"]
        password = request.POST["password"]
        # authentic
        user = authentic(request, username=username, password=password)
        if user is not None:
            login(request, user)
            return HttpResponseRedirect(reverse("index"))
        else:
            return render(request, "users/login.html", {
                "message": "Invalid credentials."
            })
    return render(request, "users/login.html")
```