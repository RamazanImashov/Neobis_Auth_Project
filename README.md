## Документация для проекта "Регистрация и управление пользователями"

### 1. Общая информация о проекте

**Название проекта:** Регистрация и управление пользователями

**Цель проекта:**
Создать API для управления пользователями, включая регистрацию, активацию, авторизацию с JWT-токенами, смену пароля, восстановление пароля, и выход. Проект построен на **Django Rest Framework (DRF)** и использует **JWT** для аутентификации, предоставляя безопасное и масштабируемое решение для пользовательских данных и управления доступом.

### 2. Архитектура и основные компоненты проекта

Проект состоит из следующих основных частей:

#### 2.1. Управление пользователями
- **Регистрация**: Пользователи могут зарегистрироваться с валидацией по email.
- **Активация аккаунта**: Активация учетной записи через уникальный код, отправляемый на email.
- **Авторизация**: Использование JWT токенов для входа и обновления токенов.
- **Управление паролем**: Возможность смены и восстановления пароля для пользователей.
  
#### 2.2. Поддержка API-документации
- **Swagger (drf-spectacular)**: Подключен для генерации документации и тестирования API.

### 3. Стек технологий

- **Backend**: Django Rest Framework (DRF)
- **Аутентификация**: JWT (rest_framework_simplejwt)
- **Документация**: drf-spectacular для генерации документации Swagger
- **База данных**: PostgreSQL (по умолчанию Django ORM может быть легко перенастроен на другую базу данных)
  
---

### 4. Основные функции и эндпоинты API

Эндпоинты сгруппированы по функционалу и доступны в `account.urls`:

#### 4.1. Регистрация и активация
- `POST /register/`: Регистрация нового пользователя.
- `POST /activate/<str:email>/<str:activation_code>/`: Активация учетной записи с использованием email и уникального кода.
- `POST /activate_code/`: Альтернативный эндпоинт активации учетной записи.

#### 4.2. Аутентификация и управление сессией
- `POST /login/`: Получение JWT-токена для аутентификации (обработка через `TokenObtainPairView`).
- `POST /refresh/`: Обновление JWT-токена.
- `POST /logout/`: Выход из системы и аннулирование токена.

#### 4.3. Управление паролем
- `POST /change_password/`: Смена пароля для авторизованных пользователей.
- `POST /lose_password/`: Запрос на сброс пароля (отправка кода подтверждения).
- `POST /lose_confirm_code/`: Подтверждение нового пароля с использованием кода, полученного по email.

### 5. Подробное описание эндпоинтов

#### Регистрация
- **URL**: `/register/`
- **Метод**: `POST`
- **Описание**: Регистрация пользователя. После успешной регистрации на указанный email отправляется код для активации.
- **Поля**: 
  - `email`: Адрес электронной почты пользователя.
  - `password`: Пароль пользователя.
  - `password_confirm`: Подтверждение пароля.

#### Активация по коду
- **URL**: `/activate/<str:email>/<str:activation_code>/`
- **Метод**: `POST`
- **Описание**: Подтверждение регистрации путем ввода кода активации, отправленного на email.

#### Авторизация (JWT-токен)
- **URL**: `/login/`
- **Метод**: `POST`
- **Описание**: Генерация JWT-токена для аутентификации.
- **Поля**:
  - `username`: Имя пользователя или email.
  - `password`: Пароль пользователя.

#### Смена пароля
- **URL**: `/change_password/`
- **Метод**: `POST`
- **Описание**: Изменение текущего пароля. Пользователь должен быть авторизован.

#### Сброс пароля
- **URL**: `/lose_password/`
- **Метод**: `POST`
- **Описание**: Запрос на сброс пароля, отправляется код подтверждения на email.
  
---

### 6. Пример использования в Django-проекте

**Файл `urls.py` (настроен для подключения приложения `account`):**
```python
from django.contrib import admin
from django.urls import path, include
from drf_spectacular.views import SpectacularSwaggerView, SpectacularAPIView
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path("admin/", admin.site.urls),
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='docs'),
    path('api/v1/account/', include('account.urls')),
]

urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

**Эндпоинты в `account/urls.py`:**
```python
from django.urls import path
from .views import (
    RegisterView, ActivationViewCode,
    LogoutView, ChangePasswordView, ForgotPasswordView,
    ForgotPasswordCompleteView, ActivationViewDjCode,
)
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path("register/", RegisterView.as_view()),
    path("activate/<str:email>/<str:activation_code>/", ActivationViewDjCode.as_view(), name="activate"),
    path("activate_code/", ActivationViewCode.as_view()),
    path("login/", TokenObtainPairView.as_view(), name="token_obtain_pair"),
    path("refresh/", TokenRefreshView.as_view(), name="token_refresh"),
    path("logout/", LogoutView.as_view(), name="auth_logout"),
    path("change_password/", ChangePasswordView.as_view()),
    path("lose_password/", ForgotPasswordView.as_view()),
    path("lose_confirm_code/", ForgotPasswordCompleteView.as_view(), name="forgot"),
]
```

### 7. Переменные окружения

Для работы проекта используются переменные окружения, которые могут быть настроены в `.env` файле:

```env
SECRET_KEY=your-secret-key
DEBUG=True
DB_NAME=your-database-name
DB_HOST=database-host
DB_USER=database-user
DB_PASS=database-password
EMAIL_HOST_USER=your-email@example.com
EMAIL_HOST_PASSWORD=email-password
```

---

### 8. Примечания

1. **Валидация и безопасность**: Все методы регистрации и аутентификации проходят валидацию. JWT токены обеспечивают защиту данных и позволяют легко управлять сессиями пользователей.
2. **API-документация**: Swagger-документация доступна по адресу `/docs/`, предоставляя интерфейс для тестирования API.

---

### Заключение

Проект **"Регистрация и управление пользователями"** предоставляет API для безопасного управления пользователями с использованием современных методов аутентификации и восстановления доступа.
