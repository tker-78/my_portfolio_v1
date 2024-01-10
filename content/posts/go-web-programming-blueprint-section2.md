---
title: "Go Web Programming Blueprint Section2"
date: 2024-01-08T09:10:17+09:00
tags: ["go"]
featured_image: ""
description: ""
---

## 第２章で学んだこと 

loginハンドラーを使って認証を行う。

```go
func loginHandler(w http.ResponseWriter, r *http.Request) {
  segs := strings.Split(r.URL.Path, "/") // urlパスを"/"で分割する
  action := segs[2]
  peovider := segs[3]
}
```

/auth/xxx/oooの場合、  
segs[2] = xxx  
segs[3] = ooo  
となる。  

Googleでのログインの場合、
/auth/login/googleでgoogleにリダイレクトしたいので、
```go
func loginHandler(w http.ResponseWriter, r *http.Request) {
  segs := strings.Split(r.URL.Path, "/")
  action := segs[2]
  provider := segs[3]
  switch action {
  case "login":
    provider, err := gomniauth.Provider(provider)
    if err != nil {
      log.Println(err)
    }
    loginURL, err := provider.GetBeginAuthURL(nil, nil)
    if err != nil {
      log.Println(err)
    }
    w.Header().Set("Location", loginURL)
    w.WriteHeader(http.StatusTemporaryRedirect)
  default:
    w.WriteHeader(http.StatusFound)
    fmt.Fprintf(w, "アクション%sには非対応です", action)
  }
}
```

localhost:8080/auth/login/googleにアクセスすると、下記のURLにリダイレクトされる。

```
http://localhost:8080/auth/callback/google?code=4%2F0AfJohXk1xQcE4ZCpXCOKj-SB5DOJjS5yzQ5HCBRKQqdrQcCo_-y4Cd9Ul3J64_1hKeuucQ&scope=email+profile+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.profile+openid&authuser=1&prompt=consent#
```

loginHandlerにcallbackのアクションが呼ばれた場合の処理を追加する。
```go
func loginHandler(w http.ResponseWriter, r *http.Request) {
  ...(省略)
  switch action {
    case "callback":
      provider, err := gomniauth.Provider(provider)
      if err != nil {
        log.Println(err)
      }
      creds, err := provider.CompleteAuth(objx.MustFromURLQuery(r.URL.RawQuery))
      if err != nil {
        log.Fatalln("認証を完了できませんでした", provider, "-", err)
      }
      user, err := provider.GetUser(creds)
      if err != nil {
        log.Fatalln("ユーザーの取得に失敗しました", provider, "-", err)
      }
      authCookieValue := objx.New(map[string]interface{}{
        "name": user.Name(),
      }).MustBase64()
      http.SetCookie(w, &http.Cookie{
        Name: "auth",
        Value: authCookieValue,
        Path: "/"
      })
      w.Header()["Location"] = []string{"/chat"}
      w.WriteHeader(http.StatusTemporaryRedirect)
  }
}
```

cookieからユーザー名を取り出す。
```go
func (t *templateHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  ...(省略)
  data := map[string]interface{}{
    "Host": r.Host,
  }
  if authCookie, err := r.Cookie("auth"); err == nil {
    data["UserSata"] = objx.MustFromBase64(authCookie.Value)
  }
  ...(省略)
}
```


入力したテキストにユーザー名と日時の情報を持たせるために、
message構造体を作成する。

```go
type message struct {
  Name string
  Message string
  When time.Time
}
```


messageでclientとroomを繋ぐ。  
```go
type client struct {
  send: chan *message
  ...
  userData: map[string]interface{}
}

type room struct {
  forward: chan *message
}
```

clientのread(), write()からJSONを扱うように変更する。
```go
func (c *client) read() {
  for {
    var msg *message
    if err := c.socket.ReadJson(&msg); err == nil {
      msg.When = time.Now()
      msg.Name = c.userDate["name"].(string)
      c.room.forward <- msg
    } else {
      break
    }
  }
  c.socket.Close()
}

func (c *client) write() {
  for msg := range c.send {
    if err := c.socket.WriteJSON(&msg); err != nil {
      break
    }
  }
  c.socket.Close()
}
```

jqueryからのjson呼び出しを実装する。
```js
socket.send(JSON.stringify({"Message": msgBox.val()}));

socket.onmessage = function(e) {
  var msg = JSON.parse(e.data);
  messages.append(
    $("<li>").append(
      $("<strong>").text(msg.Name + ": "),
      $("<span>").text(msg.Message),
      $("<span>").text(" " + msg.When),
    )
  )
}
```

日時のフォーマットをするために、message.MarshalJSON()メソッドを定義して、
Whenをstringに変換して返す。

```go
func (msg *message) MarshalJSON() ([]byte, error) {
  val, err := json.Marshal(&struct{
    {
      Name: string,
      Message: string,
      When: string,
    }{
      Name: msg.Name,
      Message: msg.Message,
      When: msg.When.Format("2006年01月02日 15時04分")
    }
  })
  if err != nil {
    log.Println(err)
    return val, err
  }
  return val, err
}
```



























