# Django Notes
## HTTP

- Hyper Text Transfer Protocol.
- Django writes to server where clients make requests.
- Server processes request and returns a response
- Key takeaway - users make requests and the server makes responses.

HTTP Status Codes
| Status code    | Description |
| --------: | :------- |
| 200  | ok    |
| 301 | Moved      |

## Installation
```bash
$ pip3 install Django
```
```bash
$ django-admin startproject PROJECT_NAME
```
## Modules

### Main Project Directory

#### manage.py
- Used to run commands with django. Not generally edited.

Run the web server:
```bash
$ python manage.py runserver
``` 
Create new app:
```bash
$ python manage.py startapp APP_NAME
``` 

#### settings.py

- Important config settings for appications.
- All apps must be installed here.

```python
# Application definition:
INSTALLED_APPS = [
    'APP_NAME', # Installation of new app.
    ....
]
``` 

#### urls.py
- Contains urls for all apps in the project.

```python
# Allowed applications.
urlpatterns = [
    # Leads to admin app.
    path('admin/', admin.site.urls),
    # Leads to created app and includes urls in said app.
    path('APP_NAME/', include("APP_NAME.urls")),
]
``` 

### App Directory

#### views.py
- Handles functions that take clients requests and then returns the servers response.
- Allows for logic to be added to rendered templates making webpages more dynamic.

Upon request from client return a HttpResponse of "Hello, world!".
```python
from django.http import HttpResponse

# Create your views here
def index(request):
    return HttpResponse("Hello, world!");
```

Upon request, render an HTML file.
```python
from django.http import render

# Create your views here
def index(request):
    return render(request, "APP_NAME/index.html")
```

Upon request, render HTML, give context by providing a dictionary.
```python
def greet(request, name):
    return render(request, "APP_NAME/greet.html", {
        # Reference name : Argument
        "name": name.capitalize(),
    })
```

We can recieve different kinds of requests, with django we can specify how to deal with them differently. In this example the function can be called with a "GET" or "POST" request.
```python
def add(request):
    if request.method == "POST":
        # Create a form variable with all data the user submitted.
        form = NewTaskForm(request.POST)
        if form.is_valid():
            form.task = cleaned_data["task"]
            tasks.append(task)
            # Reverse engineer where the url "tasks:index" is lcoated and redirect the client.
            return HttpResponseRedirect(reverse("tasks:index"))
        else:
            return render(request, "task/add.html", {
                # Send old form as opposed to new to display errors to user!
                "form": form 
            })
    
    else:
        # If a "GET" request is sent render a new blank form.
        return render(request, "tasks/add.html", {
            "form": NewTaskForm()
        })
```
Server and client sided validation is important! The client could be operating on a older version of our application!

A big issue in the form with django is that anyone will see the information posted. To fix this we can use sessions.
```python
def index(request):
    # Checking if tasks are in session.
    if "tasks" not in request.session["tasks"]
    return render(request, "tasks/index.html", {
        "tasks" : request.session["tasks"]
    })
```
Note you must run python manage.py migrate or you will get a strange error.

Make sure to add the task to session!
task = form.cleaned_data["task"]

#### urls.py

- Must be created.
- List of all allowable urls for this app.

```python
from django.urls import path

# Allows functions from views to be used.
from . import views

# Allowable urls
urlpatterns = [
    # path(url, function call, reference name)
    path("", views.index, name="index"),
    # Takes a input of string name in the url.
    path("<str:name>", views.greet, name="greet"),
]
```

### Templates

- Best practice to create template folder as templates/APP_NAME
- Contains all HTML content.
- index.html is best practice for default route.

Recall greet defined in views.py above. We can reference the dictionary created in HTML using jinja and display the variable passed in the url.
```HTML
<!-- Takes the key "name" and passes the argument to HTML. -->
<h1>Hello, {{ name }}</h1>
```
We can add logic statements being added to HTML with Django.
```HTML
<body>
    {% if newyear %}
        <h1>YES</h1>
    {% else %}
        <h1>NO</h1>
    {% endif %}
</body>
```

We can create lists in HTML by referencing lists created in our views.py
```HTML
<ul>
    <!-- Loop through all tasks defined in our dictionary in views.py -->
    {% for task in tasks %}
        <!-- Add each element to the list -->
        <li>{{ task }}</li>
    {% endfor %}
</ul>
```
If we want to loop over a list but it is empty we can provide logic for HTML to follow.
```HTML
{% for task in tasks %}
    <li>{{ task }}</li>
{% empty %}
    <li>No Task</li>
{% endfor %}
```

It is bad practice to simply use a link to the exact folder navigate to other pages as Django simplifies the process of switching urls, meaning in future versions of our code the link may break.<br>
Instead do this.
```HTML
<!-- uses urls.py refernce name to know where to link -->
<a href="{% url 'add' %}">text<a>
```
A error that might arise is name space collision as for example there many be in different apps a name="index" in urls.py. We can avoid this by adding the following cod eot urls.py and HTML.
```python
app_name = "tasks"
urlpatterns = [
    path("", views.index, name="index"),
]
```
```HTML
<a href="{% url 'tasks:index' %}">text<a>
```

Forms with Django, note that POST request sends data to the application whereas GET generally just gets a address. Also note that forms in DJango need to be CRSF verified (avoids someone forging a request from another website onto our server).
```HTML
<form action="{% url 'task:add' %}" method="POST">
    <!-- Generate unique token -->
    {% csrf_token %}
    <input type="text" name="task">
    <input type="submit">
</form>
```
We can also use python and django directly to create forms in views.py.
```python
from django import forms

class NewTaskForm(forms.Form):
    tasks = forms.CharField(label="New Task")

def add(request):
    return render(request, "tasks/add.html", {
        "form": NewTaskForm()
    })
```
Now we can modify our add.html
```HTML
<form action="{% url 'task:add' %}" method="POST">
    {% csrf_token %}
    {{ form }}
    <input type="submit">
</form>
```
The benefit here is we can modify the Class NewTaskForm to add new inputs without touching the html file. Also, we can add additional arguments like max/min values.

With Django we can use template inheritance to create a layout.html that all other templates can inherit.
```HTML
<!DOCTYPE html>
....
    <body>
        <!-- This is what will be modified in other templates. Note the name of block can be anything.-->
        {% block body %}
        {% endblock %}
    </body>
...
```
To inherit the layout.
```HTML
{% extends "APP_NAME/layout.html" %}
```

### Static

- Best practice to name the folder static/APP_NAME
- Stores CSS.

Load static files, and link to stylesheet.
```HTML
{% load static %}

....
<!-- Tells Django styles.css is located in static -->
<link href="{% static 'newyear/styles.css' %}" rel="stylesheet">
```
