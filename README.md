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

```json
{
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
}
```





<table cellpadding="10">
  <tr>
    <td><img width="974" height="518" alt="image" src="https://github.com/user-attachments/assets/aa5bebb5-0f22-4dab-9533-98fb70640ce5" /></td>
  </tr>
</table>

