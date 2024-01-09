¥
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

