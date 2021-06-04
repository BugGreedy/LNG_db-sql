## SQL入門編2: SQLを仕事に使おう

### 目次
[2−1_仕事にもSQLを使おう](#2−1_仕事にもSQLを使おう)</br>
[2-2_SQLの書き方のポイント](#2-2_SQLの書き方のポイント)</br>
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

- 複数のカラムを指定する時はカンマを入れないといけない。</br>
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

### 2-3_