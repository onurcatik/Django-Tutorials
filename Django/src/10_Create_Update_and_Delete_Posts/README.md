# Gönderi Oluşturma, Güncelleme ve Silme

Bu eğitimde, kullanıcıların Django web uygulamamızda blog gönderileri oluşturma, güncelleme ve silme yeteneğini nasıl ekleyeceğimizi ele alacağız. Ayrıca, bu işlemleri daha verimli bir şekilde yönetmek için sınıf tabanlı görünümleri (CBV'ler) kullanmayı da inceleyeceğiz. İçerik, alanın standartlarını yansıtan titizlikte ve kesinlikte olacaktır.

## Ön Koşullar

İlerlemeye başlamadan önce, blog uygulamanızla çalışan bir Django projesine ve aşağıdaki ayarlara sahip olduğunuzdan emin olun:

- Django kurulu (`pip install django`)
- Blog uygulamanızda tanımlı bir `Post` modeli
- Django görünümleri ve şablonları ile temel aşinalık

## Sınıf Tabanlı Görünümler Genel Bakış

Django'daki sınıf tabanlı görünümler, yaygın web geliştirme görevlerini yönetmek için güçlü ve esnek bir yol sunar. Yazmanız gereken kod miktarını azaltan yerleşik işlevlerle birlikte gelirler. Kullanacağımız ana sınıf tabanlı görünümler şunlardır:

- `ListView` nesnelerin bir listesini görüntülemek için
- `DetailView` tek bir nesnenin ayrıntılarını görüntülemek için
- `CreateView` yeni bir nesne oluşturmak için
- `UpdateView` mevcut bir nesneyi güncellemek için
- `DeleteView` bir nesneyi silmek için

## Adım 1: ListView Kurulumu

Gönderileri görüntülemek için mevcut işlev tabanlı görünümümüzü bir sınıf tabanlı `ListView` olarak dönüştürerek başlayacağız.

### views.py

```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/home.html'
    context_object_name = 'posts'
    ordering = ['-date_posted']
```

### urls.py

```python
from django.urls import path
from .views import PostListView

urlpatterns = [
    path('', PostListView.as_view(), name='blog-home'),
    # diğer yollar
]
```

### home.html

`home.html` şablonunuzun `ListView` tarafından sağlanan `posts` bağlam değişkenini işleyebildiğinden emin olun.

```html
{% for post in posts %}
    <article>
        <h2><a href="{% url 'post-detail' post.id %}">{{ post.title }}</a></h2>
        <p>{{ post.content }}</p>
    </article>
{% endfor %}
```

## Adım 2: DetailView Kurulumu

Bir gönderinin ayrıntılarını görüntülemek için `DetailView`'i kuracağız.

### views.py

`PostDetailView` ekleyin:

```python
class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
```

### urls.py

`PostDetailView` için bir yol ekleyin:

```python
urlpatterns = [
    path('', PostListView.as_view(), name='blog-home'),
    path('post/<int:pk>/', PostDetailView.as_view(), name='post-detail'),
    # diğer yollar
]
```

### post_detail.html

Tek bir gönderiyi görüntülemek için bir şablon oluşturun:

```html
<article>
    <h2>{{ object.title }}</h2>
    <p>{{ object.content }}</p>
</article>
```

## Adım 3: CreateView Kurulumu

Kullanıcıların yeni gönderiler oluşturmasına izin vermek için `CreateView`'i kullanacağız.

### views.py

`PostCreateView` ekleyin:

```python
class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

### urls.py

`PostCreateView` için bir yol ekleyin:

```python
urlpatterns = [
    path('', PostListView.as_view(), name='blog-home'),
    path('post/<int:pk>/', PostDetailView.as_view(), name='post-detail'),
    path('post/new/', PostCreateView.as_view(), name='post-create'),
    # diğer yollar
]
```

### post_form.html

Gönderi oluşturma ve güncelleme için bir form şablonu oluşturun:

```html
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Kaydet</button>
</form>
```

## Adım 4: UpdateView Kurulumu

Kullanıcıların gönderilerini güncellemelerine izin vermek için `UpdateView`'i kullanacağız.

### views.py

`PostUpdateView` ekleyin:

```python
class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)

    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
```

### urls.py

`PostUpdateView` için bir yol ekleyin:

```python
urlpatterns = [
    path('', PostListView.as_view(), name='blog-home'),
    path('post/<int:pk>/', PostDetailView.as_view(), name='post-detail'),
    path('post/new/', PostCreateView.as_view(), name='post-create'),
    path('post/<int:pk>/update/', PostUpdateView.as_view(), name='post-update'),
    # diğer yollar
]
```

## Adım 5: DeleteView Kurulumu

Kullanıcıların gönderilerini silmelerine izin vermek için `DeleteView`'i kullanacağız.

### views.py

`PostDeleteView` ekleyin:

```python
class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = '/'

    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
```

### urls.py

`PostDeleteView` için bir yol ekleyin:

```python
urlpatterns = [
    path('', PostListView.as_view(), name='blog-home'),
    path('post/<int:pk>/', PostDetailView.as_view(), name='post-detail'),
    path('post/new/', PostCreateView.as_view(), name='post-create'),
    path('post/<int:pk>/update/', PostUpdateView.as_view(), name='post-update'),
    path('post/<int:pk>/delete/', PostDeleteView.as_view(), name='post-delete'),
    # diğer yollar
]
```

### post_confirm_delete.html

Bir gönderinin silinmesini onaylayan bir şablon oluşturun:

```html
<form method="post">
    {% csrf_token %}
    <p>Bu gönderiyi silmek istediğinizden emin misiniz?</p>
    <button type="submit">Sil</button>
    <a href="{% url 'post-detail' object.pk %}">İptal</a>
</form>
```

## Sonuç

Bu eğitimde, kullanıcıların gönderi oluşturma, güncelleme ve silme işlevlerini ekledik. Django'nun sınıf tabanlı görünümleri, yaygın web geliştirme görevlerini yönetmek için kod yazma süresini ve miktarını azaltan güçlü ve esnek bir yol sunar.

