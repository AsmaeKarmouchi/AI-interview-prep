# Questions et Réponses d'Entretien Django - Guide Complet

## 📚 Questions Théoriques avec Réponses

### Architecture et Concepts de Base

**1. Qu'est-ce que Django et quels sont ses avantages ?**

**Réponse :** Django est un framework web Python de haut niveau qui encourage le développement rapide et la conception propre. Ses avantages :
- **Batteries incluses** : ORM, admin, authentification intégrés
- **Sécurité** : Protection CSRF, XSS, SQL injection par défaut  
- **Scalabilité** : Utilisé par Instagram, Pinterest, Mozilla
- **DRY (Don't Repeat Yourself)** : Réutilisation maximale du code
- **Documentation excellente** et grande communauté
- **Architecture MVT** claire et maintenable

**2. Expliquez l'architecture MVT (Model-View-Template)**

**Réponse :**
- **Model** : Couche de données, définit la structure et la logique métier
- **View** : Logique de traitement, reçoit les requêtes et retourne les réponses  
- **Template** : Couche de présentation, génère le HTML dynamique
- **URL Dispatcher** : Route les URLs vers les vues appropriées

```python
# Model
class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()

# View  
def article_list(request):
    articles = Article.objects.all()
    return render(request, 'articles.html', {'articles': articles})

# Template (articles.html)
{% for article in articles %}
    <h2>{{ article.title }}</h2>
    <p>{{ article.content }}</p>
{% endfor %}
```

**3. Différence entre Django et Flask ?**

**Réponse :**

| Django | Flask |
|--------|-------|
| Framework complet (batteries incluses) | Micro-framework minimaliste |
| ORM intégré | ORM optionnel (SQLAlchemy) |
| Structure de projet imposée | Structure libre |
| Admin interface incluse | Pas d'admin par défaut |
| Authentification intégrée | Extensions nécessaires |
| Meilleur pour grandes applications | Meilleur pour petites applications/APIs |

**4. Qu'est-ce que le principe DRY ?**

**Réponse :** Don't Repeat Yourself - éviter la duplication de code :
- Utilisation d'**héritage de templates** : `{% extends 'base.html' %}`
- **Modèles abstraits** pour partager des champs communs
- **Mixins** dans les vues pour partager la logique
- **Template tags/filters** personnalisés pour la logique réutilisable

### Models et Base de Données

**5. Différence entre `null=True` et `blank=True` ?**

**Réponse :**
- **`null=True`** : Autorise NULL en base de données (niveau BDD)
- **`blank=True`** : Autorise champ vide dans les formulaires (niveau validation)

```python
# Recommandations :
class Article(models.Model):
    title = models.CharField(max_length=200)  # Obligatoire
    subtitle = models.CharField(max_length=200, blank=True, default='')  # Texte optionnel
    published_date = models.DateTimeField(null=True, blank=True)  # Date optionnelle
    content = models.TextField(blank=True, default='')  # Éviter null=True sur TextField
```

**6. Quels sont les différents types de relations ?**

**Réponse :**

```python
# OneToMany (ForeignKey)
class Category(models.Model):
    name = models.CharField(max_length=100)

class Article(models.Model):
    category = models.ForeignKey(Category, on_delete=models.CASCADE)

# OneToOne  
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()

# ManyToMany
class Article(models.Model):
    tags = models.ManyToManyField('Tag', blank=True)
    authors = models.ManyToManyField(User, through='AuthorRole')

# ManyToMany avec table intermédiaire
class AuthorRole(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    role = models.CharField(max_length=50)  # 'primary', 'contributor'
    created_at = models.DateTimeField(auto_now_add=True)
```

**7. Différence entre `select_related` et `prefetch_related` ?**

**Réponse :**

```python
# select_related : JOIN SQL (ForeignKey, OneToOne)
# UNE seule requête avec JOIN
articles = Article.objects.select_related('category', 'author')
for article in articles:
    print(article.category.name)  # Pas de requête supplémentaire

# prefetch_related : Requêtes séparées (ManyToMany, reverse FK)  
# Deux requêtes : une pour articles, une pour tags
articles = Article.objects.prefetch_related('tags')
for article in articles:
    for tag in article.tags.all():  # Pas de requête supplémentaire
        print(tag.name)

# Combinaison optimale
Article.objects.select_related('category').prefetch_related('tags', 'comments__author')
```

**8. Qu'est-ce que le QuerySet et ses méthodes ?**

**Réponse :** Un QuerySet est une collection d'objets de la base de données, évalué de manière paresseuse.

```python
# Méthodes qui retournent un QuerySet (lazy)
qs = Article.objects.filter(published=True)
qs = qs.exclude(title__icontains='draft')  
qs = qs.order_by('-created_at')
qs = qs[:10]  # Pas encore de requête SQL

# Méthodes qui évaluent le QuerySet
list(qs)  # Exécute la requête
qs.count()
qs.exists()
qs.first()
for article in qs:  # Évaluation

# Méthodes avancées
Article.objects.aggregate(avg_views=Avg('views'))
Article.objects.annotate(comment_count=Count('comments'))
Article.objects.values('title', 'category__name')  # Dict au lieu d'objets
Article.objects.values_list('title', flat=True)  # Liste de valeurs
```

### Views et URLs

**9. Différence entre Function-Based Views et Class-Based Views ?**

**Réponse :**

```python
# Function-Based View (FBV)
def article_list(request):
    if request.method == 'GET':
        articles = Article.objects.filter(published=True)
        return render(request, 'articles/list.html', {'articles': articles})
    elif request.method == 'POST':
        # Logique POST
        pass

# Class-Based View (CBV) - Plus réutilisable
class ArticleListView(ListView):
    model = Article
    template_name = 'articles/list.html'
    context_object_name = 'articles'
    queryset = Article.objects.filter(published=True)
    paginate_by = 10
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        return context

# CBV Générique avec mixins
class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    form_class = ArticleForm
    success_url = reverse_lazy('article:list')
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)
```

**10. Decorators Django courants ?**

**Réponse :**

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.decorators.http import require_http_methods
from django.views.decorators.cache import cache_page
from django.views.decorators.csrf import csrf_exempt

@login_required
@permission_required('articles.add_article')
@require_http_methods(["GET", "POST"])
def create_article(request):
    pass

@cache_page(60 * 15)  # Cache 15 minutes
def article_list(request):
    pass

# Decorator personnalisé
def staff_required(view_func):
    def wrapper(request, *args, **kwargs):
        if not request.user.is_staff:
            return HttpResponseForbidden()
        return view_func(request, *args, **kwargs)
    return wrapper

@staff_required
def admin_view(request):
    pass
```

**11. Système de routage URL Django ?**

**Réponse :**

```python
# urls.py principal
urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls', namespace='articles')),
    path('api/v1/', include('api.urls')),
]

# articles/urls.py
app_name = 'articles'
urlpatterns = [
    path('', ArticleListView.as_view(), name='list'),
    path('<int:pk>/', ArticleDetailView.as_view(), name='detail'),
    path('<slug:slug>/', ArticleDetailView.as_view(), name='detail_slug'),
    path('category/<str:category>/', CategoryView.as_view(), name='category'),
    path('create/', ArticleCreateView.as_view(), name='create'),
]

# Dans les templates
{% url 'articles:detail' article.pk %}
{% url 'articles:category' category='tech' %}

# Dans les vues
from django.urls import reverse, reverse_lazy
redirect_url = reverse('articles:detail', kwargs={'pk': article.pk})
success_url = reverse_lazy('articles:list')
```

### Templates et Static Files

**12. Système de templates Django ?**

**Réponse :**

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Mon Site{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <nav>{% include 'partials/navigation.html' %}</nav>
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    {% block extra_js %}{% endblock %}
</body>
</html>

<!-- article_list.html -->
{% extends 'base.html' %}
{% load custom_tags %}

{% block title %}Articles - {{ block.super }}{% endblock %}

{% block content %}
    {% for article in articles %}
        <article>
            <h2>{{ article.title|title }}</h2>
            <p>{{ article.content|truncatewords:30 }}</p>
            <small>{{ article.created_at|date:"d M Y" }}</small>
            {% if article.is_published %}
                <span class="published">Publié</span>
            {% endif %}
        </article>
    {% empty %}
        <p>Aucun article disponible.</p>
    {% endfor %}
    
    <!-- Pagination -->
    {% if is_paginated %}
        <div class="pagination">
            {% if page_obj.has_previous %}
                <a href="?page={{ page_obj.previous_page_number }}">Précédent</a>
            {% endif %}
            
            Page {{ page_obj.number }} sur {{ page_obj.paginator.num_pages }}
            
            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">Suivant</a>
            {% endif %}
        </div>
    {% endif %}
{% endblock %}
```

**13. Template Tags et Filters personnalisés ?**

**Réponse :**

```python
# templatetags/custom_tags.py
from django import template
from django.utils.safestring import mark_safe
import markdown

register = template.Library()

# Simple tag
@register.simple_tag
def current_year():
    from datetime import datetime
    return datetime.now().year

# Filter
@register.filter
def markdown_to_html(value):
    return mark_safe(markdown.markdown(value))

# Inclusion tag
@register.inclusion_tag('partials/recent_articles.html')
def recent_articles(count=5):
    articles = Article.objects.filter(published=True)[:count]
    return {'articles': articles}

# Tag avec paramètres
@register.simple_tag
def url_replace(request, **kwargs):
    dict_ = request.GET.copy()
    for k, v in kwargs.items():
        dict_[k] = v
    return dict_.urlencode()

# Usage dans les templates
{% load custom_tags %}

<p>Copyright {% current_year %}</p>
{{ article.content|markdown_to_html }}
{% recent_articles 3 %}
<a href="?{% url_replace request page=2 %}">Page 2</a>
```

### Formulaires et Validation

**14. Django Forms vs ModelForm ?**

**Réponse :**

```python
# Form classique
class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    subject = forms.CharField(max_length=200)
    message = forms.CharField(widget=forms.Textarea)
    
    def clean_email(self):
        email = self.cleaned_data['email']
        if not email.endswith('@company.com'):
            raise forms.ValidationError('Email doit être du domaine company.com')
        return email
    
    def clean(self):
        cleaned_data = super().clean()
        name = cleaned_data.get('name')
        subject = cleaned_data.get('subject')
        
        if name and subject and name.lower() in subject.lower():
            raise forms.ValidationError('Le sujet ne peut contenir le nom')
        return cleaned_data

# ModelForm - Plus DRY
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'category', 'tags']
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
            'tags': forms.CheckboxSelectMultiple(),
        }
        labels = {
            'title': 'Titre de l\'article',
        }
    
    def __init__(self, *args, **kwargs):
        self.user = kwargs.pop('user', None)
        super().__init__(*args, **kwargs)
        
        # Filtrer les catégories selon l'utilisateur
        if self.user:
            self.fields['category'].queryset = Category.objects.filter(
                owner=self.user
            )
    
    def save(self, commit=True):
        article = super().save(commit=False)
        if self.user:
            article.author = self.user
        if commit:
            article.save()
            self.save_m2m()  # Important pour ManyToMany
        return article

# Dans la vue
def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST, user=request.user)
        if form.is_valid():
            article = form.save()
            messages.success(request, 'Article créé avec succès!')
            return redirect('articles:detail', pk=article.pk)
    else:
        form = ArticleForm(user=request.user)
    
    return render(request, 'articles/form.html', {'form': form})
```

### Sécurité

**15. Mesures de sécurité Django ?**

**Réponse :**

```python
# settings.py - Configuration sécurité
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000  # 1 an
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'

# Protection CSRF automatique
# Dans les templates
<form method="post">
    {% csrf_token %}
    <!-- champs du formulaire -->
</form>

# Protection contre l'injection SQL (ORM automatique)
# ✅ Sécurisé
Article.objects.filter(title__icontains=user_input)

# ❌ Dangereux - éviter raw SQL
Article.objects.raw("SELECT * FROM articles WHERE title LIKE '%{}%'".format(user_input))

# Protection XSS (échappement automatique)
<!-- Dans les templates, échappement automatique -->
{{ user_comment }}  <!-- Sécurisé -->
{{ user_comment|safe }}  <!-- Dangereux -->

# Permissions et authentification
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class ArticleCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    permission_required = 'articles.add_article'
    
    def dispatch(self, request, *args, **kwargs):
        # Vérifications supplémentaires
        if not request.user.is_verified:
            return redirect('verify_account')
        return super().dispatch(request, *args, **kwargs)
```

### Performance et Cache

**16. Optimisation des performances Django ?**

**Réponse :**

```python
# 1. Optimisation des requêtes
# ❌ Problème N+1
def bad_view(request):
    articles = Article.objects.all()
    for article in articles:
        print(article.author.username)  # Requête pour chaque article
        print(article.comments.count())  # Requête pour chaque article

# ✅ Optimisé  
def good_view(request):
    articles = Article.objects.select_related('author').prefetch_related('comments')
    for article in articles:
        print(article.author.username)  # Pas de requête supplémentaire
        print(article.comments.count())  # Pas de requête supplémentaire

# 2. Utilisation du cache
from django.core.cache import cache
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache 15 minutes
def article_list(request):
    return render(request, 'articles.html')

# Cache manuel
def get_popular_articles():
    cache_key = 'popular_articles'
    articles = cache.get(cache_key)
    
    if articles is None:
        articles = Article.objects.filter(views__gte=1000).order_by('-views')[:10]
        cache.set(cache_key, articles, 300)  # Cache 5 minutes
    
    return articles

# 3. Database indexes
class Article(models.Model):
    title = models.CharField(max_length=200, db_index=True)
    slug = models.SlugField(unique=True, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['created_at', 'published']),
            models.Index(fields=['author', '-created_at']),
        ]

# 4. Pagination efficace
from django.core.paginator import Paginator

def article_list(request):
    articles = Article.objects.select_related('author').order_by('-created_at')
    paginator = Paginator(articles, 20)  # 20 articles par page
    
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    return render(request, 'articles.html', {'page_obj': page_obj})

# 5. Requêtes conditionnelles
articles = Article.objects.all()

# Utiliser only() pour limiter les champs
articles = articles.only('title', 'slug', 'created_at')

# Utiliser defer() pour exclure des champs lourds  
articles = articles.defer('content')

# Utiliser exists() au lieu de count()
if Article.objects.filter(author=user).exists():
    # Au lieu de : if Article.objects.filter(author=user).count() > 0
    pass
```

## 🚀 Questions Avancées avec Réponses

**17. Custom Managers et QuerySets ?**

**Réponse :**

```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(published=True)

class ArticleQuerySet(models.QuerySet):
    def published(self):
        return self.filter(published=True)
    
    def by_author(self, author):
        return self.filter(author=author)
    
    def recent(self, days=7):
        from datetime import datetime, timedelta
        return self.filter(created_at__gte=datetime.now() - timedelta(days=days))

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published = models.BooleanField(default=False)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    objects = models.Manager()  # Manager par défaut
    published_objects = PublishedManager()  # Manager personnalisé
    
    # Ou avec QuerySet personnalisé
    objects = ArticleQuerySet.as_manager()

# Usage
Article.published_objects.all()  # Seulement les publiés
Article.objects.published().by_author(user).recent(30)  # Chaînable
```

**18. Signaux Django ?**

**Réponse :**

```python
from django.db.models.signals import post_save, pre_delete, m2m_changed
from django.dispatch import receiver
from django.core.mail import send_mail

# Signal post_save
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)

# Signal pre_delete  
@receiver(pre_delete, sender=Article)
def backup_article(sender, instance, **kwargs):
    # Sauvegarder l'article avant suppression
    ArticleBackup.objects.create(
        original_id=instance.id,
        title=instance.title,
        content=instance.content
    )

# Signal m2m_changed
@receiver(m2m_changed, sender=Article.tags.through)
def article_tags_changed(sender, instance, action, pk_set, **kwargs):
    if action == 'post_add':
        # Notifier quand des tags sont ajoutés
        send_mail(
            'Tags ajoutés',
            f'Tags ajoutés à l\'article {instance.title}',
            'from@example.com',
            ['admin@example.com']
        )

# Signal personnalisé
import django.dispatch
article_viewed = django.dispatch.Signal()

# Dans la vue
def article_detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    article_viewed.send(sender=Article, instance=article, user=request.user)
    return render(request, 'article_detail.html', {'article': article})

@receiver(article_viewed, sender=Article)
def increment_view_count(sender, instance, user, **kwargs):
    instance.views += 1
    instance.save(update_fields=['views'])
```

**19. Middleware personnalisé ?**

**Réponse :**

```python
import time
import logging
from django.utils.deprecation import MiddlewareMixin

class RequestLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # Code exécuté avant la vue
        start_time = time.time()
        
        response = self.get_response(request)
        
        # Code exécuté après la vue
        duration = time.time() - start_time
        logging.info(f'{request.method} {request.path} - {response.status_code} - {duration:.2f}s')
        
        return response

    def process_exception(self, request, exception):
        # Gérer les exceptions
        logging.error(f'Exception for {request.path}: {exception}')
        return None

# Middleware avec toutes les méthodes
class CustomMiddleware(MiddlewareMixin):
    def process_request(self, request):
        # Traiter la requête avant qu'elle atteigne la vue
        if request.path.startswith('/admin/') and not request.user.is_staff:
            return HttpResponseForbidden()
        return None

    def process_view(self, request, view_func, view_args, view_kwargs):
        # Appelé juste avant que Django appelle la vue
        return None

    def process_template_response(self, request, response):
        # Traiter les réponses avec un attribut 'template_name'
        return response

    def process_response(self, request, response):
        # Traiter toutes les réponses
        response['X-Custom-Header'] = 'MyValue'
        return response

# Dans settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'myapp.middleware.RequestLoggingMiddleware',
    'myapp.middleware.CustomMiddleware',
    # ...
]
```

**20. Tests Django complets ?**

**Réponse :**

```python
from django.test import TestCase, Client
from django.contrib.auth.models import User
from django.urls import reverse
from unittest.mock import patch, Mock

class ArticleModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.article = Article.objects.create(
            title='Test Article',
            content='Test content',
            author=self.user,
            published=True
        )

    def test_article_str_representation(self):
        self.assertEqual(str(self.article), 'Test Article')

    def test_article_slug_generation(self):
        # Test avec signal
        article = Article.objects.create(
            title='Another Test Article',
            content='Content',
            author=self.user
        )
        self.assertEqual(article.slug, 'another-test-article')

    def test_article_manager(self):
        # Test custom manager
        published_count = Article.published_objects.count()
        self.assertEqual(published_count, 1)
        
        # Créer un article non publié
        Article.objects.create(
            title='Unpublished',
            content='Content',
            author=self.user,
            published=False
        )
        
        # Vérifier que le manager ne le retourne pas
        self.assertEqual(Article.published_objects.count(), 1)

class ArticleViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.article = Article.objects.create(
            title='Test Article',
            content='Test content',
            author=self.user,
            published=True
        )

    def test_article_list_view(self):
        response = self.client.get(reverse('articles:list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test Article')
        self.assertQuerysetEqual(
            response.context['articles'],
            [self.article]
        )

    def test_article_detail_view(self):
        response = self.client.get(
            reverse('articles:detail', kwargs={'pk': self.article.pk})
        )
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.context['article'], self.article)

    def test_article_create_view_requires_login(self):
        response = self.client.get(reverse('articles:create'))
        self.assertEqual(response.status_code, 302)  # Redirect to login

    def test_article_create_view_authenticated(self):
        self.client.login(username='testuser', password='testpass123')
        
        response = self.client.post(reverse('articles:create'), {
            'title': 'New Article',
            'content': 'New content',
        })
        
        self.assertEqual(response.status_code, 302)  # Redirect after creation
        self.assertTrue(Article.objects.filter(title='New Article').exists())

class ArticleFormTest(TestCase):
    def test_valid_form(self):
        form_data = {
            'title': 'Test Title',
            'content': 'Test content',
        }
        form = ArticleForm(data=form_data)
        self.assertTrue(form.is_valid())

    def test_form_save(self):
        user = User.objects.create_user(username='test', password='test')
        form = ArticleForm({
            'title': 'Test',
            'content': 'Content'
        }, user=user)
        
        self.assertTrue(form.is_valid())
        article = form.save()
        self.assertEqual(article.author, user)

# Tests avec mocking
class EmailTest(TestCase):
    @patch('myapp.tasks.send_email.delay')
    def test_article_notification(self, mock_send_email):
        # Test que l'email est envoyé lors de la création d'un article
        user = User.objects.create_user(username='test', password='test')
        Article.objects.create(
            title='Test',
            content='Content',
            author=user,
            published=True
        )
        
        mock_send_email.assert_called_once()

# Tests d'intégration
class ArticleIntegrationTest(TestCase):
    def test_full_article_workflow(self):
        # Créer un utilisateur
        user = User.objects.create_user(username='test', password='test')
        self.client.login(username='test', password='test')
        
        # Créer un article
        response = self.client.post(reverse('articles:create'), {
            'title': 'Integration Test',
            'content': 'Test content',
        })
        
        article = Article.objects.get(title='Integration Test')
        
        # Vérifier la redirection
        self.assertRedirects(
            response, 
            reverse('articles:detail', kwargs={'pk': article.pk})
        )
        
        # Vérifier l'affichage
        response = self.client.get(
            reverse('articles:detail', kwargs={'pk': article.pk})
        )
        self.assertContains(response, 'Integration Test')

# Factory pour les tests
class ArticleFactory:
    @staticmethod
    def create_article(**kwargs