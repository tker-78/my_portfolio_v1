---
title: "Real World Http Section3"
date: 2024-01-04T10:28:35+09:00
tags: ["http", "go"]
featured_image: "{{ resources.Get "images/real_world_http.png"}}"
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

(注意)本文では`iouti`パッケージを使っているが、ioutilはGo 1.16でdepricateになったので、
`io`パッケージを使用する。  



