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
## テーブル作成
```sql
CREATE TABLE weather (
    city            varchar(80),

    temp_lo         int,           -- 最低気温
    temp_hi         int,           -- 最高気温
    prcp            real,          -- 降水量
    date            date
);
```

## テーブル削除
```sql
DROP TABLE tablename;
```


## 行の挿入
```sql
## columnを指定しないパターン
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');

## columnを指定するパターン
INSERT INTO weather (date, city, temp_hi, temp_lo)
    VALUES ('1994-11-29', 'Hayward', 54, 37);
```

## COPYコマンドによるデータのロード
大量データをテキストファイルからロードできる
```sql
COPY weather FROM '/home/user/weather.txt';
```

## 問い合わせ
```sql
## 
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
```

## テーブル間を結合して問い合わせを行う
```sql
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

## 集約関数
```sql
SELECT max(temp_lo) FROM weather;

### これはできない
# WHEREはどの行を集約処理に渡すのかを決定する
# WHEREは集約を演算する前に入力行を選択する
SELECT city FROM weather WHERE temp_lo = max(temp_lo);


### 上記をやるには副問い合わせを使って集約関数を条件に使う
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);

### GROUP句と組み合わせて上記を求めることもできる
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;

### HAVINGを使ってグループ化された行にフィルタをかける
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;

### WHEREとHAVINGの違い
# WHEREは、グループや集約を演算する前に入力行を選択する
# HAVINGは、グループと集約を演算した後に、グループ化された行を選択する
```


## 更新
```sql
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```

## 削除
```sql
DELETE FROM weather WHERE city = 'Hayward';
```

# 高度な諸機能
## View
任意の問い合わせに対してビューを作り、通常のテーブルのように参照できる問い合わせに名前をつけることができる。他のビューに対するビューの作成もできる。Viewを自由に利用することはSQLデータベースの良い設計における重要な項目。
```sql
CREATE VIEW myview AS
    SELECT city, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;
```

## 外部キー
```sql
CREATE TABLE cities (
        city     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);

## これはエラーになる
INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');

ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".
```

## トランザクション
PostgreSQLは全てのSQL文をトランザクション内で実行する。
トランザクションは、`BEGIN` と `COMMIT` で囲んで設定する。

```SQL
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';

-- 等々
COMMIT;
```

## ウインドウ関数
ここがわかりやすかった [window関数を使いこなす 〜分析のためのSQL〜](https://qiita.com/HiromuMasuda0228/items/0b20d461f1a80bd30cfc)
- 現在の行に関するテーブル全体を舐める計算をする
- 集約関数と考え方は似ている。集約関数は1行に集約するが、window関数では対象行はそのまま残る
- 対象の行全てに対して処理が行われる

```sql
SELECT
  depname,
  empid,
  salary,
  AVG(salary) OVER (PARTITION BY depname)
FROM emp_info;

  depname  | empid | salary |  avg_salary
-----------+-------+--------+-------------
 develop   |    11 |   5200 |        5020
 develop   |     7 |   4200 |        5020
 develop   |     9 |   4500 |        5020
 develop   |     8 |   6000 |        5020
 develop   |    10 |   5200 |        5020
 personnel |     5 |   3500 |        3700
 personnel |     2 |   3900 |        3700
 sales     |     3 |   4800 |        4866
 sales     |     1 |   5000 |        4866
 sales     |     4 |   4800 |        4866
 ```

>OVERとPARTITION BYという2つの関数が出てきました。続いてこれらを解説します。
>OVERはwindow関数を使いますよーというサインです。OVERの後に、どのようにwindowを作るのかということを定義します。
>PARTITIONでwindow、つまりどの範囲でグループを作るか指定します。

>AVG(salary) OVER (PARTITION BY depname)は、depnameでグループを作った上で、自分が属するdepnameのsalaryのAVGをちょうだいと言っています。


## 継承
テーブルを継承してデータを扱うことができる便利な機能。
ただし、一意制約や外部キーと統合されてないので万能ではない。

```sql
CREATE TABLE cities (
  name       text,
  population real,

  elevation  int     -- （フィート単位）
);

# INHERITSでcitiesテーブルを継承し、加えて state カラムを持つ
CREATE TABLE capitals (
  state      char(2)
) INHERITS (cities);
```

```sql
# 以下の問い合わせは下層テーブル capitalsが持つ stateも含めた結果を返す
SELECT name, elevation
  FROM cities
  WHERE elevation > 500;


# 以下の問い合わせでは、ONLY句でcitiesテーブルのみ参照させている
SELECT name, elevation
    FROM ONLY cities
    WHERE elevation > 500;

```