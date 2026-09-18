# Pipeline-Python  
# Pipeline CI на Python в GitHub Actions
### Цель — создать учебный пример CI для простого Python-приложения

Вы научитесь:

- Настроить CI для Python проектов
- Научиться контейнеризировать приложения с Docker
- Сборку Docker-образа
- Сохранение артефактов для локального использования
- CI (Continuous Integration - непрерывная интеграция)

Автоматически проверяет код при каждом push/PR:
линтинг (flake8) - автоматическая проверка исходного кода
тесты (pytest)
сборка Docker-образа для проверки (без публикации)
## 1. Создайте на GitHub новый публичный репозиторий my-python-app с README.md
Склонируйте его себе, откройте в VS Code и создайте командой структуру будущего проекта:

        mkdir -p .github/workflows myapp tests && touch .github/workflows/ci.yml myapp/{__init__.py,app.py} tests/test_app.py setup.py requirements.txt Dockerfile README.md

## 2. Файл myapp/app.py
```
def add(a: int, b: int) -> int:
    """Возвращает сумму двух чисел."""
    return a + b

def main():
    print("Hello from my Python app!")

if __name__ == "__main__":
    main()
```

## 3. Файл setup.py для включения development mode
```

```
## 4. Файл tests/test_app.py
```
from myapp.app import add

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```
## 5. Файл requirements.txt
```
pytest
flake8
```
## 6. Файл Dockerfile
```
# Используем официальный образ Python
FROM python:3.11-slim
# Устанавливаем рабочую директорию
WORKDIR /app
# Копируем файл с зависимостями
COPY requirements.txt .
# Устанавливаем зависимости
RUN pip install --no-cache-dir -r requirements.txt
# Копируем весь проект
COPY . .
# Команда по умолчанию (запуск приложения)
CMD ["python", "myapp/app.py"]
```
## 7. Файл .github/workflows/ci.yml
```
name: CI for Python App

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    name: Lint & Test
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11", "3.12"]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 pytest
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Install package in development mode
        run: pip install -e .

      - name: Lint with flake8
        run: |
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

      - name: Test with pytest
        run: pytest tests/

  docker-build:
    name: Build Docker Image (no push)
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t my-python-app:test .
```

![alt text](image.png)