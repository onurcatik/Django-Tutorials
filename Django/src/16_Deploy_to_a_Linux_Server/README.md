# Bir Linux Sunucusuna Dağıtma

Bu eğitimde, bir Django uygulamasını bir Linux sunucusuna nasıl dağıtacağınızı ayrıntılı ve kapsamlı bir şekilde ele alacağız. Bu süreç, sunucunuzu yapılandırmaktan uygulamanızı güvence altına almaya kadar birçok adımı içerir. Linode tarafından barındırılan sanal özel sunucuyu (VPS) kullanacağız, ancak talimatlar DigitalOcean veya AWS gibi diğer sağlayıcılar için de geçerlidir.

## İçindekiler
1. Ön Gereksinimler
2. Sunucuyu Kurma
3. Sunucuyu Yapılandırma
4. SSH Anahtar Tabanlı Kimlik Doğrulama Kurma
5. Apache ve mod_wsgi Kurulumu ve Yapılandırması
6. Django Uygulamasını Hazırlama
7. Django'yu Üretim İçin Yapılandırma
8. Uygulamayı Dağıtma
9. Dağıtım Sonrası Adımlar
10. Sonuç

## 1. Ön Gereksinimler
Başlamadan önce, aşağıdakilere sahip olduğunuzdan emin olun:
- Dağıtıma hazır bir Django uygulaması.
- SSH erişimi olan bir VPS (Linode, DigitalOcean, AWS, vb.).
- Temel Linux komutları ve dosya izinleri bilgisi.
- Yerel makinenizde yüklü bir SSH istemcisi.

## 2. Sunucuyu Kurma
Öncelikle, Linux sunucunuzu oluşturun ve yapılandırın. Bu eğitim için Ubuntu 20.04 kullanacağız.

### Bir VPS Oluşturma
1. VPS sağlayıcınıza giriş yapın ve yeni bir sunucu örneği oluşturun.
2. İşletim sistemi olarak Ubuntu 20.04'ü seçin.
3. Uygulamanızın gereksinimlerine göre istediğiniz planı seçin.
4. Sunucu için bir root şifresi belirleyin.

### Sunucuya SSH ile Erişim
```bash
ssh root@your_server_ip
```
`your_server_ip` yerine sunucunuzun IP adresini girin.

### Sunucuyu Güncelleme
```bash
apt-get update
apt-get upgrade -y
```

## 3. Sunucuyu Yapılandırma
### Hostname Ayarlama
```bash
hostnamectl set-hostname your_hostname
```
`your_hostname` yerine istediğiniz bir isim girin.

### Sınırlı Kullanıcı Ekleme
```bash
adduser your_username
usermod -aG sudo your_username
```
`your_username` yerine istediğiniz kullanıcı adını girin.

### Yeni Kullanıcıya Geçiş
```bash
exit
ssh your_username@your_server_ip
```

## 4. SSH Anahtar Tabanlı Kimlik Doğrulama Kurma
### SSH Anahtarlarını Oluşturma (yerel makinede)
```bash
ssh-keygen -b 4096
```

### Genel Anahtarı Sunucuya Kopyalama
```bash
ssh-copy-id your_username@your_server_ip
```

### Root Girişi ve Şifre Kimlik Doğrulamasını Devre Dışı Bırakma
SSH yapılandırma dosyasını düzenleyin:
```bash
sudo nano /etc/ssh/sshd_config
```
Aşağıdakileri ayarlayın:
```
PermitRootLogin no
PasswordAuthentication no
```
SSH hizmetini yeniden başlatın:
```bash
sudo systemctl restart ssh
```

## 5. Apache ve mod_wsgi Kurulumu ve Yapılandırması
### Apache ve mod_wsgi Kurulumu
```bash
sudo apt-get install apache2 libapache2-mod-wsgi-py3 -y
```

### Django için Apache'yi Yapılandırma
Django projeniz için yeni bir yapılandırma dosyası oluşturun:
```bash
sudo nano /etc/apache2/sites-available/your_project.conf
```
Aşağıdaki yapılandırmayı ekleyin:
```apache
<VirtualHost *:80>
    ServerAdmin admin@your_domain.com
    ServerName your_domain.com
    ServerAlias www.your_domain.com

    DocumentRoot /var/www/your_project

    Alias /static /var/www/your_project/static
    <Directory /var/www/your_project/static>
        Require all granted
    </Directory>

    Alias /media /var/www/your_project/media
    <Directory /var/www/your_project/media>
        Require all granted
    </Directory>

    <Directory /var/www/your_project/your_project>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>

    WSGIDaemonProcess your_project python-path=/var/www/your_project python-home=/var/www/your_project/venv
    WSGIProcessGroup your_project
    WSGIScriptAlias / /var/www/your_project/your_project/wsgi.py
</VirtualHost>
```
Siteyi etkinleştirin ve Apache'yi yeniden başlatın:
```bash
sudo a2ensite your_project.conf
sudo systemctl restart apache2
```

## 6. Django Uygulamasını Hazırlama
### Sanal Ortam Oluşturma
```bash
cd /var/www/your_project
python3 -m venv venv
source venv/bin/activate
```

### Bağımlılıkları Yükleme
```bash
pip install -r requirements.txt
```

### Statik Dosyaları Toplama
```bash
python manage.py collectstatic
```

## 7. Django'yu Üretim İçin Yapılandırma
### settings.py Dosyasını Güncelleme
- `DEBUG = False` olarak ayarlayın.
- Sunucunuzun IP adresini `ALLOWED_HOSTS` listesine ekleyin.
- Statik ve medya dosyalarını yapılandırın:
  ```python
  STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
  MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
  ```

### Hassas Bilgileri Güvenceye Alma
Hassas bilgileri ortam değişkenlerinde veya güvenli bir yapılandırma dosyasında saklayın.

### Veritabanını Yapılandırma
Veritabanı ayarlarının üretim için yapılandırıldığından emin olun.

## 8. Uygulamayı Dağıtma
### Veritabanını Göç Ettirme
```bash
python manage.py migrate
```

### Süper Kullanıcı Oluşturma
```bash
python manage.py createsuperuser
```

## 9. Dağıtım Sonrası Adımlar
### Güvenlik Duvarını Yapılandırma
```bash
sudo ufw allow 'Apache Full'
sudo ufw enable
```

### HTTPS'i Etkinleştirme (İsteğe Bağlı)
Let's Encrypt kullanarak HTTPS'i etkinleştirin:
```bash
sudo apt-get install certbot python3-certbot-apache -y
sudo certbot --apache
```

## 10. Sonuç
Django uygulamanızı başarıyla bir Linux sunucusuna dağıttınız. Bu kurulum, uygulamanızın çalıştırılması için sağlam bir ortam sağlar ve sunucu yapılandırmanızı yönetme esnekliği sunar.

Daha fazla optimizasyon ve güvenlik için, bir yük dengeleyici yapılandırma, otomatik yedekleme ayarlama ve sunucu performansını izleme gibi ek önlemler düşünün. En iyi uygulamaları izleyerek, uygulamanızın güvenli, ölçeklenebilir ve sürdürülebilir olmasını sağlarsınız.