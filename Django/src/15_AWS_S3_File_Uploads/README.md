# Dosya Yüklemeleri için AWS S3 Kullanımı

Bu derste, Django uygulamamızı dosya depolaması için Amazon S3 ile nasıl entegre edeceğimizi inceleyeceğiz. AWS S3 kullanmak, dosya yüklemelerini yönetmek için uygun maliyetli, ölçeklenebilir ve güvenli bir çözüm sunar. Bu eğitim, AWS S3'ü kurma, Django'yu dosya depolama için nasıl yapılandıracağımız ve uygulamamızın dosya yüklemelerini verimli bir şekilde nasıl ele alacağımız konusunda size rehberlik edecektir.

## `models.py` dosyasında düzeltme

Başlamadan önce, `users/models.py` dosyamızdaki `save` metodunda bir düzeltme yapalım. Düzeltme, `save` metodunun uyumlu olmasını sağlamak için pozisyonel ve anahtar kelime argümanlarını kabul etmelidir.

```python
# users/models.py
from django.db import models

class Profile(models.Model):
    # Model alanlarınız
    ...

    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
```

Bu değişiklik, `save` metoduna iletilen herhangi bir rastgele pozisyonel veya anahtar kelime argümanının doğru şekilde işlenmesini sağlar.

## AWS S3'ü Kurma

### Adım 1: AWS Hesabı Oluşturma

Henüz bir AWS hesabınız yoksa, [AWS](https://aws.amazon.com) adresini ziyaret edin ve kaydolun. Bir hesabınız olduğunda, AWS Yönetim Konsolu'na giriş yapın.

### Adım 2: Bir S3 Kovası Oluşturma

1. AWS Yönetim Konsolu'nda S3 hizmetine gidin ve arama çubuğuna "S3" yazın.
2. "Create bucket" (Kova oluştur) butonuna tıklayın ve benzersiz bir kova adı verin (örneğin, `django-blog-files`). Adın tüm AWS hesaplarında benzersiz olması gerektiğini unutmayın.
3. Bir bölge seçin ve varsayılan ayarları kabul edin veya gerektiği gibi özelleştirin. "Create bucket" (Kova oluştur) butonuna tıklayın.

### Adım 3: Kova İzinlerini Yapılandırma

Kovanızın "Permissions" (İzinler) sekmesine gidin ve Cross-Origin Resource Sharing (CORS) ayarlarını yapılandırın. Aşağıdaki yapılandırmayı kullanın:

```json
[
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "POST", "PUT", "DELETE"],
        "AllowedOrigins": ["*"],
        "ExposeHeaders": []
    }
]
```

### Adım 4: Programatik Erişim için IAM Kullanıcısı Oluşturma

1. AWS Yönetim Konsolu'nda IAM hizmetine gidin.
2. "Users" (Kullanıcılar) ve ardından "Add user" (Kullanıcı ekle) butonuna tıklayın.
3. Bir kullanıcı adı verin (örneğin, `django_user`) ve "Programmatic access" (Programatik erişim) seçeneğini işaretleyin.
4. Kullanıcıya `AmazonS3FullAccess` politikasını ekleyin.
5. Kullanıcı oluşturma sürecini tamamlayın ve Access Key ID (Erişim Anahtarı Kimliği) ve Secret Access Key (Gizli Erişim Anahtarı) bilgilerini kaydedin. Bu kimlik bilgileri, uygulamanızın AWS ile kimlik doğrulaması yapması için kullanılacaktır.

## Django'yu S3 Kullanacak Şekilde Yapılandırma

### Adım 5: Gerekli Paketleri Kurma

`boto3` ve `django-storages` paketlerini pip ile kurun:

```bash
pip install boto3 django-storages
```

### Adım 6: Django Ayarlarını Güncelleme

`settings.py` dosyanızda `INSTALLED_APPS` kısmına `storages` ekleyin ve AWS kimlik bilgilerinizi ve S3 kova ayarlarınızı yapılandırın:

```python
# settings.py
import os

INSTALLED_APPS = [
    # diğer yüklü uygulamalar
    'storages',
]

AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')

AWS_S3_FILE_OVERWRITE = False
AWS_DEFAULT_ACL = None
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

### Adım 7: Ortam Değişkenlerini Ayarlama

AWS kimlik bilgilerini ortam değişkenleri olarak ayarlayın. Unix tabanlı bir sistemde, `~/.bash_profile` veya eşdeğerine aşağıdakileri ekleyin:

```bash
export AWS_ACCESS_KEY_ID=your_access_key_id
export AWS_SECRET_ACCESS_KEY=your_secret_access_key
export AWS_STORAGE_BUCKET_NAME=django-blog-files
```

Profil dosyasını kaynak alarak değişiklikleri uygulayın:

```bash
source ~/.bash_profile
```

### Adım 8: `models.py` Dosyasını Değiştirme

Daha önce `save` metodunda resim boyutlandırma işlemi yaptıysanız, bu işlevselliği yorum satırı yapın veya kaldırın, çünkü S3 ile çalışmaz.

```python
# users/models.py
from django.db import models

class Profile(models.Model):
    # Model alanlarınız
    ...

    # Resim boyutlandırma işlemini içeren save metodunu yorum satırı yapın veya kaldırın
    # def save(self, *args, **kwargs):
    #     super().save(*args, **kwargs)
```

### Adım 9: Mevcut Dosyaları S3'e Taşıma

Yerel dosya sisteminizdeki mevcut dosyaları manuel olarak S3 kovasına yükleyin. AWS Yönetim Konsolu'nda kovaya gidin ve `media` klasöründe bulunan dosyaları yükleyin.

## Uygulamayı Çalıştırma

Django geliştirme sunucunuzu başlatın ve dosya yüklemelerinin ve geri alımlarının düzgün çalıştığını doğrulayın:

```bash
python manage.py runserver
```

Resim URL'lerini inceleyerek dosyaların S3 kovasından sunulduğunu kontrol edin.

## Sonuç

Bu eğitimle, Django uygulamanızı dosya depolaması için AWS S3 kullanacak şekilde yapılandırmayı başardınız. Bu yapılandırma, uygulamanızın ölçeklenebilirliğini, güvenliğini ve performansını artırır. Gelecekteki geliştirmeler, otomatik resim boyutlandırma için AWS Lambda işlevlerini uygulamayı içerebilir. 