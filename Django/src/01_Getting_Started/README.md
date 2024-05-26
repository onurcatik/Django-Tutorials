# Başlarken

## Giriş

Bu kapsamlı eğitimde, Django framework'ü ve Python kullanarak tam özellikli bir web uygulamasının nasıl oluşturulacağını öğreneceğiz. Django, kutudan çıktığı anda geniş işlevsellik sunan popüler bir framework'tür ve web geliştirmeyi verimli ve keyifli hale getirir. Kullanıcıların yazı oluşturabildiği, profillerini yönetebildiği ve kimlik doğrulama sistemi ile etkileşime girebildiği bir blog tarzı uygulama oluşturacağız.

## Uygulamanın Genel Görünümü

Oluşturacağımız uygulama şunları içermektedir:
- Kullanıcı kimlik doğrulama sistemi (kayıt, giriş, çıkış, şifre sıfırlama)
- Profil yönetimi (profil bilgilerini ve profil resimlerini güncelleme)
- Yazı yönetimi (yazı oluşturma, güncelleme, silme)
- Arka uç yönetimi için admin arayüzü

Bu eğitim, veritabanlarıyla çalışma, formları kullanma ve e-posta gönderme gibi çeşitli Django özelliklerini kapsayacaktır.

## Ön Koşullar

Başlamadan önce, sisteminizde Python ve pip'in kurulu olduğundan emin olun. Ayrıca, bu proje için bir sanal ortam kullanmanız, bağımlılıkları yönetmek ve diğer projelerle çakışmaları önlemek için önerilir.

## Ortamın Kurulumu

1. **Sanal Ortam Oluşturma**

   Projeniz için yeni bir dizin oluşturun ve bu dizine gidin. Ardından, bir sanal ortam oluşturun:
   ```bash
   python -m venv env
   ```
   Sanal ortamı etkinleştirin:
   - Windows'ta:
     ```bash
     .\env\Scripts\activate
     ```
   - macOS/Linux'ta:
     ```bash
     source env/bin/activate
     ```

2. **Django'yu Yükleme**

   Sanal ortam etkin durumdayken, Django'yu yükleyin:
   ```bash
   pip install django
   ```

   Kurulumu doğrulayın:
   ```bash
   python -m django --version
   ```
   Django'nun 2.1 veya daha yüksek bir sürümüne sahip olduğunuzdan emin olun.

## Bir Django Projesi Oluşturma

1. **Yeni Bir Django Projesi Başlatma**

   `django-admin` komutunu kullanarak yeni bir Django projesi oluşturun:
   ```bash
   django-admin startproject django_project
   ```

   Proje dizinine gidin:
   ```bash
   cd django_project
   ```

2. **Proje Yapısı**

   `startproject` komutu temel bir proje yapısı oluşturur:

   ```
   django_project/
       manage.py
       django_project/
           __init__.py
           settings.py
           urls.py
           wsgi.py
   ```

   - `manage.py`: Django projenizle etkileşimde bulunmanızı sağlayan bir komut satırı aracıdır.
   - `settings.py`: Proje için yapılandırma ayarlarını içerir.
   - `urls.py`: URL bildirimlerini içerir.
   - `wsgi.py`: Projeyi WSGI uyumlu web sunucularında dağıtmak için kullanılır.

## Geliştirme Sunucusunu Çalıştırma

1. **Sunucuyu Çalıştırma**

   Django geliştirme sunucusunu başlatın:
   ```bash
   python manage.py runserver
   ```

   Sunucunun çalıştığını ve `http://127.0.0.1:8000/` adresinden erişilebileceğini belirten bir çıktı görmelisiniz.

2. **Varsayılan Django Sayfasına Erişme**

   Web tarayıcınızı açın ve `http://127.0.0.1:8000/` adresine gidin. Varsayılan Django karşılama sayfasını görmelisiniz.

3. **Yönetim Arayüzü**

   Yönetim arayüzüne erişmek için `http://127.0.0.1:8000/admin/` adresine gidin. Bunu sonraki adımlarda yapılandıracağız.

## Proje Yapısını Anlama

1. **Manage.py**

   `manage.py` betiği, Django projenizle etkileşime girmenizi sağlayan bir komut satırı aracıdır. Sunucuyu başlatmak, uygulamalar oluşturmak ve diğer görevleri yapmak için kullanabilirsiniz.

2. **Settings.py**

   `settings.py` dosyası, Django projeniz için tüm yapılandırmayı içerir. Önemli ayarlar arasında:
   - `SECRET_KEY`: Kriptografik imzalama için gizli bir anahtar.
   - `DEBUG`: Hata ayıklama modunu açan/kapatabilen bir boolean.
   - `INSTALLED_APPS`: Yüklü uygulamaların bir listesi.

3. **Urls.py**

   `urls.py` dosyası, URL desenlerini ve eşlemeleri tanımlar. Varsayılan olarak, yönetim arayüzü için URL desenini içerir.

4. **Wsgi.py**

   `wsgi.py` dosyası, Django projesini WSGI uyumlu web sunucularında dağıtmak için kullanılır.

## Sonuç

Bu bölümde, geliştirme ortamını kurduk, Django'yu yükledik, yeni bir proje oluşturduk ve proje yapısını inceledik. Bir sonraki bölümde, projemiz içinde bir uygulama oluşturmayı ve temel rotaları kurmayı öğreneceğiz.

Bir sonraki adımlarda Django uygulamamızı oluşturmaya devam edeceğiz. Herhangi bir sorunuz veya geri bildiriminiz varsa, lütfen bize ulaşın.

## Kod Parçacıkları

Bu eğitimde kullanılan bazı önemli komutlar:

- Sanal ortam oluşturma:
  ```bash
  python -m venv env
  ```
- Sanal ortamı etkinleştirme:
  - Windows'ta:
    ```bash
    .\env\Scripts\activate
    ```
  - macOS/Linux'ta:
    ```bash
    source env/bin/activate
    ```
- Django'yu yükleme:
  ```bash
  pip install django
  ```
- Yeni bir Django projesi başlatma:
  ```bash
  django-admin startproject django_project
  ```
- Geliştirme sunucusunu çalıştırma:
  ```bash
  python manage.py runserver
  ```

Bir sonraki eğitimde, temel rotaları kurarak ve bir uygulama oluşturarak tam özellikli web uygulamamızı oluşturmaya devam edeceğiz. Herhangi bir sorunuz veya geri bildiriminiz varsa, lütfen ulaşmaktan çekinmeyin.