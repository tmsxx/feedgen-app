# FeedGen - Інструкція з розгортання

## Що це
Веб-сервіс для генерації оптимізованих заголовків та описів товарного фіду Google Merchant Center за допомогою Claude AI. Підтримує будь-яку нішу через JSON-конфіги.

## Структура проєкту
```
feedgen-app/
├── app.py              # Бекенд (FastAPI)
├── requirements.txt    # Залежності Python
├── Dockerfile          # Для контейнерного деплою
├── templates/
│   └── index.html      # Головна сторінка
└── static/
    ├── style.css       # Стилі
    └── app.js          # Фронтенд логіка
```

---

## Варіант 1: Railway (рекомендовано для старту)

### Крок 1 — GitHub
1. Створи репозиторій на github.com (наприклад, `feedgen`)
2. Завантаж всі файли з `feedgen-app/` в репозиторій

### Крок 2 — Railway
1. Зареєструйся на https://railway.app (можна через GitHub)
2. New Project → Deploy from GitHub repo → вибери свій репозиторій
3. Railway автоматично знайде Dockerfile і задеплоїть

### Крок 3 — API-ключ
1. Зареєструйся на https://platform.claude.com
2. Створи API-ключ в Settings → API Keys
3. В Railway: Settings → Variables → додай:
   ```
   ANTHROPIC_API_KEY=sk-ant-api03-...
   ```

### Крок 4 — Домен
1. Railway → Settings → Networking → Generate Domain
2. Отримаєш URL типу `feedgen-production.up.railway.app`

**Вартість:** Railway Hobby план $5/міс (500 годин виконання). Для MVP більш ніж достатньо.

---

## Варіант 2: Render

### Кроки аналогічні Railway:
1. Зареєструйся на https://render.com
2. New → Web Service → Connect GitHub repo
3. Environment: Docker
4. Environment Variables: додай `ANTHROPIC_API_KEY`
5. Render сам збілдить і задеплоїть

**Вартість:** Є безкоштовний план (cold starts через 15 хв неактивності). Starter план $7/міс.

---

## Варіант 3: Локально (для тестування)

### Крок 1 — Встанови Python 3.10+
Перевір: `python3 --version`

### Крок 2 — Встанови залежності
```bash
cd feedgen-app
pip install -r requirements.txt
```

### Крок 3 — Налаштуй API-ключ
```bash
export ANTHROPIC_API_KEY=sk-ant-api03-...
```

### Крок 4 — Запусти
```bash
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

Відкрий http://localhost:8000

---

## Як користуватися

1. Відкрий сайт
2. Завантаж .xlsx файл з товарним фідом
3. (Опціонально) Завантаж JSON-конфіг ніші
4. Вибери модель і мову
5. Натисни "Запустити генерацію"
6. Дочекайся завершення і скачай результат

## JSON-конфіг ніші

Без конфігу сервіс використовує універсальний промпт. З конфігом — генерація буде заточена під конкретну нішу (як наш Etnodim JSON).

Конфіг — це JSON файл з правилами:
- Які атрибути використовувати
- Як маппити product type
- Які стилі та ключові слова
- Заборонені слова
- Tone of voice бренду

Приклад: файл `Etnodim_Feed_Generation_Prompt.json` — повноцінний конфіг для ніші вишитого одягу.

---

## Claude API — вартість

| Модель | Input (1M токенів) | Output (1M токенів) | 1000 товарів |
|--------|-------------------|---------------------|--------------|
| Haiku 4.5 | $1 | $5 | ~$1.30 |
| Sonnet 4 | $3 | $15 | ~$4 |

Batch API дає знижку 50%. Prompt caching економить до 90% на системному промпті.

---

## Масштабування

Коли сервіс виросте:
- **Авторизація:** додати логін/реєстрацію (NextAuth, Clerk)
- **БД:** зберігати історію генерацій (PostgreSQL)
- **Черга:** для великих фідів — Celery / Redis
- **Кеш:** prompt caching через Anthropic API
- **Batch API:** для фідів 10k+ товарів
