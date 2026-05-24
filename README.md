# Medical Records Management System (userver)

REST API сервис для управления медицинскими записями на фреймворке **userver** (Вариант 20).

## Функциональность

- Регистрация и аутентификация пользователей (JWT)
- Поиск пользователей по маске имени
- Регистрация пациентов
- Поиск пациентов по ФИО
- Создание медицинских записей
- Получение истории записей пациента
- Получение записи по уникальному коду

## Технологии

- C++20
- **userver** (асинхронный фреймворк Яндекса)
- JWT аутентификация
- In-memory хранение данных
- Docker

## Требования

- Docker (для простого запуска)
- Или Linux с установленным userver (для разработки)

## Быстрый старт (Docker)

Самый простой способ запустить сервис - использовать Docker.

### Windows / macOS / Linux

1. Установите Docker Desktop с официального сайта
2. Склонируйте репозиторий:
   ```
   git clone https://github.com/Chigatu/medical-records-system-userver.git
   cd medical-records-system-userver
   ```
3. Запустите сервис:
   ```
   docker-compose up --build
   ```
4. Проверьте работоспособность:
   ```
   curl http://localhost:8080/health
   ```

## Разработка (с использованием Docker)

```
# Запустить контейнер с userver
docker run --rm -it \
  --user "$(id -u):$(id -g)" \
  --security-opt seccomp=unconfined \
  -p 8080:8080 \
  -v "$PWD":/work \
  -w /work \
  --entrypoint bash \
  ghcr.io/userver-framework/ubuntu-22.04-userver:latest

# Внутри контейнера
cd medical-api
export CCACHE_DISABLE=1
make build-debug
./build-debug/medical_api --config configs/static_config.yaml --config_vars configs/config_vars.yaml
```

## API Endpoints

| Метод | URL | Описание | Требуется JWT |
|-------|-----|----------|---------------|
| GET | /health | Проверка работоспособности | Нет |
| POST | /api/auth/register | Регистрация пользователя | Нет |
| POST | /api/auth/login | Вход в систему | Нет |
| POST | /api/patients | Создание пациента | Да |
| GET | /api/patients/search?fullName={name} | Поиск пациентов по ФИО | Нет |
| POST | /api/medical-records | Создание медицинской записи | Да |
| GET | /api/medical-records/patient/{id} | Получение истории записей пациента | Да |

## Примеры запросов

### 1. Регистрация пользователя

```
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "login": "dr.ivanov",
    "email": "ivanov@hospital.ru",
    "firstName": "Иван",
    "lastName": "Иванов",
    "password": "secret123"
  }'
```

### 2. Вход в систему

```
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"login":"dr.ivanov","password":"secret123"}'
```

### 3. Создание пациента (требуется JWT)

```
TOKEN="your-token-from-login"

curl -X POST http://localhost:8080/api/patients \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "firstName": "Мария",
    "lastName": "Петрова",
    "phone": "+79991234567",
    "address": "г. Москва",
    "birthDate": "1990-05-15",
    "snils": "123-456-789 01",
    "policyNumber": "1234567890123456"
  }'
```

### 4. Создание медицинской записи (требуется JWT)

```
curl -X POST http://localhost:8080/api/medical-records \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "patientId": 1,
    "doctorId": 1,
    "diagnosisCode": "J06.9",
    "diagnosisDescription": "Острая респираторная инфекция",
    "complaints": "Кашель, температура 38.5"
  }'
```

## Тестирование

```
./test_api_userver.sh
```

## Структура проекта

```
medical-api/
├── src/
│   ├── handlers/        # HTTP обработчики
│   ├── models/          # Доменные модели
│   ├── repositories/    # In-memory репозитории
│   ├── services/        # Бизнес-логика
│   ├── dto/             # Data Transfer Objects
│   └── main.cpp         # Точка входа
├── configs/             # Конфигурация userver
├── CMakeLists.txt       # Сборка
├── Dockerfile           # Docker образ
├── docker-compose.yaml  # Docker Compose
├── test_api_userver.sh  # Автотесты
└── README.md            # Документация
```
