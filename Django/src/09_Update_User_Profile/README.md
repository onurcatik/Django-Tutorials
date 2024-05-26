# Kullanıcı Profilini Güncelleme

## Giriş

Bu derste, kullanıcı profil sayfamızı tamamlayarak, kullanıcıların bilgilerini güncelleyebilmelerini ve yeni bir profil resmi yükleyebilmelerini sağlayacağız. Ayrıca, yüklenen resimlerin otomatik olarak yeniden boyutlandırılmasını sağlayacağız, böylece sistemimizde aşırı büyük dosyaların bulunmasını önleyeceğiz. Bu ders, Django'da form oluşturma ve işleme, görünümler ve şablonlar ile resim işleme konularını kapsayacaktır.

## Kullanıcı ve Profil Güncellemeleri İçin Formlar Oluşturma

Kullanıcı bilgilerini ve profil resimlerini güncellemek için formlar oluşturmamız gerekiyor. Veritabanı modellerimizle etkileşimi yönetmek için Django'nun ModelForm'larını kullanacağız.

### `forms.py` Dosyasını Güncelleme

Öncelikle, `User` ve `Profile` modellerini güncellemek için formlar oluşturacağız.

```python
# users/forms.py

from django import forms
from django.contrib.auth.models import User
from .models import Profile

class UserUpdateForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ['username', 'email']

class ProfileUpdateForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['image']
```

### `Profile` Görünümünü Düzenleme

Sonra, bu formları işlemek için profil görünümümüzü güncelleyeceğiz.

```python
# users/views.py

from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import UserUpdateForm, ProfileUpdateForm

def profile(request):
    if request.method == 'POST':
        u_form = UserUpdateForm(request.POST, instance=request.user)
        p_form = ProfileUpdateForm(request.POST, request.FILES, instance=request.user.profile)
        if u_form.is_valid() and p_form.is_valid():
            u_form.save()
            p_form.save()
            messages.success(request, 'Hesabınız güncellendi!')
            return redirect('profile')
    else:
        u_form = UserUpdateForm(instance=request.user)
        p_form = ProfileUpdateForm(instance=request.user.profile)

    context = {
        'u_form': u_form,
        'p_form': p_form
    }

    return render(request, 'users/profile.html', context)
```

## Şablonu Güncelleme

Bu formları göstermek için profil şablonumuzu değiştireceğiz.

```html
<!-- users/templates/users/profile.html -->

{% extends "base_generic.html" %}
{% load crispy_forms_tags %}

{% block content %}
  <div class="profile-page">
    <h2>Profil Bilgileri</h2>
    <form method="POST" enctype="multipart/form-data">
      {% csrf_token %}
      {{ u_form|crispy }}
      {{ p_form|crispy }}
      <button type="submit" class="btn btn-primary">Güncelle</button>
    </form>
  </div>
{% endblock %}
```

`crispy_forms`'un doğru bir şekilde yüklendiğinden ve formlarımızı render etmek için kullanıldığından emin olun.

## Yüklenen Resimleri Yeniden Boyutlandırma

Profil resimleri yüklenirken yeniden boyutlandırmak için Pillow kütüphanesini kullanacağız.

### Pillow Kurulumu

Eğer yüklü değilse, Pillow'u pip ile yükleyin:

```sh
pip install pillow
```

### Profil Modelinde Save Yöntemini Geçersiz Kılma

Resimleri yeniden boyutlandırmak için `Profile` modelinin `save` yöntemini geçersiz kılacağız.

```python
# users/models.py

from django.db import models
from django.contrib.auth.models import User
from PIL import Image

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    image = models.ImageField(default='default.jpg', upload_to='profile_pics')

    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        img = Image.open(self.image.path)

        if img.height > 300 or img.width > 300:
            output_size = (300, 300)
            img.thumbnail(output_size)
            img.save(self.image.path)
```

## Fonksiyonelliği Test Etme

Geliştirme sunucunuzu yeniden başlatın ve profil sayfasına gidin. Mevcut kullanıcı bilgilerinin formda önceden doldurulduğundan emin olun. Kullanıcı adını, e-posta adresini ve profil resmini güncelleyip formu gönderin. Bilgilerin güncellendiğini ve profil resminin yeniden boyutlandırıldığını doğrulayın.

```sh
python manage.py runserver
```

## Sonuç

Bu derste, kullanıcıların profil bilgilerini güncelleyebilmelerini ve yeni bir profil resmi yükleyebilmelerini sağlayan bir özellik uyguladık. Resimlerin otomatik olarak yeniden boyutlandırılmasını sağlayarak depolama ve performans optimizasyonu gerçekleştirdik. Bu kapsamlı yaklaşım, form işleme, görünüm düzenleme ve şablon güncellemeyi kapsayarak Django web geliştirme alanında beklenen titizlik ve standartları yansıtmaktadır.

#