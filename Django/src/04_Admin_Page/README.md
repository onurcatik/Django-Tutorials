# Yönetici Sayfası

## Giriş

Bu kapsamlı Django eğitim serisinin bu bölümünde, Django Yönetici arayüzüne nasıl erişileceğini ve nasıl kullanılacağını keşfedeceğiz. Django Yönetici arayüzü, sitenizdeki verileri yönetmek için vazgeçilmez bir araçtır. Veri oluşturma, güncelleme ve silme işlemlerini verimli bir şekilde gerçekleştirmek için grafiksel bir kullanıcı arayüzü (GUI) sağlar.

## Ön Koşullar

Devam etmeden önce, çalışan bir Django projesine ve çalışan bir geliştirme sunucusuna sahip olduğunuzdan emin olun. Henüz bir Django projesi kurmadıysanız, bu eğitim serisinin önceki bölümlerine başvurun.

## Yönetici Sayfasına Erişim

### Geliştirme Sunucusunu Başlatma

Başlamak için, Django geliştirme sunucusunu başlatın. Terminalinizi açın ve proje dizininize gidin. Aşağıdaki komutu çalıştırın:

```bash
python manage.py runserver
```

Bu komut sunucuyu başlatacak ve sitenize web tarayıcınızda `http://127.0.0.1:8000/` adresinden erişebilirsiniz.

### Yönetici Sayfasına Gitmek

Yönetici sayfasına erişmek için sitenizin URL'sine `/admin` ekleyin, böylece `http://127.0.0.1:8000/admin` adresine erişmiş olursunuz. Django yönetici giriş sayfası ile karşılaşacaksınız.

### Süper Kullanıcı Oluşturma

Yönetici arayüzüne ilk kez giriş yapmak için bir süper kullanıcı oluşturmalısınız. Bu süper kullanıcı hesabı, yönetici arayüzüne tam erişime sahip olacaktır. Terminalinizi açın ve geliştirme sunucusunu durdurun (çalışıyorsa) `Ctrl+C` tuşlarına basarak. Ardından aşağıdaki komutu çalıştırın:

```bash
python manage.py createsuperuser
```

Süper kullanıcı için bir kullanıcı adı, e-posta adresi ve parola girmeniz istenecektir. Gerekli bilgileri girdikten sonra süper kullanıcı hesabı oluşturulacaktır.

### Göçleri Çalıştırma

Eğer `OperationalError: no such table: auth_user` hatası alırsanız, gerekli veritabanı tablolarının henüz oluşturulmamış olduğunu gösterir. Bunu çözmek için aşağıdaki göç komutlarını çalıştırın:

```bash
python manage.py makemigrations
python manage.py migrate
```

`makemigrations` komutu göç dosyalarını hazırlar, `migrate` komutu ise bu göçleri veritabanına uygular ve gerekli tabloları oluşturur.

### Yönetici Sayfasına Giriş Yapma

Süper kullanıcı oluşturduktan ve göçleri çalıştırdıktan sonra geliştirme sunucusunu yeniden başlatın:

```bash
python manage.py runserver
```

`http://127.0.0.1:8000/admin` adresine tekrar gidin ve oluşturduğunuz süper kullanıcı kimlik bilgileriyle giriş yapın. Artık Django yönetici arayüzüne erişiminiz olacak.

## Yönetici Arayüzünü Keşfetme

### Genel Bakış

Django yönetici arayüzü, sitenizin verilerini yönetmek için çeşitli işlevler sağlar. Giriş yaptıktan sonra, kullanıcılar ve gruplar gibi farklı modelleri yönetmek için seçenekler göreceksiniz.

### Kullanıcılar ve Grupları Yönetme

#### Kullanıcılar

Kullanıcıları görüntülemek ve yönetmek için "Kullanıcılar" bağlantısına tıklayın. Oluşturduğunuz süper kullanıcı da dahil olmak üzere bir kullanıcı listesi göreceksiniz. Bir kullanıcıya tıklayarak detaylarını görüntüleyebilir ve düzenleyebilirsiniz. Django parolaları güvenli bir şekilde hashleyerek düz metin olarak saklamaz.

#### Gruplar

Gruplar, kullanıcıları kategorize etmenize ve toplu olarak izinler atamanıza olanak tanır. Grupları yönetmek için "Gruplar" bağlantısına tıklayın. Yeni gruplar ekleyebilir ve gerekli izinleri atayabilirsiniz.

### Yeni Bir Kullanıcı Oluşturma

Yeni bir kullanıcı oluşturmak için sağ üst köşedeki "Kullanıcı ekle" bağlantısına tıklayın. Kullanıcı adı ve parola gibi gerekli alanları doldurun, ardından "Kaydet" butonuna tıklayın. Ayrıca e-posta adresi ve kullanıcı izinleri gibi ek detaylar belirleyebilirsiniz.

### Son İşlemler

Yönetici arayüzü, sağ tarafta "Son İşlemler" paneli sağlayarak yapılan son değişiklikleri gösterir. Bu özellik, sitenizin verileri üzerindeki değişiklikleri takip etmenize yardımcı olur.

## Sonuç

Bu eğitimde, Django yönetici arayüzünü keşfettik, bu güçlü aracın sitenizin verilerini yönetmek için nasıl kullanılacağını öğrendik. Süper kullanıcı oluşturmayı, yönetici sayfasına giriş yapmayı ve kullanıcıları ve grupları yönetmeyi öğrendik. Yönetici arayüzü, birçok arka uç işlemini basitleştirir ve önemli ölçüde geliştirme çabası tasarrufu sağlar.

Bu serinin bir sonraki bölümünde, veritabanları ile çalışmayı, web sitemiz için gerçek veriler oluşturmayı ve bu verileri yönetici arayüzü ile entegre etmeyi inceleyeceğiz. Bu sayede, yönetici arayüzü içinde veri oluşturma, güncelleme ve silme işlemlerini etkin bir şekilde yönetebileceğiz.

