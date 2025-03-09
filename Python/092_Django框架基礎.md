[上一章：Web開發進階](091_Web開發進階.md) | [下一章：Django框架進階](093_Django框架進階.md)

# Python Django框架基礎 🌐

## Django項目

### 1. 項目結構

```python
# 創建Django項目
django-admin startproject myproject
cd myproject

# 創建應用
python manage.py startapp myapp

# 項目結構
myproject/
    ├── manage.py
    ├── myproject/
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── myapp/
        ├── __init__.py
        ├── admin.py
        ├── apps.py
        ├── models.py
        ├── views.py
        ├── urls.py
        └── templates/
            └── myapp/
                ├── base.html
                ├── index.html
                └── user_detail.html
```

### 2. 基本配置

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # 添加應用
]

# 數據庫配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# 模板配置
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# 靜態文件配置
STATIC_URL = 'static/'
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

### 3. URL配置

```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]

# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('users/', views.user_list, name='user_list'),
    path('users/<int:user_id>/', views.user_detail, name='user_detail'),
]
```

### 4. 視圖實現

```python
# myapp/views.py
from django.shortcuts import render, get_object_or_404
from django.http import HttpResponse, JsonResponse
from .models import User

def index(request):
    """首頁視圖"""
    return render(request, 'myapp/index.html')

def user_list(request):
    """用戶列表視圖"""
    users = User.objects.all()
    return render(request, 'myapp/user_list.html', {'users': users})

def user_detail(request, user_id):
    """用戶詳情視圖"""
    user = get_object_or_404(User, id=user_id)
    return render(request, 'myapp/user_detail.html', {'user': user})

def api_users(request):
    """API視圖"""
    users = User.objects.all()
    data = [{'id': user.id, 'name': user.name, 'email': user.email} for user in users]
    return JsonResponse(data, safe=False)
```

### 5. 模型定義

```python
# myapp/models.py
from django.db import models

class User(models.Model):
    """用戶模型"""
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def __str__(self):
        return self.name
    
    class Meta:
        ordering = ['-created_at']
```

### 6. 模板設計

```html
<!-- myapp/templates/myapp/base.html -->
<!DOCTYPE html>
<html>
<head>
    {% raw %}
    <title>{% block title %}My Django App{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    {% endraw %}
</head>
<body>
    <nav>
        {% raw %}
        <a href="{% url 'index' %}">Home</a>
        <a href="{% url 'user_list' %}">Users</a>
        {% endraw %}
    </nav>
    
    <main>
        {% raw %}
        {% block content %}
        {% endraw %}
    </main>
    
    <footer>
        <p>&copy; 2024 My Django App</p>
    </footer>
</body>
</html>

<!-- myapp/templates/myapp/index.html -->
{% raw %}
{% extends 'myapp/base.html' %}

{% block title %}Home{% endblock %}

{% block content %}
<h1>Welcome to My Django App</h1>
<p>This is the home page.</p>
{% endblock %}
{% endraw %}

<!-- myapp/templates/myapp/user_list.html -->
{% raw %}
{% extends 'myapp/base.html' %}

{% block title %}Users{% endblock %}

{% block content %}
<h1>Users</h1>
<ul>
    {% for user in users %}
    <li>
        <a href="{% url 'user_detail' user.id %}">{{ user.name }}</a>
        <span>{{ user.email }}</span>
    </li>
    {% endfor %}
</ul>
{% endblock %}
{% endraw %}

<!-- myapp/templates/myapp/user_detail.html -->
{% raw %}
{% extends 'myapp/base.html' %}

{% block title %}{{ user.name }}{% endblock %}

{% block content %}
<h1>{{ user.name }}</h1>
<p>Email: {{ user.email }}</p>
<p>Created: {{ user.created_at }}</p>
<p>Updated: {{ user.updated_at }}</p>
{% endblock %}
{% endraw %}
```

## 練習題

1. **Django項目**
   開發一個Django項目：
   - 創建項目結構
   - 配置基本設置
   - 實現基本功能
   - 優化項目組織

2. **視圖實現**
   創建視圖實現：
   - 處理請求響應
   - 渲染模板
   - 處理數據
   - 提供API

3. **模型設計**
   實現模型設計：
   - 定義數據模型
   - 創建數據庫
   - 實現CRUD
   - 優化查詢

## 小提醒 💡

1. 項目結構
   - 組織代碼
   - 管理配置
   - 處理靜態文件
   - 優化部署

2. 視圖設計
   - 處理請求
   - 渲染模板
   - 處理數據
   - 優化性能

3. 模型設計
   - 定義字段
   - 設置關係
   - 優化查詢
   - 處理遷移

4. 調試技巧
   - 使用開發服務器
   - 分析錯誤信息
   - 優化關鍵路徑
   - 監控應用狀態

[上一章：Web開發進階](091_Web開發進階.md) | [下一章：Django框架進階](093_Django框架進階.md) 