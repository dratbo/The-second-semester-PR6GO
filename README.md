<h1 align="center"> Привет! Я <a target="_blank"> Кармеев Артур из группы ЭФМО-01-25 </a> 
<img src="https://github.com/blackcater/blackcater/raw/main/images/Hi.gif" height="32"/></h1>
<h3 align="center"> Данная практика была интересной 🤔 </h3>

<h3 align="center"> Практическая работа №6: Реализация защиты от CSRF/XSS. Работа с secure cookies </h3>

Структура работы:
    
    └── pz6-web-security/
        ├── go.mod
        ├── templates/
        │   ├── hello.html
        │   └── profile.html
        ├── .idea/
        │   ├── .gitignore
        │   ├── modules.xml
        │   ├── pz6-web-security.iml
        │   └── workspace.xml
        ├── internal/
        │   ├── store/
        │   │   └── store.go
        │   ├── httpapi/
        │   │   └── handler.go
        │   └── auth/
        │       ├── cookie.go
        │       └── csrf.go
        └── cmd/
            └── server/
                └── main.go

## 1. Немножко теории 📖 

- CSRF — это ситуация, когда браузер пользователя отправляет запрос от его имени на целевой сайт без осознанного намерения самого пользователя. Это возможно потому, что браузер автоматически прикладывает cookies к запросу, если сайт и политика браузера это позволяют.

- XSS – это внедрение вредоносного клиентского кода, чаще всего JavaScript, в страницу, которую затем увидит пользователь в браузере.

- HttpOnly запрещает доступ к cookie из JavaScript. Защищает от кражи cookie через XSS.

- Secure требует отправки cookie только по HTTPS. Защищает от перехвата в незащищённых сетях.

- SameSite ограничивает передачу cookie в межсайтовых запросах (значения `Strict`/`Lax`/`None`). Снижает риск CSRF.

## 2. Фрагменты кода

- установка cookie

```go
const SessionCookieName = "session_id"

func SetSessionCookie(w http.ResponseWriter, value string) {
    http.SetCookie(w, &http.Cookie{
        Name:     SessionCookieName,
        Value:    value,
        Path:     "/",
        HttpOnly: true,
        Secure:   false,
        SameSite: http.SameSiteLaxMode,
        MaxAge:   3600,
    })
}
```

- генерация CSRF-токена

```go
func RandomToken(size int) (string, error) {
    buf := make([]byte, size)
    if _, err := rand.Read(buf); err != nil {
        return "", err
    }
    return hex.EncodeToString(buf), nil
}
```

- проверка CSRF-токена

```go
tokenFromForm := r.FormValue("csrf_token")
if tokenFromForm == "" || tokenFromForm != profile.CSRFToken {
    http.Error(w, "invalid csrf token", http.StatusForbidden)
    return
}
```

- безопасный HTML-шаблон

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Приветствие</title>
</head>
<body>
    <h1>Здравствуйте, {{.Name}}!</h1>
    <p>Это безопасный вывод имени пользователя через шаблон.</p>
    <p><a href="/profile">Вернуться к профилю</a></p>
</body>
</html>
```

- опасный XSS-пример

```go
func unsafeHello(w http.ResponseWriter, name string) {
    html := "<html><body><h1>Здравствуйте, " + name + "!</h1></body></html>"
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    w.Write([]byte(html))
}
```

## 3. Результаты проверки

### 3.1 Запуск приложения

<table cellpadding="10">
  <tr>
    <td><img width="974" height="135" alt="image" src="https://github.com/user-attachments/assets/dc05c67a-74bc-4bc0-bc3b-e4cdbec8e634" /></td>
  </tr>
</table>

### 3.2 Проверка сценария входа и установки cookie

<table cellpadding="10">
  <tr>
    <td><img width="974" height="523" alt="image" src="https://github.com/user-attachments/assets/f1aca929-9f52-42d7-a1c2-5ae7ede8fc6a" /></td>
  </tr>
</table>

### 3.3 Проверка CSRF-защиты

Меняем имя “Студент” на “Артур”

<table cellpadding="10">
  <tr>
    <td><img width="974" height="516" alt="image" src="https://github.com/user-attachments/assets/3835f5af-89a3-4641-ae0f-8f7d0eaa8a6c" /></td>
  </tr>
</table>

### 3.4 Проверка ошибки CSRF

Удаляем элемент где указан `csrf_token`

<table cellpadding="10">
  <tr>
    <td><img width="974" height="518" alt="image" src="https://github.com/user-attachments/assets/aab028f7-ade2-47e4-a9bf-4c045db4a334" /></td>
  </tr>
</table>

Меняем `Новое имя` на `тест` и нажимаем сохранить

<table cellpadding="10">
  <tr>
    <td><img width="974" height="518" alt="image" src="https://github.com/user-attachments/assets/9bf1de1d-0a75-4628-a320-7b0416ed7d1b" /></td>
  </tr>
</table>

Вывод ошибки:

<table cellpadding="10">
  <tr>
    <td><img width="974" height="516" alt="image" src="https://github.com/user-attachments/assets/2e5dc913-5b0c-4fc3-ae56-cedb86150f60" /></td>
  </tr>
</table>

### 3.5 Демонстрация XSS-риска

Вставляем `<script>alert('xss')</script>` в `Новое имя`

<table cellpadding="10">
  <tr>
    <td><img width="974" height="516" alt="image" src="https://github.com/user-attachments/assets/8e15dc14-6929-41f8-bc7c-85cb8ed7953e" /></td>
  </tr>
</table>

При безопасной реализации вы должны увидеть сам текст, а не выполнение скрипта. То есть страница не должна запускать alert, а должна отобразить введённую строку как обычное содержимое.
Именно это и показывает, что шаблон выводит данные безопасно.

<table cellpadding="10">
  <tr>
    <td><img width="974" height="514" alt="image" src="https://github.com/user-attachments/assets/5cbafcf4-ae71-46f1-9096-031011408913" /></td>
  </tr>
</table>

## 4. Что получилось в результате

В ходе практической работы реализовали учебное web-приложение, в котором:
- используется cookie для хранения идентификатора сессии;
- cookie настроена безопаснее, чем обычная «голая» cookie;
- форма защищена CSRF-токеном;
- сервер проверяет токен перед изменением данных;
- имя пользователя отображается через шаблон безопасным способом;
- XSS-угроза разобрана на контрасте между опасным и безопасным подходом.

## 5. Что важно понять по итогам работы

CSRF и XSS — это не «экзотические» угрозы, а очень практические классы уязвимостей.
CSRF возникает там, где сервер излишне доверяет автоматической отправке cookies браузером.
XSS возникает там, где приложение излишне доверяет данным, которые потом показывает в HTML.
Безопасное backend-приложение должно:
- осторожно использовать cookies;
- не считать наличие cookie достаточным доказательством намерения пользователя;
- экранировать пользовательский ввод при выводе в HTML;
- минимизировать доверие к данным со стороны клиента.

## 6. Доп задание 🧙‍♂️

Вариант 1. Сделать logout
Добавьте маршрут:
`GET /logout`
который очищает session cookie и завершает «сессию» пользователя.

### 6.1 Добавим функцию удаления cookie в internal/auth/cookie.go

<table cellpadding="10">
  <tr>
    <td><img width="974" height="407" alt="image" src="https://github.com/user-attachments/assets/fb18284f-3da2-47da-b323-a3ba8ba1b0d9" /></td>
  </tr>
</table>

### 6.2 Добавим метод Delete в internal/store/store.go

<table cellpadding="10">
  <tr>
    <td><img width="974" height="259" alt="image" src="https://github.com/user-attachments/assets/b7a6dfeb-7b24-4016-bfb9-8839f9bb8443" /></td>
  </tr>
</table>

### 6.3 Добавим обработчик Logout в internal/httpapi/handler.go

<table cellpadding="10">
  <tr>
    <td><img width="974" height="475" alt="image" src="https://github.com/user-attachments/assets/8dba660c-8e8f-4add-a1b1-569d5906aae3" /></td>
  </tr>
</table>

### 6.4 Зарегистрируем маршрут в cmd/server/main.go

<table cellpadding="10">
  <tr>
    <td><img width="653" height="199" alt="image" src="https://github.com/user-attachments/assets/79685b75-6ce5-43bd-b8bc-07bdd9a78215" /></td>
  </tr>
</table>

### 6.5 Проверка 

Вот профиль с изменённым именем

<table cellpadding="10">
  <tr>
    <td><img width="974" height="526" alt="image" src="https://github.com/user-attachments/assets/8efd23dc-0253-433a-859e-f19134716f61" /></td>
  </tr>
</table>

Переходим по `http://localhost:8080/logout`

<table cellpadding="10">
  <tr>
    <td><img width="974" height="520" alt="image" src="https://github.com/user-attachments/assets/414f16b3-158e-4405-9950-aff33a92fe43" /></td>
  </tr>
</table>

Logout сработал, но сразу же автоматически запустилась новая сессия (На скриншотах видно разное значение csrf_token)

---

Можем проверить через `Network` в коде элемента (я в заранее поставил галочку у `Preserve log`, чтобы журнал сохранялся) 

<table cellpadding="10">
  <tr>
    <td><img width="974" height="523" alt="image" src="https://github.com/user-attachments/assets/40386a3f-1e60-438a-9dc7-98b4d6379b56" /></td>
  </tr>
</table>

В списке запросов видно:
```
GET /logout → статус 302 (редирект) → затем GET /login → статус 302 → GET /profile.
```

Теперь перейдем к действительно сложному...

## 7. Контрольные вопросы 😈

Вопрос 1. Что такое CSRF?

CSRF (Cross-Site Request Forgery, межсайтовая подделка запроса) — это ситуация, когда браузер пользователя отправляет запрос от его имени на целевой сайт без осознанного намерения самого пользователя. Это возможно, потому что браузер автоматически прикладывает cookies к запросу, если сайт и политика браузера это позволяют. Сервер ошибочно считает запрос легитимным из-за наличия правильной cookie.

Вопрос 2. Почему наличие cookie не гарантирует, что запрос действительно инициировал пользователь?

Cookie автоматически добавляется браузером к каждому запросу на соответствующий сайт. Злоумышленник может заставить браузер пользователя (например, через специальную страницу или отправку формы) выполнить запрос, и браузер сам приложит cookie. Сервер не может отличить такой поддельный запрос от запроса, который пользователь сделал намеренно, если не использует дополнительные проверки (например, CSRF-токен). Наличие cookie доказывает только то, что пользователь ранее авторизовался, но не доказывает его намерение выполнить текущее действие.

Вопрос 3. Что такое XSS?

XSS (Cross-Site Scripting, межсайтовый скриптинг) — это внедрение вредоносного клиентского кода (чаще всего JavaScript) в страницу, которую затем увидит пользователь в браузере. Классический сценарий: пользователь отправляет комментарий или имя, сервер сохраняет текст, а затем выводит его в HTML без экранирования. Браузер интерпретирует этот текст не как обычные данные, а как HTML или script, и выполняет вредоносный код.

Вопрос 4. Чем CSRF отличается от XSS?

- CSRF использует доверие сервера к браузеру пользователя – сервер ошибочно считает запрос легитимным, потому что в нём есть нужная cookie.

- XSS использует доверие браузера к данным, пришедшим от сервера – браузер ошибочно считает вредоносный фрагмент частью нормальной страницы и выполняет его.

Иными словами: CSRF атакует механику авторизованного запроса, а XSS атакует отображение данных в браузере.

Вопрос 5. Для чего нужен CSRF-токен?

CSRF-токен — это дополнительный секретный параметр, который сервер генерирует, встраивает в форму (например, в скрытое поле) и сохраняет на сервере. При отправке формы сервер проверяет, совпадает ли полученный токен с тем, который был сохранён для данной сессии. Токен не может быть украден или подставлен злоумышленником в межсайтовом запросе, потому что он не передаётся автоматически браузером (в отличие от cookie). Это позволяет серверу убедиться, что запрос действительно инициирован пользователем с той же страницы, а не пришёл с чужого сайта.

Вопрос 6. Что делает атрибут HttpOnly у cookie?

HttpOnly запрещает доступ к cookie из JavaScript в браузере. Это снижает риск кражи cookie через XSS-атаки, так как вредоносный скрипт не сможет прочитать значение такой cookie и отправить злоумышленнику.

Вопрос 7. Для чего нужен атрибут Secure?

Атрибут Secure говорит браузеру отправлять cookie только по защищённому протоколу HTTPS. Если установлен `Secure: true`, браузер никогда не передаст эту cookie по обычному HTTP. Это предотвращает перехват cookie при атаках типа «человек посередине» (Man-in-the-Middle) в незащищённых сетях.

Вопрос 8. Какую роль играет SameSite?

SameSite ограничивает передачу cookie в межсайтовых сценариях. Он может принимать значения `Strict`, `Lax` или `None`. `SameSite=Lax` (используется в работе) разрешает отправку cookie при переходе по ссылке с другого сайта, но запрещает при отправке формы или POST-запросах. Это помогает уменьшить риск CSRF-атак, так как злоумышленнику сложнее заставить браузер отправить cookie вместе с поддельным запросом с чужого сайта.

Вопрос 9. Почему нельзя вставлять пользовательский ввод в HTML через конкатенацию строк?

Если склеивать строки, например:
`"<h1>Здравствуйте, " + имяПользователя + "!</h1>"`
и в `имяПользователя` попадёт HTML-код или тег `<script>`, браузер интерпретирует его как часть разметки и выполнит. Злоумышленник может ввести `<script>alert('xss')</script>`, и такой скрипт будет выполнен на странице у всех пользователей, которые увидят этот ввод. Это приводит к краже cookie, подмене интерфейса, выполнению действий от лица пользователя и другим серьёзным последствиям.

Вопрос 10. Почему шаблоны безопаснее ручной сборки HTML?

Стандартные HTML-шаблоны (например, в Go `html/template`) автоматически экранируют (эскейпят) специальные символы: `<`, `>`, `&`, `'`, `"`. Поэтому даже если пользователь ввёл `<script>alert(1)</script>`, в итоговой HTML-странице этот текст будет отображён буквально как `&lt;script&gt;alert(1)&lt;/script&gt;` и браузер не выполнит его как код. Ручная сборка через конкатенацию строк не обеспечивает такого экранирования, что создаёт риск XSS.
