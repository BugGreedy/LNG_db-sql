## SQL入門編2: SQLを仕事に使おう

### 目次
[2−1_仕事にもSQLを使おう](#2−1_仕事にもSQLを使おう)</br>
[2-2_SQLの書き方のポイント](#2-2_SQLの書き方のポイント)</br>
[2-3_ログ解析してみよう](#2-3_ログ解析してみよう)</br>
[2-4_アクティブユーザーを調べよう](#2-4_アクティブユーザーを調べよう)</br>
[2-5_データを集計しよう](#2-5_データを集計しよう)</br>
[2-6_ユーザーの年齢を計算をしよう](#2-6_ユーザーの年齢を計算をしよう)</br>
[2-7_テキストを検索しよう](#2-7_テキストを検索しよう)</br>
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
  仮に退会したタイミングを記録する`deleted_at`のようなカラムがあった際、現在のアクティブユーザーのレコードには`null`が入っている。つまりこのカラムには何もない状態になっている。</br>
  退会ユーザー数を計測する際はテーブルからこの`null`のレコードを取り除いた数を集計する。</br>
  - `IS NULL`：空のカラム。
  ```sql
  SELECT COUNT(*) AS アクティブユーザー数
  FROM users
  WHERE deleted_at IS NULL;
  ```
</br>

- イベントログに記録があるユーザーだけを調べる。
  上記の"退会したユーザー数"からの算出でアクティブユーザーを割り出すと登録はしたがプレイはしていない幽霊ユーザー数がカウントされてしまう。</br>
  そこでイベントログに記録があるユーザーだけを計測し、アクティブユーザー数を調べる。</br>
  </br>
  まずイベントログにあるユーザーIDを重複なしで計測する。
  ```sql
  SELECT DISTINCT userID AS アクティブユーザー
  FROM eventlog;
  ```
  - `DISTINCT`:重複したデータを除外する。
  </br>

  これを用いてユーザー先ほどのNULLを取り除く記述と合わせて計測してみる。
  ```sql
  SELECT COUNT(DISTINCT userID) AS アクティブユーザー
  FROM eventlog;
    INNER JOIN users ON users.userID = eventlog.userID
  WHERE deleted_at IS NULL;
  ```

- 日毎のアクティブユーザー数を計測する。
  ```sql
  SELECT
    DATE(eventlog.startTime) AS 日付,
    COUNT(DISTINCT eventlog.userID) AS アクティブユーザー
  FROM eventlog
    INNER JOIN users ON users.userID = eventlog.userID
  WHERE deleted_at IS NULL
  GROUP BY DATE(eventlog.startTime);
  ```
</br>

***

### 2-5_データを集計しよう
- sql上で集計(合算)をする</br>
  ```sql
  SELECT
  	eventlog.userID AS ユーザーID,
  	SUM(events.increase_exp) AS 獲得経験値合計
  FROM
  	eventlog
  	INNER JOIN events ON events.eventID = eventlog.eventID
  GROUP BY eventlog.userID;
  ```
  - `SUM()`で対象を和算できる。
</br>

- 平均を求める。</br>
  ```sql
  SELECT
  	eventlog.userID AS ユーザーID,
  	SUM(events.increase_exp) AS 獲得経験値合計,
  	AVG(events.increase_exp) AS 獲得経験値平均
  FROM
  	eventlog
  	INNER JOIN events ON events.eventID = eventlog.eventID
  GROUP BY eventlog.userID; 
  ```
  - `AVG()`:平均を求める。
</br>

- 集計結果から表示するデータを指定する。
  ```sql
  -- 獲得合計値が3000以上のレコードのみ表示
  SELECT
  	eventlog.userID AS ユーザーID,
  	SUM(events.increase_exp) AS 獲得経験値合計,
  	AVG(events.increase_exp) AS 獲得経験値平均
  FROM
  	eventlog
  	INNER JOIN events ON events.eventID = eventlog.eventID
  GROUP BY eventlog.userID
  HAVING SUM(events.increase_exp) >= 3000;
  ```
  - `HAVING`: GROUP BYのあとでも記述できる条件追加。WHEREだとGROUP BYの前に記述しないと行けないので本件では使えない。</br>
    - ※`HAVING`と`ORDER BY`が両方ある場合は`HAVING`を先に記述する。</br>
      例：
      ```sql
      GROUP BY eventlog.userID
      HAVING SUM(events.increase_gold) >= 50
      ORDER BY eventlog.userID;
      ```
</br>

**SQLの動作の流れ**</br>
1. FROM      対象テーブルからデータを取り出す
2. WHERE     条件に一致するレコードを読み込み
3. GROUP BY  グループ化
4. HAVING    集計結果から絞り込み
5. SELECT    指定したカラムだけを表示</br>
</br>

- ユーザーごとのプレイ開始日と終了日を調べてみる</br>
  各ユーザーの各プレイごとのログイン時間(startTime)とログアウト時間(endTime)が記録されている。</br>
  この場合求めるのは"最初(最小日時)のログイン時間" と"最後(最大日時)のログアウト時間"である。
  ```sql
  SELECT
  	eventlog.userID AS ユーザーID,
  	MIN(startTime) AS 開始日,
  	MAX(endTime) AS 終了日
  FROM
  	eventlog
  GROUP BY eventlog.userID ORDER BY userID;  -- グループ化後でも並び替えできる。
  ```
  - `MIN()`：指定カラムの最小を求める
  - `MAX()`：指定カラムの最大値を求める

***

### 2-6_ユーザーの年齢を計算をしよう
SQLにおいて四則演算を使用できる。</br>
例：プレイ期間を計測する。
```sql
SELECT
	userID AS ユーザーID,
	MIN(startTime) AS 開始日,
	MAX(endTime) AS 終了日,
	DATE(MAX(endTime)) - DATE(MIN(startTime)) + 1 AS プレイ期間(日数)
FROM
	eventlog
GROUP BY userID;
```
</br>
次にユーザーの年例を調べてみる。</br>

まずユーザーの生年月日を調べる。
```sql
SELECT
	userID AS ユーザーID,
    birth AS 生年月日
FROM
	users;
```
現在の年数から取得した生年月日を引き、年齢を調べる。</br>
- `YEAR()`：引数の年数を取得する。</br>
- `CURRENT_DATE()`：現在の日時を取得する。</br>
```sql
SELECT
	userID AS ユーザーID,
	YEAR(CURRENT_DATE()) AS 現在年,
    birth AS 生年月日,
    YEAR(CURRENT_DATE()) - YEAR(birth) AS 数え年
FROM
	users;
```
ただし、これでは数え年なので満年齢がわからない。</br>
ここで用いるのが`TIMESTAMPDIFF関数`
- `TIMESTAMPDIFF(引数1,時間1,時間2)`:時間1と時間2の差を引数1の単位(YEAR/DATE/MONTH/HOURなど)を取得する。
```sql
SELECT
	userID AS ユーザーID,
	YEAR(CURRENT_DATE()) AS 現在年,
  birth AS 生年月日,
  YEAR(CURRENT_DATE()) - YEAR(birth) AS 数え年,
  TIMESTAMPDIFF(YEAR,birth,CURRENT_DATE()) AS 満年齢
FROM
	users;
```
↓出力結果
```
ユーザーID	現在年	生年月日	数え年	満年齢
1	2021	1995-04-03	26	26
2	2021	1980-10-19	41	40
3	2021	1978-04-21	43	43
4	2021	1980-10-02	41	40
```

***

### 2-7_テキストを検索しよう
