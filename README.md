# Organoid-Analyzer
我来指导您如何运行这个项目。

1. 首先创建并激活虚拟环境：

```bash
# 创建虚拟环境
python -m venv venv

# 在Windows上激活虚拟环境
venv\Scripts\activate

# 在Linux/Mac上激活虚拟环境
source venv/bin/activate
```

2. 安装所需依赖，创建 `requirements.txt`：

```python:requirements.txt
Django==4.2.7
djangorestframework==3.14.0
djangorestframework-simplejwt==5.3.0
django-cors-headers==4.3.0
opencv-python==4.8.1.78
numpy==1.26.2
python-magic==0.4.27
Pillow==10.1.0
psycopg2-binary==2.9.9
```

3. 安装依赖：

```bash
pip install -r requirements.txt
```

4. 创建项目目录结构：

```bash
organoid_analyzer/
├── manage.py
├── requirements.txt
├── organoid_analyzer/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── api/
│   ├── __init__.py
│   ├── views.py
│   ├── serializers.py
│   ├── permissions.py
│   └── middleware.py
├── users/
│   ├── __init__.py
│   └── models.py
├── data/
│   ├── __init__.py
│   └── models.py
└── plugins/
    ├── __init__.py
    ├── models.py
    ├── plugin_manager.py
    ├── base_plugin.py
    └── examples/
        ├── __init__.py
        ├── image_enhancement.py
        └── organoid_counter.py
```

5. 配置数据库，在 `settings.py` 中添加：

```python:organoid_analyzer/settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'organoid_db',
        'USER': 'postgres',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

# 如果您想使用SQLite进行测试，可以使用：
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

6. 初始化数据库：

```bash
# 创建数据库迁移
python manage.py makemigrations users
python manage.py makemigrations data
python manage.py makemigrations plugins

# 应用迁移
python manage.py migrate

# 创建超级用户
python manage.py createsuperuser
```

7. 注册默认插件，创建初始化脚本：

```python:organoid_analyzer/plugins/management/commands/init_plugins.py
from django.core.management.base import BaseCommand
from plugins.plugin_manager import PluginManager
from plugins.examples.organoid_counter import OrganoidCounterPlugin
from plugins.examples.image_enhancement import ImageEnhancementPlugin

class Command(BaseCommand):
    help = '初始化默认插件'

    def handle(self, *args, **kwargs):
        plugin_manager = PluginManager()
        
        # 注册默认插件
        plugin_manager.register_plugin(OrganoidCounterPlugin)
        plugin_manager.register_plugin(ImageEnhancementPlugin)
        
        self.stdout.write(self.style.SUCCESS('成功初始化默认插件'))
```

8. 运行初始化插件命令：

```bash
python manage.py init_plugins
```

9. 运行开发服务器：

```bash
python manage.py runserver
```

10. 测试API：

```bash
# 获取JWT令牌
curl -X POST http://localhost:8000/api/token/ \
     -H "Content-Type: application/json" \
     -d '{"username": "admin", "password": "your_password"}'

# 使用令牌访问API
curl -X GET http://localhost:8000/api/datasets/ \
     -H "Authorization: Bearer your_jwt_token"

# 上传图像并应用插件
curl -X POST http://localhost:8000/api/files/1/apply_plugin/ \
     -H "Authorization: Bearer your_jwt_token" \
     -H "Content-Type: application/json" \
     -d '{"plugin_name": "organoid_counter"}'
```

11. 创建测试数据：

```python:organoid_analyzer/data/management/commands/create_test_data.py
from django.core.management.base import BaseCommand
from django.contrib.auth import get_user_model
from data.models import Dataset, DataFile

User = get_user_model()

class Command(BaseCommand):
    help = '创建测试数据'

    def handle(self, *args, **kwargs):
        # 创建测试用户
        user, created = User.objects.get_or_create(
            username='researcher',
            defaults={
                'email': 'researcher@example.com',
                'role': 'researcher'
            }
        )
        if created:
            user.set_password('test123')
            user.save()
            
        # 创建测试数据集
        dataset = Dataset.objects.create(
            name='测试数据集',
            description='用于测试的类器官数据集',
            created_by=user
        )
        
        self.stdout.write(self.style.SUCCESS('成功创建测试数据'))
```

12. 运行测试数据创建命令：

```bash
python manage.py create_test_data
```

主要API端点：

- `POST /api/token/` - 获取JWT令牌
- `GET /api/datasets/` - 获取数据集列表
- `POST /api/datasets/{id}/upload_files/` - 上传文件到数据集
- `POST /api/files/{id}/apply_plugin/` - 对文件应用插件分析

需要我详细解释任何部分吗？或者需要其他帮助？
