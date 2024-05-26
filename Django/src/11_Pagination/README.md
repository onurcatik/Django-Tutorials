# Python Django Eğitimi: Tam Özellikli Web Uygulaması Bölüm 11 - Sayfalama

Bu eğitimde, web uygulamamızı sayfalama özelliği ekleyerek geliştireceğiz. Bu özellik, çok sayıda gönderiyi daha verimli bir şekilde yönetmemize ve görüntülememize yardımcı olacaktır. Ayrıca, belirli bir kullanıcı tarafından filtrelenen gönderileri içeren ve bu sayfanın da sayfalı olmasını sağlayan bir sayfa oluşturacağız.

## Gösterim İçin Gönderiler Eklemek

Öncelikle, sayfalamayı görmek için yeterli sayıda gönderiye ihtiyacımız var. Gönderileri manuel olarak oluşturabilir veya bir betik kullanarak birden fazla gönderi ekleyebilirsiniz. Burada, gönderileri JSON dosyası kullanarak Django kabuğunda eklemek için bir betik kullanacağız.

### Django Kabuğu Kullanarak Gönderiler Eklemek

1. Örnek veriler içeren bir JSON dosyası (`posts.json`) hazırlayın. Bu dosyayı `manage.py` dosyanızın bulunduğu dizine yerleştirin.

2. Django kabuğunu açın ve aşağıdaki betiği çalıştırarak gönderileri yükleyin:

```python
import json
from blog.models import Post

# JSON dosyasını aç ve yükle
with open('posts.json', 'r') as f:
    posts_json = json.load(f)

# Gönderiler üzerinde döngü yaparak veritabanına ekle
for post_data in posts_json:
    post = Post(
        title=post_data['title'],
        content=post_data['content'],
        author_id=post_data['user_id']
    )
    post.save()
```

## Sayfalama Uygulama

### Görünümlerde Sayfalama Ayarları

Django, sayfalama için `Paginator` sınıfını sağlar. Sınıf tabanlı görünümlerimizi sayfalama eklemek için değiştireceğiz.

1. `views.py` dosyanızı açın.

2. Anasayfa görünümünü (genellikle bir `ListView`) bulun ve `paginate_by` özniteliğini ekleyin:

```python
from django.views.generic import ListView
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/home.html'
    context_object_name = 'posts'
    ordering = ['-date_posted']
    paginate_by = 5  # Tercihinize göre bu sayıyı ayarlayın
```

### Şablonlarda Sayfalama Bağlantıları Oluşturma

Sayfalama bağlantılarını göstermek için, şablonumuzu güncelleyerek gezinme kontrolleri eklememiz gerekiyor.

1. `home.html` şablonunuzu açın.

2. Aşağıdaki kodu ekleyerek sayfalama bağlantılarını gösterin:

```html
{% if is_paginated %}
    <nav aria-label="Page navigation">
        <ul class="pagination">
            {% if page_obj.has_previous %}
                <li class="page-item">
                    <a class="page-link" href="?page=1">İlk</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Önceki</a>
                </li>
            {% endif %}

            {% for num in page_obj.paginator.page_range %}
                {% if num == page_obj.number %}
                    <li class="page-item active">
                        <a class="page-link">{{ num }}</a>
                    </li>
                {% elif num > page_obj.number|add:'-3' and num < page_obj.number|add:'3' %}
                    <li class="page-item">
                        <a class="page-link" href="?page={{ num }}">{{ num }}</a>
                    </li>
                {% endif %}
            {% endfor %}

            {% if page_obj.has_next %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ page_obj.next_page_number }}">Sonraki</a>
                </li>
                <li class="page-item">
                    <a class="page-link" href="?page={{ page_obj.paginator.num_pages }}">Son</a>
                </li>
            {% endif %}
        </ul>
    </nav>
{% endif %}
```

### Kullanıcıya Göre Gönderileri Filtreleme

Belirli bir kullanıcıya göre gönderileri filtrelemek ve sayfalama yapmak için yeni bir görünüm ve şablon oluşturacağız.

1. `views.py` dosyanızda, aşağıdaki görünümü ekleyin:

```python
from django.contrib.auth.models import User
from django.shortcuts import get_object_or_404

class UserPostListView(ListView):
    model = Post
    template_name = 'blog/user_posts.html'
    context_object_name = 'posts'
    paginate_by = 5

    def get_queryset(self):
        user = get_object_or_404(User, username=self.kwargs.get('username'))
        return Post.objects.filter(author=user).order_by('-date_posted')
```

2. `urls.py` dosyanızda, bu görünüm için bir yol ekleyin:

```python
from django.urls import path
from .views import UserPostListView

urlpatterns = [
    path('user/<str:username>', UserPostListView.as_view(), name='user-posts'),
]
```

3. Yeni bir şablon `user_posts.html` oluşturun ve anasayfa şablonunuza benzer bir yapıyı kullanın, ancak başlık olarak gönderilerin belirli bir kullanıcı tarafından filtrelendiğini belirtin:

```html
{% extends "blog/base.html" %}
{% block content %}
  <h1>{{ view.kwargs.username }} tarafından gönderilenler</h1>
  {% for post in posts %}
    <article class="media content-section">
      <img class="rounded-circle article-img" src="{{ post.author.profile.image.url }}">
      <div class="media-body">
        <div class="article-metadata">
          <a class="mr-2" href="{% url 'user-posts' post.author.username %}">{{ post.author }}</a>
          <small class="text-muted">{{ post.date_posted|date:"F d, Y" }}</small>
        </div>
        <h2><a class="article-title" href="{% url 'post-detail' post.id %}">{{ post.title }}</a></h2>
        <p class="article-content">{{ post.content }}</p>
      </div>
    </article>
  {% endfor %}
  {% include 'blog/pagination.html' %}
{% endblock content %}
```

## Sonuç

Bu adımlarla, Django uygulamamızda sayfalama özelliğini başarıyla uygulamış olduk. Ayrıca, belirli kullanıcılar tarafından gönderilen gönderileri filtreleyen ve bu listelerin de sayfalı olmasını sağlayan bir görünüm oluşturduk. Sayfalama, web uygulamanızın performansını ve kullanılabilirliğini artırarak büyük veri setlerini yönetilebilir parçalara ayırır.