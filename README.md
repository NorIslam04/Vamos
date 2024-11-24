# Approche avec Django Templates et Forms :

pythonCopy# models.py
from django.db import models

class Post(models.Model):
    message = models.TextField()
    poster = models.ForeignKey('auth.User', on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

# forms.py
from django import forms

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['message']

# views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required

@login_required
def forum_view(request):
    posts = Post.objects.all().order_by('-created_at')
    form = PostForm()
    
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.poster = request.user
            post.save()
            return redirect('forum')
            
    return render(request, 'forum.html', {
        'posts': posts,
        'form': form
    })
htmlCopy<!-- templates/forum.html -->
{% extends 'base.html' %}

{% block content %}
<div class="post-form">
    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Poster</button>
    </form>
</div>

<div class="posts">
    {% for post in posts %}
    <div class="post-card">
        <div class="post-header">
            <span class="post-author">@{{ post.poster.username }}</span>
        </div>
        <div class="post-content">
            {{ post.message }}
        </div>
        <div class="post-footer">
            <div class="post-actions">
                <button>💬</button>
                <button>❤️</button>
            </div>
        </div>
    </div>
    {% endfor %}
</div>
{% endblock %}

Pour le système de profil :

pythonCopy# models.py
class Profile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE)
    bio = models.TextField(blank=True)
    followers = models.ManyToManyField('auth.User', related_name='following')

# forms.py
class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ['bio']

# views.py
@login_required
def profile_view(request):
    profile = request.user.profile
    form = ProfileForm(instance=profile)
    
    if request.method == 'POST':
        form = ProfileForm(request.POST, instance=profile)
        if form.is_valid():
            form.save()
            return redirect('profile')
            
    return render(request, 'profile.html', {
        'profile': profile,
        'form': form
    })
htmlCopy<!-- templates/profile.html -->
{% extends 'base.html' %}

{% block content %}
<div class="profile">
    <div class="profile-header">
        <h2>@{{ user.username }}</h2>
    </div>
    
    <div class="profile-stats">
        <div class="stat">
            <span>Followers</span>
            <span>{{ user.profile.followers.count }}</span>
        </div>
        <div class="stat">
            <span>Following</span>
            <span>{{ user.following.count }}</span>
        </div>
    </div>

    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Mettre à jour</button>
    </form>
</div>
{% endblock %}

Pour le système de recherche :

pythonCopy# views.py
def search_users(request):
    query = request.GET.get('q', '')
    users = User.objects.filter(username__icontains=query)
    return render(request, 'search.html', {'users': users})
htmlCopy<!-- templates/search.html -->
{% extends 'base.html' %}

{% block content %}
<div class="search">
    <form method="GET">
        <input type="text" name="q" placeholder="Rechercher un utilisateur...">
        <button type="submit">Rechercher</button>
    </form>

    <div class="user-list">
        {% for user in users %}
        <div class="user-item">
            <span>@{{ user.username }}</span>
            <form method="POST" action="{% url 'follow_user' user.id %}">
                {% csrf_token %}
                <button type="submit">Suivre</button>
            </form>
        </div>
        {% endfor %}
    </div>
</div>
{% endblock %}

URLs et configuration :

pythonCopy# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('forum/', views.forum_view, name='forum'),
    path('profile/', views.profile_view, name='profile'),
    path('search/', views.search_users, name='search'),
    path('follow/<int:user_id>/', views.follow_user, name='follow_user'),
]

Ajoutez du CSS pour le style :

pythonCopy# settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    BASE_DIR / 'static'
]
cssCopy/* static/css/style.css */
.post-card {
    border: 1px solid #ddd;
    margin: 10px 0;
    padding: 15px;
    border-radius: 8px;
}

.profile {
    max-width: 600px;
    margin: 0 auto;
}

.user-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    border-bottom: 1px solid #eee;
}
htmlCopy<!-- templates/base.html -->
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <nav>
        <a href="{% url 'forum' %}">Forum</a>
        <a href="{% url 'profile' %}">Profile</a>
        <a href="{% url 'search' %}">Rechercher</a>
    </nav>
    
    {% block content %}
    {% endblock %}
</body>
</html>
Avantages de cette approche :

Plus simple à mettre en place
Pas besoin de JavaScript complexe
Rendu côté serveur plus rapide
Meilleure SEO
Moins de code à maintenir

Vous voulez que je détaille une partie particulière ?
