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

Do šablony `base.css` vložíme odkazy na Javascript a CSS knihovny (uložení do static files je bonusové cvičení).

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.8.0/Chart.min.js" integrity="sha256-Uv9BNBucvCPipKQ2NS9wYpJmi8DTOEfTA/nH2aoJALw=" crossorigin="anonymous"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.8.0/Chart.min.css" integrity="sha256-aa0xaJgmK/X74WM224KMQeNQC2xYKwlAt08oZqjeF0E=" crossorigin="anonymous" />
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
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

Předchozí graf má nevýhodu v tom, že uživatel nemůže nijak ovlivnit, co je zobrazeno. Můžeme ale využít filtry, které jsme si ukázali v předchozí lekce, a nechat uživatele vybrat záznamy, které se mají v grafu zobrazit. K tomu stačí upravit `OpportunityListView`. Hlavní změnou je, že tentokrát nepracujeme se všemi záznamy, ale pouze s těmi, které vyhovují filtru, který nastavil uživatel. Tyto záznamy najdeme v atributu `self.object_list`. Kromě toho, že atribut nemusí obsahovat všechny záznamy pro daný model, s ním můžeme pracovat stejně jako v předchozím případě (jedná se opět o `QuerySet`) a napojit na něj metodu `filter()`. Další operaci provádět nebudeme, tj. zobrazíme data bez agregace.

Filtry je možné použít i bez tabulky, tj. můžeme pomocí filtrů ovládat pouze graf. Stačilo by odebrat kód na vykreslení tabulky ze šablony a smazat `SingleTableMixin` z mateřských tříd.

```py
class OpportunityListView(LoginRequiredMixin, SingleTableMixin, FilterView):
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["qs"] = self.object_list.filter(value__isnull=False)
        return context
```

Přidáme kód pro zobrazení grafu.

```js
$(document).ready(function(){
    var ctx = document.getElementById('graf').getContext('2d');
    var myChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: [{%for row in qs%}'{{row.company}}',{%endfor%}],
            datasets: [{
                label: 'Opportunity value',
                data: [{%for row in qs%}{{row.value}},{%endfor%}]
            }]
        }
    });
});
```

A nesmíme zapomenout na `<canvas>`.

```html
<canvas id="graf"></canvas>
```

### Čtení na doma: Agregace

Pokud bychom chtěli pomocí filtrů zobrazit celkovou hodnotu obchodních případů pro jednotlivé firmy, musíme provést agregaci. Tentokrát na ni musíme jít z druhé strany, tj. od obchodních případů, protože ty jsou v atributu `self.object_list`. Abychom přidali název firmy jako další sloupeček, podle kterého má být provedena agregace, použijeme metodu `values()` a jako parametr vložíme `company__name`, tedy název firmy. Nakonec přidáme již známou metodu `annotate()`.

```py
context["qs"] = self.object_list.filter(value__isnull=False)\
    .values("company__name").annotate(value=Sum("value"))
```

# Dump dat

Data z Django můžeme exportovat pomocí příkazu `python manage.py dumpdata`. Ten umí vypsat vybrané záznamy na obrazovku nebo do textového souboru, z něj je pak můžeme importovat zpět, což se hodí třeba v případě, že chceme aplikaci nasadit na server nebo ji přesouváme z jednoho serveru na druhý.

Zpravidla nechceme provést export celé databáze, ale jen části. Máme dvě možnosti, jak to provést - vybráním modelů (nebo celých aplikací), jejich žáznamy chceme exportovat, nebo naopak vyloučením těch, které nechceme (a všechny ostatní nevyloučené tím pádem chceme). Při exportu dat je důležité myslet především na cizí klíče. Pokud bychom totiž chtěly provést import záznamu, který se odkazuje na neexistující záznam nějakého jiného modelu, import bude neúspěšný.

Začneme tím, že provedeme export. Chceme provést export modelů `auth.Group`, `auth.User` a všech modelů v aplikaci `crm`.

```
python -Xutf8 manage.py dumpdata auth.Group auth.User crm --indent=4 --output=data.json
```

Před nahráváním je potřeba upravit signál, který vytváří objekt modelu `Employee` při vytvoření nového uživatele. Ten by totiž neměl probíhat během importu dat, tam automatické vytváření záznamů nechceme, aby se "netlouklo" k importem. Zabráníme tomu pomocí úpravy importu. Ve slovníku dalších parametrů `kwargs` máme parametr `raw`, který má hodnotu `True`, pokud dochází k importu dat. Přidáme tedy do podmínky, aby byl záznam modelu `Employee` vytvořen pouze v případě, že nedochází k importu, tj. v případě, že parametr `raw` má hodnotu `False`.

```py
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created and kwargs.get('raw', False):
        Employee.objects.create(user=instance)
```

Nyní můžeme provést import dat. Abychom si ověřili, že import skutečně funguje, můžeme stávající databázi přejmenovat, pomocí 

```
python manage.py migrate
```

vytvořit novou a do ní nahrát data.

```
python manage.py loaddata data.json
```

Po nahrání by v aplikaci mělo fungovat přihlášení a měla by být dostupná data.

# Čtení na doma: Přepínání časového pásma

# Cvičení

## Graf Sales Pipeline

Přidej na titulní stránku graf, který je označovaný jako *Sales Pipeline*. Graf zobrazuje celkový objem obchodních případů v jednotlivých stavech, tj. jaká je celková hodnota případů ve stavu `Prospecting`, jaká je ve stavu `Analysis` atd.

Grafy můžeme mít vedle sebe. Nejprve vlož do metody `get_context_data()` další dotaz. Nejprve je potřeba říct, podle kterého sloupce provádíme agregaci - to řekneme metodou `values()`, jak je uvedeno níže. Jakmile víme, podle čeho řádky sloučit, pak už postupujeme stejně jako v lekci - provedeme sumu podle hodnoty obchodního případu a vyfiltrujeme pouze řádky, který není součet prázdné číslo.

```py
context["qs2"] = models.Opportunity.objects.values("status")\
    # Na další řádek přidej annotate pro pole value
    # Na další řádek přidej filtr, že hodnota nemá být null
```

Dále do blocku `scripts` šablony `index.html` vlož kód, který vygeneruje druhý graf.

```js
$(document).ready(function(){
    // Kód z lekce
    var ctx = document.getElementById('graf').getContext('2d');
    var myChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: [{%for row in qs%}'{{row}}',{%endfor%}],
            datasets: [{
                label: 'Opportunity value',
                data: [{%for row in qs%}{{row.opportunity_value}},{%endfor%}]
            }]
        }
    });
    // Nový graf
    var ctx2 = document.getElementById('graf2').getContext('2d');
    var myChart2 = new Chart(ctx2, {
        type: 'bar',
        data: {
            labels: , //Tady chceme status pro každý řádek
            datasets: [{
                label: 'Opportunity value',
                data: // Tady chceme hodnotu pro každý řádek
            }]
        }
    });
});
</script>
```

A nezapomeň přidat další `<canvas>` s id `graf2`.

## Bonus: Převod do statických souborů

Převeď do statických souborů Javascript skripty, které tvá aplikace nyní stahauje z internetu. Skripty i CSS soubory ideálně ukládej do podsložek podle jmen modulů, abys v budoucnu snadno odlišil(a) vlastní kód od importovaných knihoven. Např. pro Bootstrap můžeš ve složce `static` vytvořit podsložku `bootstrap`, tam uložit soubory `bootstrap.bundle.min.js` a `bootstrap.min.css` a poté nahradit odkazy přímo zadané v tagu `<link>` a `<script>` odkazem získaným pomocí Django tagu `static`.
