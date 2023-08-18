# Case-Sensitive-Usernames

Enforcing case sensitivity in usernames requires modifying your Django authentication system to ensure that usernames are treated as case-sensitive. By default, Django's authentication system treats usernames as case-insensitive. Here's how you can make usernames case-sensitive:

1. **Custom User Model**:
   If you are using Django's default `User` model for authentication, you will need to create a custom user model. This allows you to override the username field's behavior.

2. **Custom Username Field**:
   Create a custom username field by inheriting from `models.CharField` and overriding its `db_index` and `unique` attributes. Set `db_index` to `True` to improve lookup efficiency, and set `unique` to `True` to enforce uniqueness (including case sensitivity).

3. **Update Authentication Backend**:
   Modify your authentication backend to use the case-sensitive username field for authentication.

Here's a general guide to implementing case-sensitive usernames:

**Step 1: Create a Custom User Model**

```python
# models.py

from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.db import models

class CustomUserManager(BaseUserManager):
    def create_user(self, username, password=None, **extra_fields):
        if not username:
            raise ValueError('The Username field must be set')
        username = self.model.normalize_username(username)
        user = self.model(username=username, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, username, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self.create_user(username, password, **extra_fields)

class CustomUser(AbstractUser):
    username = models.CharField(
        max_length=30,
        unique=True,  # Enforces case-sensitive uniqueness
        db_index=True,  # Improves lookup efficiency
    )
    
    # Add any other fields you need
    
    objects = CustomUserManager()
```

---

**Step 2: Update Authentication Backend**

Create a custom authentication backend to use the case-sensitive username for authentication:

```python
# authentication.py

from django.contrib.auth.backends import ModelBackend
from django.contrib.auth import get_user_model

class CaseSensitiveModelBackend(ModelBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        UserModel = get_user_model()
        try:
            user = UserModel.objects.get(username=username)
        except UserModel.DoesNotExist:
            return None
        if user.check_password(password):
            return user
        return None
```

----

**Step 3: Use the Custom Authentication Backend**

In your `settings.py`, specify your custom authentication backend:

```python
# settings.py

AUTHENTICATION_BACKENDS = ['your_app.authentication.CaseSensitiveModelBackend']
```

Replace `'your_app'` with the actual name of your app where you've defined the custom authentication backend.


After implementing these steps, your Django application will treat usernames as case-sensitive, preventing the insertion of "Mena" if "mena" already exists. Make sure to adjust the code according to your project structure and specific requirements.

---

Happy coding :computer: :clinking_glasses:

Best regards,

Yara Elmalah 😊

In the world of code, we unleash our Nen, channeling our aura into transformative creations :sauropod: :fire:	
