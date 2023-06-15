[![simple](https://img.shields.io/badge/build-Simple%20Authentication-blue)](README.md)
[![custom](https://img.shields.io/badge/build-Custom%20Authentication-blue)](README_custom.md)

above mentioned button can be used to switch between Simple and Custom authentication.
# Simple authentication and adding your own custom fields in SignUp page

Here, I had written short description about custom fields with steps by steps code.If you only want to see then click [here](https://github.com/QuaackQuaack/Authentication-with-django/tree/main/custom-authentication)

## Quick note

### Virtual Environment and Installing Django	
In real world, different user have different version of package install in their machine. Python and Django are installed globally in our PC so, it will be pain to install and again reinstall everytime. In this case, Virtual Environment is our ally. It will help us to create a environment where we can avoid installing package globally.

```python
python -m venv <.virtual_environment_name> 
```

let us keep virtual environment name as  ***.venv***, then to activate virtual Environment we need to follow quite different approach for both Unix(Mac) and windows

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
don't forgot to put period(.) at the end of code because this will help us in easy file management of Django. 

#### Creating an app
Django is famous for it's management of multiple apps, for now we will deal with one app i.e accounts. To add new app, 
```python
python manage.py startapp accounts 
```

after that we need to register it.For that go to `django_project/settings.py` in this file, you can see *Installed app* around line 33, there you need to register our app.Now, for CustomUser we also need to mention our model ie ***CustomUser***

```python
INSTALLED_APPS = [
    .......,
    'accounts.apps.AccountsConfig',
    ]
    .....
    .....
AUTH_USER_MODEL = 'accounts.CustomUser'
```

# Basic Authenticate App
Now, tight your seat belts our journey is about to start. One of the easiest and clear way to implement **Custom authentication** in django is by using default forms like **UserCreationForm and UserChangeForm**. Without any delay let's start to write code.

At first, let us set our path to **templates** and manage our **Redirect_URl**, go to `django_project/settings` and here 
```python
TEMPLATES = [
    {
        ...
        "DIRS": [BASE_DIR / "templates"],
        ...
    }
]
```
Here, we are providing path for our templates which we will use later on in our program. Now to redirect us to **home** after login and logout, we need to tell django where to send user. Staying in same file i.e `django_project/setting` at EOF, write 

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


for now, make one directory called `templates` where we will setup our code for templates.

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
Here, we encounter different new words. First of all, at the top of line **extends** help us to extend wrapper of **base.html** to other templates and we used **if** to distinguish whether the user is authenticated or not, with this we will provide user login, logout features. Now, moving towards next templates, we will write out code for signup, login and logout.

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
This is basic login templates. Here, we used form tag and also, you may have noticed **csrf_token** which is quite important to make our login more secure and prevent from exploitation of our website.We used **as_p** to represent our form, but there are many other way too like, **as_ul**, **as_table**.Now, Logout and SignUp also follows same format.

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

### Managing URL
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

---

### Models
In django, Models plays vital role in database.We need to be abit cautious. Here for we are adding custom field ie. ***age***
```vim 
accounts/models.py
```
---
```python
from django.contrib.auth.models import AbstractUser
from django.db import models 

class CustomUser(AbstractUser):
    age = models.PositiveIntegerField(null = True, blank = True)
```

### Forms 
Now, there are two ways to deal with custom fields at first when we SignUp for the first time and later on editing as a ***superuser***.For these, we are using ***UserCreationForm*** and ***UserChangeForm*** respectively.
Our code will look like this 
``` vim 
accounts/forms.py
```
---
``` python
from django.contrib.auth.forms import UserCreationForm, UserChangeForm
from .models import CustomUser

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm):
        model = CustomUser
        fields = UserCreationForm.Meta.fields + ("age",)

class CustomUserChangeForm(UserChangeForm):
    class Meta:
        model = CustomUser
        fields = UserChangeForm.Meta.fields
```
Here first of all we imported, ***UserCreationForm*** and ***UserChangeForm***, and then model which we made before. After that, we declare two separate class for each form i.e Change and Create. Here, we can notice ***class Meta*** it helps to declare characteristics of class. 

### Admin
It is one of the important pretty messy, but don't worry we will go through it :)
After making our CustomUser we need to update our ***admin*** panel where we will work with ***list_display***, which are need to list in our interface, ***fieldsets*** fields that are used in editing users and atlast ***add_fieldsets*** that are used when creating a user or AKA ***for SignUp*** page. Our code will look like this 
```vim 
accounts/admin.py 
```
---
``` python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin

from .forms import CustomUserCreationForm, CustomUserChangeForm
from .models import CustomUser

class CustomUserAdmin(UserAdmin):
    add_form = CustomUserCreationForm
    form = CustomUserChangeForm
    model = CustomUser
    list_display = [
        "email",
        "username",
        "age",
        "is_staff",
        ]
    fieldsets = UserAdmin.fieldsets + ((None, {"fields": ("age",)}),)
    add_fieldsets = UserAdmin.add_fieldsets + ((None, {"fields": ("age",)}),)

admin.site.register(CustomUser, CustomUserAdmin)
```
### Migrations
Migrations play important role in django. Here I will say they will play important role in Database.But if you want to get into it in more details then follow this link [here](https://stackoverflow.com/a/45784615)

```python
python manage.py makemigrations
python manage.py migrates 
```
Note: one of the important habit is to write name of package, after ***makemigrations*** for example 
``` 
python manage.py makemigrations accounts 
``` 
here We are only dealing with one app so , we don't need to worry about it.

Now, We are ready to go, type 
```python 
python manage.py runserver 
```
and go to 
```
localhost:8000
``` 
to see our page.

Meow~~
