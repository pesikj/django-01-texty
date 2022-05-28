# Registrace uživatele

Registrace uživatele je v podstatě vytvoření záznamu. Při registraci uživatele je vytvořený záznam modelu `User`, který je součástí frameworku Django. Registrace uživatele se drobně liší pouze zadáváním hesla, které uživatele donutíme zadávat dvakrát (kvůli kontrole) a neukládáme ho do databáze přímo, ale jako hash.

Jako první vytvoříme formulář, tj. vložíme novou třídu do souboru `forms.py`. Tato třída bude dědit od třídy `UserCreationForm`, tj. od speciální třídy s formulářem pro registraci uživatele.

U některých aplikací, především u těch určených pro koncové uživatele, je trendem používat e-mail jako uživatelské heslo. Django má na tyto dvě informace dvě pole - jedno na e-mail a jedno na uživatelské jméno. Můžeme ale udělat jednoduchý trik, a to je označení pole `username` popisem `Email`. Uživatel se pak bude přihlašovat pomocí e-mailu a nemusí si pamatovat nebo ukládat uživatelské jméno. Do seznamu polí pak vložíme `username`, `password1` a `password2`.

```py
from django.contrib.auth.forms import UserCreationForm

class RegisterUserForm(UserCreationForm):
    username = CharField(required=True, label='Email')

    class Meta:
        model = User
        fields = ("username", "password1", "password2")
```

Dále vytvoříme pohled. Ten bude dědit od `CreateView` a jako atribut `form_class` nastavíme hodnotu `RegisterUserForm`.

```py
from crm.forms import RegisterUserForm

class RegisterView(CreateView):
    form_class = RegisterUserForm
    success_url = reverse_lazy("login")
    template_name = "registration/register.html"
```

Dále vytvoříme šablonu.

```html
{% extends "base.html" %}
{% block content %}
<h2>Register</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Register</button>
</form>
{% endblock %}
```

A nakonec musíme do seznamu adres `urlpatterns` v souboru `crm/urls.py` vložit adresu, která nás odkáže na námi vytvořený pohled.

```py
path('register', views.RegisterView.as_view(), name='register'),
```

Tím máme registraci hotovou. Účet uživatele je po registraci aktivní a uživatel se může rovnou přihlásit. V některých případech můžeme chtít ověřit, že se jedná o skutečného uživatele a nikoli o bota, který chce zahltit naši aplikaci. To můžeme udělat způsoby:

- odesláním kontrolního e-mailu s odkazem, na který uživatel musí kliknout,
- přidáním recaptcha (např. [Google reCAPTCHA](https://www.google.com/recaptcha/about/)),
- položením nějaké otázky uživateli, např. na výpočet jednoduchého příkladu.

# Modul django-crispy-forms

Pokud bychom chtěli výrazně upravit vzhled formuláře, můžeme použít modul `django-crispy-forms`. Ten umožní rozvrhnout formulář a nastavit mu konkrétní CSS třídy přímo ve třídě formuláře, což je většinou přehlednější než nastavení vzhledu v šabloně. K modulu `django-crispy-forms` existuje další užitečný modul, a to `crispy-bootstrap5`, který je vyladěný speciálně pro Bootstrap 5

Moduly je potřeba nejprve nainstalovat, a to přes vývojové prostředí nebo příkazem

```
pip install django-crispy-forms
pip install crispy-bootstrap5
```

Moduly musíme přidat do seznamu `INSTALLED_APPS`.

```py
INSTALLED_APPS = (
    # Ostatní hodnoty necháme v seznamu
    "crispy_forms",
    "crispy_bootstrap5",
)
```

Následně nastavíme konstanty `CRISPY_TEMPLATE_PACK` a `CRISPY_ALLOWED_TEMPLATE_PACKS`, též v souboru `settings.py`.

```py
CRISPY_ALLOWED_TEMPLATE_PACKS = "bootstrap5"

CRISPY_TEMPLATE_PACK = "bootstrap5"
```

Následně přidáme metodu `__init__()` ke třídě `CompanyForm`. Následně vkládáme třídy `Div()`, které vytváření HTML tagy `<div>` v samotné šabloně. Vytvoříme jeden tag `Div()`, kterému nastavíme atribut `css_class` na hodnotu `row` - ten symbolizuje jeden "řádek" v šabloně. Následně vložíme samostatné `Div()` pro každé pole formuláře. Na první místo vložíme název pole a dále jako parametr `css_class` vložíme třídu, která bude definovat šířku pole.

```py
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Layout, ButtonHolder, Submit, Div


class CompanyForm(ModelForm):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.layout = Layout(
            Div(
                Div("name", css_class="col-sm-4"),
                Div("status", css_class="col-sm-2"),
                Div("identification_number", css_class="col-sm-4"),
                Div("email", css_class="col-sm-4"),
                Div("phone_number", css_class="col-sm-4"),
                css_class="row",
            ),
            ButtonHolder(
                Submit('submit', 'Submit', css_class='button')
            )
        )

```

# Django REST API

Některé aplikace nechtějí s uživateli komunikovat pouze pomocí prohlížeče a zobrazením webové stránky, ale i poskytováním dat v textovém formátu. Tato data nemají žádné zbytečné informace o formátování, velikosti písem atd., ale pouze texty, čísla či další data. Takové rozhraní je označováno jako API. Ideálním formátem je JSON, který je možné snadno zpracovat třeba v Pythonu.

Pro práci s API můžeme využít modul `djangorestframework`, který nabízí opravdu široké možnosti, jaká API vytvořit. Modul je potřeba nejprve nainstalovat a přidat do seznamu `INSTALLED_APPS`.

```py
INSTALLED_APPS = [
    # Do seznamu vložím novou položku
    'rest_framework',
]
```

Při vytváření API jako první vytvoříme serializer, což je v podstatě popis struktury našich dat. Uvažujme, že chceme poskytovat informace o obchodních příležitostech. Vytvoříme tedy třídu s názvem `OpportunitySerializer`. Do ní vložíme třídu `Meta`, které nastavíme atributy `model` a `fields`. S polem `status` a `value` není problém, ale chceme zobrazit i název společnosti a uživatele. K tomu musíme použít speciální pole `serializers.StringRelatedField`, které vezme navázaný model a převede ho na řetězec.

```py
from rest_framework import serializers
from crm.models import Company, Opportunity

class OpportunitySerializer(serializers.ModelSerializer):
    company = serializers.StringRelatedField()
    sales_manager = serializers.StringRelatedField()

    class Meta:
        model = Opportunity
        fields = ['company', 'status', 'sales_manager', 'value']
```

Dále vytvoříme pohled `OpportunityViewSet` v souboru `views.py`, který tentokrát dědí od třídy `viewsets.ModelViewSet`. Modelu nastavíme attribut `queryset`, což jsou záznamy, které chceme v pohledu zobrazit, a `serializer_class`.

```py
from crm.serializers import OpportunitySerializer
from rest_framework import viewsets

class OpportunityViewSet(viewsets.ModelViewSet):
    queryset = models.Opportunity.objects.all()
    serializer_class = OpportunitySerializer
```

Adresu tentokrát vytvoříme trochu jinak. Do souboru `crm/urls.py` nejprve přidáme vytvoření objektu `router` (směrovače), do kterého zaregistrujeme adresu na náš pohled `OpportunityViewSet`. Dále přidáme do seznamu `urlpatterns` adresu, která tentokrát nejde na konkrétní adresu, ale na router.

```py
router = routers.DefaultRouter()
router.register(r'opportunities', views.OpportunityViewSet)

urlpatterns = [
    # Ostatní adresy nechám
    path('register', views.RegisterView.as_view(), name='register'),
    path('api/', include(router.urls)),
]
```

Nyní si v prohlížeči můžeme otevřít stránku [http://localhost:8000/api/opportunities/](http://localhost:8000/api/opportunities/), což je náhled informací, které naše API poskytuje. API můžeme vyzkoušet i v modulu `requests`, který je na komunikaci s API ideálním. Modul musíme opět nainstalovat.

Následně vytvoříme nový program. Tento program je naprosto samostatný, není tedy součástí Django. Program vrátí informace o obchodních případech v čistě textové podobě.

```py
import requests
response = requests.get("http://localhost:8000/api/opportunities/")
print(response.text)
```

# Cvičení

## Firmy přes API

Umožni prohlížení seznamu firem přes API. Vytvoř nový serializer, který bude obsahovat pole `name`, `status`, `identification_number`, `email`. Vytvoř nový pohled a přidej adresu do routeru. Otestuje pomocí modulu `request`, že API funguje.

## Bonus: Formulář pro obchodní případ

Uprav formulář pro obchodní případ, aby též využíval moduly `django-crispy-forms` a `crispy-bootstrap5`.
