## SQL入門編1: SQLの基本文法を学ぶ

### 目次
[1-1_データベースを知ろう](#1−1_データベースを知ろう)</br>
[1-2_データベースを準備しよう](#1-2_データベースを準備しよう)</br>
[1-3_テーブルの中身を見てみよう](#1-3_テーブルの中身を見てみよう)</br>
[1-4_いろいろな情報を取り出そう](#1-4_いろいろな情報を取り出そう)</br>
[1-5_データを追加・更新・削除しよう](#1-5_データを追加・更新・削除しよう)</br>
[1-6_2つのテーブルを結合しよう](#1-6_2つのテーブルを結合しよう)</br>
[1-7_結合したテーブルを操作しよう](#1-7_結合したテーブルを操作しよう)</br>

***

### 1−1_データベースを知ろう
データベースとは大量のデータを保管し、効率よく取り扱うツールの事である。</br>
データをまとめたものを更に扱うツールもデータベースという。</br>
データベースにおいてデータを格納する表を**テーブル**という。</br>
テーブルにおける行(横の段)を**レコード**という。</br>
テーブルにおける列(縦の段)を**カラム/フィールド**という。</br>
</br>

| カラム1           | カラム2           | カラム3           |
| ----------------- | ----------------- | ----------------- |
| レコード1/カラム1 | レコード1/カラム2 | レコード1/カラム3 |
| レコード2/カラム1 | レコード2/カラム2 | レコード2/カラム3 |
| レコード3/カラム1 | レコード3/カラム2 | レコード3/カラム3 |
</br>

また、複数のテーブルを関連付ける事を**リレーション**という。
このようなDBを**RDB(リレーショナル・データベース)**という。
RDBを用いる事で大量のデータを少ない記述で管理する事ができる。</br>
例：</br>
</br>

playersテーブル
| Name        | LV   | Job |
| ----------- | ---- | --- |
| Takashi     | 10   | 1   |
| Takeshi     | 2020 | 2   |
| Takeru      | 30   | 3   |

↑
リレーション
↓

jobsテーブル
| Name    | VIT | STR | DEF | DEX |
| ------- | --- | --- | --- | --- |
| Warrior | 250 | 25  | 20  | 10  |
| Knight  | 200 | 20  | 30  | 8   |
| Bishop  | 150 | 15  | 25  | 7   |

</br>

データベースに対する指示や問い合わせを行う言語を**SQL**という。</br>
たとえば`SELECT * FROM players`はplayerテーブルが全てのデータを選択して表示するという指示。</br>
</br>

さまざまなデータベース</br>
- **MySQL**：もっともよく使われているDB。
- **SQLite**：スマートフォンのアプリの中にDBを組み込む場合などに用いられる。
- **PostgreSQL**
- **Oracle Database**：企業情報などのDB。
- **Microsoft SQL Server**</br>
また、AWSやGCPは**No SQL**と呼ばれるDB(ようはSaasのDB)なども使用されている。</br>
</br>

このレッスンでの内容
1. データベースの準備をする。
2. データベースの中身を見る。
3. いろいろな情報を取り出す。
4. データを追加・更新・削除する。
5. 2つのテーブルを結合する。
6. 結合したテーブルを操作する。
</br>

***

### 1-2_データベースを準備しよう
MySQLとphpMyAdminを使って基本的なデータベースを作成していく。</br>
</br>

1. phpMyAdminを開いてDBを作成。</br>
   [phpMyAdmin](http://localhost:8888/phpMyAdmin/)にアクセスして、データベースタグをクリック。</br>
   データベース名(今回は`LNG_db-sql`とする)を記入し、`utf8_general_ci`を選択して作成をクリック。</br>
   - この`utf8_general_ci`は**照合順序**と呼ばれるもので、日本語を扱うためこの`utf8_general_ci`を選択している。</br>
   </br>

2. データベースにテーブルを作成</br>
   データベースが作成できたら、SQLタブをクリック。するとSQL文の入力欄が表示される。</br>
   ここに作成したい内容のSQL文を入力する。今回は`sql_sample.txt`の内容をペーストする。</br>
   SQL文が入寮できたら実行ボタンをクリック。</br>
   これで`LNG_db-sql`のDBの配下にplayersテーブルとJobテーブルができた。</br>
</br>

***

### 1-3_テーブルの中身を見てみよう
今回は下記のような内容を実行する。
1. SQL文を書いてみる
2. SQLの基本構文を理解する
3. 一部のカラムだけ取得する
4. 一部の行だけ取得する
5. 条件に合った行だけ取得する
</br>

1. SQL文を書いてみる
   phpMyAdminのSQLタブをクリック。</br>
   記入欄に下記のSQL文を記述。</br>

   ```SQL
   -- 全てのデータを取り出す
   SELECT * FROM players;
   ```
   これを実行する事でplayersテーブルの全てのデータを取り出すことができる。</br>
   </br>
   `SELECT * FROM players;`について
   - `SELECT`はDBからデータを取り出す命令。
   - `*`の箇所は本来はカラムを指定する箇所。`*`は全てのカラムという意味のもの。(ワイルドカード)
    - 備考:[ワイルドカード](https://wa3.i-3-i.info/word11613.html)
   - `FROM`は対象となるテーブルを指定している。
  </br>
   **SQL文**について
   - 基本的に命令文は大文字でかく。テーブル名などは作成時のものを使用するが、小文字で書く場合が多い。
   - 命令の最後に`;`を記述する。
   - `--`でコメントを記載できる。`--`のあとには**スペースが必要なので注意。**
</br>

2. 一部のカラムだけ取得する。</br>
   次にplayersテーブルのnameとlevelのみ取り出すSQL文を記述する。
   ```sql
   -- 一部のカラムだけ取り出す
   SELECT name,level FROM players;
   ```
   これでplayersテーブルの全てのレコードのnameとlevelカラムだけ表示できる。
</br>

3. 一部の行だけ取得する。</br>
   次にlevelが7以上のレコードのみを取り出すSQL文を記述する。
   ```sql
   -- 一部の行/レコードのみを取り出す
   SELECT * FROM players WHERE level >= 7;
   ```
   これで条件に合ったレコードのみ取得することができる。</br>
   `WHERE 条件`でその条件に合ったものだけを指定できる。</br>
   - `WHERE`の条件指定
     一般の比較演算子の記号が使える。</br>
     また、`a <> b`で**aとbが等しくないとき**という条件の設定が可能。</br>
     例：
     ```sql
     -- 勇者以外のジョブを取得する。
     SELECT * FROM players WHERE job_id <> 6;
      ```
      これで勇者(job_id = 6)以外を取得できる。</br>
      </br>
      また、`AND`や`OR`を用いることで複数の条件を組み合わせることもできる。
      例：
      ```sql
      -- 勇者以外でレベルが7以上のプレイヤーの、名前とレベルだけを取得する。
      SELECT name,level FROM players WHERE level >= 7 AND job_id <> 6;

</br>

***

### 1-4_いろいろな情報を取り出そう
今回は次のような内容を実行する。
1. データ件数を表示する
2. 条件に合ったデータの件数を表示する
3. 並び替える
4. 上位3件だけを表示する
5. 職業ごとの人数を調べる
</br>

1. データ件数を表示する
   ```sql
   SELECT COUNT(*) FROM players;
   ```
   ↓出力結果
   ```sql
   COUNT(*)
   10
   ```
</br>

2. 条件に合ったデータの件数を表示する
   ```sql
   -- jobが勇者の数を調べる
   SELECT COUNT(*) FROM players WHERE job_id = 6;
   ```
   ↓出力結果
   ```sql
   COUNT(*)
   2
   ```
</br>

3. 並び替える
   ```sql
   -- データをそのカラムに基づいて並び替える
   SELECT * FROM players ORDER BY level;
    ```
    - `ORDER BY カラム名`でカラム名の昇順に並び替える。</br>
      また、降順(大きい順)にする場合はカラム名の後に`DESC`を追記する。(DESC:descending(下り)の略)</br>
    例：
    ```sql
    SELECT * FROM players ORDER BY level DESC;
    ```
</br>

4. 上位3件だけを表示する</br>
   ```sql
   -- levelが高い順に3件表示
   SELECT * FROM players ORDER BY level DESC LIMIT 3;
   ```
   - `LIMIT`はリミット命令という。表示する数を指定する。</br>

5. 職業ごとの人数を調べる</br>
   ```sql
   -- グループ化を用いて、職業ごとの人数を集計する。
   SELECT job_id, COUNT(*) FROM players GROUP BY job_id;
   ```
   - `GROUP BY` 指定のカラムをまとめる。</br>
</br>

***

### 1-5_データを追加・更新・削除しよう
1. データを追加する
2. 一度に複数のデータを追加する
3. データを更新する
4. データを削除する
5. 条件に合ったデータを削除する
</br>

1. データを追加する</br>
   ```sql
    -- playerテーブルにデータを追加する
    INSERT INTO players(id,name,level,job_id)VALUES(11,"モグラ1号",1,1);
    ```
    これでデータが追加される。</br>
    - `INSERT INTO 追加先のテーブル(id,カラム)VALUES(id,カラムに入れるvalue);`</br>
      ここでidが既存のものだとエラーが生じてデータを挿入できない。</br>
    </br>
    次にデータの挿入と表示を同時に行ってみる。
    ```sql
    INSERT INTO players(id,name,level,job_id)VALUES(12,"モグラ2号",1,1);
    SELECT * FROM players;
    ```
    このようにSQL文は一度の入力で複数のSQLを実行できる。</br>
</br>

2. 一度に複数のデータを追加する
   ```sql
   INSERT INTO players(id,name,level,job_id)
   VALUES
    (13,"モグラ3号”,1,1,),
    (14,"モグラ4号”,1,1,)
   ;
   SELECT * FROM players;
   ```
</br>

3. データを更新する</br>
   すでに記録されているデータの値を上書きして更新する</br>
   ```sql
   UPDATE players SET level = 10 WHERE id = 11;
   SELECT * FROM players;
   ```
   これでモグラ1号のlevelが10になった。</br>
   - `UPDATE テーブル名 SET 更新する内容 WHERE 更新する対象;`</br>
   また、下記のような対象に何か加えるというような更新の仕方も可能。
   ```sql
   UPDATE players SET level = level + 1 WHERE id = 12;
   SELECT * FROM players;
   ```
   これでモグラ2号のlevelが2になった。</br>
</br>

4. データを削除する
   ```sql
   DELETE FROM players WHERE id = 12;
   SELECT * FROM players;
   ```
</br>

5. 条件に合ったデータを削除する</br>
   `id=11`以上のデータを削除する。</br>
   ```sql
   DELETE FROM players WHERE id >= 11;
   SELECT * FROM players;
   ```
</br>

***

### 1-6_2つのテーブルを結合しよう
たとえばプレイヤーと職業といったデータは、一つのテーブルとして記述することも可能だが、
そのステータスや職業名など同じデータを重複して記録する必要があるため膨大なデータ量を￥となる。</br>
テーブルを分割して管理し、結合して使うことで効率的なデータの運用ができる。</br>
</br>

**内部結合**
```sql
-- テーブルを結合して表示する(内部結合)
SELECT * FROM players INNER JOIN jobs ON jobs.id = players.job_id;
```
※この時、対象カラムのvalueがnullのレコードは表示されない。</br>
つまり2つのテーブルに共通するデータのみが表示される。</br>
このような表示状態を**内部結合**という。</br>
</br>

**外部結合**</br>
次に、対象カラムのvalueがnullのレコードも表示されるように記述してみる。
```sql
-- テーブルを結合して表示する(左結合)
SELECT * FROM players LEFT JOIN jobs ON jobs.id = players.job_id;
```
今度は対象カラムの`job_id`がnullのレコードも表示された。</br>
このときの表示状態を**左結合**という。</br>
今回はplayersテーブル(FROMで指定されたテーブル)に含まれる情報を全て表示している。</br>
</br>

さらに次はjobsテーブルに含まれるデータを結合して表示してみる。
```sql
-- テーブルを結合して表示する(右結合)
SELECT * FROM players RIGHT JOIN jobs ON jobs.id = players.job_id;
```
今度はplayersテーブルにはない”僧侶"ジョブが表示された。</br>
このときの表示を**右結合**という。</br>
今回はjobsテーブル(JOINで指定されたテーブル)に含まれる情報を全て表示している。</br>
</br>

また、この他に左右いずれかに存在しているレコードを全て表示する**完全外部結合**というものがある。</br>
</br>

***

### 1-7_結合したテーブルを操作しよう
結合したカラムに対してのDBの基本操作を実行してみる。</br>
今回実行するのは次の項目
1. 結合したテーブルで、指定したカラムだけ表示する
2. 結合したテーブルで、条件に合った行だけ表示する
3. 結合したテーブルで、集計する</br>
</br>

1. 結合したテーブルで、指定したカラムだけ表示する
   ```sql
   SELECT name,level,vitality FROM players INNER JOIN jobs ON jobs.id = players.job_id;
   ```
</br>

2. 結合したテーブルで、条件に合った行だけ表示する
   ```sql
   SELECT * FROM players INNER JOIN jobs ON jobs.id = players.job_id WHERE level >= 5;
   ```
</br>

3. 結合したテーブルで、集計する
   ```sql
   SELECT job_id,job_name, COUNT(*) FROM players INNER JOIN jobs on jobs.id = players.job_id GROUP BY job_id;
   ```
