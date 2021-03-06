# 実践的なsql

## 所得税の計算
- よくある集合関数の使い方を理解できているかどうか。少し捻らないと分からない。

>|課税される所得金額|税率|控除額(円)|
>|---------------|---|-----|
>|1,000円 から 1,949,000円まで|5％|0円|
>|1,950,000円 から 3,299,000円まで|10％|97,500円|
>|3,300,000円 から 6,949,000円まで|20％|427,500円|
>|6,950,000円 から 8,999,000円まで|23％|636,000円|
>|9,000,000円 から 17,999,000円まで|33％|1,536,000円|
>|18,000,000円 から 39,999,000円まで|40％|2,796,000円|
>|40,000,000円 以上|45％|4,796,000円|
(下記国税庁のページより引用。単位は1000円刻み)

これをデータベース上でテーブルとして表現すると下のようになる。
```
-- テーブル定義
CREATE TABLE IncomeTax(
  income int,
  tax real,
  deduction int
);

-- 値放り込み
INSERT INTO IncomeTax VALUES(1000,0.05,0);
INSERT INTO IncomeTax VALUES(1950000,0.10,97500);
INSERT INTO IncomeTax VALUES(3300000,0.20,427500);
INSERT INTO IncomeTax VALUES(6950000,0.23,636000);
INSERT INTO IncomeTax VALUES(9000000,0.33,1536000);
INSERT INTO IncomeTax VALUES(18000000,0.40,2796000);
INSERT INTO IncomeTax VALUES(40000000,0.45,4796000);
```
これをuser.csvに記載されているデータと付き合わして、氏名(名前+苗字),所得,所得税として払うべき金額(円)(小数点切り上げ),控除額,使った税率を検索するクエリを求めよ。 \
なお、user.csvの所得はドル表記、給料は所得控除済み、1$=130円とし、使っているdbはsqlite3とする。
```
-- テーブル定義
CREATE TABLE user(
  firstname text,
  lastname test,
  income int
);

-- Random Data Generatorで作成済みのユーザーデータを流し込み。
.mode csv
.import ./user.csv user
```


```
SELECT user.firstname || ' ' || user.lastname
       ,user.income * 130
       ,round(user.income * 130 * incometax.tax + 0.5) -- sqliteには切り上げ関数がないので、0.5プラスすることにより切り上げ処理
       ,incometax.deduction
       ,incometax.tax 
FROM user INNER JOIN incometax
  ON user.income * 130 < incometax.income
GROUP BY user.firstname
        ,user.lastname
        ,user.income;
```
- 同じ要領で特定の金額以上の商品を買ったら売り上げ率%もできる。

## ネットワークビジネスの組織図
- 木構造のデータを用いることになる。実践的なデータベースをある程度触っていないと難しいか。

distributer.csvとorganization.csvを付き合わせて、 \
distributer.csvでid=1の人の組織図を作成せよ。 \
1段目、2段目と分かるようにすること。

|id|firstname|lastname|
|---|----|---|

```
CREATE TABLE distributer(
  id int primary key,
  firstname text,
  lastname test
);

-- Random Data Generatorで作成済みのユーザーデータを流し込み。
.mode csv
.import ./distributer.csv distributer

```

Organization
|distributer_id|downstream_id|
|----|----|

```
CREATE TABLE Organization(
  distributer_id int,
  downstream_id int
);

-- 作成済みのユーザーデータを流し込み。
.mode csv
.import ./organization.csv Organization

```

SQL
```
WITH RECURSIVE r AS (
    SELECT *
          ,1 as hierarchy
    FROM Organization
    WHERE distributer_id = 1
    UNION
    SELECT Organization.*
          ,hierarchy + 1 as hierarchy
    FROM Organization, r 
    WHERE Organization.distributer_id = r.downstream_id
)
SELECT distributer.firstname || ' ' || distributer.lastname
      ,r.distributer_id
      ,r.hierarchy
FROM r INNER JOIN distributer
  ON r.distributer_id = distributer.id
GROUP BY distributer.firstname
      ,distributer.lastname
      ,r.distributer_id
      ,r.hierarchy
ORDER BY r.hierarchy
```

## 病理システムの統計
- 集合関数、分析関数をよく用いる。

## 権限マスタ
- 大規模なシステムでよくある。職種ごとの権限と役職ごとの権限、会社ごとの権限があるため複雑。


## 在庫テーブル、材料テーブル、レシピテーブル
- 材料から何か物を作るという作業に必要なテーブル郡。レシピと在庫を見て作れない物があれば発注するようにメールを送るはよくあるパターン。

在庫テーブルから
# レファレンス
[国税庁](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shotoku/2260.htm)

[Random Data Generator](http://randat.com/)

[アムウェイのボーナスについて](https://topteam-world.com/2017/09/17/post-1545/)

# Random Data Generatorの仕様
- CRLFで出力される
- UTF8で出力
- id列は自分で作成する必要あり。
