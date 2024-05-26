# Kullanıcı Kaydı

Bu derste, Django uygulamamızı kullanıcı kayıt sistemi ekleyerek geliştirmeye devam edeceğiz. Bu sistem, kullanıcıların bir hesap oluşturmasına olanak tanıyacak ve birçok web uygulaması için temel bir özelliktir. Amacımız, kullanıcıların web sitesinin ön ucundan kayıt olabilecekleri bir kayıt sayfası oluşturmaktır.

### Kullanıcı Kayıt Sayfası Oluşturma

#### Adım 1: Kullanıcı Yönetimi için Yeni Bir Uygulama Oluşturma

Öncelikle, kullanıcı yönetimine ayrılmış yeni bir uygulama oluşturmalıyız. Bu, kullanıcı ile ilgili mantığın kapsüllenmiş ve uygulamanın diğer bölümlerinden ayrı olmasını sağlar.

Proje dizininize (manage.py'nin bulunduğu yere) gidin ve yeni bir uygulama oluşturun:

```bash
python manage.py startapp users
```

Uygulamayı oluşturduktan sonra, `INSTALLED_APPS` listesine ekleyin:

```python
INSTALLED_APPS = [
    ...
    'users.apps.UsersConfig',
    ...
]
```

#### Adım 2: Kullanıcı Kaydı Görünümü Oluşturma

`users/views.py` dosyasını açın ve kullanıcı kayıt sayfası için yeni bir görünüm oluşturun. Bu amaçla Django'nun yerleşik `UserCreationForm` formunu kullanacağız.

```python
from django.shortcuts import render, redirect
from django.contrib import messages
from django.contrib.auth.forms import UserCreationForm

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'{username} için hesap oluşturuldu!')
            return redirect('blog-home')
    else:
        form = UserCreationForm()
    return render(request, 'users/register.html', {'form': form})
```

#### Adım 3: Kayıt Sayfası için Şablon Oluşturma

Sonraki adım olarak, kayıt sayfası için bir şablon oluşturun. `users` uygulama dizininde `templates` adlı bir klasör ve onun içinde `users` adlı bir alt klasör oluşturun. Son olarak, bu dizinde `register.html` adlı bir dosya oluşturun.

`users/templates/users/register.html`:

```html
{% extends "blog/base.html" %}
{% load crispy_forms_tags %}

{% block content %}
  <div class="content-section">
    <form method="POST">
      {% csrf_token %}
      <fieldset class="form-group">
        <legend class="border-bottom mb-4">Bugün Katıl</legend>
        {{ form|crispy }}
      </fieldset>
      <div class="form-group">
        <button type="submit" class="btn btn-outline-info">Kaydol</button>
      </div>
    </form>
    <div class="border-top pt-3">
      <small class="text-muted">
        Zaten bir hesabınız var mı? <a href="#">Giriş Yap</a>
      </small>
    </div>
  </div>
{% endblock %}
```

#### Adım 4: URL Kalıplarını Güncelleme

Proje ana `urls.py` dosyanıza kayıt sayfası için bir URL kalıbı ekleyin:

```python
from django.contrib import admin
from django.urls import path
from users import views as user_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('register/', user_views.register, name='register'),
    path('', include('blog.urls')),
]
```

#### Adım 5: Crispy Forms'u Yükleme ve Yapılandırma

Crispy Forms, form renderlamayı Bootstrap ile basitleştiren üçüncü taraf bir Django uygulamasıdır. Crispy Forms'u pip ile yükleyin:

```bash
pip install django-crispy-forms
```

`INSTALLED_APPS` listenize `crispy_forms` ekleyin ve şablon paketi olarak Bootstrap 4'ü ayarlayın:

```python
INSTALLED_APPS = [
    ...
    'crispy_forms',
    ...
]

CRISPY_TEMPLATE_PACK = 'bootstrap4'
```

#### Adım 6: Özel Kullanıcı Kayıt Formu Oluşturma

Kullanıcı kayıt formuna e-posta alanı gibi daha fazla alan eklemek için `UserCreationForm` sınıfını genişleterek özel bir form oluşturun.

`users/forms.py`:

```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm

class UserRegisterForm(UserCreationForm):
    email = forms.EmailField()

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']
```

Görünümü, özel kayıt formunu kullanacak şekilde güncelleyin:

`users/views.py`:

```python
from .forms import UserRegisterForm

def register(request):
    if request.method == 'POST':
        form = UserRegisterForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'{username} için hesap oluşturuldu!')
            return redirect('blog-home')
    else:
        form = UserRegisterForm()
    return render(request, 'users/register.html', {'form': form})
```

#### Adım 7: Flash Mesajlarını Gösterme

Flash mesajlarını temel şablonda göstermek için `blog/templates/blog/base.html` dosyasını güncelleyin:

```html
{% if messages %}
  {% for message in messages %}
    <div class="alert alert-{{ message.tags }}">
      {{ message }}
    </div>
  {% endfor %}
{% endif %}
```

### Sonuç

Artık Django uygulamanızda tamamen işlevsel bir kullanıcı kayıt sistemine sahipsiniz. Kullanıcılar ön uçtan hesap oluşturabilir ve bilgileri veritabanında saklanır. Bu derste, yeni bir uygulama oluşturmayı, görünümleri ve şablonları ayarlamayı, Django formlarını kullanmayı ve Crispy Forms ile form renderlamayı ele aldık.

