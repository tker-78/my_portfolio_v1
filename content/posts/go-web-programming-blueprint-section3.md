---
title: "Go Web Programming Blueprint Section3"
date: 2024-01-13T06:58:04+09:00
tags: ["go", "jQuery"]
featured_image: ""
description: ""
---

## 第３章で学んだこと
### gomniauthによるアバター情報の取得
`gomniauth`によって、ユーザー情報にアバター情報も含まれるため、
その情報を取り出してフロントに渡してあげることで、アバター画像を表示できる。  

message.go
```go
type message struct {
  ...
  AvatarURL string
}
```

client.go
```go
func (c *client) read() {
  ...
  if avatarURL, ok := c.userData(["avatar_url"]); ok {
    msg.AvatarURL = avatarURL.(string)
  }
  c.room.forward <- msg
}
```

cookieにアバター情報を書き込む.  
auth.go
```go
authCookieValue := objx.New(map[string]interface{}{
  "name": user.Name(),
  "avatar_url": user.AvatarURL(),
}).MustBase64()
```

フロント側の実装。
chat.html
```js
socket.onmessage = function(e){
  var msg = JSON.parse(e.data);
  messages.append(
    $("<li>").append(
      $("<img>").attr("title", msg.Name).css({
        width: 50,
        verticalAlign: "middle",
        borderRadius: "50%",
      }).attr("src", msg.AvatarURL),
      ...
    )
  )
}
```


### アバター画像のアップロード

画像のアップロードには、
`uploadHandler()`を実装する。

```go
func uploadHandler(w http.ResponseWriter, req *http.Request) {
  // useridの取得

  // file, headerの取得

  // fileからbytesデータの読み込み

  // ファイル名の設定

  // ファイルの保存

  // リダイレクト
}
```

main.go
```go
mux.Handle("/upload", &templateHandler{filename: "upload.html"})
mux.HandleFunc("/uploader", uploadHandler)
mux.Handle("/avatars/", http.StripPrefix("/avatars/",http.FileServer(http.Dir("./avatars"))))
```

upload.html
```html
<form action="/uploader" role="form" enctype="multipart/form-data" method="post">
  <input type="hidden" name="userid" value="{{.UserData.userid}}"/ >
  <div class="form-group">
    <label for="message">ファイル選択</label>
    <input type="file" name="avatarFile" />
  </div>
  <input type="submit" value="アップロード" class="btn btn-primary" />
</form>
```
















