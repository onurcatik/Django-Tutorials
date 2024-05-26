# Şablonlar

## Giriş

Bu eğitimde, Django şablonlarını kullanarak daha karmaşık HTML kodu döndürmeyi ve bu şablonlara değişkenler geçmeyi öğreneceğiz. Bu süreç, HTML yapısını Python kodundan ayırarak web uygulamanızın düzenini ve sürdürülebilirliğini artırır.

## Şablon Dizini Kurulumu

Öncelikle, Django uygulamanızın dizininde bir `templates` dizini olduğundan emin olun. Bu `templates` dizini içinde, uygulamanızla aynı adı taşıyan başka bir alt dizin oluşturun. Bu yapı, Django'nun şablonlarınızı tanıyıp bulmasını sağlar.

### Dizin Yapısı:
```
proje_adi/
│
├── uygulama_adi/
│   ├── templates/
│   │   └── uygulama_adi/
│   │       ├── home.html
│   │       └── about.html
```

## Şablonların Oluşturulması

### Ana Sayfa Şablonu (`home.html`)

`home.html` dosyasını `uygulama_adi/templates/uygulama_adi/` dizininde oluşturun. İşte minimal bir örnek:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Django Blog - Ana Sayfa</title>
</head>
<body>
    <h1>Blog Ana Sayfa!</h1>
</body>
</html>
```

### Hakkında Şablonu (`about.html`)

Benzer şekilde `about.html` dosyasını oluşturun:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Django Blog - Hakkında</title>
</head>
<body>
    <h1>Hakkında Sayfası!</h1>
</body>
</html>
```

## Yüklenen Uygulamalar (Installed Apps) Yapılandırması

Uygulamanızı, projenizin `settings.py` dosyasındaki `INSTALLED_APPS` listesine ekleyin. Bu adım, Django'nun şablonlarınızı tanıması için gereklidir.

```python
# proje_adi/settings.py

INSTALLED_APPS = [
    ...
    'uygulama_adi.apps.UygulamaAdiConfig',
]
```

## Görünümlerde (Views) Şablonları İşleme

Görünümlerinizi, basit HTTP yanıtları döndürmek yerine şablonları işleyecek şekilde güncelleyin.

### Görünümler (`views.py`)

```python
from django.shortcuts import render

def home(request):
    return render(request, 'uygulama_adi/home.html')

def about(request):
    return render(request, 'uygulama_adi/about.html')
```

## Şablonlara Bağlam (Context) Geçme

Verileri şablonlara `render` fonksiyonunun `context` parametresi aracılığıyla geçebilirsiniz.

### Bağlam ile Örnek

`home` görünümünü bağlam verileri içerecek şekilde değiştirin:
```python
from django.shortcuts import render

posts = [
    {
        'author': 'CoreyMS',
        'title': 'Blog Post 1',
        'content': 'İlk gönderi içeriği',
        'date_posted': '27 Ağustos 2018'
    },
    {
        'author': 'Jane Doe',
        'title': 'Blog Post 2',
        'content': 'İkinci gönderi içeriği',
        'date_posted': '28 Ağustos 2018'
    }
]

def home(request):
    context = {
        'posts': posts
    }
    return render(request, 'uygulama_adi/home.html', context)
```

### Bağlam Değişkenleri ile Şablon

`home.html` dosyasını gönderileri gösterecek şekilde güncelleyin:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Django Blog - Ana Sayfa</title>
</head>
<body>
    <h1>Blog Ana Sayfa!</h1>
    {% for post in posts %}
        <article>
            <h2>{{ post.title }}</h2>
            <p>yazan {{ post.author }} tarihinde {{ post.date_posted }}</p>
            <div>{{ post.content }}</div>
        </article>
    {% endfor %}
</body>
</html>
```

## Şablon Mirası (Template Inheritance)

Yinelemeyi önlemek için, diğer şablonların miras alabileceği bir ana şablon oluşturun.

### Ana Şablon (`base.html`)

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Django Blog{% endblock %}</title>
    <link rel="stylesheet" type="text/css" href="{% static 'uygulama_adi/main.css' %}">
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="{% url 'home' %}">Ana Sayfa</a></li>
                <li><a href="{% url 'about' %}">Hakkında</a></li>
            </ul>
        </nav>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```


### Ana Şablonu Genişletme

`home.html` ve `about.html` dosyalarını `base.html` şablonunu genişletecek şekilde güncelleyin.

#### `home.html`

```html
{% extends "uygulama_adi/base.html" %}
{% block title %}Ana Sayfa{% endblock %}
{% block content %}
<h1>Blog Ana Sayfa!</h1>
{% for post in posts %}
    <article>
        <h2>{{ post.title }}</h2>
        <p>yazan {{ post.author }} tarihinde {{ post.date_posted }}</p>
        <div>{{ post.content }}</div>
    </article>
{% endfor %}
{% endblock %}
```

#### `about.html`

```html
{% extends "uygulama_adi/base.html" %}
{% block title %}Hakkında{% endblock %}
{% block content %}
<h1>Hakkında Sayfası!</h1>
{% endblock %}
```

## Sonuç

Bu eğitimde, Django şablonlarını kullanarak HTML işlemenin ve şablonlara veri geçirmenin nasıl yapıldığını gösterdik. Ayrıca, şablon mirasının nasıl kullanıldığını ve bunun kodun bakımını nasıl kolaylaştırdığını inceledik. Serinin bir sonraki bölümünde, Django yönetici arayüzünü nasıl kullanacağımızı ve bu arayüzün veritabanı yönetimini nasıl kolaylaştırdığını öğreneceğiz.