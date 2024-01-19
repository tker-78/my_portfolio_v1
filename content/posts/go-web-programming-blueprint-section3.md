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



### アップロードしたアバター画像を適用する

authとlocalの画像のどちらを適用するか切り替えられるようにする。  

Avatarインターフェースを作り、room構造体にavatarフィールドを追加する。  
main.goでのroomオブジェクト作成時に、avatarオブジェクトを渡すことで、
どちらの画像を適用するか指定できるようにする。

avatar.go
```go
type Avatar interface{
  GetAvatarURL(c *client) (string, error)
}

type FileSystemAvatar struct{}
type AuthAvatar struct{}

var fileSystemAvatar FileSysytemAvatar
var authAvatar AuthAvatar

func (_ FileSystemAvatar) GetAvatarURL(c *client) (url string, err error) {
  if userid, ok := c.userData["userid"]; ok {
    if useridStr, ok := userid.(string); ok {
      return "/avatars/" + useridStr + ".jpg", nil
    }
  }
  return "", nil
}

func (_ AuthAvatar) GetAvatarURL(c *client) (url string, err error) {
  if avatarURL, ok := c.userData["avatar_url"]; ok {
    avatarURLStr := avatarURL.(string)
    url = avatarURLStr
    return url, nil
  }
  return "", err
}
```

room.go
```go
func newRoom(avatar Avatar) *room {
  ...
  avatar: avatar
}
```

そして、client.read()メソッドないのアバター情報の読み出しの箇所を変更する。

client.go
```go
msg.AvatarURL, _ := c.room.avatar.GetAvatarURL(c)
```


これで、newRoom()に渡すavatarを指定することで、どちらの画像を適用するか指定できる。



## Google Cloud Runへのデプロイ

```
$ gcloud config set project <project_name>
$ gcloud services enable compute.googleapis.com \
  run.googleapis.com artifactregistry.googleapis.com \
  cloudbuild.googleapis.com servicenetworking.googleapis.com
$ gcloud iam service-accounts create blueprint-service-account \
  --display-name="Blueprint Service Account" 
```



環境変数はデプロイ時に指定してあげるか、
コンソールから登録する。
```
$ gcloud run deploy blueprint \
  --region=us-central1 \
  --source=. \
  --service-account="blueprint-service-account@blueprint-410621.iam.gserviceaccount.com" \
  --allow-unauthenticated \
  --set-env-vars security_key="" \
  --set-env-vars client_id="xxx.apps.googleusercontent.com" \
  --set-env-vars client_secret="" \
  --set-env-vars url="https://<app url>/auth/callback/google" 
```


websocketの接続には、`wss://`スキームを指定する。(`ws://`では不安全のため接続できない。)
localでの実行の際は、`ws`で接続する。  





