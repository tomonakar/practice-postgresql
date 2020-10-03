# PostgreSQLの練習
- 参照リファレンス: https://www.postgresql.jp/document/12/html/index.html

# 環境構築
- brewで入れる場合は素直に `brew install postgresql`
- 今回はdockerイメージを利用した
  - Dockerfileを作成
  - docker-compose.ymlを作成
  - 参考(docker環境構築)：https://crudzoo.com/blog/docker-postgres
  - 参考(docker-hub): https://hub.docker.com/_/postgres/

# 基本コマンド
### docker関連
```sh
# コンテナ起動
docker-compose up -d

# コンテナ停止
docker-compose down

# コンテナに接続
docker-compose exec db bash

# postgresqlに接続
psql -U admin

# キャッシュのクリア
#### -v　でボリュームイメージを削除
docker-compose down -v
docker-compose build --no-cache
docker-compose up -
```

