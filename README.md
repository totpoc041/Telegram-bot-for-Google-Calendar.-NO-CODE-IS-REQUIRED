# Telegram-bot-for-Google-Calendar.-NO-CODE-IS-REQUIRED
Бессерверный Telegram-бот для интеграции с Google Календарем. Работает на Google Apps Script без серверов и баз данных. Автоматически извлекает время из сообщений и работает через Webhook

---

# 📅 Telegram ➔ Google Calendar Bot (Serverless)

Легковесный и полностью бессерверный (serverless) Telegram-бот для быстрого добавления событий в Google Календарь. В бота кидается Дата+Время+Описание и он сам заносит встречу в календарь

Бот работает на базе **Google Apps Script (GAS)**. Ему не нужны VPS, Docker-контейнеры, базы данных или платные подписки. Код исполняется прямо в инфраструктуре Google с нативным доступом к вашему календарю.

## ✨ Особенности

* **Zero Infrastructure:** Не нужно арендовать сервер или настраивать окружение.
* **100% Free:** Работает в рамках бесплатных лимитов Google-аккаунта.
* **Простой парсинг:** Понимает человекочитаемый формат `ДД.ММ ЧЧ:ММ Название`.
* **Безопасность:** Скрипт имеет доступ только к вашему календарю, данные не проходят через сторонние сервисы (например, IFTTT или Make).

---

## 🚀 Быстрый старт (Установка за 5 минут)

### Шаг 1. Создание Telegram-бота

1. Откройте Telegram и найдите официального бота [@BotFather](https://t.me/BotFather).
2. Отправьте команду `/newbot` и следуйте инструкциям для создания бота.
3. Сохраните полученный **Токен** (строка вида `123456789:ABCdefGhIJKlmNoPQRstuVWXyz`).

### Шаг 2. Настройка Google Apps Script

1. Перейдите на [Google Apps Script](https://script.google.com/) и создайте **Новый проект** (New project).
2. Очистите редактор и вставьте следующий код:

```javascript
// ================= НАСТРОЙКИ =================
// Вставьте сюда токен, полученный от BotFather
const BOT_TOKEN = 'ВАШ_ТОКЕН_ОТ_BOTFATHER'; 

// 'primary' означает ваш основной Google-календарь. 
// Можно заменить на ID другого календаря, если нужно.
const CALENDAR_ID = 'primary'; 
// =============================================

function doPost(e) {
  if (!e || !e.postData || !e.postData.contents) return HtmlService.createHtmlOutput('ok');
  
  const update = JSON.parse(e.postData.contents);
  if (!update.message || !update.message.text) return HtmlService.createHtmlOutput('ok');
  
  const text = update.message.text;
  const chatId = update.message.chat.id;

  // Ожидаемый формат: ДД.ММ ЧЧ:ММ Название события
  const regex = /(\d{1,2})\.(\d{1,2})\s+(\d{1,2}):(\d{2})\s+(.+)/;
  const match = text.match(regex);

  if (match) {
    const day = parseInt(match[1], 10);
    const month = parseInt(match[2], 10) - 1; // В JS месяцы считаются с нуля
    const hours = parseInt(match[3], 10);
    const minutes = parseInt(match[4], 10);
    const title = match[5].trim(); 
    
    const year = new Date().getFullYear();
    
    // По умолчанию создается событие длительностью 1 час
    const startTime = new Date(year, month, day, hours, minutes);
    const endTime = new Date(startTime.getTime() + 60 * 60 * 1000); 

    try {
      CalendarApp.getCalendarById(CALENDAR_ID).createEvent(title, startTime, endTime);
      sendMessage(chatId, `✅ Добавлено:\n${title}\n📅 ${match[1]}.${match[2]} в ${match[3]}:${match[4]}`);
    } catch (error) {
      sendMessage(chatId, `❌ Ошибка создания: ${error.message}`);
    }
  } else {
    sendMessage(chatId, "⚠️ Формат не распознан. Отправьте в виде:\nДД.ММ ЧЧ:ММ Название\n(Например: 11.06 16:00 Техническое интервью)");
  }
  
  return HtmlService.createHtmlOutput('ok');
}

function sendMessage(chatId, text) {
  const url = `https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`;
  const payload = { chat_id: chatId, text: text };
  const options = {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload)
  };
  UrlFetchApp.fetch(url, options);
}

```

3. Замените значение `BOT_TOKEN` на ваш токен.
4. Нажмите иконку 💾 (Сохранить).

### Шаг 3. Деплой (Публикация Web App)

1. В правом верхнем углу нажмите синюю кнопку **Deploy** ➔ **New deployment**.
2. Нажмите на шестеренку слева от «Select type» и выбери **Web app**.
3. **ВАЖНО:** В поле *Who has access* обязательно выберите **Anyone** (Все). Иначе Telegram не сможет отправлять данные скрипту.
4. Нажмите **Deploy**.
5. Google попросит авторизовать скрипт: нажмите *Review permissions* ➔ выберите свой аккаунт ➔ *Advanced* ➔ *Go to [Project Name]* ➔ *Allow*.
6. Скопируйте полученный **Web app URL** (заканчивается на `/exec`).

### Шаг 4. Установка Webhook

Чтобы Telegram знал, куда пересылать сообщения бота, нужно связать его токен с вашим Web app URL.

Соберите следующую ссылку и перейдите по ней в браузере:

```text
https://api.telegram.org/bot<ВАШ_ТОКЕН>/setWebhook?url=<ВАШ_WEB_APP_URL>

```

*(Обратите внимание: слово `bot` перед токеном убирать нельзя!)*

**Пример успешного ответа в браузере:**

```json
{"ok":true,"result":true,"description":"Webhook was set"}

```

Всё готово! Откройте вашего бота в Telegram и отправьте тестовое сообщение.

---

## 📱 Использование

Отправьте или перешлите боту сообщение строго в следующем формате:
`ДД.ММ ЧЧ:ММ Название события`

**Примеры:**

* `12.06 14:00 Собеседование DevOps`
* `05.09 09:30 Встреча с командой`

Бот ответит подтверждением, и событие мгновенно появится в Google Календаре. Продолжительность встречи по умолчанию — 1 час.

---

## 🛠 Обновление кода (Важно!)

Если вы решите изменить код (например, поменять токен или логику парсинга), **простое сохранение файла не обновит работу бота**. Вебхук всегда смотрит на опубликованную версию.

Чтобы применить изменения без потери связи с Telegram:

1. Нажмите **Deploy** ➔ **Manage deployments**.
2. Выберите ваше развертывание слева.
3. Нажмите иконку ✏️ (Редактировать) справа вверху.
4. В поле **Version** обязательно выберите **New version**.
5. Нажмите **Deploy**.

---
