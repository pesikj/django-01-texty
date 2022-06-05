# Statické soubory

Statickými soubory většinou myslíme obrázky, skripty v jazyce JavaScript nebo CSS soubory. Ty můžeme pro přehlednost ukládat do speciální složky. 

Pro práci se statickými soubory je potřeba mít v seznamu `INSTALLED_APPS` hodnotu `"django.contrib.staticfiles"`, tu už bychom tam ale měli mít automaticky.

Dále nastavíme hodnotu `STATIC_URL`, což je adresa, pomocí které přistupujeme ke statickým souborům. Pokud chceme, aby byly statické soubory dostupné pod adresou `static/`, nastavíme následující hodnotu:

```py
STATIC_URL = 'static/'
```

Zkusme nyní přesunout náš soubor se styly. Vytvoříme soubor `css/base.css` a vložíme do něj styly, které v naší aplikaci jsou definované v tagu `<style>` v souboru `base.html`. Obsah souboru `css/base.css` může například být

```css
.cell-with-link {
    font-weight: bold;
    text-decoration-line: none;
}
```

Dále musíme říct, v jakém adresáři jsou souboru uložené. To provedeme pomocí nastavení konstanty `STATICFILES_DIRS`. Konstanta je seznam, protože v aplikaci může být více adresářů se statickými soubory. My nastavíme jen jednou hodnotu, a to `BASE_DIR / "static"`. První část `BASE_DIR` je hlavní adresář naší aplikace, který Django vkládá do konfiguračního souboru automaticky.

```py
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
```

Abychom mohli statické soubory používat, musíme do každé šablony, kde je používáme, nahrát tag `static`. 

```
{% load static %}
```

Dále napojíme náš soubor `style.css` pomocí tagu `<link>`. V tagu jako adresu použijeme tag `static` a do něj vložíme cestu k požadovanému souboru uvnitř adresáře `static`.

```html
<link rel="stylesheet" type="text/css" href="{% static 'css/style.css' %}">
```

Při nahrávání aplikace na server je často výhodné mít všechny statické soubory pohromadě v jednom adresáři, aby byly odlišené od kódu aplikace a byla usnadněna konfigurace webového serveru. Cílový adresář, kde by měly být statické soubory shromážděny, nastavíme pomocí konstanty `STATIC_ROOT`. Můžeme například použít nový adresář `staticfiles`. Tento adresář bude čistě jako *systémový* nebo *výstupní*, nic do něj ručně nemusíme nahrávat.

```py
import os

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

K umístění statických souborů do adresáře spustíme příkaz

```
python manage.py collectstatic
```

Tímto příkazem budou statické soubory nasměrovány do jednoho adresáře a na něj pak můžeme přesměrovat například webový server.

# Grafy a vizualizace

Ke generování grafů existuje obrovské množství knihoven, které můžeme použít.

První možností je modul `matplotlib`, který probíráme na kurzu Python pro data 1. Modul umí vygenerovat obrázek v nějakém formátu (např. PNG). My můžeme modul použít tak, že si obrázek vygenerujeme v aplikaci a vložíme jej do webové stránky. Určitou výhodou takového přístupu je, že graf je skutečně pouze obrázek a nijak nereaguje na to, když na něj uživatel najede myší. 

Pokud například chceme, aby se po najetí na nějaký datový bod zobrazil jeho popisek, můžeme využít některé z javascriptových knihoven.

- Široce užívanou knihovnou je [Chart.js](https://www.chartjs.org/), která je poměrně jednoduchá na užití. Lze využití modul [django-chartjs](https://github.com/peopledoc/django-chartjs), který pomáhá s vkládáním grafů do Django aplikace.
- Mnohem sofistikovanější je knihovna [plotly](https://plotly.com/). Hodí se například pro zobrazování výstupů pro algoritmy strojového učení. Pro `plotly` modul existuje prostředí [Dash](https://plotly.com/dash/), které je zaměřený na tvorbu interaktivních dashboardů.
- Další knihovnou je [Highcharts](https://www.highcharts.com/).

My si vyzkoušíme modul Chart.js na jednoduchém příkladu. Graf vložíme do stránky přímo i s daty.

Nejprve pro větší přehlednost přidáme do stránky další blok, který pojmenujeme `scripts`. Do šablony `base.html` vložíme kód pro označení umístěno bloku.

```
{%block scripts %}

{% endblock %}
```

Nejprve si připravíme data k zobrazení. Do kontextu pohledu `IndexView` vložíme hodnotu `qs`, která bude obsahovat všechny obchodní případy, které mají zadanou hodnotu. Filtr bohužel musíme přidat, protože Javascript a Python používají pro nevyplněné hodnoty jiné označení a Javascript by `None` pochopil jako název neexistující proměnné.

```py
class IndexView(TemplateView):
    template_name = "index.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["qs"] = models.Opportunity.objects.filter(value__isnull=False)
        return context
```

Do šablony `index.html` vložíme kód pro vygenerování grafu. Atributem `type` nastavíme typ grafu (`bar` označuje sloupcový graf, další dostupné typy grafů jsou [v dokumentaci](https://www.chartjs.org/docs/latest/)) a do atributu `data` vložíme data. Do vnořeného atributu `labels` vložíme popisy řádků, což může být v našem případě název společnosti, ke které se obchodní případ vztahuje. Název musíme vložit do uvozovek. Dále do vnořeného atributu `datasets` vložíme atributy `label` (což je popisek datové řady) a `data`, což je seznam číselných hodnot, které určí výšky sloupců grafu.

```js
{% block scripts %}
<script>
$(document).ready(function(){
    var ctx = document.getElementById('graf').getContext('2d');
    var myChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: [{%for row in qs%}'{{row.company}}',{%endfor%}],
            datasets: [{
                label: 'Value of opportunities',
                data: [{%for row in qs%}{{row.value}},{%endfor%}]
            }]
        }
    });
});
</script>
{% endblock %}
```

Jako poslední vložíme do stránky tag `<canvas>`, kterému musíme nastavit `id` na hodnotu `graf` (musí odpovídat řetězci, který je v metodě `getElementById()`).

```html
<canvas id="graf"></canvas>
```

## Agregace

V řadě případů potřebujeme s daty provést nějakou agregaci, než je zobrazíme. Typickým příkladem je graf celkové obchody obchodních případů za jednotlivé obchodníky. Agregace znamená, že data jsou "pospojovaná" dohromady. Počet řádků v datech (a tím pádem i počet sloupců grafu) bude odpovídat počtu obchodníků a číselná hodnota (výška sloupce) bude odpovídat celkové hodnotě obchodních případů jednotlivých obchodníků.

Vezmeme tedy všechny uživatele a ke každému uživateli pomocí metody `annotate` přidáme sloupec, který bude obsahovat hodnotu obchodních případů (`opportunity__value`) a pomocí funkce `Sum` zajistíme jejich součet. K dispozici máme i další agregace, například počet nebo průměrnou hodnotu. Pozor na rozdíl počtu podtržítek - dvě podtržítka za sebou v případě funkce `Sum` znamenají odkaz na navázaný model.

```py
from django.contrib.auth.models import User
from django.db.models import Sum

# Upravíme metodu get_context_data()
context["qs"] = User.objects.annotate(opportunity_value=Sum('opportunity__value'))\
            .filter(opportunity_value__isnull=False)
```

Dále upravíme skript - pro popisy řádků náme stačí `row`, protože bude použit výchozí převod na řetězec (v případě modelu `User` je to uživatelské jméno).

```js
labels: [{%for row in qs%}'{{row}}',{%endfor%}],
```

V případě dat použijeme přidaný sloupec `opportunity_value`.

```js
data: [{%for row in qs%}{{row.opportunity_value}},{%endfor%}]
```

## Grafy a filtry

```py
class OpportunityListView(LoginRequiredMixin, SingleTableMixin, FilterView):
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["qs"] = self.object_list.filter(value__isnull=False)
        return context
```

### Čtení na doma: Agregace 

```py
context["qs"] = self.object_list.filter(value__isnull=False)\
    .values("company__name").annotate(value=Sum("value"))
```

# Dump dat

```
django-admin dumpdata
```

```
python -Xutf8 manage.py dumpdata crm.Company crm.Opportunity --indent=4 --output=data.json


# Přepínání časového pásma

