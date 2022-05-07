# Závislosti

Většina projektů v Pythonu (nejen webových) potřebuje před spuštěním doinstalovat nějaké moduly. Aby byla instalace co nejjednodušší, je dobrou praxí vkládat seznam potřebných modulů do souboru `requirements.txt` v nejvyšším adresáři aplikace. Ideální je přidat do seznamu moduly včetně čísla verze, které se píše za dva symboly rovná se. Tím je zajištěno, že projekt půjde spustit i po uvolnění nových verzí modulů, které už by s naším kódem nemusely být kompatibilní.

```
Django==4.0.4
python-decouple==3.6
```

Je samozřejmě též dobrou praxí, zvláště u aktivně vyvíjených projektů, postupně přecházet (upgradovat) na nové verze.

# Překlady

Django je určené i pro rozsáhlé projekty a má připravený nástroj pro překlad textů do různých jazyků.


