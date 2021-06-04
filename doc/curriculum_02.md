## SQL入門編2: SQLを仕事に使おう

### 目次
[2−1_仕事にもSQLを使おう](#2−1_仕事にもSQLを使おう)</br>
[2-2_SQLの書き方のポイント](#2-2_SQLの書き方のポイント)</br>
[2-3_ログ解析してみよう](#2-3_ログ解析してみよう)</br>
[2-4_アクティブユーザーを調べよう](#2-4_アクティブユーザーを調べよう)</br>
</br>

***

### 2−1_仕事にもSQLを使おう
テーブルの結合は3つ以上のテーブルでもできる。</br>
```sql
SELECT * FROM users
INNER JOIN jobs ON jobs.jobID = users.jobID
INNER JOIN area ON area.areaID = users.areaID;
```
</br>

このようなSQLのコードの事を**問い合わせ、クエリー**などと呼ぶ。</br>
またクエリーを実行した後の表を**ビュー**という。</br>
</br>

***

### 2-2_SQLの書き方のポイント
**SQL文の基本**</br>

- 基本的にはセミコロンまでが一つのかたまり。</br>
  例：
  ```SQL
  SELECT *
  FROM users
    INNER JOIN jobs ON jobs.jobID = users.jobID
    INNER JOIN area ON area.areaID = users.areaID;
  ```
  など、わかりやすいところで改行して良い。</br>
  </br>

- 複数のカラムを指定する時は**カンマを入れないといけない。**</br>
  </br>

- 結合テーブルのどちらにもあるカラム(`job_id`や単純に`ID`など)はその表示したいカラムのテーブル名を一緒に記述する必要がある。</br>
  例：
  ```sql
  -- 書き方のポイント
  SELECT userID,name,area.areaID
  FROM users
    INNER JOIN jobs ON jobs.jobID = users.jobID
    INNER JOIN area ON area.areaID = users.areaID;
  ```
  </br>

- `WHERE 条件式`で複数の条件をつけるときは`AND`でつなぐ。</br>
  例：
  ```sql
  SELECT userID,name,area.areaID
    FROM users
      INNER JOIN jobs ON jobs.jobID = users.jobID
      INNER JOIN area ON area.areaID = users.areaID
    WHERE
      userID >= 10
      AND userID < 20; 
  ```
</br>

***

### 2-3_ログ解析してみよう
**DATE関数**：`DATE(日時の値)`日時表示のデータを日付にする
```sql
SELECT startTime,logID FROM eventlog;
-- 出力結果 → 2015-02-01 20:00:00	1

SELECT DATE(startTime),logID FROM eventlog;
-- 出力結果 → 2015-02-01	1
```
</br>

- 日時別にアクセス数を求める。
  ```sql
  SELECT DATE(startTime),COUNT(logID)
  FROM eventlog
  GROUP BY DATE(startTime);
  ```
  
</br>

- 指定期間のアクセス数を求める。
  ```sql
  SELECT DATE(startTime),COUNT(logID)
  FROM eventlog
  WHERE DATE(startTime) BETWEEN '2015-04-01' AND '2015-04-30'
  GROUP BY DATE(startTime);
  ```
  - `BETWEEN 日付1 AND 日付2`：日付1から日付2の期間の条件式
</br>

- 月次のアクセス数を求める。
  ```sql
  SELECT DATE_FORMAT(startTime,'%Y-%m'),COUNT(logID)
  FROM eventlog
  GROUP BY DATE_FORMAT(startTime,'%Y-%m');
  ```
  - `DATE_FORMAT(日時の値,'%Y-%mなどの指定する形')`： %Y-%mなどの指定する形の引数の表示にする。
    出力結果は 2015-02 のようになる。</br>
</br>

***

### 2-4_アクティブユーザーを調べよう
- カラム名の表示を変更できる。</br>
  `SELECT COUNT(*) FROM users;`などを実行した際、カラム名が`COUNT(*)`などになる。</br>
  この時`AS`を用いる事でカラム名を変更できる。</br>
  ```sql
  SELECT COUNT(*) AS アクティブユーザー数 FROM users;
  ```
  カラム名は`アクティブユーザー数`と表示される。
</br>

- 退会したユーザーを計測する</br>
  仮に退会したタイミングを記録する`deletedTime`のようなカラムがあった際、現在のアクティブユーザーのレコードには`null`が入っている。つまりこのカラムには何もない状態になっている。</br>
  退会ユーザー数を計測する際はテーブルからこの`null`のレコードを取り除いた数を集計する。</br>
  - `deleted_at 取り除く対象`：取り除く対象を取り除く。
  - `IS NULL`：空のカラム。
  ```sql
  SELECT COUNT(*) AS アクティブユーザー数
  FROM users
  WHERE deleted_at IS NULL;
  ```
</br>



