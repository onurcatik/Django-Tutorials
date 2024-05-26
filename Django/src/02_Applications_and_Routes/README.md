## Uygulamalar ve Yönlendirmeler

### Giriş

Bu derste, Django projemize bir blog uygulaması ekleyerek URL yönlendirmelerini kuracağız. Bu sayede, kullanıcıları web uygulamamızdaki belirli sayfalara yönlendirebileceğiz. Ders, blog uygulamasının oluşturulması, URL yönlendirmelerinin haritalanması ve temel görünümlerin (views) render edilmesini ayrıntılı bir şekilde ele alacaktır.

### Blog Uygulaması Oluşturma

Django projeleri, her biri farklı işlevlerden sorumlu birden fazla uygulama içerebilir. Bu modüler yaklaşım, kod organizasyonunu ve yeniden kullanılabilirliği artırır. Örneğin, tek bir Django projesi, bir blog uygulaması, bir mağaza uygulaması ve daha fazlasını içerebilir.

Mevcut projemiz içinde yeni bir uygulama oluşturmak için proje dizinine (manage.py dosyasının bulunduğu dizin) gidin ve aşağıdaki komutu çalıştırın:

```bash
python manage.py startapp blog
```

Bu komut, blog uygulaması için gerekli dosya ve dizin yapısını oluşturacaktır.

### Dizin Yapısı

Blog uygulamasını oluşturduktan sonra, proje dizininiz aşağıdaki gibi görünmelidir:

```
myproject/
    manage.py
    myproject/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    blog/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
        models.py
        tests.py
        views.py
```

`blog` dizini, uygulamamızın işlevselliği için gerekli dosyaları içerir.

### Görünümler (Views) Oluşturma

Görünümler, Django'da farklı web sayfaları için mantığı (logic) yöneten temel bileşenlerdir. Blogun ana sayfası için basit bir görünüm oluşturacağız.

`blog` dizinindeki `views.py` dosyasını açın ve aşağıdaki kodu ekleyin:

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse('<h1>Blog Ana Sayfası</h1>')
```

Bu kod, bir HTTP yanıtı döndüren `home` adlı bir fonksiyon tanımlar ve basit bir HTML başlığı içerir.

### URL'leri Haritalama

Görünümü bir URL üzerinden erişilebilir hale getirmek için, URL'yi görünüm fonksiyonuna bağlamamız gerekir. `blog` dizininde `urls.py` adlı yeni bir dosya oluşturun ve aşağıdaki kodu ekleyin:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='blog-home'),
]
```

Bu kod, blogun kök URL'sini `home` görünüm fonksiyonuna bağlayan bir URL düzeni oluşturur.

Daha sonra, blog uygulamasının URL yapılandırmalarını projenin ana URL yapılandırmasına dahil etmemiz gerekiyor. Proje dizinindeki `urls.py` dosyasını açın ve şu şekilde düzenleyin:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]
```

Bu, `/blog/` isteklerinin `blog` uygulamasındaki URL desenleri tarafından yönetilmesini sağlar.

### Geliştirme Sunucusunu Çalıştırma

Kurulumumuzu test etmek için aşağıdaki komutla geliştirme sunucusunu çalıştırın:

```bash
python manage.py runserver
```

Tarayıcınızda `http://localhost:8000/blog/` adresine gidin. Sayfada "Blog Ana Sayfası" mesajını görmelisiniz.

### Daha Fazla Görünüm Eklemek

Blogun "Hakkında" sayfası için başka bir görünüm ekleyelim. Öncelikle, `views.py` dosyasına yeni bir görünüm fonksiyonu ekleyin:

```python
def about(request):
    return HttpResponse('<h1>Blog Hakkında</h1>')
```

Ardından, `blog` dizinindeki `urls.py` dosyasını güncelleyerek "Hakkında" sayfası için bir URL deseni ekleyin:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='blog-home'),
    path('about/', views.about, name='blog-about'),
]
```

Geliştirme sunucusunu yeniden başlatın ve `http://localhost:8000/blog/about/` adresine giderek "Hakkında" sayfasının çalıştığını doğrulayın.

### Blogu Ana Sayfa Yapmak

Eğer blogunuzu tüm sitenizin ana sayfası yapmak istiyorsanız, ana URL yapılandırmasını ayarlamanız gerekir. Proje dizinindeki `urls.py` dosyasını şu şekilde güncelleyin:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

Şimdi `http://localhost:8000/` adresine gidildiğinde blogun ana sayfası görüntülenecektir.

### Sonuç

Bu derste, yeni bir blog uygulaması oluşturduk, farklı sayfalar için görünümler tanımladık ve bu görünümleri URL'lerle eşleştirdik. Bu temel kurulum, Django projemizde daha karmaşık özellikler oluşturmaya başlamamız için gereklidir.

Serinin bir sonraki bölümünde, şablonlar kullanarak daha karmaşık HTML içerikleri nasıl döndüreceğimizi ve bu şablonlara değişkenleri nasıl aktaracağımızı öğreneceğiz.

### Kod Parçacıkları

#### Blog Uygulaması Oluşturma

```bash
python manage.py startapp blog
```

#### `views.py` Dosyasında Görünümleri Tanımlama

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse('<h1>Blog Ana Sayfası</h1>')

def about(request):
    return HttpResponse('<h1>Blog Hakkında</h1>')
```

#### `blog/urls.py` Dosyasında URL'leri Haritalama

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='blog-home'),
    path('about/', views.about, name='blog-about'),
]
```

#### Blog URL'lerini `myproject/urls.py` Dosyasına Dahil Etme

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]
```

#### Blogu Ana Sayfa Yapmak

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

Bu adımları izleyerek, Django projenizde işlevsel bir blog uygulaması oluşturmuş olacaksınız ve bu uygulama daha fazla geliştirme ve iyileştirme için hazır olacaktır.