[上一章：Django框架基礎](092_Django框架基礎.md) | [下一章：數據分析基礎](094_數據分析基礎.md)

# Python Django框架進階 🌐

## 高級特性

### 1. 類視圖

```python
# myapp/views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import User

class UserListView(ListView):
    """用戶列表視圖"""
    model = User
    template_name = 'myapp/user_list.html'
    context_object_name = 'users'
    paginate_by = 10
    
    def get_queryset(self):
        """獲取查詢集"""
        queryset = super().get_queryset()
        search = self.request.GET.get('search')
        if search:
            queryset = queryset.filter(name__icontains=search)
        return queryset

class UserDetailView(DetailView):
    """用戶詳情視圖"""
    model = User
    template_name = 'myapp/user_detail.html'
    context_object_name = 'user'

class UserCreateView(CreateView):
    """創建用戶視圖"""
    model = User
    template_name = 'myapp/user_form.html'
    fields = ['name', 'email']
    success_url = reverse_lazy('user_list')

class UserUpdateView(UpdateView):
    """更新用戶視圖"""
    model = User
    template_name = 'myapp/user_form.html'
    fields = ['name', 'email']
    success_url = reverse_lazy('user_list')

class UserDeleteView(DeleteView):
    """刪除用戶視圖"""
    model = User
    template_name = 'myapp/user_confirm_delete.html'
    success_url = reverse_lazy('user_list')
```

### 2. 表單處理

```python
# myapp/forms.py
from django import forms
from .models import User

class UserForm(forms.ModelForm):
    """用戶表單"""
    class Meta:
        model = User
        fields = ['name', 'email']
        widgets = {
            'name': forms.TextInput(attrs={'class': 'form-control'}),
            'email': forms.EmailInput(attrs={'class': 'form-control'}),
        }
    
    def clean_email(self):
        """驗證郵箱"""
        email = self.cleaned_data['email']
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError('This email is already registered.')
        return email

# myapp/views.py
from django.shortcuts import render, redirect
from .forms import UserForm

def create_user(request):
    """創建用戶視圖"""
    if request.method == 'POST':
        form = UserForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('user_list')
    else:
        form = UserForm()
    return render(request, 'myapp/user_form.html', {'form': form})
```

### 3. 中間件

```python
# myapp/middleware.py
import time
from django.http import HttpResponse
from django.urls import reverse

class TimingMiddleware:
    """計時中間件"""
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # 處理請求前
        start_time = time.time()
        
        # 處理請求
        response = self.get_response(request)
        
        # 處理響應後
        duration = time.time() - start_time
        print(f"Request to {request.path} took {duration:.2f} seconds")
        
        return response

class AuthMiddleware:
    """認證中間件"""
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # 檢查是否需要認證
        if request.path.startswith('/admin/') and not request.user.is_authenticated:
            return HttpResponse('Please login first', status=401)
        
        return self.get_response(request)

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'myapp.middleware.TimingMiddleware',
    'myapp.middleware.AuthMiddleware',
]
```

### 4. 信號處理

```python
# myapp/signals.py
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import User
import logging

logger = logging.getLogger(__name__)

@receiver(post_save, sender=User)
def user_saved(sender, instance, created, **kwargs):
    """用戶保存信號處理"""
    if created:
        logger.info(f"New user created: {instance.name}")
    else:
        logger.info(f"User updated: {instance.name}")

@receiver(post_delete, sender=User)
def user_deleted(sender, instance, **kwargs):
    """用戶刪除信號處理"""
    logger.info(f"User deleted: {instance.name}")

# myapp/apps.py
from django.apps import AppConfig

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        """應用啟動時加載信號"""
        import myapp.signals
```

### 5. 緩存處理

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}

# myapp/views.py
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

@method_decorator(cache_page(60 * 15), name='dispatch')
class UserListView(ListView):
    """緩存的用戶列表視圖"""
    model = User
    template_name = 'myapp/user_list.html'
    context_object_name = 'users'

# myapp/models.py
from django.core.cache import cache

class User(models.Model):
    """用戶模型"""
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    
    def save(self, *args, **kwargs):
        """保存時更新緩存"""
        super().save(*args, **kwargs)
        cache.delete('user_list')
    
    def delete(self, *args, **kwargs):
        """刪除時更新緩存"""
        super().delete(*args, **kwargs)
        cache.delete('user_list')
```

## 練習題

1. **類視圖**
   開發類視圖：
   - 實現CRUD操作
   - 處理表單
   - 優化查詢
   - 提供分頁

2. **中間件**
   創建中間件：
   - 處理請求響應
   - 實現認證
   - 優化性能
   - 提供監控

3. **信號處理**
   實現信號處理：
   - 監聽模型事件
   - 處理業務邏輯
   - 優化性能
   - 提供日誌

## 小提醒 💡

1. 類視圖
   - 選擇合適視圖
   - 處理表單
   - 優化查詢
   - 提供分頁

2. 中間件
   - 處理請求
   - 實現認證
   - 優化性能
   - 提供監控

3. 信號處理
   - 監聽事件
   - 處理邏輯
   - 優化性能
   - 提供日誌

4. 調試技巧
   - 使用開發工具
   - 分析性能
   - 優化關鍵路徑
   - 監控應用狀態

[上一章：Django框架基礎](092_Django框架基礎.md) | [下一章：數據分析基礎](094_數據分析基礎.md) 