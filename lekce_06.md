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

Django využívá sadu nástrojů gettext, které je potřeba nainstalovat. Instalační balíček pro Windows si můžeš stáhnout [zde](https://mlocati.github.io/articles/gettext-iconv-windows.html). Ve většině případů je nejlepší kombinace `64 bit` a `static`. Po instalaci je potřeba restartovat vývojové prostředí. Postup instalace na MacOS spočívá v následujících příkazech:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install gettext
brew link gettext --force
```

A konečně instalace na Ubuntu spočívá v následujícím příkaze:

```
sudo apt-get install gettext
```

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

Všimni si, že v importu je část `as _`. Tím je funkce `gettext` pro potřeba aktuálního souboru přejmenovaná na `_`. Tato úprava, kterou doporučují i vývojáři Django v oficiální dokumentaci, je čistě estetická a pomáhá větší přehlednosti kódu.

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

## Nastavení jazyka

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

Dále je potřeba doplnit seznam dostupných jazyků. Pokud počítáme s češtinou a angličtinou, pak doplníme:

```py
LANGUAGES = [
    ["cs", "Czech"],
    ["en", "English"]
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

## Překlady v šablonách

Zatím jsme vyřešili pouze překlady textů v pohledech, zajímavější je ale zajistit překlady textů v šablonách. Důležité je vždy do šablony (ideálně na začátek, **pod** tag `{% extends %}`, pokud se v šabloně vyskytuje) vložit tag `{% load i18n %}`. To nám umožní využívat tag `{% translate %}`. Do něj vždy vložíme řetězec, který je určený k překladu, např. `{% translate "Home" %}`.

```html
{% load i18n %}

<a class="nav-link" href="{% url 'index' %}">{% translate "Home" %}</a>
<a class="nav-link" href="{% url 'company_create' %}">{% translate "Create Company" %}</a>
<a class="nav-link" href="{% url 'company_list' %}">{% translate "Companies" %}</a>
```

Následně opět zadáme příkaz

```
django-admin makemessages -l cs
```

Nové řetězce se objeví v souboru `django.po` a můžeme k nim přidat překlady.

```
#: .\crm\templates\base.html:25
msgid "Home"
msgstr "Domů"

#: .\crm\templates\base.html:28
msgid "Create Company"
msgstr "Vytvořit společnost"

#: .\crm\templates\base.html:31
msgid "Companies"
msgstr "Společnosti"
```

Poté provedeme kompilace zpráv příkazem

```
django-admin compilemessages
```

a restartujeme server.

## Překlad polí u modelů

V případě překladu názvů polí modelů je situace podobná, pouze použijeme funkci `gettext_lazy`. Ta zajistí načtení překladu až ve chvíli, kdy je řetězec skutečně použit, například při renderování (vykreslování) šablony.

```py
from django.utils.translation import gettext_lazy as _
```

Následně jako první parametr u polí přidáme název, který je opět opatřený podtržítkem, aby byl zajistěn překlad. Netýká se to pole `address`.

```py
    name = models.CharField(_("Name"), max_length=20)
    status = models.CharField(_("Status"), max_length=2, default="N", choices=status_choices)
    phone_number = models.CharField(_("Phone Number"),max_length=20, null=True, blank=True)
    email = models.CharField(_("Email"),max_length=50, null=True, blank=True)
    identification_number = models.CharField(_("Identification Number"),max_length=100)
    address = models.ForeignKey(Address, on_delete=models.SET_NULL, null=True, blank=True)
```

Pole s cizím klíčem má totiž název stejný, jako navázaný model. Je tedy lepší rovnou přejmenovat celý model, a to pomocí vnořené třídy `Meta`. Název lze specifikovat pro jednotné i množné číslo, což jsou (asi, lingvisté nechť mě případně opraví :-) ) ve většině jazyků různá slova.

```py
class Address(models.Model):
    street = models.CharField(max_length=200, blank=True, null=True)
    zip_code = models.CharField(max_length=10, blank=True, null=True)
    city = models.CharField(max_length=100, blank=True, null=True)

    class Meta:
        verbose_name = _("Address")
        verbose_name_plural = _("Addresses")
```

Tím získáme i názvy políček u modelů česky.

# Formuláře

Nyní si pohrajeme s formuláři. U formulářu je častá validace, která zabrání tomu, aby uživatelé zadávali do polí nesmysl (např. záporné nákupy zboží, neúplná čísla bankovních účtů, nesmyslná data narození atd.). Ověření, zda je nějaká hodnota ve formuláři smysluplná, zajistíme pomocí metody `clean()`.

## Validace formulářů

My provedeme ověření, zda uživatel zadal IČO ve správné délce. Nejprve ale musíme vytvořit formulář.

Vytvoříme formulář `CompanyForm` jako třídu, která dědí od třídy `ModelForm`. Do třídy přidáme vnořenou třídu `Meta`, která obsahuje dva atributy - `model` a `fields` (seznam polí). Hodnoty atributů nemusíme vymýšlet, ale můžeme je zkopírovat z pohledu `CompanyCreateView`.

Následně přidáme metodu `clean_identification_number()`, která provede kontrolu IČO. Nejprve načteme hodnotu, kterou zadal uživatel, z pole `cleaned_data`. V něm jsou hodnoty zadané uživatelem. Následně provedeme kontrolu délky řetězce. Pokud je jiná než 8, vyvoláme tzv. výjimku `ValidationError` typu pomocí slova `raise`. Výjimka je událost, která je zpravidla důsledkem nějaké chyby a je signálem, že aktuálně prováděná operace by měla být přerušena. Vyvoláním výjimky tedy zabráníme tomu, aby byla firma se špatným IČO vložena do databáze, a místo toho je zvolen alternativní postup, což je zobrazení chyby ve formuláři. Uživatel následně bude mít možnost chybu opravit a pokusit se o uložení znovu.

```py
from django.forms import ModelForm, ValidationError
from crm.models import Employee, Company

class CompanyForm(ModelForm):
    def clean_identification_number(self):
        identification_number = self.cleaned_data['identification_number']

        if len(identification_number) != 8:
            raise ValidationError(_("The identification number has incorrect length."))
        return identification_number

    class Meta:
        model = Company
        fields = ["name", "status", "phone_number", "email", "identification_number"]
```   

Abychom provázali formulář a pohled, přidáme k pohledu `CompanyCreateView` atribut `form_class` a smažeme atributy `model` a `fields`, protože tyto hodnoty jsou načtené prostřednictvím formuláře.

```py
class CompanyCreateView(LoginRequiredMixin, SuccessMessageMixin, CreateView):
    template_name = "company/create_company.html"
    form_class  = CompanyForm
    success_url = reverse_lazy("index")
    # Translators: This message is shown after successful creation of a company
    success_message = _("Company created!")
```

# Cvičení

## Překlady

Proveď překlad názvu textových polí a hlášky o úspěšném vytvoření u obchodních případů. Dále pomocí vnořené třídy `Meta` zařiď překlady modelu `Company` a `Contact`.

Přelož možnosti stavu u společnosti. To provedeš stejně jako u jiných řetězců, pouze pozor na to, že musíš překládat pouze slovní názvy, nikoli zkratky (tj. pouze `New`, nikoli `N`).

### Řešení

```py
class Company(models.Model):
    # Překlady stavů u firmy - vždy překládám druhou hodnotu vloženého řetězce, první nechám být
    status_choices = (
        ("N", _("New")),
        ("L", _("Lead")),
        ("O", _("Opportunity")),
        ("C", _("Active Customer")),
        ("FC", _("Former Customer")),
        ("I", _("Inactive")),
    )
    name = models.CharField(_("Name"), max_length=20)
    status = models.CharField(_("Status"), max_length=2, default="N", choices=status_choices)
    phone_number = models.CharField(_("Phone Number"),max_length=20, null=True, blank=True)
    email = models.CharField(_("Email"),max_length=50, null=True, blank=True)
    identification_number = models.CharField(_("Identification Number"),max_length=100)
    address = models.ForeignKey(Address, on_delete=models.SET_NULL, null=True, blank=True)

    def __str__(self):
        return self.name

    # Vložím třídu Meta kvůli překladu Foreign Key
    class Meta:
        verbose_name = _("Company")
        verbose_name_plural = _("Companies")

class Contact(models.Model):
    primary_company = models.ForeignKey(Company, on_delete=models.SET_NULL, null=True)
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    phone_number = models.CharField(max_length=20)
    email = models.CharField(max_length=50)

    # Vložím třídu Meta kvůli překladu Foreign Key
    class Meta:
        verbose_name = _("Contact")
        verbose_name_plural = _("Contacts")

class Opportunity(models.Model):
    status_choices = (
        ("1", "Prospecting"),
        ("2", "Analysis"),
        ("3", "Proposal"),
        ("4", "Negotiation"),
        ("5", "Closed Won"),
        ("0", "Closed Lost"),
    )

    # Překlad polí je zařízen třídami Meta, u modelu User to za nás zařídí Django
    company = models.ForeignKey(Company, on_delete=models.RESTRICT)
    sales_manager = models.ForeignKey(User, on_delete=models.RESTRICT)
    primary_contact = models.ForeignKey(Contact, on_delete=models.SET_NULL, null=True, blank=True)
    # Zde mám přeložené názvy
    description = models.TextField(_("Description"), null=True)
    status = models.CharField(_("Status"), max_length=2, default="1", choices=status_choices)
    value = models.DecimalField(_("Value"), max_digits=10, decimal_places=2, null=True)
```

## Validace čísla

Do metody clean_identification_number přidej kontrolu, zda hodnota obsahuje pouze čísla. K tomu můžeš využít metodu `isdigit()`, kterou zavoláš pomocí tečkové notace, např. `retezec.isdigit()`. Metoda vrací hodnotu `True`, pokud řetězec obsahuje pouze čísla. 

### Řešení

```py
    def clean_identification_number(self):
        identification_number = self.cleaned_data['identification_number']

        if len(identification_number) != 8:
            raise ValidationError(_("The identification number has incorrect length."))
        # Pokud v hodnotě, kterou zadal uživatel, nejsou jen čísla...
        if not identification_number.isdigit():
            # Vyvolám chybu
            raise ValidationError(_("The identification number must contain only numbers."))
        return identification_number
```

## Bonus: Validace e-mailu

Ověř, že `email` obsahuje zavináč. To můžeš zařídit pomocí operátoru `in`, který využiješ v podmínce. Podobnou úlohu jsme si ukazovali v kurzu Úvod do programování 1. Pokud hodnota, kterou zadal uživatel, zavináč neobsahuje, upozorni ho na chybu. Pozor na to, že pole `email` je nepovinné. Pokud není vyplněno, je na jeho místě ve slovníku `cleaned_data` hodnota `None`. Zkontroluj nejprve, zda hodnota není `None` a přítomnost zavináče řeš až v případě, že e-mail byl vyplněn.

Podmínku, zda byl vyplněn e-mail můžeš sestavit např. takto:

```py
    email = self.cleaned_data['email']
    if email:
        # Sem dopiš kontrolu zavináče
        pass
    return email
```

### Řešení

Vložíme metodu `clean_email`, která zkontroluje, zda je v řetězci zavináš.

```py
    def clean_email(self):
        email = self.cleaned_data['email']
        if email:
            if "@" not in email:
                raise ValidationError(_("Email does not contain @."))
        return email
```

## Bonus: Validace telefonního čísla

Důležitá jsou i telefonní čísla, která by měla mít správnou délku. Uvažuj následující dva formáty za správné:

- formát `+420734123456` (tj. řetězec začíná mezinárodní předvolbou `+420` a dále obsahuje 9 čísel, celkem tedy `+` a 12 čísel),
- formát `734123456` (tj. 9 znaků).

Pokud uživatel nezadá číslo v platném formátu, vypiš chybu. Kontroluj pouze **počet znaků**, nikoli to, že znaky jsou čísla.

### Řešení

```py
    def clean_phone_number(self):
        phone_number = self.cleaned_data['phone_number']
        if phone_number:
            if phone_number.startswith("+420"):
                if len(phone_number) != 13:
                    raise ValidationError(_("Incorrect format of phone number"))
            else:
                if len(phone_number) != 9:
                    raise ValidationError(_("Incorrect format of phone number"))
        return phone_number
```

## Nepovinný úkol na doma

Vyzkoušej tag `blocktranslate`, který může být využit k překladu delších textů. Níže je příklad jeho využití. Zkus tento tag s nějakým textem vložit na titulní stránku. Podívej se,
jak se zobrazí v souboru s překlady a nějaký překlad mu vytvoř. Pozor na to, že pokud první řetězec obsahuje zalomení řádku (`\n`), musí být zalomení řádku i v překladu. Pro zjednodušení vlož řetězec pouze jako jeden řádek.

```
{% blocktranslate %}Welcome in our system. This system helps company to be efficient in its business project.{% endblocktranslate %}
```