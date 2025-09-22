# Django 5.2 – Trabalhando com Formulários (Resumo)

Este README resume os principais pontos da documentação oficial de formulários do Django 5.2, traduzidos e organizados para consulta rápida. Inclui também exemplos práticos de uso com validação, customização e integração.

---

## 🌍 Pilares Básicos

- Formulários HTML usam `<form>` com campos (`<input>`, `<textarea>`, etc.).
- **GET** → ações seguras (ex: buscas).
- **POST** → ações que modificam estado (ex: salvar no banco).
- **Proteção CSRF** obrigatória em POST → `{% csrf_token %}` no template.

---

## 🏗️ Classes de Formulário no Django

- **`forms.Form`** → define formulários manuais, campos e validações.
- **`forms.ModelForm`** → vinculado a um modelo Django, gera campos automaticamente.
- **Fields** → validam dados, convertem tipos e geram HTML via *widgets*.

---

## 🔄 Ciclo de Vida de um Formulário

1. **GET** → renderiza formulário vazio.
2. **POST** → instancia com dados submetidos (`request.POST` / `request.FILES`).
3. **Validação** → `form.is_valid()` → se válido, usar `form.cleaned_data`; senão, exibir erros.
4. **Renderização** → no template via `{{ form }}`, `{{ form.as_p }}` ou campo por campo.

---

## 🎨 Renderização e Customização

- **Padrão**: `{{ form.as_p }}`, `{{ form.as_ul }}`, `{{ form.as_table }}`.
- **Manual (recomendado)**: iterar sobre `form` ou `form.fields` para aplicar HTML/CSS customizados.
- **Widgets**: alteram a saída HTML (ex: `PasswordInput`, `EmailInput`).
- **Renderers**: controlam globalmente a forma como formulários são renderizados.

Atributos úteis em templates:
- `field.errors` → erros do campo.
- `field.label_tag` → `<label>` associado.
- `field.help_text` → texto de ajuda.
- `form.non_field_errors` → erros de validação geral.

---

## ✅ Validação

- **`clean_<field>()`** → valida campo individual.
- **`clean()`** → validação envolvendo múltiplos campos.
- **`ValidationError`** → para invalidar com mensagens amigáveis.
- **Ordem**: validação de campo → `clean()` → erros → `form.errors`.

---

## ⚠️ Gerenciamento de Erros

- Erros de campo → aparecem junto ao campo.
- Erros gerais → via `form.non_field_errors()`.
- Mensagens podem ser únicas, listas ou dicionários.

---

## 📂 Upload de Arquivos

- Necessário usar **`request.FILES`** junto de `request.POST`.
- Campos como `FileField` e `ImageField` lidam automaticamente com uploads.

---

## 🛠️ Dicas Úteis

- Formulários não incluem `<form>` ou `<button>` → você deve adicionar no template.
- É possível iterar sobre `form.visible_fields()` e `form.hidden_fields()`.
- Validações personalizadas podem ser combinadas com validador(es) de campo.

---

## 💡 Exemplos Práticos

### 1. Formulário Customizado (forms.Form)
```python
from django import forms
from django.core.exceptions import ValidationError

class RegisterForm(forms.Form):
    username = forms.CharField(max_length=30)
    email = forms.EmailField()
    password = forms.CharField(widget=forms.PasswordInput)
    confirm_password = forms.CharField(widget=forms.PasswordInput)

    def clean_username(self):
        if " " in self.cleaned_data["username"]:
            raise ValidationError("Usuário não pode conter espaços.")
        return self.cleaned_data["username"]

    def clean(self):
        cleaned = super().clean()
        if cleaned.get("password") != cleaned.get("confirm_password"):
            raise ValidationError("As senhas não coincidem.")
        return cleaned


# Resumo da Documentação de Formulários no Django 5.2

## Principais conceitos

### 1. Formulários HTML básicos

-   Elementos como `<input>`, `<select>`, `<textarea>` etc. são usados
    para capturar dados do usuário.\
-   Dois atributos essenciais no `<form>`:
    -   `action`: para qual URL os dados serão enviados.\
    -   `method`: normalmente `GET` ou `POST`.

### 2. Uso de GET vs POST

-   **POST**: para requisições que mudam o estado do sistema, envolvem
    dados sensíveis ou muitas informações.\
-   **GET**: para buscas ou situações onde não há alteração do estado.
    URLs são reproduzíveis e podem ser salvas em favoritos, mas são
    menos seguras para dados sensíveis.

### 3. Classe `Form` do Django

-   Define os campos, tipos, rótulos e validações do formulário.\
-   `ModelForm`: facilita criar formulários baseados em modelos do banco
    de dados.

### 4. Instanciação, renderização e processamento

-   Formulários "vazios" (unbound) e formulários com dados (bound) após
    submissão.\
-   Fluxo em uma view:
    -   Exibir formulário vazio para GET.\
    -   Popular com `request.POST` para POST e chamar `is_valid()`.\
    -   Se válido: usar `cleaned_data` para processar.\
    -   Se inválido: retornar formulário ao template com erros.

### 5. Templates para formulários

-   Renderizar com `{{ form }}` ou campos individualmente.\
-   Em requisições POST: necessário `{% csrf_token %}` no template para
    proteção contra CSRF.

### 6. Widgets, validação e tipo de dado

-   Cada campo tem um widget padrão (ex: `CharField` → `TextInput`).\
-   É possível customizar widgets.\
-   Campos como `EmailField`, `URLField`, `IntegerField` validam e
    convertem dados automaticamente para tipos Python.

### 7. Erros de formulário

-   **Erros de campo**: exibidos próximos ao campo correspondente.\
-   **Erros não relacionados a campo (non-field errors)**: exibidos
    separadamente.

### 8. Looping sobre campos no template

-   Possível iterar sobre os campos (`for field in form`) para
    renderização personalizada.\
-   Separar campos visíveis de ocultos: `form.visible_fields()` e
    `form.hidden_fields()`.
