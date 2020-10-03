# PostgreSQLの練習
- 参照リファレンス: https://www.postgresql.jp/document/12/html/index.html

# 環境構築
- brewで入れる場合は素直に `brew install postgresql`
- 今回はdockerイメージを利用した
  - Dockerfileを作成
  - docker-compose.ymlを作成
  - 参考(docker環境構築)：https://crudzoo.com/blog/docker-postgres
  - 参考(docker-hub): https://hub.docker.com/_/postgres/

# docker関連
```sh
# コンテナ起動
docker-compose up -d

# コンテナ停止
docker-compose down

# コンテナに接続
docker-compose exec db bash

# dbコマンド実行
<コマンド> -U admin

# adminユーザでpostgresqlに接続
psql -U admin

# キャッシュのクリア
#### -v　でボリュームイメージを削除
docker-compose down -v
docker-compose build --no-cache
docker-compose up -
```

# 基本コマンド
```
# db作成
createdb <db名>

# db削除
dropdb <db名>
```

# 基本的なSQL
- テーブル作成
```sql
CREATE TABLE weather (
    city            varchar(80),

    temp_lo         int,           -- 最低気温
    temp_hi         int,           -- 最高気温
    prcp            real,          -- 降水量
    date            date
);
```

- テーブル削除
```sql
DROP TABLE tablename;
```


- 行の挿入
```sql
## columnを指定しないパターン
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');

## columnを指定するパターン
INSERT INTO weather (date, city, temp_hi, temp_lo)
    VALUES ('1994-11-29', 'Hayward', 54, 37);
```

- COPYコマンドによるデータのロード
  - 大量データをテキストファイルからロードできる
```sql
COPY weather FROM '/home/user/weather.txt';
```


- SELECT
```sql
SELECT * FROM weather;

## 列指定
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;

## 式の指定
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;

## where句による条件指定
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;

## 結果のソート
SELECT * FROM weather
    ORDER BY city, temp_lo;

## 重複行の除外
SELECT DISTINCT city
    FROM weather;

## 重複行を除外してソートも行う
SELECT DISTINCT city
    FROM weather
    ORDER BY city;

## テーブル間を結合して問い合わせを行う
## 問い合わせは全ての列名を修飾するのが良いと考えられている（列名が増えた場合に影響を受けなくて済むため）
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather, cities
    WHERE cities.name = weather.city;

## 内部結合
## 2つのテーブルで指定したカラムが一致するレコードだけ取得する
SELECT *
    FROM weather INNER JOIN cities ON (weather.city = cities.name);

## 左外部結合
## 左側のテーブルを軸にテーブルを結合する(どちらかのテーブルにしか存在しないものも取得する)
SELECT *
    FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);

## 別名を利用した入力量の省略
SELECT *
    FROM weather w, cities c
    WHERE w.city = c.name;

## 自己結合
## テーブルを自分自身に対して結合することができる
## 以下では、他の気象データの気温範囲内にある気象データを全て取り出している
SELECT W1.city, W1.temp_lo AS low, W1.temp_hi AS high,
    W2.city, W2.temp_lo AS low, W2.temp_hi AS high
    FROM weather W1, weather W2
    WHERE W1.temp_lo < W2.temp_lo
    AND W1.temp_hi > W2.temp_hi;

```