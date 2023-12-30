## GCPにデプロイ
GCP(Cloud Run)にデプロイする。

[ここ](https://qiita.com/mikkegt/items/74898c082cd9fa77c5a1)を参考にする。

Dockerfileを作成する。
```dockerfile
# Use a nginx Docker image
FROM nginx

# Copy the static HTMLs to the nginx directory
COPY public /usr/share/nginx/html

# Copy the nginx configuration template to the nginx config directory
COPY nginx/default.template /etc/nginx/conf.d/default.template

# Substitute the environment variables and generate the final config
CMD envsubst < /etc/nginx/conf.d/default.template > /etc/nginx/conf.d/default.conf && exec nginx -g 'daemon off;'

```


nginx/default.templateを作成する。
```
server {
    listen       ${PORT};
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

Dockerを起動する。
```bash
$ docker build -t my_portfolio_v1 .
$ docker run -p 1312:1312 -e PORT=1312 my_portfolio_v1
```

コンテナイメージをsubmitする。
```bash
$ gcloud builds submit --tag gcr.io/my-portfolio-v1-409721/my-portfolio-v1-container-image
```

CI/CDの構築がよくわからないので、GitHub Pagesを使った方が良さそう。




## GitHub Pagesにデプロイ

[HUGOのドキュメント](https://gohugo.io/hosting-and-deployment/hosting-on-github/)を参考にする。

