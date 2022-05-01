# Úprava administrátorského rozhraní

U administrátorského rozhraní jsme zatím pouze povolovali nové modely. Administrátorské rozhraní můžeme ale též přizpůsobit našim potřebám a můžeme například určovat, která políčka budou zobrazena ve formulářích.

## Úprava formuláře

První možností je výběr polí, která budou zobrazena ve formuláři. K určení polí nejprve vytvoříme třídu `CompanyAdmin` (název třídy nutně nemusí obsahovat název modelu, propojení třídy formuláře a modelu zajistíme až níže), která dědí od třídy `admin.ModelAdmin`. Té poté přidáme atribut `fields`, což je seznam polí, která chceme mít ve formuláři.

```py
class CompanyAdmin(admin.ModelAdmin):
    fields = ['name', 'phone_number', 'email', 'address']

admin.site.register(crm.models.Company, CompanyAdmin)
```

V některých případech můžeme chtít přidat nějaká pole pouze pro čtení. Taková pole mohou být například `status` (ten je určený tím, jestli s firmou máme nějaké obchodní případy a případně jaké) nebo `identification_number`, které by měla mít firma pevně přidělené. Tato pole přidáme do seznamu `fields` a navíc je přidáme do nového seznamu `readonly_fields`.

```py
class CompanyAdmin(admin.ModelAdmin):
    fields = ["name", "status", "phone_number", "email", "address", "identification_number"]
    readonly_fields = ["status", "identification_number"]
```

Kromě polí na formuláři můžeme provést úpravu i seznamu záznamů. Ve výchozím nastavení se zobrazují pouze hodnoty v poli `name`, v případě jiných modelů pak název modelu a primární klíč (např. `Opportunity object (5)`). V případě, že implementujeme metodu `__str__` pro model, zobrazí se hodnota, kterou metoda vrací.

Do tabulky můžeme přidat další sloupce pomocí atributu `list_display`. Přidejme k poli `name` pole `status` a `email`.

```py
class CompanyAdmin(admin.ModelAdmin):
    fields = ["name", "status", "phone_number", "email", "address", "identification_number"]
    readonly_fields = ["status", "identification_number"]
    list_display = ["name", "status", "email"]
```

Dále se nám může hodit filtrování. Můžeme chtít v tabulce například vidět pouze zákazníky nebo naopak nové společnosti, které jsme zatím nekontaktovali. Přidání filtru zařídíme pomocí atributu `list_filter`.

```py
class CompanyAdmin(admin.ModelAdmin):
    fields = ["name", "status", "phone_number", "email", "address", "identification_number"]
    readonly_fields = ["status", "identification_number"]
    list_display = ["name", "status", "email"]
    list_filter = ["status"]
```

Další užitečnou funkcí je vyhledávání. Vyhledávací pole zatím vůbec nevidíme, protože jsme neurčili, která pole by měla být při vyhledávání použita. Tato pole určíme pomocí atributu `search_fields`. Nastavme tam pole `name`, `email` a `identification_number`. Zkusme ale ještě jednu věc - vyhledávání pomocí polí v navázaných modelech. Můžeme například chtít vyhledávat pomocí pole `description` u všech navázaných obchodních případů (model `Opportunity`). Toto pole není přímo součástí modelu `Company`, ale modelu `Opportunity`. Název pole, pomocí kterého přistupujeme z firmy k obchodnímu případu, je ve výchozím nastavení stejný jako název navázaného modelu, tj. `opportunity`. Dále musíme přidat **dvě** podtržítka a název pole u navázaného modelu. Tj. výsledný název je `opportunity__description`.

```py
class CompanyAdmin(admin.ModelAdmin):
    fields = ["name", "status", "phone_number", "email", "address", "identification_number"]
    readonly_fields = ["status", "identification_number"]
    list_display = ["name", "status", "email"]
    list_filter = ["status"]
    search_fields = ["name", "email", "identification_number", "opportunity__description"]
```

V administrátorském rozhraní nyní přibylo vyhledávací políčko, které vyhledává podle hodnot v zadaných polích.

# Závislosti

```
Django==4.0.4
python-decouple==3.6
```
