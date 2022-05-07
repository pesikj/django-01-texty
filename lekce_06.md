# Závislosti

Většina projektů v Pythonu (nejen webových) potřebuje před spuštěním doinstalovat nějaké moduly. Aby byla instalace co nejjednodušší, je dobrou praxí vkládat seznam potřebných modulů do souboru `requirements.txt` v nejvyšším adresáři aplikace. Ideální je přidat do seznamu moduly včetně čísla verze, které se píše za dva symboly rovná se. Tím je zajištěno, že projekt půjde spustit i po uvolnění nových verzí modulů, které už by s naším kódem nemusely být kompatibilní.

```
Django==4.0.4
python-decouple==3.6
```

Je samozřejmě též dobrou praxí, zvláště u aktivně vyvíjených projektů, postupně přecházet (upgradovat) na nové verze.

# Překlady

Pokud vyvíjíme web pro uživatele z různých zemí, většínou neřešíme pouze překlady, ale i časová pásma, formát data, čísel atd. My se nyní zaměříme na překlad webu, protože to je nejdůležitější krok.

Aby se různé jazyky nepletly v textu, Django umožňuje generovat textové soubory, do kterého jsou automaticky importovány označené řetězce. Pokud každý řetězec pak v textovém souboru vytvoříme jeho překlad. Textových souborů s překlady můžeme mít libovolné množství, záleží čístě na tom, kolik jazykových verzí chceme vytvořit.

```py
from django.utils.translation import gettext as _

class CompanyCreateView(LoginRequiredMixin, SuccessMessageMixin, CreateView):
    model = models.Company
    template_name = "company/create_company.html"
    fields = ["name", "status", "phone_number", "email", "identification_number"]
    success_url = reverse_lazy("index")
    # Translators: This message is shown after successful creation of a company
    success_message = _("Company created!")
```

```py
LOCALE_PATHS = [
    "locale"
]
```

```
mkdir locale
```

```
django-admin makemessages -l cz
```

```
#. Translators: This message is shown after successful creation of a company
#: .\crm\views.py:20
msgid "Company created!"
msgstr ""
```

```
#. Translators: This message is shown after successful creation of a company
#: .\crm\views.py:20
msgid "Company created!"
msgstr "Společnost vytvořena!"
```

```html
{% load i18n %}

<a class="nav-link" href="{% url 'index' %}">{% translate "Home" %}</a>
<a class="nav-link" href="{% url 'company_create' %}">{% translate "Create Company" %}</a>

```

```
#: .\crm\templates\base.html:25
msgid "Home"
msgstr "Domů"

#: .\crm\templates\base.html:28
msgid "Create Company"
msgstr "Vytvořit společnost"
```

```
django-admin compilemessages
```
