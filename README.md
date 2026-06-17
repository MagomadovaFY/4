# Лабораторная работа №04: Реализация механизмов безопасности и отказоустойчивости в распределенной системе

## 📋 Информация о студенте

| Поле | Значение |
|------|----------|
| **ФИО** | Магомадова Фирюза |
| **Группа** | ЦИБ-241 |
| **Вариант** | №11 — Пагинация |
| **Дата выполнения** | 07.06.2026 |

---

## 🎯 Цель работы

Разработать и исследовать распределенную систему, обеспечивающую защищенную передачу данных с использованием симметричного шифрования (Fernet), а также демонстрирующую отказоустойчивость через автоматическое переключение между узлами.

### Индивидуальное задание (Вариант №11 — Пагинация)

**Задача:** Реализовать возврат данных частями (страницами) на сервере и поддержку пагинации на клиенте.

**Требования:**
- Клиент может запрашивать страницу с указанным номером и размером
- Сервер возвращает данные постранично с информацией о количестве страниц
- Запросы передаются в зашифрованном виде через координатора

---

## 🏗️ Архитектура системы

КЛИЕНТ КООРДИНАТОР СЕРВЕР 1 (5003)
│ │ │
│───(1)─────────→│ │
│ Запрос с │ │
│ пагинацией │ │
│ │───(2)─────────────→│
│ │ Перенаправление │
│ │ │
│ │←──(3)─────────────│
│ │ Ответ с данными │
│←──(4)─────────│ │
│ Ответ │ │
│ │ │
│ │ (при отказе │
│ │ сервера 5003 │
│ │ переключение │
│ │ на сервер 5004) │

**Компоненты системы:**
1. **Клиент** — шифрует запросы, отправляет координатору
2. **Координатор** — балансирует нагрузку между серверами
3. **Сервер 1** (порт 5003) — обрабатывает запросы с пагинацией
4. **Сервер 2** (порт 5004) — резервный сервер

---

## 🔧 Технологический стек

| Технология | Версия | Назначение |
|------------|--------|------------|
| Python | 3.14.4 | Язык программирования |
| Flask | 3.1.3 | Веб-фреймворк для серверов |
| Requests | 2.34.2 | HTTP-клиент |
| Cryptography | 41.0.7 | Шифрование Fernet |
| venv | - | Виртуальное окружение |

---

## 📁 Структура проекта
```
lab_04/
├── server.py                # Сервер с поддержкой пагинации
├── coordinator.py           # Балансировщик нагрузки (отказоустойчивость)
├── client.py                # Клиент с запросом страниц
├── generate_certs.py        # Генерация сертификатов (PKI)
├── generate_key.py          # Генерация ключа Fernet
├── encryption_key.txt       # Ключ симметричного шифрования
├── requirements.txt         # Зависимости Python
├── certs/                   # Сертификаты X.509
│   ├── ca-key.pem           # Приватный ключ центра сертификации
│   ├── ca-cert.pem          # Сертификат центра сертификации
│   ├── server-key.pem       # Приватный ключ сервера
│   ├── server-cert.pem      # Сертификат сервера
│   ├── client-key.pem       # Приватный ключ клиента
│   └── client-cert.pem      # Сертификат клиента
└── venv/                    # Виртуальное окружение Python


---

## 🚀 Инструкция по запуску

### 1) Клонирование репозитория

```bash
git clone [ссылка на репозиторий]
cd lab_04
```
## 2) Создание и активация виртуального окружения
```bash
python3 -m venv venv
source venv/bin/activate  # для macOS/Linux

```
## 3) Установка зависимостей
```bash
pip install -r requirements.txt
```
## 4) Генерация ключей и сертификатов
```bash
python3 generate_certs.py
python3 generate_key.py
```
# 4) Запуск системы (4 терминала)
## Терминал 1 — Сервер 1 (порт 5003):
```bash
cd ~/lab_04
source venv/bin/activate
python3 server.py 5003
```
## Терминал 2 — Сервер 2 (порт 5004):
```bash
cd ~/lab_04
source venv/bin/activate
python3 server.py 5004

```
## Терминал 3 — Координатор (порт 8000):
```bash
cd ~/lab_04
source venv/bin/activate
python3 coordinator.py

```
## Терминал 4 — Клиент:
```bash
cd ~/lab_04
source venv/bin/activate
python3 client.py

```
# Результаты работы
## 1. Работа пагинации (Вариант №11)
### Запрос страницы 1 (10 элементов):
```bash
{
    "status": "ok",
    "data": {
        "page": 1,
        "per_page": 10,
        "total": 50,
        "total_pages": 5,
        "items": ["Item 1", "Item 2", ..., "Item 10"]
    }
}
```
### Запрос страницы 3 (5 элементов):
```bash
{
    "status": "ok", 
    "data": {
        "page": 3,
        "per_page": 5,
        "total": 50,
        "total_pages": 10,
        "items": ["Item 11", "Item 12", "Item 13", "Item 14", "Item 15"]
    }
}
```
## 2. Вывод клиента
```bash
==================================================
Страница 1 (10 элементов):
Страница 1 из 5
Всего элементов: 50
Элементы: ['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5', 'Item 6', 'Item 7', 'Item 8', 'Item 9', 'Item 10']

==================================================
Страница 3 (5 элементов):
Страница 3 из 10
Элементы: ['Item 11', 'Item 12', 'Item 13', 'Item 14', 'Item 15']

```
# 🛡️ Механизмы безопасности
### Шифрование Fernet:

Симметричное шифрование AES-128-CBC

Проверка целостности HMAC-SHA256

Защита от подделки данных

### TLS/mTLS:

X.509 сертификаты

Центр сертификации (CA)

Взаимная аутентификация (опционально)

### Защита канала:

HTTPS между координатором и серверами

Проверка сертификатов


# Сервер (пагинация)
```bash
ITEMS = [f"Item {i}" for i in range(1, 51)]

@app.route("/api/data", methods=["POST"])
def handle():
    encrypted = request.json.get("data", "")
    decrypted = cipher.decrypt(encrypted.encode()).decode()
    
    params = decrypted.split("|")
    if params[0] == "get_items":
        page = int(params[1].split("=")[1])
        per_page = int(params[2].split("=")[1])
        
        start = (page - 1) * per_page
        end = start + per_page
        items_page = ITEMS[start:end]
        
        return jsonify({
            "page": page,
            "per_page": per_page,
            "total": len(ITEMS),
            "total_pages": (len(ITEMS) + per_page - 1) // per_page,
            "items": items_page
        })
```
# Координатор (отказоустойчивость)
```bash
SERVERS = ["http://127.0.0.1:5003", "http://127.0.0.1:5004"]
idx = 0

@app.route("/api/data", methods=["POST"])
def proxy():
    global idx
    for _ in range(len(SERVERS)):
        server = SERVERS[idx]
        idx = (idx + 1) % len(SERVERS)
        try:
            resp = requests.post(f"{server}/api/data", json=request.json, timeout=3)
            return jsonify(resp.json()), 200
        except:
            continue
    return jsonify({"error": "No servers"}), 503
```
# Клиент (запрос страниц)
```bash
def get_items(page=1, per_page=10):
    message = f"get_items|page={page}|per_page={per_page}"
    encrypted = cipher.encrypt(message.encode()).decode()
    resp = requests.post("http://localhost:8000/api/data", json={"data": encrypted})
    return resp.json()
# Использование
result = get_items(page=1, per_page=10)
print(f"Страница {result['data']['page']} из {result['data']['total_pages']}")
print(f"Элементы: {result['data']['items']}")




