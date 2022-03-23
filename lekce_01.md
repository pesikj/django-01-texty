# Naše aplikace

V rámci kurzu vytvoříme aplikaci označovanou jako CRM, tedy aplikaci, do které jsou ukládány informace související s řízením obchodu a projektů. Základ naší aplikace budou informace o firmách (potenciálních nebo aktivních zákaznících), lidech, obchodních případech, obchodních schůzkách a produktech.

Začneme vytvořením projektu, který můžeme pojmenovat např. python-crm. Na začátku musíme udělat posloupnost rutinních kroků:

- založit projekt ve vývojovém prostředí,
- do virtuálního prostředí nainstalovat Django.

Poté vytvoříme projekt pomocí příkazu

```
django-admin startproject company_manager
```

```
cd company_manager
python manage.py runserver
```

Objeví se hláška

```
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
```

Tu můžeme zatím ignorovat, migraci provedeme spolu s migrací našich modelů.

```
python manage.py startapp crm
```

```
INSTALLED_APPS = [
    "crm.apps.CrmConfig",
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Následně vytvoříme modely, které reprezentují firmy (`Account`), lidi/kontakty z ostatních firem (`Contact`) a obchodní příležitosti (`Opportunity`). U firem a obchodních příležitostí si budeme evidovat stavy, protože ty jsou pro nás důležité (potřebujeme např. vědět, jestli je nějaká firma už náš zákazník či zda jsme ji zatím vůbec nekontaktovali).

```py
from django.db import models
from django.contrib.auth.models import User


class Address(models.Model):
    street = models.CharField(max_length=200, blank=True, null=True)
    zip_code = models.CharField(max_length=10, blank=True, null=True)
    city = models.CharField(max_length=100, blank=True, null=True)


class Account(models.Model):
    status_choices = (
        ("A", "Account"),
        ("L", "Lead"),
        ("O", "Opportunity"),
        ("C", "Active Customer"),
        ("FC", "Former Customer"),
        ("I", "Inactive Account"),
    )
    status = models.CharField(max_length=2, default="A", choices=status_choices)
    phone_number = models.CharField(max_length=20)
    email = models.CharField(max_length=50)
    identification_number = models.CharField(max_length=100)


class Contact(models.Model):
    primary_account = models.ForeignKey(Account, on_delete=models.SET_NULL, null=True)
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    phone_number = models.CharField(max_length=20)
    email = models.CharField(max_length=50)


class Opportunity(models.Model):
    status_choices = (
        ("1", "Prospecting"),
        ("2", "Analysis"),
        ("3", "Proposal"),
        ("4", "Negotiation"),
        ("5", "Closed Won"),
        ("0", "Closed Lost"),
    )

    account = models.ForeignKey(Account, on_delete=models.RESTRICT)
    sales_manager = models.ForeignKey(User, on_delete=models.RESTRICT)
    primary_contact = models.ForeignKey(Contact, on_delete=models.SET_NULL, null=True)
    description = models.TextField(null=True)
    status = models.CharField(max_length=2, default="1", choices=status_choices)
```

V dalším kroku vytoříme uvítací obrazovku (pohled `IndexView`) a pohled na vytvoření firmy, který pojmenujeme `AccountCreateView`.

```py
from django.views.generic import CreateView, TemplateView
import crm.models as models

class IndexView(TemplateView):
    template_name = "index.html"

class AccountCreateView(CreateView):
    model = models.Account
    template_name = "account/create_account.html"
    fields = ["status", "phone_number", "email", "identification_number"]

```

Musíme oběma pohledům vytvořit adresy, aby byly dostupné pro uživatele.

```py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('account/create', views.AccountCreateView.as_view(), name='account_create'),
]
```

A do "hlavního" souboru s adresami přidáme adresu na aplikaci `crm`.

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include("crm.urls")),
]
```

Vytvoříme "základní" šablonu `base.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Company Manager</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
</head>
<body>
<nav class="navbar navbar-expand-sm bg-light">
  <ul class="navbar-nav">
    <li class="nav-item">
      <a class="nav-link" href="{% url 'index' %}">Home</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="{% url 'account_create' %}">Opportunities</a>
    </li>

</nav>
{% block content %}

{% endblock %}

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
</body>
</html>
```

Dále přidáme uvítací obrazovku, zatím bez textů.

```html
{% extends "base.html" %}
{% block content %}
<div class="jumbotron text-center">
  <h1>Welcome in Company Manager</h1>
</div>

<div class="container">
  <div class="row">
    <div class="col-sm-4">
      <h3>Column 1</h3>
      <p>Lorem ipsum dolor..</p>
    </div>
  </div>
</div>
{% endblock %}
```

A poslední šablona je pro vytvoření firmy.

```html
{% extends "base.html" %}
{% block content %}
<h1>Create account</h1>
<form >
{{ form.as_p }}
<button type="submit" class="btn btn-primary">Primary</button>
</form>
{% endblock %}
```
