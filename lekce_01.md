# Naše aplikace

V rámci kurzu vytvoříme aplikaci označovanou jako CRM, tedy aplikaci, do které jsou ukládány informace související s řízením obchodu a projektů. Základ naší aplikace budou informace o firmách (potenciálních nebo aktivních zákaznících), lidech (obchodních kontaktech) a obchodních případech. Do budoucna přidáme třeba záznamy o schůzkách nebo tvorbě marketingových kampaní.

Začneme vytvořením projektu, který můžeme pojmenovat např. python-crm. Na začátku musíme udělat posloupnost rutinních kroků:

- založit projekt ve vývojovém prostředí,
- do virtuálního prostředí nainstalovat Django.

V prostředí PyCharmu vytvoříme nový projekt volbou `File` -> `New Project`, případně tlačítkem `New Project` na uvítací obrazovce. Kromě názvu projektu nemusíme nic měnit a můžeme kliknout na tlačítko `OK`. `django` nainstalujeme pomocí panelu `Python Packages` (tlačítko na jeho otevření je v dolní liště), do vyhledávacího pole zadáme `django` a po úspěšném vyhledání balíčku klikneme na tlačítko `Install`.

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


class Company(models.Model):
    status_choices = (
        ("N", "New"),
        ("L", "Lead"),
        ("O", "Opportunity"),
        ("C", "Active Customer"),
        ("FC", "Former Customer"),
        ("I", "Inactive"),
    )
    name = models.CharField(max_length=20)
    status = models.CharField(max_length=2, default="N", choices=status_choices)
    phone_number = models.CharField(max_length=20, null=True, blank=True)
    email = models.CharField(max_length=50, null=True, blank=True)
    identification_number = models.CharField(max_length=100)
    address = models.ForeignKey(Address, on_delete=models.SET_NULL, null=True)


class Contact(models.Model):
    primary_company = models.ForeignKey(Company, on_delete=models.SET_NULL, null=True)
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

    company = models.ForeignKey(Company, on_delete=models.RESTRICT)
    sales_manager = models.ForeignKey(User, on_delete=models.RESTRICT)
    primary_contact = models.ForeignKey(Contact, on_delete=models.SET_NULL, null=True)
    description = models.TextField(null=True)
    status = models.CharField(max_length=2, default="1", choices=status_choices)

```

V dalším kroku vytoříme uvítací obrazovku (pohled `IndexView`) a pohled na vytvoření firmy, který pojmenujeme `CompanyCreateView`.

```py
from django.views.generic import CreateView, ListView, TemplateView
import crm.models as models
from django.urls import reverse_lazy

class IndexView(TemplateView):
    template_name = "index.html"

class CompanyCreateView(CreateView):
    model = models.Company
    template_name = "company/create_company.html"
    fields = ["name", "status", "phone_number", "email", "identification_number"]
    success_url = reverse_lazy("index")

```

Importy v PyCharmu jsou červeně podtržené. Je třeba PyCharmu vysvětlit, že hlavním (`root`) adresářem projektu je `company_manager` a v něm má hledat modul `crm`. To lze vyřešit tím, že na adresář `company_manager` klikneme pravým tlačítkem a zvolíme možnost `Mark directory as` -> `Source root`.

Musíme oběma pohledům vytvořit adresy, aby byly dostupné pro uživatele.

Aktuální verze projektu je v repositáři [django-01-projekt-lekce01](https://github.com/pesikj/django-01-projekt-lekce01).

```py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('company/create', views.CompanyCreateView.as_view(), name='company_create'),
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

Zatím nemáme pohled a šablonu na seznam firem, v aplikaci tedy nemůžeme zkontrolovat, zda bylo vytvoření úspěšné. Můžeme ale použít administrátorské rozhraní. Nejprve zaregistrujeme model `Company`, aby byl v rozhraní viditelný.

```html
from django.contrib import admin
import crm.models

admin.site.register(crm.models.Company)
```

Dále musíme vytvořit uživatele, který bude mít do administrátorského rozhraní přístup. To zařídíme pomocí příkazu `python manage.py createsuperuser`. Zadáme uživatelské jméno a heslo (e-mail můžeme nechat nevyplněný). Administrátorské rozhraní si otevřeme na adrese [http://localhost:8000/admin/](http://localhost:8000/admin/), zadáme uživatelské jméno a heslo a zkontrolujeme, zda v seznamu firem vidíme námi vytvořené záznamy.

## Nahrání na Git

Nyní můžeme náš projekt nahrát na Git. Z menu `VCS` vybereme volbu `Share on GitHub`. V okně `Share Project On Github` zadáme název projektu a rozhodneme, zda chceme uložit projekt jako soukromý nebo veřejný.

# Úkoly

## Seznam firem

Vytvoř stránku, kde uživatel uvidí všechny firmy, které jsou v aplikaci uložené. Tento pohled by měl být založený na pohledu `ListView` a podobný pohled jsme již vytvářeli v kruzu [Python pro web](https://kodim.cz/czechitas/progr2-python/python-pro-web/sablony-a-pohledy/#vytvoreni-seznamu-kurzu). Pro jednotlivé firmy stačí vypsat jejich názvy do nečíslovaného seznamu (`<ul>`).

## Obchodní případy

U modelu obchodního případu (`Opportunity`) přidej hodnotu, která bude typu `DecimalField`. Vytvoř soubor s migrací a proveď migraci své databáze.

Přidej do své aplikace pohled a šablonu na vytvoření obchodního případu. Následně vytvoř URL adresu aby bylo možné stránku otevřít. Před testem formuláře si pomocí administrátorského rozhraní vytvoř nového uživatele, který bude reprezentovat obchodníka.

## Bonus

Přidej pohled, šablonu a URL adresu pro stránku se seznamem obchodních případů.


# Odkazy

* [Django](https://www.djangoproject.com/)
* [Bootstrap](https://getbootstrap.com/)
* [Bootstrap 5 Navbars](https://www.w3schools.com/bootstrap5/bootstrap_navbar.php)

