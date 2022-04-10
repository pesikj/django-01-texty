# Rozšíření modelu uživatele

U uživatele chceme často evidovat více informací, než kolik umožňuje klasický `User` model. U firemních aplikací je často potřeba uložit oddělení, telefonní číslo, pobočku či nadřízeného zaměstnance, u běžných komerčních aplikací pak třeba doručovací adresu, odkazy na profily na sociálních sítích atd.

Django nabízí několik možností, jak rozšířit možnosti modelu `User`. Jednou z nich je vytvoření vlastní verze modelu, to je ale používáno především v situaci, kdy máme specifické požadavky (např. provádíme import uživatelů z jiné databáze). Pokud náme jde o přidání dodatečných polí, je možné přidat samostatný model (např. `Employee`) a k němu přidat vazbu `OneToOne` na model `Employee`. Dále k modelu `Employee` přidáme oddělení a telefonní číslo.

```py
class Employee(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    department = models.CharField(max_length=100, blank=True)
    phone_number = models.CharField(max_length=20, blank=True)
```

Záznam modelu `Employee` by v ideálním případě měl být vytvořen při vytvoření záznamu mudelu `User`. To můžeme zařídit automaticky pomocí **signálů**. Signály jsou vysílány při různých operacích (např. uložení záznamu) a pokud se přihlásíme k odběru signálu, můžeme při operaci vždy provést nějakou akci. Vytvoříme tedy funkci `create_user_profile`, kterou označíme tzv. **dekorátorem** `@receiver` (přijímač). Funkce `create_user_profile()` bude mít parametry `sender` (odesílatel signálu), `instance` (upravovaný záznam), `created` (pravdivostní hodnota, která udává, jestli došlo k vytvoření nebo úpravě již existujícího záznamu) a pro případné další parametry je přítomen `**kwargs`.

```py
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Employee.objects.create(user=instance)
```

Protože jsme vytvořili nový model, musíme provést migraci databáze.

```
python manage.py makemigrations
python manage.py migrate
```

Dále zaregistrujeme model pro zobrazení v administrátorském rozhraní.

```
admin.site.register(crm.models.Employee)
```

## Úprava informací o uživateli

Nejčastější případ úpravy dat o uživateli je, že si data upravuje sám uživatel. Aby si uživatel mohl své údaje upravit, přidáme na úpravu pohled `EmployeeUpdateView`. Využijeme `UpdateView`, přidáme ale metodu `get_object()`. Tato metoda je u třídy `UpdateView` čte ID záznamu, který upravujeme, z adresy. To by v našem případě nebylo správně, protože uživatel by měl mít možnost upravovat pouze své údaje (pokud není administrátor). Upravíme tedy metodu `get_object()`. Využijeme atribut `request`, což je objekt třídy `HttpRequest` a obsahuje různé informace o požadavku uživatele. Kompletní popis této třídy je možné najít [v dokumentaci](https://docs.djangoproject.com/en/4.0/ref/request-response/). Jedním z klíčových atrbitů požadavku je uživatel, který požadavek vyvolal. S uživatelem již umíme pracovat na úrovni šablony (např. při kontrole, zda je uživatel přihlášen), nyní ho využijeme v metodě `get_object()`. Atribut `user` je záznam třídy `User` a můžeme tak přistoupit přes jeho atribut `employee` k záznamu třídy `Employee`, což je přesně to, co potřebujeme.

```py
class EmployeeUpdateView(LoginRequiredMixin, UpdateView):
    template_name = "employee/update_employee.html"
    fields = ['department', 'phone_number']
    success_url = reverse_lazy("employee_update")

    def get_object(self, queryset=None):
        return self.request.user.employee
```

## Nepovinné čtení na doma - kombinace úprav obou modelů

```py
from django.forms import ModelForm
from crm.models import Employee
from django.contrib.auth.models import User


class UserForm(ModelForm):
    class Meta:
        model = User
        fields = ('first_name', 'last_name', 'email')

class EmployeeForm(ModelForm):
    class Meta:
        model = Employee
        fields = ('department', 'phone_number')
```

```py
class EmployeeUpdateView(LoginRequiredMixin, UpdateView):
    template_name = "employee/update_employee.html"
    form_class = EmployeeForm
    success_url = reverse_lazy("employee_update")

    def get_object(self, queryset=None):
        return self.request.user.employee

    def get_context_data(self, **kwargs):
        context = super().get_context_data()
        context["user_form"] = UserForm(instance=self.request.user)
        return context

    def post(self, request, *args, **kwargs):
        user_form = UserForm(request.POST, instance=request.user)
        if user_form.is_valid():
            user_form.save()
        return super().post(request, *args, **kwargs)
```


# Zprávy

```py
from django.contrib.messages.views import SuccessMessageMixin

class EmployeeUpdateView(LoginRequiredMixin, SuccessMessageMixin, UpdateView):
    template_name = "employee/update_employee.html"
    fields = ['department', 'phone_number']
    success_url = reverse_lazy("employee_update")
    success_message = "Data was updated successfully"

    def get_object(self, queryset=None):
        return self.request.user.employee
```

```html
{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <div class="alert alert-primary {{ message.tags }}" role="alert">
        {{ message }}
    </div>
    {% endfor %}
</ul>
{% endif %}
```

https://getbootstrap.com/docs/4.0/components/alerts/

```py
from django.contrib.messages import constants as messages

MESSAGE_TAGS = {
    messages.DEBUG: 'alert-secondary',
    messages.INFO: 'alert-info',
    messages.SUCCESS: 'alert-success',
    messages.WARNING: 'alert-warning',
    messages.ERROR: 'alert-danger',
}
```

https://ordinarycoders.com/blog/article/django-messages-framework

# Překlady
