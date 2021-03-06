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

Django nabízí vlastní nástroj na automatické testování. Zaměříme se především na testování pohledů. Logika testu pohledu se skládá z následujících dvou kroků:

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

Test spustíme příkazem `python manage.py test`. Django projde všechny třídy, které dědí od `TestCase`, a spustí všechny jejich metody, které začínají slovem `test`. Ke třídě můžeme přidat i metody, které se jmenují jinak, ale ty nebudou automaticky spuštěny (mohou to být např. metody na nějaké speciální připravné práce). Každé spuštění testu je zpravidla **samostatné**, tj. pokud chceme testovat dva různé pohledy, je lepší na to použít dvě různé metody.

Důležité je, že každý test je zcela **samostatný**. Pokud tedy například v jedné metodě vytvoříme firmu, ta firma nebude dostupná v jiné metodě, která testuje pohled na seznam firem. Pokud bychom chtěli otestovat pohled na seznam firem, můžeme si vytvořit firmu ručně v rámci "přípravných prací".

Vytvořme nyní sofistikovanější test, který otevěří, že funguje vytvoření společnosti pomocí vyplnění formuláře. Tentokrát simulujeme akci POST, tj. voláme metodu `self.client.post()`, do které jako slovník vložíme hodnoty, jejichž vyplnění do formuláře simulujeme. Opět zkontrolujeme, zda je vrácen kód 200. Dále zkontrolujeme, že je firma v databázi, tj. počet záznamů v databázi je 1. Pro větší specifičnost testu by bylo možné například testovat, kolik je v databázi firem s konkrétním `identification_number`. Tím bychom nemuseli test upravovat, pokud v rámci příprav přidáme vytvoření firmy.

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

Testovat můžeme i zabezpečení aplikace. Uvažujme, že se uživatel nepřihlásí. V takovém případě by neměl mít možnost například si otevřít stránku se seznamem firem. Pokud by se o to pokusil, měl by být přesměrován na stránku s přihlašovacím formulářem. To můžeme testovat pomocí metody `assertRedirects()`, která kontroluje, zda byl uživatel přesměrován na správnou cílovou straánku. Přesnou adresu cílové stránky můžeme najít, pokud si stejnou akci vyzkoušíme v prohlížeči. Kromě adresy samotné přihlašovací stránky, tj. `/accounts/login/`, je v adrese ještě parametr `next`, který zajistí, že po přihlášení je uživatel "vrácen" na stránku, která ho na přihlašovací formulář přesměrovala.

```py
   def test_company_list_not_signed(self):
        response = self.client.get(reverse("company_list"), follow=True)
        self.assertRedirects(response, '/accounts/login/?next=/company/list')
```

Testovat můžeme i obsah stránky. Zkusme nyní otestovat ještě pohled se seznamem firem. Do metody `setUp()` přidáme vytvoření firmy a následně pomocí metody `assertContains()` zkontrolujeme, že vrácená HTML stránka obsahuje název testovací firmy.

```py
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user('jirka', 'jirka@mojefirma.cz', 'tajne-heslo')
        Company.objects.create(name="THE MAMA AI", phone_number="723 000000", identification_number="1000000")

    def test_company_list(self):
        self.client.login(username='jirka', password='tajne-heslo')
        response = self.client.get(reverse("company_list"))
        self.assertContains(response, "THE MAMA AI")
```

# Cvičení

## Obchodní případy pro adminy

- V administrátorském rozhraní uprav seznam obchodních případů tak, aby obsahovala pole `status` a `value`.
- Umožni uživatelům filtrování obchodních případů podle jejich statusu (pole `status`).
- Umožni uživatelům vyhledávání obchodních případů podle obsahu podle `description`.

### Řešení příkladu

Pokud u modelu nemáte pole `value`, které reprezentuje hodnotu potenciálního obchodního případu, tak si je prosím doplňte. Pokud máte pole pojmenované jinak, tak použijte váš název.

```py
class OpportunityAdmin(admin.ModelAdmin):
    list_display = ["status", "value"]
    list_filter = ["status"]
    search_fields = ["description"]
admin.site.register(models.Opportunity, OpportunityAdmin)
```

## Test vytváření obchodních případů

Přidej nyní automatické testy (můžeš je vložit jako další metody do třídy `CRMViewTests`, pouze nezapomeň dát na začátek názvu slovo `test`), které ověří, že obsluha obchodních případů funguje.

- Přidání obchodního případu by mělo vyžadovat oprávnění `add_opportunity`. V metodě `setUp()` můžeš toto oprávnění uživateli přidělit:

```py
# Tento řádek s importem je potřeba přidat
from django.contrib.auth.models import Permission

class CRMViewTests(TestCase):
    def setUp(self):
        # Tři řádky níže už bys v metodě setUp() měl(a) mít
        self.client = Client()
        self.user = User.objects.create_user('jirka', 'jirka@mojefirma.cz', 'tajne-heslo')
        Company.objects.create(name="Test company", phone_number="723 000000", identification_number="1000000")
        # Tento řádek musíš přidat do své metody setUp()
        self.user.user_permissions.add(Permission.objects.get(codename='add_opportunity'))
```

Přidej automatický test, který ověří, že jde přidat obchodní případ. Použij metodu `post`, do které vlož hodnoty všech povinných polí (můžeš samozřejmě přidat i nepovinná pole). Do polí `company` vlož hodnotu 1 (primární klíč vytvořeného obchodního případu) a do pole `sales_manager` též hodnotu 1 (primární klíč vytvořeného uživatele).

Ověř, že je vrácen kód 200. Ověř, že v databázi je nyní nový obchodní případ.

### Řešení příkladu

Toto je jedno z možných řešení. Do slovníku `data` můžete samozřejmě přidat i nějaké nepovinné hodnoty.

```py
def test_post_opportunity_create(self):
    self.client.login(username="jirka", password="tajne-heslo")
    response = self.client.post(reverse("opportunity_create"),
                                data={"company": "1",
                                        "sales_manager": "1",
                                        "status": "1"},
                                follow=True)
    self.assertEqual(response.status_code, 200)
    self.assertEqual(Opportunity.objects.count(), 1)
```

Níže vidíte, jak je nastavený model `Opportunity`, pro který tento test funguje. Povinná jsou pouze pole `company`, `sales_manager` a `status`, ostatní mají nastaveno `blank=True`, mohou tedy být nevyplněná.

Atribut `status_choices` je pouze číselník, není to pole, při vyplňování formuláře si ho tedy nevšímáme.

```py
class Opportunity(models.Model):
    status_choices = (
        ("1", "Prospecting"),
        ("2", "Analysis"),
        ("3", "Proposal"),
        ("4", "Negotiation"),
        ("5", "Closed Won"),
        ("0", "Closed Lost")
    )

    company = models.ForeignKey(Company, on_delete=models.RESTRICT)
    sales_manager = models.ForeignKey(User, on_delete=models.RESTRICT)
    primary_contact = models.ForeignKey(Contact, on_delete=models.SET_NULL, null=True, blank=True)
    description = models.TextField(null=True, blank=True)
    status = models.CharField(max_length=2, default="1", choices=status_choices)
    value = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
```

## Bonus 1

Do administrátorského rozhraní ke každému obchodnímu případu přidej název firmy, na kterou je navázán. To můžeš udělat přidáním metody `__str__()` k modelu `Company`. 

### Řešení

Přidáme metodu `__str()__`, která obecně říká, jak převést objekt na řetězec. V tomto případě je firma převedená na řetězec tak, že se použije její název (`name`).

```py
class Company(models.Model):
    status_choices = (
        ("N", "New"),
        ("L", "Lead"),
        ("O", "Opportunity"),
        ("C", "Active Customer"),
        ("FC", "Former Customer"),
        ("I", "Inactive")
    )

    name = models.CharField(max_length=50)
    status = models.CharField(max_length=2, default="N", choices=status_choices)
    phone_number = models.CharField(max_length=20, null=True, blank=True)
    email = models.CharField(max_length=50, null=True, blank=True)
    identification_number = models.CharField(max_length=100)
    address = models.ForeignKey("Address", on_delete=models.SET_NULL, null=True)

    # Toto je potřeba přidat
    def __str__(self):
        return self.name
```

Jako druhý krok přidáme pole `company` do seznamu `list_display`.

```py
class OpportunityAdmin(admin.ModelAdmin):
    # Sem přidáme hodnotu company
    list_display = ["status", "value", "company"]
    list_filter = ["status"]
    search_fields = ["description"]
admin.site.register(models.Opportunity, OpportunityAdmin)
```
