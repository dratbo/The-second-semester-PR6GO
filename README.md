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

Вывод:

<table cellpadding="10">
  <tr>
    <td><img width="974" height="516" alt="image" src="https://github.com/user-attachments/assets/2e5dc913-5b0c-4fc3-ae56-cedb86150f60" /></td>
  </tr>
</table>

