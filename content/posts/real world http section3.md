---
title: "Real World Http Section3"
date: 2024-01-04T10:28:35+09:00
tags: ["http", "go"]
featured_image: "{{ resources.Get 'images/real_world_http.png'}}"
description: ""
---

## 第３章で学んだこと

httpリクエストの基本は`curl`コマンド. 

```bash
$ curl http://localhost:8080
```
これと等価なgoでのhttpリクエストは、

```go
func main() {
  resp, err := http.Get("http://locahost:8080")
  if err != nil {
    panic(err)
  }
  defer resp.Body.Close()
  body, err := io.ReadAll(resp.Body)
  if err != nil {
    panic(err)
  }
  log.Println(string(body))
}
```

(注意)本文では`iouti`パッケージを使っているが、ioutilはGo 1.16でdepricatedになったので、
`io`パッケージを使用する。  


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



