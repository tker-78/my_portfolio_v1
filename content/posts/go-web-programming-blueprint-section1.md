---
title: "Go Web Programming Blueprint Section1"
date: 2024-01-07T16:49:07+09:00
tags: ["go", "websocket"]
featured_image: ""
description: ""
---

## 第１章で学んだこと
### template
templateを扱うtemplateHandlerを定義する。  

```go
type templateHandler struct {
  once sync.Once,
  filename string,
  tmpl *template.Template
}
```

下記のメソッドを追加することで、templateHandlerがserverのインターフェースを満たすようにする。
こうすることで、templateHandlerをHandlerFuncとして渡すことができるようになる。  

また、`sync.Once`を使うことで、複数のgoroutineがserveHTTP()を呼び出したとしても、
実行されることは1回のみであることを保証できる。  

```go
func (t *templateHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  t.Once.Do(func() {
    t.tmpl = template.Must(template.ParseFiles(filepath.Join("templates", t.filename)))
  })
  t.tmpl.Execute(w,r)
}
```

```go
func main() {
  http.Handle("/", &templateHandler{filename: "chat.html"})
  http.ListenAndServe(":8080", nil) // localhost:8080でサーバーを起動
}
```

マルチプレクサを使う場合は下記のように書く.  
```go
func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &templateHandler{filename: "chat.html"})
	server := &http.Server{
		Addr:    ":8080",
		Handler: mux,
	}
	server.ListenAndServe()

}
```



### websocket

websocketの通信には`gorilla/websocket`を使う。

room.ServeHTTP()が呼び出されると、
clientがroom.joinに送信されるようにする。
その後、goroutineをつかって、client.Write()を実行することで、
clientからのメッセージの受信を待機する。  




WebSocketを利用するためには、websocket.Upgrader型を使ってHTTP接続をアップグレードする必要がある。

```go
var upgrader = &websocket.Upgrader{ ReadBufferSize: 1024, WriteBufferSize: 1024 }
```

room.ServeHTTP()メソッドでhtp接続をハンドルし、
接続が成立した際に、チャネルが待機に入るようにする。

```go
func (r *room) ServeHTTP(w http.ResponseWriter, req *http.Request) {
  socket, err := upgrader.Upgrade(w, req, nil)
  if err != nil {
    log.Println(err)
    return
  }
  client := &client{
    socket: socket,
    send: make(chan []byte, 1024),
    room: r,
  }
  r.join <-client
  defer func() {r.leave <- client}()
  
  // read()とwrite()のどちらをgoroutineとしてもOK(main関数もそれ自体がgoroutineなので)
  go client.write()
  client.read()
}
```

クライアント側では、jqueryを使ってインタラクションを実装する。
下記のコードでwebsocket接続が開始する。 

```js
socket = new WebSocket("ws://localhost:8080/room");
```

### プログラムのビルド

chapter1/chat内で、
```bash
$ go build -o chat
```
として、chat内に実行ファイルを作成する。

```bash
$ ./chat # プログラムの実行
```



以上.  

























