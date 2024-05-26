# Heroku Kullanarak Dağıtım

Bu eğitimde, Django uygulamanızı internet üzerinde erişilebilir hale getirmek için Heroku'ya nasıl dağıtacağınızı ele alacağız. Bu kılavuz, Django uygulamanızı Heroku'ya verimli bir şekilde dağıtmanız için gereken adımları detaylı bir şekilde açıklayacaktır.

## Önkoşullar

Başlamadan önce aşağıdakilere sahip olduğunuzdan emin olun:

1. **Heroku Hesabı**: [Heroku](https://signup.heroku.com/) adresinden kaydolun.
2. **Git Sürüm Kontrolü**: Git'in yüklü olduğundan emin olun. Gerekirse [Git kurulum kılavuzu](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)'na bakın.
3. **Heroku CLI**: Heroku Komut Satırı Arayüzünü yükleyin. Talimatlar için [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) sayfasını takip edin.

## Adım 1: Heroku Hesabı Oluşturma

Heroku web sitesinde bir hesap oluşturun. Kaydolduktan sonra, aşağıdaki komutu çalıştırarak Heroku CLI üzerinden oturum açtığınızdan emin olun:
```bash
heroku login
```

## Adım 2: Django Projenizi Hazırlayın

Django proje dizininize gidin ve sanal bir ortam kurduğunuzdan emin olun. Gerekli bağımlılıkları yükleyin ve bunları listelemek için bir `requirements.txt` dosyası oluşturun:
```bash
pip install gunicorn
pip freeze > requirements.txt
```

## Adım 3: Git'i Başlatın ve `.gitignore` Dosyası Oluşturun

Proje dizininizde bir Git deposu başlatın ve bir `.gitignore` dosyası oluşturun:
```bash
git init
echo "venv/" > .gitignore
echo "*.pyc" >> .gitignore
echo "__pycache__/" >> .gitignore
echo "db.sqlite3" >> .gitignore
```

## Adım 4: Heroku Uygulaması Oluşturun

Yeni bir Heroku uygulaması oluşturun:
```bash
heroku create your-app-name
```

## Adım 5: Heroku İçin Ayarları Yapılandırma

Django `settings.py` dosyanızı gerekli yapılandırmaları içerecek şekilde değiştirin:

1. **Statik Dosyalar**:
    ```python
    import os

    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    STATIC_URL = '/static/'

    # Dizin var olduğundan emin olun
    os.makedirs(STATIC_ROOT, exist_ok=True)
    ```

2. **İzin Verilen Hostlar**:
    ```python
    ALLOWED_HOSTS = ['your-app-name.herokuapp.com']
    ```

3. **Veritabanı Yapılandırması**:
    `dj-database-url` paketini yükleyin ve `requirements.txt` dosyanıza ekleyin:
    ```bash
    pip install dj-database-url
    pip freeze > requirements.txt
    ```
    `settings.py` dosyasını Heroku tarafından sağlanan `DATABASE_URL` ortam değişkenini kullanacak şekilde değiştirin:
    ```python
    import dj_database_url

    DATABASES = {
        'default': dj_database_url.config(default='sqlite:///db.sqlite3')
    }
    ```

## Adım 6: `Procfile` Oluşturma

Proje dizininizin köküne bir `Procfile` oluşturun:
```bash
echo "web: gunicorn your_project_name.wsgi" > Procfile
```

## Adım 7: Heroku'ya Dağıtım

Kodunuzu Heroku'ya ekleyin, işleyin ve gönderin:
```bash
git add .
git commit -m "Initial commit"
git push heroku master
```

## Adım 8: Veritabanını Göç Ettirin

Heroku'da göçleri çalıştırın:
```bash
heroku run python manage.py migrate
```

## Adım 9: Süper Kullanıcı Oluşturun

Django yönetici arayüzüne erişim sağlamak için bir süper kullanıcı oluşturun:
```bash
heroku run python manage.py createsuperuser
```

## Adım 10: Hata Ayıklama ve Ortam Değişkenleri

1. **Debug Ayarı**:
    Üretimde `DEBUG`'u `False` olarak ayarlayın. Bunu yönetmek için ortam değişkenlerini kullanın:
    ```python
    DEBUG = os.environ.get('DEBUG', 'False') == 'True'
    ```

2. **Gizli Anahtar**:
    Gizli anahtarınızı bir ortam değişkenine taşıyın:
    ```python
    SECRET_KEY = os.environ.get('SECRET_KEY', 'your-default-secret-key')
    ```
    Bu değişkenleri Heroku'da ayarlayın:
    ```bash
    heroku config:set SECRET_KEY=your-secret-key
    heroku config:set DEBUG=False
    ```

## Adım 11: Dağıtımı Kontrol Edin

Uygulamanızı bir web tarayıcısında açın:
```bash
heroku open
```

## Ek Notlar

- **Statik Dosyalar**: Heroku her dağıtımda dosya sistemini temizler. Kullanıcı tarafından yüklenen dosyalar için AWS S3 gibi bir depolama hizmeti kullanın.
- **Özel Alan Adları**: Özel bir alan adı için [Heroku özel alan adları belgelerine](https://devcenter.heroku.com/articles/custom-domains) bakın.

## Sonuç

Django uygulamanızı Heroku'ya dağıtmak, sunucu yönetiminin karmaşıklıklarını ortadan kaldırarak dağıtım sürecini basitleştirir. Bu adımları izleyerek, uygulamanız artık Heroku'da canlı olmalıdır ve kullanıcılar tarafından erişilebilir hale gelmelidir.

Daha fazla bilgi ve gelişmiş yapılandırmalar için [Heroku belgelerine](https://devcenter.heroku.com/) ve [Django dağıtım kontrol listesine](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/) başvurun.

## Örnek Kod Parçacıkları

Süreç boyunca size yardımcı olacak bazı örnek kod parçacıkları:

### Procfile
```plaintext
web: gunicorn your_project_name.wsgi
```

### .gitignore
```plaintext
venv/
*.pyc
__pycache__/
db.sqlite3
```

### settings.py (Veritabanı Yapılandırması)
```python
import dj_database_url

DATABASES = {
    'default': dj_database_url.config(default='sqlite:///db.sqlite3')
}
```

### settings.py (Statik Dosyalar Yapılandırması)
```python
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = '/static/'

# Dizin var olduğundan emin olun
os.makedirs(STATIC_ROOT, exist_ok=True)
```

Django uygulamanızı Heroku'ya dağıtmak oldukça basit bir süreçtir ve bu eğitimi izleyerek çalışır durumda bir dağıtım yapmanız mümkündür. Herhangi bir sorunla karşılaşırsanız, Heroku belgelerine başvurun veya topluluk forumlarında sorular sorun.