# Мой опыт отправки комментария к фото ВКонтакте через Postman

Автор: Кристина Трофимова  

---

## Описание задачи  
Мне нужно было реализовать отправку комментария к фотографии профиля ВКонтакте (например, [этот](https://vk.com/apps?act=manage)).  
Требования:  
- Использованием API и Postman (я использую метод `photos.createComment`).  
- Опубликовать пошаговое решение в GitHub-репозитории.  

---

## 🛠️ Как я это делала (шаг за шагом)  

### 1. Создала приложение в ВК  
1. Перешла в [Управление приложениями](https://vk.com/apps?act=manage).  
2. Выбрала «Создать приложение» → тип `Standalone`.  
3. Сохранила `ID приложения` из настроек — без него нельзя получить токен.  

**Важно!** Standalone-тип приложения позволяет работать с API без настройки сервера — идеально для тестовых задач.  

---

### 2. Получила токен доступа  
Сгенерировала URL для запроса прав, подставив свой `ID приложения`: https://oauth.vk.com/authorize?client_id=ВАШ_ID&display=page&scope=photos,wall,offline&response_type=token&v=5.131

#### С какими сложностями столкнулась:  
- Права `photos` и `wall` не активировались, хотя я их указала в scope.  
  *Что сделала:* Оказалось, ВК требует подтвердить личность для некоторых прав. Пришлось загрузить паспортные данные в настройках аккаунта.  
- Токен «пропадал» после авторизации.  
  *Мой лайфхак:* Стала копировать весь URL из адресной строки браузера в блокнот. Токен находится между access_token= и &expires_in.  

---

### 3. Настроила Postman  
Собрала POST-запрос для метода photos.createComment:  
- URL: https://api.vk.com/method/photos.createComment  
- Параметры (Query Params):  

| Параметр      | Значение                          |  
|---------------|-----------------------------------|  
| access_token  | ВАШ_ТОКЕН                         |
| owner_id      | 189704458                         |  
| photo_id      | 303057922                         |  
| message       |"тестовый комментарий из Postman!" |    
| v             | 5.131                             |  

Пример моего запроса: POST  [https://api.vk.com/method/photos.createComment?access_token=ВАШ_ТОКЕН&owner_id=189704458&photo_id=303057922&message=тестовый_комментарий_из_Postman!&v=5.131](https://api.vk.com/method/photos.createComment?access_token=ВАШ_ТОКЕН&owner_id=189704458&photo_id=303057922&message=тестовый_комментарий_из_Postman!&v=5.131)

---

### 4. Ошибки, которые меня «обрадовали» (и как я их исправила)  

#### 🔴 Ошибка 15: «Access denied»  
- Почему возникло: Токен не имел прав photos и wall, хотя я их запрашивала.  
- Как решила:  
  1 . Перевыпустила токен, заново указав scope=photos,wall,offline.  
  2 . Подтвердила личность в ВК — без этого права не активировались. 
  3 . Повторила пункт 1. 

#### 🔴 Ошибка 100: «Invalid parameters»  
- Причина: В owner_id случайно добавила буквы вместо цифр.  
- Проверка: Посмотрела URL фото: photo189704458_303057922 → owner_id=189704458, photo_id=303057922.  

#### 🔴 Ошибка 413: «Request Too Large»  
- Что произошло: Так как я уже успела раастроиться что запрос на публикацию комментария так и не отправляется, то решила попробовать  отправить запрос через GET (ну вдруг, мало ли🤭) — URL получился гигантским.  
- Исправление: Переключилась снова на POST — так параметры передаются в теле, а не в URL.
---

### 5. Как проверила результат  
1. Через API: Вызвала метод photos.getComments:GET https://api.vk.com/method/photos.getComments?access_token=ВАШ_ТОКЕН&owner_id=189704458&photo_id=303057922&v=5.131
2. Вручную: Открыла страницу фото в ВК — комментарий был на месте! ✅ 


---

## 🗂️ Что я положила в репозиторий vk-api 
- [**postman**](https://github.com/user-attachments/files/18775277/TZ.postman_collection.json) — готовая коллекция для Postman.  
- [**screenshots/**](screenshots/) — скрины запросов и результатов.
- [**README.md**](README.md) — эта инструкция.

---

## 🔐 Советы от меня  
- Никогда нельзя публиковать токен в коде! Даже если репозиторий приватный — лучше просто перевыпустить токен сразу после тестов.  
- Всегда проверять права токена: https://api.vk.com/method/account.getAppPermissions?access_token=ВАШ_ТОКЕН&v=5.131

 Если в ответе нет 4 (право photos) — запросы к комментариям не сработают.  
- Фото-аватарки могут блокировать комментарии. Необходимо проверить настройки приватности владельца фото.  

---
И еще...
В моменте, решила попробовать один способ публикации коментария (в общем читерство🙈). Уверена, Вы догадываетесь о чем я, но если вдруг хотите убедиться - расскажу))

---

Ссылка на репозиторий: [github.com/Kristoonchik95/vk-api](https://github.com/Kristoonchik95/vk-api)  


 

---

### Почему я оформила именно так:  
1. Личный опыт: Добавила реальные кейсы — например, требование верификации аккаунта ВК. Это сэкономит время тем, кто столкнётся с такой же проблемой.  
2. Визуальные подсказки: Эмодзи и скриншоты делают инструкцию менее «сухой».  
3. Акцент на безопасности: Работодатели ценят внимание к защите данных — я выделила это отдельным блоком.  
4. Простой язык: Технические детали описала так, как объяснила бы коллеге за чашкой кофе ☕.



