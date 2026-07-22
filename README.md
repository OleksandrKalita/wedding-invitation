# Сайт-запрошення Діана & Родіон

## 1. Деплой на Railway

1. Створи новий репозиторій на GitHub і заливай туди всі файли з цієї папки
   (`server.js`, `package.json`, `public/index.html`).
2. На [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → обери цей репозиторій.
3. Railway сам визначить Node.js проєкт і виконає `npm install` + `npm start`.
4. Після деплою Railway видасть публічний домен типу
   `https://your-project.up.railway.app` — це і є посилання на сайт.
5. Якщо хочеш власний домен — у налаштуваннях сервісу Railway є вкладка **Settings → Networking → Custom Domain**.

Альтернатива без GitHub: встанови Railway CLI (`npm i -g @railway/cli`), потім
у цій папці виконай `railway login`, `railway init`, `railway up`.

## 2. Прийом форми в Telegram через n8n

### Крок 1 — створи бота
У Telegram напиши **@BotFather** → `/newbot` → отримаєш **токен бота**.

### Крок 2 — дізнайся chat_id
- Якщо відповіді мають приходити тобі особисто: напиши своєму боту будь-яке повідомлення,
  потім відкрий у браузері
  `https://api.telegram.org/bot<ТВІЙ_ТОКЕН>/getUpdates` і знайди там `"chat":{"id": ...}`.
- Якщо хочеш групу — додай бота в групу, напиши там повідомлення і так само подивись `getUpdates`.

### Крок 3 — workflow в n8n
1. **Webhook** нода:
   - HTTP Method: `POST`
   - Path: наприклад `wedding-rsvp`
   - Respond: `Immediately` (або через окрему Respond to Webhook ноду, повернувши `200 OK`)
2. **Telegram** нода (Send Message), підключена після Webhook:
   - Credential: встав токен бота
   - Chat ID: той, що отримав вище
   - Text (приклад):
     ```
     🎉 Нова відповідь на запрошення!

     Ім'я: {{$json["body"]["guest_name"]}}
     Ночівля: {{$json["body"]["staying_overnight"]}}
     Напої: {{$json["body"]["drinks"]}}
     Кількість гостей: {{$json["body"]["guest_count"]}}
     Діти: {{$json["body"]["has_kids"]}}
     Про дітей: {{$json["body"]["kids_notes"]}}
     Коментар: {{$json["body"]["comment"]}}
     ```
3. Активуй workflow (перемикач Active у правому верхньому куті).
4. Скопіюй **Production URL** з ноди Webhook (не Test URL!).

### Крок 4 — встав URL у сайт
Відкрий `public/index.html`, знайди рядок:

```js
const N8N_WEBHOOK_URL = "https://YOUR-N8N-INSTANCE.up.railway.app/webhook/wedding-rsvp";
```

і заміни на свій реальний Production URL з n8n. Заливай оновлений файл на Railway (новий commit → автоматичний редеплой).

## 3. Перевірка
Відкрий сайт, заповни форму й натисни «Отправить ответ» — повідомлення має прийти в Telegram протягом секунди.
Якщо не приходить — перевір, що workflow в n8n активний (Active), і що URL скопійований саме Production, а не Test.
