# Giriş ve Çıkış Sistemi

Bu eğitimde, bir Django uygulaması için kullanıcıların giriş yapabilmesi, çıkış yapabilmesi ve belirli sayfalara yalnızca kimliği doğrulanmış kullanıcıların erişebilmesini sağlayan bir kimlik doğrulama sistemi geliştireceğiz. İçeriği eleştirel bir şekilde ele alacak ve alanın titizliğini ve standartlarını yansıtacak şekilde hataları düzelteceğiz.

## Giriş ve Çıkış Görünümlerinin Ayarlanması

Django, kimlik doğrulama işlemlerini yönetmek için yerleşik görünümler sağlar, bunlar arasında giriş ve çıkış görünümleri de bulunur. Bu görünümleri proje URL modülümüzde yapılandırarak başlayacağız.

### Adım 1: Kimlik Doğrulama Görünümlerini İçe Aktarma

Öncelikle, Django'nun kimlik doğrulama sisteminden giriş ve çıkış görünümlerini içe aktarın.

```python
# Projenizin ana URL modülünde (urls.py)
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('register/', include('users.urls')),  # Kayıt rotanızın yapılandırıldığını varsayıyoruz
    path('', include('blog.urls')),  # Blog URL'lerinizin yapılandırıldığını varsayıyoruz
    path('login/', auth_views.LoginView.as_view(template_name='users/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(template_name='users/logout.html'), name='logout'),
]
```

### Adım 2: Giriş ve Çıkış Şablonları Oluşturma

Sağlanan görünümler kimlik doğrulama mantığını yönetir ancak şablonları içermez. Gerekli şablonları `users/templates/users` dizininde oluşturacağız.

#### login.html

```html
{% extends "base.html" %}
{% block content %}
  <div class="content-section">
    <form method="POST">
      {% csrf_token %}
      <fieldset class="form-group">
        <legend class="border-bottom mb-4">Giriş Yap</legend>
        {{ form|crispy }}
      </fieldset>
      <div class="form-group">
        <button class="btn btn-outline-info" type="submit">Giriş Yap</button>
      </div>
    </form>
    <div class="border-top pt-3">
      <small class="text-muted">
        Hesabınız yok mu? <a href="{% url 'register' %}">Şimdi kaydolun</a>
      </small>
    </div>
  </div>
{% endblock %}
```

#### logout.html

```html
{% extends "base.html" %}
{% block content %}
  <div class="content-section">
    <h2>Çıkış yapıldı</h2>
    <a href="{% url 'login' %}">Tekrar giriş yap</a>
  </div>
{% endblock %}
```

### Adım 3: Giriş Sonrası Yönlendirme

Varsayılan olarak, Django giriş yaptıktan sonra kullanıcıları `/accounts/profile/` adresine yönlendirir. Bunu, kullanıcıları ana sayfaya yönlendirecek şekilde değiştireceğiz.

```python
# settings.py dosyasında
LOGIN_REDIRECT_URL = 'blog-home'  # Ana sayfa URL adınızı buraya yazın
LOGIN_URL = 'login'  # urls.py dosyasındaki login rota adının aynı olduğundan emin olun
```

### Adım 4: Navigasyon Çubuğunu Güncelleme

Navigasyon çubuğunu, kullanıcının kimlik doğrulama durumuna göre farklı bağlantılar gösterecek şekilde değiştirin.

```html
<!-- base.html dosyasında -->
<nav>
  <ul>
    {% if user.is_authenticated %}
      <li><a href="{% url 'logout' %}">Çıkış Yap</a></li>
      <li><a href="{% url 'profile' %}">Profil</a></li>
    {% else %}
      <li><a href="{% url 'login' %}">Giriş Yap</a></li>
      <li><a href="{% url 'register' %}">Kayıt Ol</a></li>
    {% endif %}
  </ul>
</nav>
```

## Profil Sayfası Oluşturma

Kullanıcıların yalnızca giriş yapmışsa erişebileceği bir profil sayfası oluşturacağız.

### Adım 1: Profil Görünümünü Oluşturma

```python
# users uygulamasının views.py dosyasında
from django.contrib.auth.decorators import login_required
from django.shortcuts import render

@login_required
def profile(request):
    return render(request, 'users/profile.html')
```

### Adım 2: Profil Şablonunu Oluşturma

```html
<!-- profile.html -->
{% extends "base.html" %}
{% block content %}
  <div class="content-section">
    <h1>Profil Sayfası</h1>
    <p>Kullanıcı Adı: {{ user.username }}</p>
  </div>
{% endblock %}
```

### Adım 3: Profil URL'si Ekleme

```python
# users/urls.py dosyasında
from django.urls import path
from . import views

urlpatterns = [
    path('profile/', views.profile, name='profile'),
]
```

## Belirli Sayfalara Erişimi Kısıtlama

Profil görünümünde `login_required` dekoratörünü kullanarak, yalnızca giriş yapmış kullanıcıların bu sayfaya erişebilmesini sağladık. Eğer kullanıcı kimlik doğrulaması yapılmamışsa, giriş sayfasına yönlendirilecektir.

### Özet

Bu eğitimde, bir Django uygulaması için kapsamlı bir giriş ve çıkış sistemi uyguladık. Giriş ve çıkış işlemleri için özel şablonlar oluşturduk, kullanıcıları uygun şekilde yönlendirdik, navigasyon çubuğunu kimlik doğrulama durumuna göre güncelledik ve belirli sayfalara erişimi kısıtladık. Bu, Django web uygulamamızda güvenli ve kullanıcı dostu bir kimlik doğrulama deneyimi sağlar.

### Kod Parçacıkları

1. **URL'ler Yapılandırması**:

    ```python
    from django.contrib import admin
    from django.urls import path, include
    from django.contrib.auth import views as auth_views

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('register/', include('users.urls')),
        path('', include('blog.urls')),
        path('login/', auth_views.LoginView.as_view(template_name='users/login.html'), name='login'),
        path('logout/', auth_views.LogoutView.as_view(template_name='users/logout.html'), name='logout'),
    ]
    ```

2. **Ayarlar**:

    ```python
    LOGIN_REDIRECT_URL = 'blog-home'
    LOGIN_URL = 'login'
    ```

3. **Profil Görünümü**:

    ```python
    from django.contrib.auth.decorators import login_required
    from django.shortcuts import render

    @login_required
    def profile(request):
        return render(request, 'users/profile.html')
    ```

4. **Profil URL'si**:

    ```python
    from django.urls import path
    from . import views

    urlpatterns = [
        path('profile/', views.profile, name='profile'),
    ]
    ```

5. **Şablonlar**:

    - **login.html**:

        ```html
        {% extends "base.html" %}
        {% block content %}
          <div class="content-section">
            <form method="POST">
              {% csrf_token %}
              <fieldset class="form-group">
                <legend class="border-bottom mb-4">Giriş Yap</legend>
                {{ form|crispy }}
              </fieldset>
              <div class="form-group">
                <button class="btn btn-outline-info" type="submit">Giriş Yap</button>
              </div>
            </form>
            <div class="border-top pt-3">
              <small class="text-muted">
                Hesabınız yok mu? <a href="{% url 'register' %}">Şimdi kaydolun</a>
              </small>
            </div>
          </div>
        {% endblock %}
        ```

    - **logout.html**:

        ```html
        {% extends "base.html" %}
        {% block content %}
          <div class="content-section">
            <h2>Çıkış yapıldı</h2>
            <a href="{% url 'login' %}">Tekrar giriş yap</a>
          </div>
        {% endblock %}
        ```

    - **profile.html**:

        ```html
        {% extends "base.html" %}
        {% block content %}
          <div class="content-section">
            <h1>Profil Sayfası</h1>
            <p>Kullanıcı Adı: {{ user.username }}</p>
          </div>
        {% endblock %}
        ```

