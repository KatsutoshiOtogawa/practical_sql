# window_functions_teach
Window関数についての実践的な使い方。及び説明です。

## 所得税の計算

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
  Deduction int
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
これをuser.csvに記載されているデータと付き合わして、氏名(名前+苗字),控除額,所得税として払うべき金額(円),使った税率を検索するクエリを求めよ。
なお、user.csvの所得はドル表記、また1$=130円とする。
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

# レファレンス
[国税庁](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shotoku/2260.htm)
[Random Data Generator](http://randat.com/)
