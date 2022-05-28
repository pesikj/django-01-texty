# CK Editor

Nakonec si vyzkoušíme přidat editor, který umožní uživatelům snadno vytvářet formátovaný obsah. Takový editor se hodí například k editaci článků zpravodajského serveru nebo popisu produktu v e-shopu. Jako editor použijeme [CK Editor](https://ckeditor.com/), což je editor vytvořený v jazyce JavaScript a nabízí spoustu různých možností nastavení. K CK Editoru existuje modul `django-ckeditor`, který nám umožní snadno využít CK Editor v naší aplikaci.

```py
INSTALLED_APPS = [
    # Ostatní hodnoty necháme v seznamu
    "ckeditor",
]
```

Do naší aplikace musíme přidat zdrojový kód CK Editoru. K tomu vytvoříme adresář `static` v kořenovém adresáři Django projektu a do něj stáhneme a rozbalíme [CK editor](https://ckeditor.com/ckeditor-5/download/).

```py
STATIC_ROOT = "static/"
CKEDITOR_BASEPATH = "/static/ckeditor/ckeditor/"
```

Nyní k modelu `Company` přidáme nové pole typu `RichTextField`.

```py
from ckeditor.fields import RichTextField

class Company(models.Model):
    # Ostatní pole ponecháme
    notes = RichTextField()
```

A nakonec přidáme pole `notes` i do formuláře `CompanyForm`, kde mu nastavíme `widget=CKEditorWidget`.

```py
from ckeditor.widgets import CKEditorWidget

class CompanyForm(ModelForm):
    notes = CharField(widget=CKEditorWidget())

    class Meta:
        model = Company
        # Sem přidáme pole notes
        fields = ["name", "status", "phone_number", "email", "identification_number", "notes"]
```

# Časové zóny

Django je připravené na vývoj globálních aplikací, které používají uživatelé z různých částí světa. Kromě podpory překladů do různých jazyků je pak nutné počítat i s tím, že aplikaci budou používat lidé nacházející se v různých časových pásmech. Django tedy ve výchozím nastavení ukládá ke všem časovým údajům ukládá informace o časové zóně.

Uvažujme třeba, že bychom chtěli u každého obchodního případu ukládat, kdy byl založen. Přidáme tedy pole `created_on`, kterému nastavíme parametr `auto_now_add=True`. Do pole se pak automaticky vloží datum a čas **vytvoření** záznamu. Dále přidáme pole `updated_on`, kde nastavíme parametr `auto_now=True`. Hodnota pole bude aktualizována při **každé úpravě** záznamu.

```py
class Opportunity(models.Model):
    # Přidáme nové pole
    created_on = models.DateTimeField(auto_now_add=True, null=True)
    updated_on = models.DateTimeField(auto_now=True, null=True)
```

Nesmíme zapomenou na migraci databáze.

```
python manage.py migrate
python manage.py makemigrations
```

V administrátorském rozhraní si můžeme zobrazit pole `updated_on`.

```py
class OpportunityAdmin(admin.ModelAdmin):
    list_display = ["status", "value", "company", "updated_on"]
```

Výsledek v administrátorském rozhraní vidíme níže. Oproti datu a času, který vidíme na hodinách, je údaj posunutý. Zobrazuje se totiž v jiném časovém pásmu.

![](images/lekce_09/date_time_admin.PNG)

Časové pásmo, které je použito, je v souboru `settings.py` nastaveno v jako kontanta `TIME_ZONE`. Spolu s ní je v `settings.py` konstanta `USE_TZ`, pomocí které můžeme používání časových zón vypnout. Pokud hodnotu nastavíme jako `False`, budou všechna data ukládána a zobrazována v časové zóně, která je uvedená v hodnotě `TIME_ZONE`.

```py
TIME_ZONE = 'UTC'
USE_TZ = True
```

K časovým zónám se ještě vrátíme, ale nyní přejděme k tabulkám.

# Modul django-tables2

Pro práci s tabulkami existuje modul `django-tables2`, který za nás zařídí zobrazení tabulky v pěkném formátu, stránkování a další věci.

```py
INSTALLED_APPS = (
    ...,
    "django_tables2",
)

DJANGO_TABLES2_TEMPLATE = "django_tables2/bootstrap4.html"
```

```py
import django_tables2 as tables
from crm.models import Opportunity

class OpportunityTable(tables.Table):
    class Meta:
        model = Opportunity
        fields = ("company", "sales_manager", "status", "value", "updated_on")
```

```py
from django_tables2 import SingleTableView

class OpportunityListView(LoginRequiredMixin, SingleTableView):
    model = models.Opportunity
    table_class = tables.OpportunityTable
    template_name = "opportunity/list_opportunity.html"
```

```
{% extends "base.html" %}
{% load render_table from django_tables2 %}
{% block content %}
    <h1>Opportunities</h1>
    {% render_table table %}
{% endblock %}
```

![](images/lekce_09/table.PNG)

```py
class OpportunityTable(tables.Table):
    company = tables.LinkColumn("opportunity_update", args=[A("pk")],
                                attrs={"a": {"class": "cell-with-link"}})
```

```css
.cell-with-link {
    font-weight: bold;
    text-decoration-line: none;
}
```

## Modul django-filter

```py
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

```py
class OpportunityFilter(django_filters.FilterSet):
    class Meta:
        model = Opportunity
        fields = ['company', 'sales_manager', "status"]
```

```py
class OpportunityFilterFormHelper(FormHelper):
    form_method = 'GET'
    layout = Layout(
            Div(
                Div("company", css_class="col-sm-3"),
                Div("sales_manager", css_class="col-sm-3"),
                Div("status", css_class="col-sm-2"),
                Submit('submit', 'Filter', css_class='button'),
                css_class="row"
            )
        )
```

```py
class OpportunityListView(LoginRequiredMixin, SingleTableMixin, FilterView):
    model = models.Opportunity
    table_class = tables.OpportunityTable
    template_name = "opportunity/list_opportunity.html"
    filterset_class = OpportunityFilter

    def get_filterset(self, filterset_class):
        filterset = super().get_filterset(filterset_class)
        filterset.form.helper = OpportunityFilterFormHelper()
        return filterset
```

```
{% extends "base.html" %}
{% load render_table from django_tables2 %}
{% load crispy_forms_tags %}
{% block content %}
    <h1>Opportunities</h1>
    {% crispy filter.form %}
    {% render_table table %}
{% endblock %}
```

