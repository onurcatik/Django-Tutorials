# E-posta ve Şifre Sıfırlama

## Giriş

Bu derste, Django web uygulamamız için bir şifre sıfırlama özelliği uygulayacağız. Django, yalnızca ilgili kullanıcının şifresini sıfırlayabilmesini sağlamak için güvenli belirteçler oluşturma işlevselliğini sağlar. E-posta ile şifre sıfırlama talimatları göndermek için bu yerleşik görünümleri kullanacağız.

## URL Desenlerinin Ayarlanması

İlk olarak, Django tarafından sağlanan gerekli şifre sıfırlama görünümlerini içerecek şekilde URL yapılandırmamızı güncellememiz gerekiyor.

### Adım 1: `urls.py` Dosyasını Güncelleyin

Projenizin `urls.py` dosyasını açın ve aşağıdaki URL desenlerini ekleyin:

```python
from django.urls import path
from django.contrib.auth import views as auth_views

urlpatterns = [
    # Mevcut URL desenleri
    path('admin/', admin.site.urls),
    path('register/', include('users.urls')),

    # Şifre sıfırlama URL desenleri
    path('password-reset/', 
         auth_views.PasswordResetView.as_view(template_name='users/password_reset.html'), 
         name='password_reset'),
    path('password-reset/done/', 
         auth_views.PasswordResetDoneView.as_view(template_name='users/password_reset_done.html'), 
         name='password_reset_done'),
    path('password-reset-confirm/<uidb64>/<token>/', 
         auth_views.PasswordResetConfirmView.as_view(template_name='users/password_reset_confirm.html'), 
         name='password_reset_confirm'),
    path('password-reset-complete/', 
         auth_views.PasswordResetCompleteView.as_view(template_name='users/password_reset_complete.html'), 
         name='password_reset_complete'),
]
```

### Açıklama

1. **PasswordResetView**: Kullanıcıların şifre sıfırlama talebinde bulunmaları için e-posta adreslerini girebilecekleri bir form görüntüler.
2. **PasswordResetDoneView**: Şifre sıfırlama e-postasının gönderildiğini onaylar.
3. **PasswordResetConfirmView**: Kullanıcılara, e-posta yoluyla gönderilen belirteci kullanarak yeni bir şifre belirlemelerine olanak tanır.
4. **PasswordResetCompleteView**: Kullanıcılara şifrelerinin başarıyla sıfırlandığını bildirir.

## Şablonların Oluşturulması

Bir sonraki adımda, her bir görünüm için şablonları oluşturacağız. Bu şablonların `users/templates/users/` dizininde bulunduğundan emin olun.

### Adım 2: `password_reset.html` Dosyasını Oluşturun

`users/templates/users/` dizininde `password_reset.html` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin:

```html
{% extends "base_generic.html" %}
{% block content %}
  <h2>Şifre Sıfırla</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Şifre Sıfırlama Talebi</button>
  </form>
{% endblock %}
```

### Adım 3: `password_reset_done.html` Dosyasını Oluşturun

`users/templates/users/` dizininde `password_reset_done.html` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin:

```html
{% extends "base_generic.html" %}
{% block content %}
  <div class="alert alert-info" role="alert">
    Şifrenizi sıfırlama talimatlarını içeren bir e-posta gönderildi.
  </div>
{% endblock %}
```

### Adım 4: `password_reset_confirm.html` Dosyasını Oluşturun

`users/templates/users/` dizininde `password_reset_confirm.html` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin:

```html
{% extends "base_generic.html" %}
{% block content %}
  <h2>Yeni Şifre Belirle</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Şifreyi Sıfırla</button>
  </form>
{% endblock %}
```

### Adım 5: `password_reset_complete.html` Dosyasını Oluşturun

`users/templates/users/` dizininde `password_reset_complete.html` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin:

```html
{% extends "base_generic.html" %}
{% block content %}
  <div class="alert alert-info" role="alert">
    Şifreniz belirlendi. Şimdi <a href="{% url 'login' %}">giriş yapabilirsiniz</a>.
  </div>
{% endblock %}
```

## E-posta Ayarlarının Yapılandırılması

E-posta gönderebilmek için `settings.py` dosyamızda e-posta altyapısını yapılandırmamız gerekiyor. Bu eğitimde Gmail kullanacağız.

### Adım 6: `settings.py` Dosyasını Güncelleyin

`settings.py` dosyanıza aşağıdaki yapılandırmayı ekleyin:

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PASS')
```

`EMAIL_USER` ve `EMAIL_PASS` ortam değişkenlerinin Gmail kullanıcı adınız ve uygulama parolanızla ayarlandığından emin olun.

### Örnek: Ortam Değişkenlerinin Ayarlanması

Unix sistemlerinde, ortam değişkenlerini `.bashrc` veya `.bash_profile` dosyanızda ayarlayabilirsiniz:

```sh
export EMAIL_USER='your-email@gmail.com'
export EMAIL_PASS='your-email-password'
```

Windows'ta, ortam değişkenlerini Sistem Özellikleri aracılığıyla ayarlayabilirsiniz.

## Şifre Sıfırlama Özelliğini Test Etme

### Adım 7: Geliştirme Sunucusunu Çalıştırın

Geliştirme sunucunuzun çalıştığından emin olun:

```sh
python manage.py runserver
```

### Adım 8: Şifre Sıfırlama Formuna Erişin

Tarayıcınızda `/password-reset/` adresine gidin. E-posta adresinizi girin ve formu gönderin. Şifre sıfırlama bağlantısını içeren bir e-posta almalısınız.

### Adım 9: Şifre Sıfırlama İşlemini Tamamlayın

E-postadaki bağlantıyı takip ederek yeni bir şifre belirleyin. Yeni şifre ile giriş yapabildiğinizi doğrulayın.

## Giriş Sayfasına Şifre Sıfırlama Bağlantısı Ekleme

Kullanıcıların şifrelerini sıfırlamalarını kolaylaştırmak için giriş sayfasına "Şifremi Unuttum?" bağlantısı ekleyeceğiz.

### Adım 10: `login.html` Dosyasını Güncelleyin

`login.html` şablonunu açın ve aşağıdaki bağlantıyı ekleyin:

```html
{% extends "base_generic.html" %}
{% block content %}
  <h2>Giriş Yap</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Giriş Yap</button>
  </form>
  <p class="text-muted"><a href="{% url 'password_reset' %}">Şifremi Unuttum?</a></p>
{% endblock %}
```

## Sonuç

Bu derste, Django web uygulamasında yerleşik görünümler ve şablonları kullanarak şifre sıfırlama özelliğini nasıl uygulayacağımızı gösterdik. Bu adımları takip ederek, kullanıcıların e-posta yoluyla güvenli bir şekilde şifrelerini sıfırlayabilmelerini sağlayabilirsiniz. Bu, modern bir web uygulaması için hem işlevsellik hem de güvenlik sağlayan temel bir özelliktir.