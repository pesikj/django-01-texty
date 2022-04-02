# CSS a Bootstrap

**CSS (Cascading Style Sheet)** je stylovací jazyk, kterým HTML stránce dodáváme vzhled. Pomocí CSS nastavujeme, jak naše nadpisy vypadají, jakou mají barvu, jakým jsou písmem, jakou velikostí. Můžeme nastavit kolem bloku textu rámeček, přidat barvu pozadí a například i vytvářet efekty při najetí kurzorem myši. Přes CSS nastavíme třeba i to, že web má text ve dvou sloupcích nebo že obrázková galerie má fotky ve čtyřech řadách po pěti. CSS umí z obyčejné textové stránky udělat pastvu pro oči.

## Použití CSS

Kaskádové styly můžeme nastavit na třech místech:
- v samostatném souboru (s příponou `.css`),
- ve speciálním tagu `style`,
- v atributu libovolného tagu `style`.

První dva způsoby se používají stejným stylem - zapíšeme tag, na který chceme nastavení aplikovat, a hodnoty jednotlivých atributů zapisujeme do složených závorek. Vždy píšeme název atributu, dvojtečku, hodnotu nastavení a na závěr středník.

Například kaskádový styl níže změní pozadí a písmo všech odstavců na stránce.

```css
p {
    background-color: bisque;
    font-family: Georgia;
}
```

Pokud tento obsah vložíme do souboru `styles.css`, vložíme do tagu `head` HTML stránky tag `link` s odkazem na náš soubor kaskádových stylů.

```html
<link rel="stylesheet" href="styles.css" />
```

Stejný obsah můžeme vložit i do tagu `style` v tagu `head` HTML stránky.

Pokud bychom chtěli stejné nastavení aplikovat na jeden konkrétní odstavec, použijeme třetí možnost, tj. atribut `style`.

```html
<p style="background-color: aqua; font-family: Georgia;">
```

Nyní umíme aplikovat kaskádové styly na všechny tagy nebo na jeden vybraný. Co ale v případě, že chceme obarvit několik odstavců, ale ne všechny? K tomu slouží třídy.

Při vytváření třídy napíšeme vždy její název za tečku a následně vložíme kaskádové styly do složených závorek.

```css
.bisque-paragraph {
    background-color: bisque;
    font-family: Georgia;
}
```

Aby byla třída aplikována, musíme elementu nastavit atribut `class`.

```html
<p class="bisque-paragraph">
```

Priorita stylů je vždy (od nejvyšší): přímé nastavení stylu v tagu `style`, nastavení třídy, nastavení atributu. Pokud má element více tříd s nastavením stejného atributu, aplikuje se ten poslední v pořadí. Dále elementy přebírají nastavení od nadřazeného fontu. Např. odkaz bude mít stejné písmo jako odstavec, do kterého je vložen, pokud mu nenastavíme jiný.

V krajním případě je možné přiřadit konkrétnímu nastavení direktivu `!important`. Jde však skutečně o nástroj nejvyšší nouze, který nám může ve stylech udělat pořádnou paseku.

```css
.bisque-paragraph {
    background-color: bisque !important;
    font-family: Georgia;
}
```

## Nejpoužívanější atributy

Příklady atributů, které můžeme nastavit, jsou v níže.

| atribut | význam | příklad hodnoty |
|---|---|---|
| `background-color` | barva pozadí | `blue`, `#74992e` |
| `border` | okraj/rámeček prvku | `solid`, `dashed red` |
| `display` | způsob zobrazení | `none` |
| `list-style` | vzhled nečíslovaného seznamu | `square`, `inside`, `url('/media/examples/rocket.svg')`|

Řada elementů je sdružena do skupin, které spolu věcně souvisí. Níže si je projdeme.

### Nastavení písma

Přehled základních písem je k dispozici [https://www.w3schools.com/css/css_font.asp](zde).

| atribut | význam | příklad hodnoty |
|---|---|---|
| `color` | barva písma | `red` |
| `font-family` | písmo | `Courier New` |
| `font-size` | velikost písma | `12px`, `smaller`, `2em` |
| `font-style` | styl písma | `italic` |
| `font-weight` | tloušťka písma | `bold`, `bolder`, `600` (`normal` = `400`) |

### Nastavení okrajů

V kaskádových stylech máme dva typy okrajů:

- `padding` ("vnitřní okraj") je prostor mezi obsahem elementu a rámečkem. `padding` je svým způsobem součástí elementu, takže například má stejnou barvu jako obsah elementu.
- `margin` ("vnější okraj") je prostor mezi rámečkem a okolím. Není obarvený stejnou barvou jako obsah elementu.

![](images/lekce_02/margin-padding.png)

U obou typů okrajů nastavujeme jejich šířku, a to jak pro všechny strany najednou, tak pro jednotlivé strany. Nastavení konkrétní strany má přednost před obecným nastavením.

| atribut | význam | příklad hodnoty |
|---|---|---|
| `margin` | šířka všech vnějších okrajů | `2em`, `50px`, `10px 50px 20px` |
| `margin-bottom` | šířka horního vnějšího okraje | |
| `margin-left` | šířka levého vnějšího okraje | |
| `margin-right` | šířka pravého vnějšího okraje | |
| `margin-top` | šířka horního vnějšího okraje | |

Vnitřní okraje se nastavují stejným způsobem.

### Nastavení polohy

Polohu můžeme nastavit absolutně (vzhledem k levému hornímu okraji stránky) nebo relativně (vzhledem k okolním prvkům). Vzhledem k obrovskému množství různých zařízení a velikostí obrazovek je lepší se nastavení polohy vyhnout.

### Nastavení textu

| atribut | význam | příklad hodnoty |
|---|---|---|
| `text-align` | zarovnání textu | `left`, `right`, `justify`, `start` |
| `text-decoration` | podtržení (nebo obecně dekorace) textu | `underline`, `underline overline #FF3028;` |
| `text-indent` | odsazení prvního řádku | `1em` |
| `text-transform` | úprava velikosti textu | `capitalize`, `uppercase`, `lowercase` |

### Nastavení velikosti

| atribut | význam | příklad hodnoty |
|---|---|---|
| `height` | výška | `150px`, `6em`, `75%` |
| `šířka` | výška | `150px`, `6em`, `75%` |

### Nastavení viditelnosti

| atribut | význam | příklad hodnoty |
|---|---|---|
| `z-index` | viditelnost elementu | `1`, `5`, `auto` |

Kompletní přehled prvků je k dispozici například [zde](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Properties_Reference#common_css_properties_reference). 

Samostatnou kapitolou jsou barvy. CSS nabízí širokou škálu pojmenovaných barev, jejich přehled je npaříklad [zde](https://www.w3schools.com/cssref/css_colors.asp). Pokud by ti seznam nestačil a potřebuješ použít jinou barvu (např. kvůli dodržení grafického manuálu), můžeš si barvu namixovat např. [zde](https://htmlcolorcodes.com/).

### Tagy span a div

V souvislosti s CSS časo využíváme dva tagy - `span` a `div`. Ty slouží většinou pro "obalení" nějakých elementů stránky třídou. Element `span` je označovaný jako in-line (řádkový) a slouží k obalení nějaké malé části prvků, např. několika slov odstavci. Element `div` je block-line (blokový). Jednoduše řečeno, při použití `div` tagu je před a za tag vloženo zalomení řádků.

## Bootstrap

CSS framework je ve své podstatě sada předdefinovaných tříd, které můžeme použít na našem webu. Většina CSS frameworků nabízí obrovský rozsah tříd pro nejrůznější styly - prvky formulářů, odstavce, navigační panely, seznamy a další. Velkou výhodou frameworků je zpravidla i to, že zajistí responzivitu, prvky na stránce se tedy přizpůsobí tabletům a mobilním telefonům.

Nejpoužívanějším CSS frameworkem je [Bootstrap](https://getbootstrap.com/). Výhodou Bootstrapu je propracovaná dokumentace a velké množství návodů, tutoriálů a šablon. Alternativy k Bootstrapu jsou například [Materialize CSS](https://materializecss.com/), [Foundation](https://get.foundation/) nebo [Bulma](https://bulma.io/).

### Řádky a sloupce

V Bootstrapu jsou klíčovými elementy řádky a sloupce. Sloupce vkládáme do řádků a u sloupců nastavujeme šířku. Šířka řádku je 12.

Například třída `col-sm-12` zamená, že element bude roztažený přes celý řádek na zařízeních s malým displejem, třída `col-lg-6` znamená roztažení přes polovinu sloupce na velkých zařízeních. Obě třídy lze kombinovat, tj. element může být různě široký na různě velkých displejích.

Příklad jednoduché stránky s Bootstrapem je [zde](files/lekce_02/stranka_bootstrap.html).

## Použití Bootstrap v Django

Naše webová aplikace se bude skládat ze spousty stránek, všechny ale budou mají společné prvky (např. navigační panel, patička s kontaky, logo apod.). Tyto opakující se elementy vkládáme do stránky, kterou můžeme pojmenovat jako `base.html`.

Vytvoříme "základní" šablonu `base.html`. U této stránky je nutné vyznačit bloky, jak jsme si ukázali úvodním kurzu Django. Blok s obsahem stránky pojmenujeme `content` a vložíme tedy tag `{% block content %}` a párový tag `{% endblock %}`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Company Manager</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
</head>
<body>
{% block content %}

{% endblock %}

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
</body>
</html>
```

U každé stránky nastavíme, že je rozšířením stránky `base.html` pomocí tagu `{% extends "base.html" %}`. Začneme s uvítací obrazovkou, kde přidáme nějaký testovací text. I na této stránce musíme vyznačit blok `content`, aby došlo k propojení mezi oběma bloky.

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

Další šablona je pro stránku s vytvořením firmy.

```html
{% extends "base.html" %}
{% block content %}
<h1>Create new company</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Primary</button>
</form>
{% endblock %}
```

# Úkoly

## Navigační panel

Přidej k našemu webu navigační panel. Navigační panel vkládáme do tagu `nav` jako nečíslovaný seznam `ul`. Níže je příklad navigačního panelu s odkazem na titulní stránku.

- Základní třída pro panel je `navbar`.
- Třída `navbar-expand-sm` zařídí, že se na malých obrazovkách menu zobrazí jako vertikální.
- Třída `bg-light` určuje barvu pozadí, v tomto případě jde o světle šedou. Další možnosti pozadí jsou k dispozici [zde](https://getbootstrap.com/docs/4.0/utilities/colors/#background-color).


```html
<nav class="navbar navbar-expand-sm bg-light">
  <ul class="navbar-nav">
    <li class="nav-item">
      <a class="nav-link" href="{% url 'index' %}">Home</a>
    </li>
</nav>
```

Protože odkazů v naší aplikaci bude více a nemusely by se do panelu vejít, můžeš vytvořit menu, do kterého budeš vkládat odkazy, které spolu souvisejí (např. jedno rozbalovací menu pro firmy, druhé pro obchodní příležitosti atd.). Níže je příklad rozbahovacího menu, které vkládáme jako další položku do seznamu. 

- Rozbalovací menu je samostatný (vnořený) nečíslovaný seznam, který má třídu `dropdown`. 
- Nadpis menu je vložený jako odkaz se třídou `dropdown-toggle`. Atribut `data-toggle` atribut zajistí, že po kliknutí myší na nadpis dojde k rozbalení seznamu.
- Tag `span` se třídou `caret` zobrazí šipku u odkazu, aby bylo zřejmé, že jde o rozbalovací menu.
- Jednotlivé odkazy v menu jsou pak vloženy jako nečíslovaný seznam s třídou `dropdown-menu`.

```html
<nav class="navbar navbar-expand-sm bg-light">
  <ul class="navbar-nav">
    <li class="dropdown"><a class="dropdown-toggle" data-toggle="dropdown">Rozbalovací menu<span class="caret"></span></a>
      <ul class="dropdown-menu">
        <li><a href="#">Položka 1</a></li>
        <li><a href="#">Položka 2</a></li>
      </ul>
    </li>
  </ul>
</nav>
```
