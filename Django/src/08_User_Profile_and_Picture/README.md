#  Kullanıcı Profili ve Resim

Bu bölümde, kullanıcı profil sayfamızı geliştirerek kullanıcıların profil resmi yükleme yeteneğini ekleyeceğiz. Bu eğitim, mevcut kullanıcı modelini ek alanlarla genişletmeyi ve belirli işlemlerden sonra belirli işlevleri yürütmek için Django sinyallerini kullanmayı öğretecek. Bu tür işlevsellik, sağlam ve kullanıcı dostu bir web uygulaması oluşturmak için gereklidir.

## Kullanıcı Modelinin Genişletilmesi

Başlangıç olarak, profil resmi gibi ek alanları barındırmak için mevcut kullanıcı modelini genişletmemiz gerekiyor. Django'da bunu başarmak için iki yaygın yaklaşım vardır: özel bir Profil modeli ile Bir-Bir Bağlantı kullanmak veya yerleşik `User` modelini doğrudan genişletmek. Bu eğitimde, Bir-Bir Bağlantı yöntemini kullanacağız.

### Adım 1: Profil Modeli Oluşturma

İlk olarak, kullanıcı profillerini yönetmek için `profiles` adında yeni bir uygulama oluşturun. Terminalinizde şu komutu çalıştırın:

```bash
python manage.py startapp profiles
```

Ardından, `profiles/models.py` dosyasında `Profile` modelini tanımlayın:

```python
from django.db import models
from django.contrib.auth.models import User
from PIL import Image

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    image = models.ImageField(default='default.jpg', upload_to='profile_pics')

    def __str__(self):
        return f'{self.user.username} Profile'

    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        img = Image.open(self.image.path)

        if img.height > 300 or img.width > 300:
            output_size = (300, 300)
            img.thumbnail(output_size)
            img.save(self.image.path)
```

### Adım 2: Profilleri Otomatik Olarak Oluşturmak ve Kaydetmek İçin Bir Sinyal Oluşturma

Yeni bir `User` oluşturulduğunda bir `Profile` nesnesinin oluşturulmasını sağlamak için Django sinyallerini kullanabiliriz. Bu, `profiles/signals.py` dosyasında bir sinyal tanımlanarak gerçekleştirilebilir:

```python
from django.db.models.signals import post_save
from django.contrib.auth.models import User
from django.dispatch import receiver
from .models import Profile

@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_profile(sender, instance, **kwargs):
    instance.profile.save()
```

Bu sinyalin uygulama hazır olduğunda kaydedildiğinden emin olmak için `profiles/apps.py` dosyasında bu sinyali içe aktarın:

```python
from django.apps import AppConfig

class ProfilesConfig(AppConfig):
    name = 'profiles'

    def ready(self):
        import profiles.signals
```

`settings.py` dosyasında `INSTALLED_APPS` bölümünü `profiles` uygulamasını içerecek şekilde güncellediğinizden emin olun:

```python
INSTALLED_APPS = [
    ...
    'profiles.apps.ProfilesConfig',
    ...
]
```

### Adım 3: Kullanıcı Kaydını Profil Oluşturmayı Ele Alacak Şekilde Güncelleme

Yeni bir kullanıcı kaydederken, bir profilin de oluşturulduğundan emin olun. Kayıt formunuzu ve görünümünüzü buna göre güncelleyin. `users/forms.py` dosyasında bir profil güncelleme formu oluşturun:

```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm
from profiles.models import Profile

class UserRegisterForm(UserCreationForm):
    email = forms.EmailField()

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

class UserUpdateForm(forms.ModelForm):
    email = forms.EmailField()

    class Meta:
        model = User
        fields = ['username', 'email']

class ProfileUpdateForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['image']
```

Profil güncellemelerini ele almak için `users/views.py` dosyasındaki görünümü güncelleyin:

```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from .forms import UserRegisterForm, UserUpdateForm, ProfileUpdateForm

def register(request):
    if request.method == 'POST':
        form = UserRegisterForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'Your account has been created! You are now able to log in')
            return redirect('login')
    else:
        form = UserRegisterForm()
    return render(request, 'users/register.html', {'form': form})

@login_required
def profile(request):
    if request.method == 'POST':
        u_form = UserUpdateForm(request.POST, instance=request.user)
        p_form = ProfileUpdateForm(request.POST, request.FILES, instance=request.user.profile)
        if u_form.is_valid() and p_form.is_valid():
            u_form.save()
            p_form.save()
            messages.success(request, f'Your account has been updated!')
            return redirect('profile')
    else:
        u_form = UserUpdateForm(instance=request.user)
        p_form = ProfileUpdateForm(instance=request.user.profile)

    context = {
        'u_form': u_form,
        'p_form': p_form
    }

    return render(request, 'users/profile.html', context)
```

### Adım 4: Profil Yönetimi İçin Şablonlar Oluşturma

Kullanıcı kaydı ve profil güncellemelerini ele almak için şablonlar oluşturun. `templates/users/register.html` dosyasında:

```html
{% extends "base_generic.html" %}
{% block content %}
  <div class="container mt-5">
    <h2>Register</h2>
    <form method="POST">
      {% csrf_token %}
      {{ form|crispy }}
      <button type="submit" class="btn btn-primary">Register</button>
    </form>
  </div>
{% endblock %}
```

`templates/users/profile.html` dosyasında:

```html
{% extends "base_generic.html" %}
{% block content %}
  <div class="container mt-5">
    <h2>Profile</h2>
    <form method="POST" enctype="multipart/form-data">
      {% csrf_token %}
      {{ u_form|crispy }}
      {{ p_form|crispy }}
      <button type="submit" class="btn btn-primary">Update</button>
    </form>
  </div>
{% endblock %}
```

### Sonuç

Yukarıda belirtilen adımları izleyerek, kullanıcıların profil resimlerini yükleyip yönetebileceği bir işlevselliği Django uygulamanıza başarıyla eklediniz. Bu, bir-bir ilişkiyle kullanıcı modelini genişletmeyi, otomatik profil oluşturma için Django sinyallerini kullanmayı ve kullanıcı etkileşimi için form ve görünümler sağlamayı içerir. Bu eğitim, kullanıcı deneyimini geliştiren ve uygulamanızın bütünlüğünü koruyan sağlam özellikler oluşturmayı göstermektedir.