# Интеграция Examus и StartExam

## Авторизация
Открытие ссылки на StartExam не ведет сразу на экзамен, а выдает редирект на Экзамус

`app.examus.net/quick/start`

Экзамус ждет следующие данные:
POST:
 - `User information`: ФИО, Табельный номер
 - `Exam information`: Название экзамена, длительность, тип экзамена (c ручным прокторингом/без, с записью в календаре/без)
 - `Session information`: uid сессии (опционально), url-адрес (который должен отобразиться пользователю после начала экзамена)

Экзамус создает пользователя по этой информации (либо использует существующего), авторизует его, создает сеанс и выдает приветственную страницу.

Приветственная страница содержит:
 - Cсылку на экстеншн, который необхимо установить, если еще не установлен
 - Инструкцию для использования экстеншна

## Начало экзамена
Пользователь с помощью экстеншна начинает сессию прокторинга, проходит идентификацию личности и проверку соединения, в случае успеха система переходит на url, указанный в `Session information`.

## Конец экзамена
Кнопка "закончить экзамен" в StartExam теперь ведет на 
`app.examus.net/quick/finish`

Экзамус отмечает текущий сеанс завершенным.
В get параметрах указывается "redirect_url", на который потом происходит редирект со всеми параметрами изначального запроса

## Результаты сессии прокторинга
Когда сеанс прокторинга считается проверенным, экзамус делает запрос на некий url startExam, отправляя результаты прокторинга

Альтернативный вариант - StartExam запрашивает статус прокторинга на 
`app.examus.net/quick/result/<uid>/`

В случае с онлайн проктором, статус выставляется сразу

В случае с пост-просмотром записей, статус выставляется через некоторое время
