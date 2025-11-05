Проект: Weather Reminder (Django + DRF + PostgreSQL + aiogram)
Что это

Телеграм-бот принимает подписку на ежедневный прогноз погоды по городу и времени. Django хранит подписки и даёт REST API. По расписанию скрипт забирает список активных подписок, ходит в OpenWeather и рассылает прогнозы пользователям в TG.

Стек

Django + Django REST Framework

PostgreSQL

aiogram

requests (для OpenWeather)

(опционально в конце) docker-compose

Архитектура репозитория
weather-reminder/
  backend/
    manage.py
    backend/                # settings, urls
    subscriptions/          # app: модели, API, команды
      models.py
      serializers.py
      views.py
      urls.py
      management/commands/send_weather.py
    requirements.txt
    .env.example
  bot/
    bot.py
    requirements.txt
    .env.example
  README.md
  docker-compose.yml        # stretch

База данных: минимальная схема

Subscriber
tg_user_id: BigInteger (unique)
city: CharField(100)
time_local: TimeField (когда отправлять, один раз в день)
active: BooleanField
created_at: DateTimeField(auto_now_add=True)

При желании позже добавишь timezone и историю отправок.

API (DRF) — эндпоинты

POST /api/subscribers/ — создать/обновить подписку
тело: { "tg_user_id": 123, "city": "Vienna", "time_local": "08:30", "active": true }

GET /api/subscribers/?active=true&city=Vienna — список

PATCH /api/subscribers/<id>/ — изменить (город/время/active)

POST /api/notify/preview/ — (dev-ручка) получить текст прогноза для города на сейчас

Аутентификация для бота: простой TokenAuthentication (DRF Token) или секретный хедер.

Команды бота (aiogram)

/start — приветствие + помощь

/subscribe <город> <HH:MM> — создать/обновить подписку (пример: /subscribe Vienna 08:30)

/stop — отключить подписку

/today — прислать прогноз сейчас

/help — подсказка по форматам

Бот стучится в твой Django API:

POST /api/subscribers/ для создания/апдейта

POST /api/notify/preview/ для /today

Секреты и конфиг (.env)

backend/.env:

SECRET_KEY=...
DEBUG=True
DB_NAME=weather
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=localhost
DB_PORT=5432
OPENWEATHER_API_KEY=...
BOT_TOKEN=...          # можно хранить и в bot/.env, но дублировать не надо
API_TOKEN=...          # если выберешь DRF TokenAuth для бота


bot/.env:

BOT_TOKEN=...
API_BASE_URL=http://localhost:8000
API_TOKEN=...