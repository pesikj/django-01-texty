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

Django využívá sadu nástrojů gettext, které je potřeba nainstalovat. Instalační balíček pro Windows si můžeš stáhnout [zde](https://mlocati.github.io/articles/gettext-iconv-windows.html). Ve většině případů je nejlepší kombinace `64 bit` a `static`. Po instalaci je potřeba restartovat vývojové prostředí.

Nyní můžeme začít s označením řetězce k překladu. V případě pohledů můžeme například přeložit zprávy, které se zobrazují uživateli po úspěšném provedení nějakého úkonu.

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

Před vytvořením souboru s překlady musíme upravit nastavení, aby bylo jasné, kde budou překlady uloženy. Řekněme, že naše překlady budou v adresáři `locale` v kořenovém adresáři projektu. To provedeme vložením hodnoty `LOCALE_PATHS` do souboru `settings.py`.

```py
LOCALE_PATHS = [
    "locale"
]
```

Django si bohužel adresář samo nevytvoří, musíme to udělat za něj. Můžeme to provést přes vývojové prostředí, nástroj na prohlížení souborů nebo i z terminálu, a to pomocí příkazu `mkdir`.

```
mkdir locale
```

Následně vygenerujeme soubor s řetězci. Při generování nastavujeme, pro které jazykové mutace by měly být soubory vygenerovány. Vygenerujme pouze českou verzi, jako parametr `-l` tedy přidáme `cs`.

```
django-admin makemessages -l cs
```

Níže je soubor, který byl vygenerován (včetně komentáře).

```
#. Translators: This message is shown after successful creation of a company
#: .\crm\views.py:20
msgid "Company created!"
msgstr ""
```

Do řádky, která začíná `msgstr`, doplním do uvozovek překlad zprávy do češtiny.

```
#. Translators: This message is shown after successful creation of a company
#: .\crm\views.py:20
msgid "Company created!"
msgstr "Společnost vytvořena!"
```

Po úpravě překladu musíme provést tzv. komplilaci. Kompilace je v tomto případě převod překladů do rychleji čistelného formátu (pro počítač, nikoli pro člověka), ze kterého pak Django načte požadované texty.

```
python manage.py compilemessages
```

Po kompilace zpráv je bohužel nutné restartovat Django server, tj. v terminálu použít zkratku `Ctrl + C` a následně server znovu spustit klasickým příkazem

```
python manage.py runserver
```

Nyní se nabízí otázka, jak vlastně Django pozná, který jazyk má zobrazit? Existují 4 možnosti:

- z URL adresy,
- z tzv. cookie (cookie je informace, kterou aplikace ukládá do prohlížeče konkrétního uživatele a z něj pak může načíst jím preferovanou jazykovou verzi, případně další informace, ukážeme si dále),
- z webového prohlížeče (prohlížeče někdy odesílají informace o tom, jaký jazyk uživatel používá),
- z globálního nastavení.

Django zkouší postupně popsané zdroje a jakmile nastavení jazyka najde, použije ho a dál již nepokračuje.

Využijeme nyní druhý bod, který umožní uživateli, aby si nastavil jazyk, který chce používat. Nejprve musíme upravit nastavení a **přidat** novou hodnotu do seznamu `MIDDLEWARE` (opravdu pouze přidáváme novou hodnotu, stávající hodnoty seznamu musíme ponechat).

```py
MIDDLEWARE = [
    ...
    "django.middleware.locale.LocaleMiddleware",
]
```

Do souboru `company_manager/urls.py` (tj. do **hlavního** souboru s adresami, **nikoli** do `crm/urls.py`) do seznamu `urlpatterns` přidáme následující řádek, který zajišťuje pohledy pro práci s jazykovými verzemi, mimo jiné pohled na změnu jazykové verze.

```
path('i18n/', include('django.conf.urls.i18n')),
```

Následně můžeme do nějaké stránky (např. na titulní stránku) vložit následující formulář, který zobrazí uživateli seznam dostupných jazykových verzí, umožní mu jednu verzi si vybrat a nastavení uložit.

```
<form action="{% url 'set_language' %}" method="post">{% csrf_token %}
    <input name="next" type="hidden" value="{{ redirect_to }}">
    <select name="language">
        {% get_current_language as LANGUAGE_CODE %}
        {% get_available_languages as LANGUAGES %}
        {% get_language_info_list for LANGUAGES as languages %}
        {% for language in languages %}
        <option value="{{ language.code }}" {% if language.code== LANGUAGE_CODE %} selected{% endif %}>
            {{ language.name_local }} ({{ language.code }})
        </option>
        {% endfor %}
    </select>
    <input type="submit" value="Go">
</form>
```

Uložení si můžeme ověřit v prohlížeči. Stačí otevřít vývojářské nástroje ve webovém prohlížeči, následně vybrat záložku `Application`, v menu vlevo pak rozkliknout `Cookies` a kliknout na `http://localhost:8000/`. Pokud jsme uložili volbu jazyka, měli bychom ve sloupci `name` vidět `django_language` a ve stejném řádku ve sloupci `value` hodnotu `cs`.

Zatím jsme vyřešili pouze překlady textů v pohledech, zajímavější je ale zajistit překlady textů v šablonách.

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
