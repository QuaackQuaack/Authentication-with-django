[![simple](https://img.shields.io/badge/build-Simple%20Authentication-blue)](README.md)
[![custom](https://img.shields.io/badge/build-Custom%20Authentication-blue)](README_custom.md)

# Beginner authentication to advance customizing authentication in Django

## In this blog
 here, I will talk about setting up a simple page for authentication and also, account management with customizing authentication the help of django

In ths Blog, we will learn,
1. Simple authenticate page
2. Advanced Authenticate page with custom fields for Users


## Quick note

### Virtual Environment and Installing Django
	In real world, different user have different version of package install in their machine. Python and Django are installed globally on our PC so, it will be pain to install and again reinstall everytime. In this case, Virtaul Environment is our ally. This will help us to create a environment where we can run our project.

```python
python -m venv <.virtual_environment_name> 
```

for a while let us keep virtual environmnet name as  .venv, then to activate vitual Environment we need to follow quite different approach for both Unix(Mac) and windows

For Windows 
```python
> .venv/Scripts/Activate.ps1
(.venv) > python -m pip install django~=4.0.1 

```

For Unix
``` python
$ source .venv/bin/activate 
(.venv)$ python -m pip install django~=4.0.1
```

Starting project 
```python
django-admin startproject django_project . 
```
don't forgot to put period(.) at the end of code because this will help us in file structure management of django. 

#### Creating an app
Django is famous for it's management of mulitple accounts, for now we will only deals with one app i.e accounts. To add new app, 
` python manage.py startapp accounts `

after that we need to register it.For that go to `django_project/settings.py` there you can see *Installed app* around line 33, there you need to register our app.

```python
INSTALLED_APPS = [
    .......,
    'accounts.apps.AccountsConfig',
    ]
```


# Basic Authenticate App
Now, tight your seat belts out journey is about to start. One of the easiest and clear way to implement **Simple authenticate** in django is by using default View like **LogInView**. Without any delay let's start to write code and I am going to give you detail about that particular step.

At first, let us set our directory to **templates** and manage our **Redirect_URl**, go to `django_project/settings` and there 
``` 
TEMPLATES = [
    {
        ...
        "DIRS": [BASE_DIR / "templates"],
        ...
    }
]
```
Here, we are providing path for our templates which we will use lateron in our program. Now to redirect us to **home** after login and logout, we need to tell django where to send user. Remaining in same file i.e `django_project/setting` at EOF, write 
```python
LOGIN_REDIRECT_URL = "home" 
LOGOUT_REDIRECT_URL = "home"
```
so this much for setup. Now we will work towards templates.

### Setting Up Templates
For templates, we will go in order 
1. templates/base.html
2. templates/home.html
3. templates/registration/login.html
4. templates/registration/logout.html
5. templates/registration/singup.html


for now, make one directory called `templates` and in their we will write our all code for templates.

#### Templates/base.html
---
```HTML
<!DOCTYPE html>
<html>
<head>
    <title>Basic page</title>
</head>
<body>
    {% block content %}
    {% endblock content %}
</body>
</html>
```

#### templates/home.html
---
```HTML
{% extends "base.html" %}

{% block content %}
{% if user.is_authenticated %}
    Hi {{ user.username }}!
    <p><a href="{% url 'logout' %}">Log Out</a></p>
{% else %}
    <p>You are not logged in</p>
    <a href="{% url 'login' %}">Log In</a> |
    <a href="{% url 'signup' %}">Sign Up</a>
{% endif %}
{% endblock content %}
```
Here, we encounter different new words. First of all, at the top of line **extends** help us to extend wrapper of **base.html** to other templates. 
and we used **if** , to distinguish whether the user is authenticated or not, with this we will provide user login, logout features. Now, moving towards next templates, we will write out code for signup, login and logout.

make one directory with name **registration**
` mkdir templates/registration `

#### templates/registration/login.html
---
```HTML
{% extends "base.html" %}

{% block content %}
<h2>Log In</h2>
    <form method="post">{% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Log In</button>
</form>
{% endblock content %}
```
This is basic login templates. Here, we used form tag and **csrf_token** is quite important to make our login more secure and prevent from exploitation of our website. and we used **as_p** to represent our form, but there are many other way too like, **as_ul**, **as_table**.Now, Logout and SignUp also follows same format.

#### templates/registration/logout.html
---
```HTML
{% extends 'base.html' %}

{% block content %}
<h2>Log out</h2>
    <form method = 'post'>{% csrf_token %}
    {{ form.as_p }}
    <button type = 'submit'>Logout</button>
</form>
{% endblock content %}
```

#### templates/registration/signup.html
---
```HTML
{% extends 'base.html' %}

{% block content %}
<h2> Sign Up</h2>
    <form method = 'post'>{% csrf_token %}
    {{ form.as_p }}
    <button type = 'submit'> Sign Up</button>
</form>
{% endblock content %}
```

### Managing URl 
For login and logout, we can use default built-in **auth app** but, for SignUp we will use use our own Url and View. To route our URL, we need to go to 
#### django_project/urls.py
``` python
from django.urls import path, include 
from django.views.generic.base import TemplateView

urlpatterns = [
    ....
    path("accounts/", include("accounts.urls")), 
    path("accounts/", include("django.contrib.auth.urls")), 
    path("", TemplateView.as_view(template_name="home.html"), name="home"), 
]
```
here order is important, **accounts.urls** help us to manage our singup page, and **django.contrib.auth.urls** helps us to direct our login, logout page and at last with the help of TemplateView, we are rendering our template directly.
Now, we need to config our 
#### accounts/urls.py
```python
from django.urls import path
from .views import SignUpView

urlpatterns = [
    path("signup/", SignUpView.as_view(), name = 'signup'),
]
```
At the top, we imported path which act as power-house of our urlpatterns and after that, we imported view which is yet to be written. Here we are approaching ***URL-->Views*** approach and also, we are using class views so ***as_view()*** is used.

### Views
We are using built-in auth app for login and logout, so we only need to write view for **SignUp** only. Here we are using UserCreationForm , which has three fields i.e username, password1, password2. Password1 and password2 must be same in order to make account.

``` python 
from django.contrib.auth.forms import UserCreationForm
from django.urls import reverse_lazy
from django.views.generic import CreateView

class SignUpView(CreateView):
    form_class = UserCreationForm
    success_url = reverse_lazy("login")
    template_name = "registration/signup.html"
```

let me explain what we have done here, at first we imported ***UserCreationForm*** and then ***reverse_lazy*** which help us to load our url later on when they are available or needed.We have directed our user to login page upon successful registration.


So, in this way we can set a simple authentication page with easily understandable and less code in django. 

 
