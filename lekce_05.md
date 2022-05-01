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

# Automatické testy

Automatické testy jsou klíčovou součástí dnešních aplikací. Moderním trendem je totiž poskytovat nové funkce a nové verze uživatelům okamžitě. V případě některých aplikací tak dochází k nahrání nové verze i několikrát denně. Při takové frekvenci změn by bylo nereálné (případně příliš drahé) testovat aplikaci ručně. I drobná změna v aplikaci se totiž může projevit nečekanou chybou v úplně jiné části aplikace, proto částečné "proklikání" nových funkcí nezaručuje, že změny aplikaci nijak nerozbily.

Django nabízí vlastní nástroj na automatické testování. Logika testu se skládá z následujících dvou kroků:

1. nasimuluji nějakou uživatelskou akci (např. otevření stránky nebo vyplnění formuláře),
1. zkontroluji, zda výstup, který Django vrací, odpovídá tomu, co by mělo být vráceno.

Test píšeme jako třídu, která dědí od třídy `TestCase`. Do třídy můžeme přidat metodu `setUp()`, která provádí *přípravné práce*, které jsou před testem potřeba. Důležité je, že veškeré testy jsou prováděny **v testovací databázi**, která je vytvořena na začátku testu a s databází aplikace nemá nic společného! Pokud tedy například chceme otestovat vyplnění formuláře, který vyžaduje přihlášení, je nutné v metody `setUp()` uživatele vytvořit. Žádné změny v testovací databázi se pak neprojeví v databázi, kterou používá naše aplikace, můžeme tedy heslo bez obav vložit do zdrojového kódu, protože k přihlášení do aplikace ho nikdo použít nemůže.

Vytvoříme nyní třídu `CRMViewTests`, která v metodě `setUp()` vytvoří klienta, který bude složit k simulování akcí uživatele, a uživatele, kterého použijeme k přihlášení.

```py
from django.contrib.auth.models import User
from django.test import TestCase
from django.test.client import Client
from django.urls import reverse

from crm.models import Company


class CRMViewTests(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user('jirka', 'jirka@mojefirma.cz', 'tajne-heslo')
```

Začneme testem otevření formuláře na vytvoření firmy. U této akce zkontrolujeme, zda Django vrací [návratový kód 200](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status), což je kód s jednoduchým významem `OK`. Zde můžeme simulovat i rozbití formuláře, např. odebráním některého z polí z modelu.

```py
    def test_get_company_create(self):
        self.client.login(username='jirka', password='tajne-heslo')
        response = self.client.get(reverse('company_create'))
        self.assertEqual(response.status_code, 200)
```

Sofistikovanějším testem je test vytvoření společnosti. Tentokrát simulujeme akci POST, tj. voláme metodu `self.client.post()`, do které jako slovník vložíme hodnoty, jejichž vyplnění do formuláře simulujeme. Opět zkontrolujeme, zda je vrácen kód 200.

```py
    def test_post_company_create(self):
        self.client.login(username='jirka', password='tajne-heslo')
        response = self.client.post(reverse('company_create'),
                                    data={"name": "Test 2",
                                          "status": "N",
                                          "phone_number": "723 123456",
                                          "identification_number": "123456789"},
                                    follow=True)
        self.assertEqual(response.status_code, 200)
        self.assertEqual(Company.objects.count(), 1)
```

Testovat můžeme i zabezpečení aplikace. Uvažujme, že se uživatel nepřihlásí. V takovém případě by neměl mít možnost například si otevřít stránku se seznamem firem. Pokud by se o to pokusil, měl by být přesměrován na stránku s přihlašovacím formulářem. To můžeme testovat pomocí metody `assertRedirects`, která kontroluje, zda byl uživatel přesměrován na správnou cílovou straánku.

```py
   def test_company_list_not_signed(self):
        response = self.client.get(reverse("company_list"), follow=True)
        self.assertRedirects(response, '/accounts/login/?next=/company/list')
```

Testovat můžeme i obsah stránky. Zkusme nyní otestovat ještě pohled se seznamem firem.

```py
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user('jirka', 'jirka@mojefirma.cz', 'tajne-heslo')
        Company.objects.create(name="Test company", phone_number="723 000000", identification_number="1000000")

    def test_company_list(self):
        self.client.login(username='jirka', password='tajne-heslo')
        response = self.client.get(reverse("company_list"))
        self.assertContains(response, "Test company")
```

# Závislosti

```
Django==4.0.4
python-decouple==3.6
```
