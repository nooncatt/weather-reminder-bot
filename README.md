# Проект: Weather Reminder (Django + DRF + PostgreSQL + aiogram)

Telegram-бот + Django REST API для ежедневных уведомлений о погоде.  
Бот принимает подписку на прогноз по городу и времени, а Django хранит данные пользователей и рассылает уведомления через OpenWeather API.
## Стек технологий
- **Backend:** Django, Django REST Framework  
- **Database:** PostgreSQL  
- **Bot:** aiogram  
- **External API:** OpenWeather  
- **Other:** requests, docker-compose *(опционально)*

## Архитектура проекта
```
weather-reminder-bot/
├── backend/
│   ├── manage.py
│   ├── backend/                      # settings, urls
│   ├── subscriptions/                # app: модели, API, команды
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── management/commands/send_weather.py
│   ├── requirements.txt
│   └── .env.example
├── bot/
│   ├── bot.py
│   ├── requirements.txt
│   └── .env.example
├── README.md
└── docker-compose.yml             
````

## Структура базы данных
### **Subscriber**
| Поле | Тип | Описание |
|------|-----|-----------|
| `tg_user_id` | BigInteger (unique) | ID пользователя Telegram |
| `city` | CharField(100) | Город для прогноза |
| `time_local` | TimeField | Время отправки прогноза |
| `active` | BooleanField | Подписка активна |
| `created_at` | DateTimeField | Дата создания записи |

## Команды бота (aiogram)
| Команда                      | Действие                                                |
| ---------------------------- | ------------------------------------------------------- |
| `/start`                     | Приветствие и помощь                                    |
| `/subscribe <город> <HH:MM>` | Подписка на прогноз (пример: `/subscribe Vienna 08:30`) |
| `/stop`                      | Отключить подписку                                      |
| `/today`                     | Прислать прогноз прямо сейчас                           |
| `/help`                      | Подсказка по форматам                                   |
Бот взаимодействует с API:
* `POST /api/subscribers/` — создать/обновить подписку
* `POST /api/notify/preview/` — получить прогноз для `/today`