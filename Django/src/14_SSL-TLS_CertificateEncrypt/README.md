# Python Django Eğitimi: Let's Encrypt ile Ücretsiz SSL/TLS Sertifikası Kullanarak HTTPS Etkinleştirme

## Giriş

Web sunucunuzu HTTPS ile güvence altına almak, istemciler ve sunucular arasında iletilen verilerin korunması için çok önemlidir. Bu eğitim, Let's Encrypt'ten ücretsiz bir SSL/TLS sertifikası kullanarak Django uygulamanızda HTTPS'yi etkinleştirmenin ayrıntılı adım adım kılavuzunu sunmaktadır. Gerekli yapılandırmaları ve komutları, web sitenizin güvenliğini sağlamak için ele alacağız.

## Ön Koşullar

Devam etmeden önce aşağıdaki gereksinimlere sahip olduğunuzdan emin olun:
1. Apache web sunucusunda çalışan bir Django uygulaması.
2. Sunucunuza SSH erişimi.
3. Sunucunuzun IP adresine işaret eden kayıtlı bir alan adı.

Bu eğitimde Ubuntu 18.04 ve Apache web sunucusunu kullandığınız varsayılmaktadır.

## Adım 1: Sunucunuza SSH ile Erişim

Öncelikle, sunucunuza SSH ile bağlanın. Terminalinizi açın ve şu komutu çalıştırın:

```bash
ssh kullanici_adiniz@sunucu_ip_adresiniz
```

`kullanici_adiniz` ve `sunucu_ip_adresiniz` değerlerini kendi kullanıcı adınız ve sunucu IP adresiniz ile değiştirin.

## Adım 2: Certbot'un Kurulumu

Certbot, Let's Encrypt'ten SSL/TLS sertifikalarını almayı ve dağıtmayı kolaylaştıran bir istemcidir. Certbot ve Apache eklentisini aşağıdaki komutlarla kurun:

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-apache
```

## Adım 3: Apache Yapılandırmasının Güncellenmesi

Certbot'u çalıştırmadan önce, alan adınızı içerecek şekilde Apache yapılandırma dosyanızı güncelleyin.

Django projeniz için Apache yapılandırma dosyasını açın:

```bash
sudo nano /etc/apache2/sites-available/your_project.conf
```

`ServerName` yönergesinin alan adınıza ayarlandığından emin olun:

```apache
<VirtualHost *:80>
    ServerName your_domain.com
    # Diğer yapılandırmalar
</VirtualHost>
```

Certbot işlemi sırasında çakışmaları önlemek için geçici olarak `WSGI` yönergelerini yorum satırı haline getirin:

```apache
# WSGIDaemonProcess your_project python-path=/path/to/your_project python-home=/path/to/virtualenv
# WSGIProcessGroup your_project
# WSGIScriptAlias / /path/to/your_project/your_project/wsgi.py
```

Dosyayı kaydedip kapatın (`Ctrl + X`, ardından `Y` ve `Enter`).

## Adım 4: SSL/TLS Sertifikasını Alma ve Kurma

Certbot'u çalıştırarak sertifikayı alın ve kurun:

```bash
sudo certbot --apache
```

Certbot sizden e-posta adresinizi girmenizi ve hizmet koşullarını kabul etmenizi isteyecektir. Ayrıca, HTTP trafiğini HTTPS'ye yönlendirip yönlendirmeyeceğinizi de soracaktır. Tüm trafiği yönlendirmeyi seçin, böylece tüm bağlantılar güvenli hale gelir.

Certbot, Apache yapılandırmanızı değiştirir ve yeni bir SSL yapılandırma dosyası oluşturur.

## Adım 5: Yapılandırmanın Sonlandırılması

Certbot tamamlandıktan sonra, SSL yapılandırma dosyanızdaki `WSGI` yönergelerinin yorumlarını kaldırın:

```bash
sudo nano /etc/apache2/sites-available/your_project-le-ssl.conf
```

`WSGI` yönergelerinin yorumlarını kaldırın:

```apache
WSGIDaemonProcess your_project python-path=/path/to/your_project python-home=/path/to/virtualenv
WSGIProcessGroup your_project
WSGIScriptAlias / /path/to/your_project/your_project/wsgi.py
```

Dosyayı kaydedip kapatın.

## Adım 6: Apache'yi Yeniden Başlatma

Değişikliklerin uygulanması için Apache'yi yeniden başlatın:

```bash
sudo service apache2 restart
```

## Adım 7: HTTPS Yapılandırmasını Doğrulama

Tarayıcınızı açın ve `https://` kullanarak alan adınıza gidin. Adres çubuğunda bir asma kilit simgesi olup olmadığını kontrol ederek bağlantınızın güvenli olduğunu doğrulayın.

## Adım 8: Sertifika Yenileme İşleminin Otomatikleştirilmesi

Let's Encrypt sertifikaları 90 gün geçerlidir. Yenileme sürecini otomatik hale getirmek için bir cron görevi ekleyin:

Crontab dosyasını düzenleyin:

```bash
sudo crontab -e
```

Aşağıdaki satırı ekleyerek yenileme komutunu aylık olarak çalıştırın:

```crontab
30 4 1 * * /usr/bin/certbot renew --quiet
```

Dosyayı kaydedip kapatın.

## Sonuç

Bu adımları izleyerek, Django uygulamanızı Let's Encrypt'ten ücretsiz bir SSL/TLS sertifikası kullanarak HTTPS ile güvence altına aldınız. Bu, istemcileriniz ve sunucunuz arasındaki tüm verilerin şifrelenmesini sağlayarak kullanıcılarınıza güvenli bir tarama deneyimi sunar.

## Kod Parçacıkları

Bu eğitim boyunca kullanılan temel komutlar aşağıda hızlı referans için verilmiştir:

```bash
# Sunucunuza SSH ile bağlanın
ssh kullanici_adiniz@sunucu_ip_adresiniz

# Paket listesini güncelleyin ve Certbot'u kurun
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-apache

# Apache yapılandırma dosyasını açın
sudo nano /etc/apache2/sites-available/your_project.conf

# Certbot'u çalıştırarak sertifikayı alın ve kurun
sudo certbot --apache

# Apache'yi yeniden başlatın
sudo service apache2 restart

# Sertifika yenileme işlemi için cron görevi ekleyin
sudo crontab -e
```

Yer tutucular olan `kullanici_adiniz`, `sunucu_ip_adresiniz`, `your_project` ve `your_domain.com` değerlerini kendi gerçek değerlerinizle değiştirin.

Ciddi ve kesin bir yaklaşımı sürdürerek, Django uygulamanızda HTTPS'yi sağlam ve güvenli bir şekilde dağıtmış olursunuz.