# Python Django Eğitimi: Tam Özellikli Web Uygulaması Bölüm 5 - Veritabanı ve Geçişler

## Giriş

Bu eğitimde, uygulamamız için kendi veritabanı tablolarımızı oluşturarak sahte veriler yerine gerçek gönderiler oluşturmayı ele alacağız. Django, veritabanımızla nesne yönelimli bir şekilde etkileşimde bulunmamızı sağlayan yerleşik bir Nesne İlişkisel Eşleyici (ORM) sağlar. Bu eğitimde modellerin tanımlanması, geçişlerin çalıştırılması ve veritabanı sorgulama konuları ele alınacaktır.

## Nesne İlişkisel Eşleyici (ORM) Genel Bakış

Nesne İlişkisel Eşleyici (ORM), geliştiricilerin veritabanıyla programlama dillerinin nesne yönelimli paradigmasını kullanarak etkileşime girmesini sağlayan bir kütüphanedir. Bir ORM kullanmanın ana avantajı, veritabanı etkileşimlerini soyutlaması ve geliştiricilerin uygulama kodunu değiştirmeden farklı veritabanları arasında geçiş yapabilmesidir. Django'da, ORM veritabanı yapımızı Python sınıfları olarak tanımlamamıza olanak tanır.

### Django ORM'in Avantajları

1. **Veritabanı Bağımsızlığı**: Farklı veritabanları arasında kod tabanını değiştirmeden kolayca geçiş yapma.
2. **Otomatik Tablo Oluşturma**: Veritabanı tablolarını Python sınıfları olarak tanımlama.
3. **Sorgu Soyutlama**: Karmaşık veritabanı sorgularını ham SQL yerine Python kodu kullanarak gerçekleştirme.
4. **Model İlişkileri**: Modeller arasında (örneğin, bire-çok, çok-çok) ilişkiler tanımlama kolaylığı.

## Modellerin Tanımlanması

Django'da modeller Python sınıfları olarak tanımlanır. Her sınıf veritabanında bir tabloyu temsil eder ve sınıfın her bir özniteliği o tablodaki bir sütunu temsil eder. Django, bu modellere dayanarak gerekli SQL ifadelerini otomatik olarak oluşturur.

### Örnek: Post Modelinin Tanımlanması

Blog uygulamamız için bir `Post` modeli tanımlayarak başlayalım. Bu model, başlık, içerik, gönderim tarihi ve yazar için alanlar içerecektir.

1. Blog uygulama dizininizdeki `models.py` dosyasını açın.
2. `Post` modelini şu şekilde tanımlayın:

```python
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    date_posted = models.DateTimeField(default=timezone.now)
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.title
```

### Açıklama

- **title**: Maksimum uzunluğu 100 karakter olan bir karakter alanı.
- **content**: Gönderi içeriği için daha uzun metin girişlerine izin veren bir metin alanı.
- **date_posted**: Varsayılan değeri mevcut tarih ve saat olan bir tarih-saat alanı.
- **author**: `User` modeline bağlanan bir yabancı anahtar. `on_delete=models.CASCADE` argümanı, bir kullanıcı silindiğinde o kullanıcının gönderilerinin de silinmesini sağlar.
- **__str__ metodu**: Model örneklerini okunabilir bir formatta görüntülemek için kullanılan modelin string temsilini tanımlar.

## Geçişlerin Çalıştırılması

Modellerimizi tanımladıktan sonra, veritabanı şemasını güncellemek için geçişler oluşturmalı ve uygulamalıyız.

### Geçişlerin Oluşturulması

1. Komut satırını açın ve proje dizininize gidin.
2. Geçişleri oluşturmak için şu komutu çalıştırın:

```bash
python manage.py makemigrations
```

Bu komut, `Post` tablosunu oluşturmak için gerekli SQL ifadelerini içeren bir geçiş dosyası oluşturur.

### Geçişlerin Uygulanması

Geçişleri uygulamak ve veritabanı şemasını güncellemek için şu komutu çalıştırın:

```bash
python manage.py migrate
```

Bu komut, geçiş dosyasındaki SQL ifadelerini çalıştırarak `Post` tablosunu veritabanında oluşturur.

## Veritabanını Sorgulama

Modellerimizi tanımlayıp geçişleri uyguladıktan sonra, Django ORM kullanarak veritabanını sorgulayabiliriz.

### Django Shell Kullanımı

Django shell, veritabanı modelleriyle etkileşime geçmemizi sağlayan interaktif bir ortamdır.

1. Komut satırını açın ve Django shell'i başlatmak için şu komutu çalıştırın:

```bash
python manage.py shell
```

2. Gerekli modelleri içe aktarın:

```python
from blog.models import Post
from django.contrib.auth.models import User
```

### Gönderi Oluşturma ve Sorgulama

Yeni bir gönderi oluşturup veritabanını sorgulayalım.

1. Yeni bir kullanıcı oluşturun (kullanıcı modelinin boş olmadığını varsayarak):

```python
user = User.objects.create_user(username='testuser', password='password123')
```

2. Yeni bir gönderi oluşturun:

```python
post = Post(title='My First Post', content='This is the content of my first post', author=user)
post.save()
```

3. Tüm gönderileri sorgulayın:

```python
Post.objects.all()
```

### Belirli Gönderilerin Filtrelenmesi ve Getirilmesi

Django ORM tarafından sağlanan çeşitli sorgu yöntemlerini kullanarak belirli gönderileri filtreleyebilir ve getirebilirsiniz.

1. İlk gönderiyi getirin:

```python
Post.objects.first()
```

2. Yazara göre gönderileri filtreleyin:

```python
Post.objects.filter(author=user)
```

3. Birincil anahtara (ID) göre bir gönderi getirin:

```python
Post.objects.get(id=1)
```

## Yönetim Arayüzü

`Post` modelini Django yönetim arayüzü aracılığıyla yönetmek için `admin.py` dosyasında kaydetmemiz gerekiyor.

1. Blog uygulama dizininizdeki `admin.py` dosyasını açın.
2. `Post` modelini kaydedin:

```python
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

Modeli kaydettikten sonra, Django yönetim arayüzünü kullanarak gönderileri ekleyebilir, düzenleyebilir ve silebilirsiniz.

## Sonuç

Bu eğitimde, Django'da modellerin nasıl tanımlanacağını, geçişlerin nasıl çalıştırılacağını ve Django ORM kullanarak veritabanının nasıl sorgulanacağını öğrendik. Ayrıca, `Post` modelini Django yönetim arayüzü aracılığıyla nasıl yöneteceğimizi de inceledik. Django'nun ORM'sini kullanarak veritabanı etkileşimlerini verimli bir şekilde yönetebilir ve uygulama mantığımızı geliştirmeye odaklanabiliriz.

