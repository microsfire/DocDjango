## Guia Completo Django: Formulários, Validação, Bootstrap, ModelForm e Allauth

---

# 1. Formulários e Validação Customizada

## forms.py (Form manual)
```python
from django import forms
from django.core.exceptions import ValidationError

class RegisterForm(forms.Form):
    username = forms.CharField(
        label="Usuário",
        max_length=30,
        widget=forms.TextInput(attrs={"class": "form-control", "placeholder": "Digite seu usuário"})
    )
    email = forms.EmailField(
        label="E-mail",
        widget=forms.EmailInput(attrs={"class": "form-control", "placeholder": "Digite seu e-mail"})
    )
    password = forms.CharField(
        label="Senha",
        widget=forms.PasswordInput(attrs={"class": "form-control", "placeholder": "Digite sua senha"})
    )
    confirm_password = forms.CharField(
        label="Confirmar Senha",
        widget=forms.PasswordInput(attrs={"class": "form-control", "placeholder": "Confirme sua senha"})
    )

    def clean_username(self):
        username = self.cleaned_data.get("username")
        if " " in username:
            raise ValidationError("O nome de usuário não pode conter espaços.")
        return username

    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get("password")
        confirm_password = cleaned_data.get("confirm_password")

        if password and confirm_password and password != confirm_password:
            raise ValidationError("As senhas não coincidem.")
        return cleaned_data
```

## views.py
```python
from django.shortcuts import render
from .forms import RegisterForm

def register_view(request):
    if request.method == "POST":
        form = RegisterForm(request.POST)
        if form.is_valid():
            return render(request, "success.html", {"user": form.cleaned_data["username"]})
    else:
        form = RegisterForm()

    return render(request, "register.html", {"form": form})
```

## Template register.html (Bootstrap)
```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Cadastro</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="container mt-5">

  <h2>Formulário de Cadastro</h2>
  <form method="POST" class="mt-3">
    {% csrf_token %}

    {% if form.non_field_errors %}
      <div class="alert alert-danger">
        {{ form.non_field_errors }}
      </div>
    {% endif %}

    {% for field in form %}
      <div class="mb-3">
        {{ field.label_tag }}
        {{ field }}
        <div class="text-danger">{{ field.errors }}</div>
      </div>
    {% endfor %}

    <button type="submit" class="btn btn-primary">Cadastrar</button>
  </form>

</body>
</html>
```

---

# 2. ModelForm com Django User

## forms.py (ModelForm)
```python
from django import forms
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError

class RegisterForm(forms.ModelForm):
    password = forms.CharField(
        label="Senha",
        widget=forms.PasswordInput(attrs={"class": "form-control", "placeholder": "Digite sua senha"})
    )
    confirm_password = forms.CharField(
        label="Confirmar Senha",
        widget=forms.PasswordInput(attrs={"class": "form-control", "placeholder": "Confirme sua senha"})
    )

    class Meta:
        model = User
        fields = ["username", "email", "password"]
        widgets = {
            "username": forms.TextInput(attrs={"class": "form-control", "placeholder": "Digite seu usuário"}),
            "email": forms.EmailInput(attrs={"class": "form-control", "placeholder": "Digite seu e-mail"}),
        }

    def clean_username(self):
        username = self.cleaned_data.get("username")
        if " " in username:
            raise ValidationError("O nome de usuário não pode conter espaços.")
        if User.objects.filter(username=username).exists():
            raise ValidationError("Este nome de usuário já está em uso.")
        return username

    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get("password")
        confirm_password = cleaned_data.get("confirm_password")

        if password and confirm_password and password != confirm_password:
            raise ValidationError("As senhas não coincidem.")
        return cleaned_data

    def save(self, commit=True):
        user = super().save(commit=False)
        user.set_password(self.cleaned_data["password"])
        if commit:
            user.save()
        return user
```

## views.py
```python
from django.shortcuts import render, redirect
from .forms import RegisterForm

def register_view(request):
    if request.method == "POST":
        form = RegisterForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("login")
    else:
        form = RegisterForm()

    return render(request, "register.html", {"form": form})
```

---

# 3. Integração com Django Allauth

## Instalação
```bash
pip install django-allauth
```

## settings.py
```python
INSTALLED_APPS = [
    ...
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
    'allauth.socialaccount.providers.github',
    ...
]

AUTHENTICATION_BACKENDS = (
    'allauth.account.auth_backends.AuthenticationBackend',
    'django.contrib.auth.backends.ModelBackend',
)

SITE_ID = 1

ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_EMAIL_VERIFICATION = 'mandatory'
ACCOUNT_AUTHENTICATED_REDIRECT_URL = '/'
ACCOUNT_LOGOUT_REDIRECT_URL = '/'
ACCOUNT_SIGNUP_REDIRECT_URL = '/'
ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION = True
```

## urls.py
```python
from django.urls import path, include

urlpatterns = [
    path('accounts/', include('allauth.urls')),
    ...
]
```

## Template login.html
```html
{% extends 'base_generic.html' %}

{% block content %}
  <h2>Login</h2>
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Entrar</button>
  </form>
  <p><a href="{% url 'account:password_reset' %}">Esqueceu sua senha?</a></p>
{% endblock %}
```

## Social login
1. Adicione Social Applications no Django Admin.
2. Configure Client ID e Secret Key dos provedores.
3. Associe à(s) Site(s) corretos.
```
---

Este arquivo serve como **guia completo** para:
- Formulários Django manuais
- ModelForm com validação e hash de senha
- Integração com Bootstrap
- Registro/Login via Django Allauth

