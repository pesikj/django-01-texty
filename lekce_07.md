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
    ...
    "crispy_forms",
    "crispy_bootstrap5",
    ...
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

