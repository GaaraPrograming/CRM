# README.md

# 🧠 CRM Django App

Este proyecto es una aplicación tipo CRM construida con Django. Permite gestionar clientes, compañías, representantes de ventas e interacciones, simulando un entorno realista con medio millón de registros de interacciones.

---

## 🎯 Funcionalidades principales
- Gestión de usuarios personalizados (representantes de ventas).
- Empresas y clientes asociados.
- Medio millón de interacciones simuladas.
- Vista tipo CRM con filtros, ordenamientos y visualización de última interacción.

---

## ⚙️ Requisitos
- Python 3.8+
- Django 4+
- Faker

Instala las dependencias con:
```bash
pip install -r requirements.txt
```

Contenido de `requirements.txt`:
```txt
Django>=4.0
Faker
```

---

## ⚙️ Configuración del modelo personalizado de usuario

Para utilizar el modelo personalizado `User` del proyecto, agrega lo siguiente al `settings.py`:

```python
AUTH_USER_MODEL = 'crm_app.User'
```

> ⚠️ Asegúrate de hacer esto antes de ejecutar `makemigrations` por primera vez.

---

## 🚀 Cómo ejecutar el proyecto

1. Clona el repositorio:
```bash
git clone <url-del-repo>
cd crm_project
```

2. Crea un entorno virtual:
```bash
python -m venv env
source env/bin/activate  # En Windows: env\Scripts\activate
```

3. Instala dependencias:
```bash
pip install -r requirements.txt
```

4. Aplica migraciones:
```bash
python manage.py makemigrations
python manage.py migrate
```

5. Crea superusuario (opcional, para acceder al admin):
```bash
python manage.py createsuperuser
```

6. Pobla la base de datos con datos ficticios:
```bash
python manage.py populate_data
```

> Este comando creará:
> - 3 representantes de ventas
> - 1000 clientes distribuidos en compañías y representantes
> - 500 interacciones por cliente (500,000 en total)

7. Ejecuta el servidor:
```bash
python manage.py runserver
```

---

## 👨‍💻 Comandos incluidos en el proyecto

### `populate_data.py`
Ubicado en `crm_app/management/commands/populate_data.py`, este script genera datos realistas con [Faker](https://faker.readthedocs.io):

```python
from django.core.management.base import BaseCommand
from crm_app.models import User, Company, Customer, Interaction
from faker import Faker
import random
from datetime import datetime, timedelta

class Command(BaseCommand):
    help = "Populate DB with fake data"

    def handle(self, *args, **kwargs):
        fake = Faker()
        Interaction.objects.all().delete()
        Customer.objects.all().delete()
        Company.objects.all().delete()
        User.objects.all().delete()

        users = [User.objects.create_user(username=f"user{i}", password="1234") for i in range(3)]
        companies = [Company.objects.create(name=fake.company()) for _ in range(10)]

        customers = []
        for _ in range(1000):
            customers.append(Customer.objects.create(
                name=fake.name(),
                birth_date=fake.date_of_birth(minimum_age=18, maximum_age=70),
                company=random.choice(companies),
                representative=random.choice(users)
            ))

        interaction_types = ["Call", "Email", "SMS", "Facebook"]
        bulk = []
        for customer in customers:
            for _ in range(500):
                bulk.append(Interaction(
                    customer=customer,
                    interaction_type=random.choice(interaction_types),
                    date=fake.date_time_between(start_date="-1y", end_date="now")
                ))
        Interaction.objects.bulk_create(bulk)
        self.stdout.write(self.style.SUCCESS("Datos generados correctamente."))
```

---

## 👁️ Vista principal (CRM)

Accede a `http://127.0.0.1:8000/` para ver la lista de clientes con:
- Nombre completo
- Compañía
- Cumpleaños (formato: "February 5")
- Última interacción (ej: "1 day ago (Phone)")

### 🔍 Filtros disponibles
- Buscar por nombre
- Filtrar cumpleaños de esta semana

### ↕️ Ordenamientos disponibles
- Nombre
- Compañía
- Cumpleaños
- Fecha de última interacción

---

## 📂 Estructura del proyecto
```
crm_project/
├── crm_app/
│   ├── migrations/
│   ├── templates/crm_app/customer_list.html
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── management/commands/populate_data.py
├── crm_project/
│   ├── settings.py
│   ├── urls.py
├── manage.py
├── requirements.txt
├── README.md
```

---
