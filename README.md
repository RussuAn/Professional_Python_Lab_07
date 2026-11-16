# Система обліку студентів

Лабораторна робота №7 «Робота з базами даних у Python»  
Курс «Професійний Python», варіант №1  
Руссу Анастасія Сергіївна, ФЕП-32  
Викладач: Свелеба С. А.

## Опис роботи

Програма продовжує систему обліку студентів із лабораторної №1 та зберігає студентів і групи в SQLite. Реалізовано додавання, виведення, пошук за прізвищем і групою, оновлення бала, переведення до іншої групи, видалення, середній бал групи та найкращого студента. Шар зберігання відокремлено від бізнес-логіки через StudentRepository та GroupRepository. SQLAlchemy ORM використовується для моделей і CRUD, sqlite3 DB-API — для параметризованого пошуку, Alembic — для міграцій.

## Структура проєкту

```text
Lab_07_variant_1/
├── pyproject.toml
├── README.md
├── .gitignore
├── alembic.ini
├── migrations/
│   ├── env.py
│   └── versions/
│       ├── 001_create_tables.py
│       └── 002_add_student_email.py
├── src/student_manager/
│   ├── __init__.py
│   ├── models.py
│   ├── database.py
│   ├── repositories.py
│   ├── db_api.py
│   ├── services.py
│   └── main.py
├── tests/
│   ├── conftest.py
│   ├── test_repositories.py
│   └── test_migrations.py
└── report/
    └── Report_lab_07.docx
```

## Запуск у Windows

У PowerShell відкрийте папку `Lab_07_variant_1`, яка містить `pyproject.toml`, та послідовно виконайте:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e ".[test]"
.\.venv\Scripts\python.exe -m alembic upgrade head
.\.venv\Scripts\python.exe -m student_manager.main
```

## Тести

```powershell
.\.venv\Scripts\python.exe -m pytest -v
```

Тести використовують **окремі тимчасові SQLite бази** (`tmp_path`), перевіряють CRUD, PRIMARY/FOREIGN KEY, CHECK/UNIQUE, пошук, агрегацію та rollback масового переведення.

## Міграції

Файл `001_create_tables.py` створює таблиці `groups` та `students`. Файл `002_add_student_email.py` додає поле `email` та обмеження `UNIQUE`.

```powershell
.\.venv\Scripts\python.exe -m alembic current
.\.venv\Scripts\python.exe -m alembic downgrade 001_initial
.\.venv\Scripts\python.exe -m alembic upgrade head
```

## Обробка даних

`groups.id` та `students.id` є первинними ключами; `students.group_id` посилається на `groups.id`. Середній бал допускається в межах 0–100, порожні імена та назви груп заборонені. Для групи без студентів середній бал дорівнює `None`. Масове переведення виконується в одній транзакції: або всі зміни зберігаються через `commit`, або відхиляються через `rollback`.
