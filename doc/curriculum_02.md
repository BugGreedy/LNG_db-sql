## SQL入門編2: SQLを仕事に使おう

### 目次
[2−1_仕事にもSQLを使おう](#2−1_仕事にもSQLを使おう)</br>
[2-2_SQLの書き方のポイント](#2-2_SQLの書き方のポイント)</br>
[2-3_ログ解析してみよう](#2-3_ログ解析してみよう)</br>
[2-4_アクティブユーザーを調べよう](#2-4_アクティブユーザーを調べよう)</br>
[2-5_データを集計しよう](#2-5_データを集計しよう)</br>
[2-6_ユーザーの年齢を計算をしよう](#2-6_ユーザーの年齢を計算をしよう)</br>
[2-7_テキストを検索しよう](#2-7_テキストを検索しよう)</br>
[2-8_サブクエリでアクティブユーザー数を求めよう](#2-8_サブクエリでアクティブユーザー数を求めよう)</br>
[2-9_グループ分けしよう](#2-9_グループ分けしよう)</br>
[2-10_クロス集計してみよう](#2-10_クロス集計してみよう)</br>
[2-11_サブクエリで、平均や割合を求めよう](#2-11_サブクエリで、平均や割合を求めよう)</br>
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
  - `MAX()`：指定カラムの最大値を求める</br>
</br>


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
ここで用いるのが**TIMESTAMPDIFF関数**である。
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
</br>

***

### 2-7_テキストを検索しよう
SQL内でテキストの検索を行う際は**LIKE命令**
- `WHERE user.user_name LIKE '%マン'`：user.user_nameの中から"○○マン"というレコードだけ取得する。
```sql
-- 「○○との闘い」のみを取得する。
SELECT
	userID,
	startTime,
	events.event_summary
FROM
	eventlog
	INNER JOIN events ON events.eventID = eventlog.eventID
WHERE events.event_stage <> 0
    AND events.event_summary LIKE '%との闘い'
ORDER BY
    userID, startTime;
```
備考：`LIKE '%との闘い'`の`%`はワイルドカード。</br>
↓出力結果
```
userID	startTime	event_summary
2	2015-02-01 23:21:07	亡霊との闘い
2	2015-02-02 22:00:02	亡霊との闘い
2	2015-02-09 21:41:02	ダダメシとの闘い
2	2015-02-09 21:41:02	ザウルスとの闘い
2	2015-02-09 22:03:02	ズッポシとの闘い
2	2015-02-10 18:00:02	ズッポシとの闘い
2	2015-02-11 18:00:02	ズッポシとの闘い
2	2015-02-17 21:00:01	ズッポシとの闘い
```
</br>

次はそのイベントのURLを検索してみる。</br>
```sql
SELECT
	userID,
	startTime,
	events.event_summary,
	events.event_url
FROM
	eventlog
	INNER JOIN events ON events.eventID = eventlog.eventID
WHERE events.event_stage <> 0
    AND events.event_url LIKE '%dungeon%'
ORDER BY
    userID, startTime;
```
↓出力結果 ("dungeon"が含まれるURLのみ取得)
```
userID	startTime	event_summary	event_url
2	2015-02-01 22:53:08	パイザの洞窟	rpg/paiza/1/dungeon_0
2	2015-02-02 23:18:02	ダマスのほこら	rpg/paiza/2/dungeon_0
2	2015-02-09 21:18:02	ダダロスの廃城	rpg/paiza/2/dungeon_1
2	2015-02-09 21:55:02	ザッパスの洞窟	rpg/paiza/3/dungeon_0
2	2015-02-17 21:00:01	ダガナログの洞窟	rpg/paiza/4/dungeon_1
2	2015-02-17 21:46:01	ダガナの鉱山	rpg/paiza/4/dungeon_0
2	2015-02-17 22:46:01	ダガナログの洞窟	rpg/paiza/4/dungeon_1
4	2015-02-01 23:53:07	パイザの洞窟	rpg/paiza/1/dungeon_0
4	2015-02-09 21:18:02	ダダロスの廃城	rpg/paiza/2/dungeon_1
4	2015-03-02 22:24:02	ザッパスの洞窟	rpg/paiza/3/dungeon_0
4	2015-03-02 23:02:02	ダガナの鉱山	rpg/paiza/4/dungeon_0
```
</br>

***

### 2-8_サブクエリでアクティブユーザー数を求めよう
**サブクエリとは**
- SQLを実行した結果をまた別のSQLと組み合わせる機能。</br>
- サブクエリでは、クエリの出力結果をFROMに指定できる。</br>
```sql
SELECT day, COUNT(user)
FROM (SELECT  DISTINCT           -- FROM(sql)でsqlの結果を利用できる。
	DATE(startTime) AS day,
	eventlog.userID AS user
FROM eventlog
	INNER JOIN users ON users.userID = eventlog.userID
WHERE deleted_at IS NULL) AS ActiveUsers  -- ここでAS~と付けないとエラーになる。
GROUP BY day;
```
</br>

***

### 2-9_グループ分けしよう
SQLの**CASE命令**</br>
- 条件に合わせて出力内容を変える。</br>
  プログラミングのif文に似ている。
  ```sql
  -- CASEの基本形
  SELECT
  	userID,
  	level,
  	CASE
  		WHEN (条件式1) THEN (出力1)
  		WHEN (条件式2) THEN (出力2)
  		ElSE (出力3)
  	END
  FROM
  	users
  ```
 </br>

では、ユーザーをレベルに合わせて、クラス分けをしてみる。
```sql
-- レベル4以上を'上級'、レベル2以上を'中級'、それ以外を'初級'とする。
SELECT
	userID,level,
    CASE
        WHEN level >= 4 THEN '上級'
        WHEN level >= 2 THEN '中級'
        ELSE '初級'
    END AS クラス
FROM
	users;
```
↓出力結果
```
userID	level	クラス
1	0	初級
2	5	上級
3	1	初級
4	4	上級
5	0	初級
6	3	中級
7	1	初級
8	3	中級
9	1	初級
10	1	初級
11	0	初級
12	3	中級
13	1	初級
14	3	中級
```
</br>

**注意**</br>
ただし、この時`WHEN level >=2`を`WHEN level >=4`の前に記述してしまうと上級が表示されなくなってしまう。</br>
これはCASE文が上から実行されるからである。</br>
例：
```sql
userID	level	クラス
1	0	初級
2	5	中級  -- userID=2がレベル5なのに'中級'として出力されている。
3	1	初級
4	4	中級
5	0	初級
6	3	中級
7	1	初級
8	3	中級
```
</br>

次に、これらのクラスごとの人数を集計してみる。
```sql
SELECT
    CASE
        WHEN level >= 2 THEN '中級'
        WHEN level >= 4 THEN '上級'
        ELSE '初級'
    END AS クラス,
    COUNT(*) AS ユーザー数
	
FROM
	users
GROUP BY クラス;
```
↓出力結果
```sql
クラス	ユーザー数
初級	60
中級	40
```
</br>

***

### 2-10_クロス集計してみよう
通常のビューでは集計結果が共通のカラム構成で縦に並ぶ。例：イベント開始日/userID/クラス</br>
これに対してサブクエリを用いて、集計結果を縦横に並ぶビューを**クロス集計**という。</br>
- クロス集計表を作る手順
  1. クロス集計の元になるデータを用意する
  2. サブクエリを読み込む
  3. CASEで、特定の値だったら1にする。このときASで指定する別名(カラム名)を特定の値と同じにする
     - 例：CASE WHEN クラス = '初級' THEN 1 ELSE 0 END AS 初級

では、クロス集計をする前の手順を実行する。
```sql
SELECT
    日付,
    ユーザー,
    クラス,
    CASE WHEN クラス = '初級' THEN 1 ELSE 0 END AS 初級,
    CASE WHEN クラス = '中級' THEN 1 ELSE 0 END AS 中級,
    CASE WHEN クラス = '上級' THEN 1 ELSE 0 END AS 上級
FROM(SELECT DISTINCT
	DATE_FORMAT(startTime,'%Y-%m') AS 日付,
	eventlog.userID AS ユーザー,
	CASE
	    WHEN users.level >= 4 THEN '上級'
	    WHEN users.level >= 2 THEN '中級'
	    ELSE '初級'
	END AS クラス
FROM eventlog
	INNER JOIN users ON users.userID = eventlog.userID)
	AS  クラス分け;
```
↓出力結果
```
日付	ユーザー	クラス	初級	中級	上級
2015-02	1	初級	1	0	0
2015-02	3	初級	1	0	0
2015-02	2	上級	0	0	1
2015-02	4	上級	0	0	1
2015-02	5	初級	1	0	0
2015-02	6	中級	0	1	0
2015-02	8	中級	0	1	0
2015-02	7	初級	1	0	0
2015-02	9	初級	1	0	0
2015-02	10	初級	1	0	0
2015-03	11	初級	1	0	0
2015-03	12	中級	0	1	0
```
これで各クラスに対するカラムを追加できた。</br>
あとはこのクラスごとのカラムを集計して日付ごとにグループ化を行う。
```sql
SELECT
    日付,
    SUM(CASE WHEN クラス = '初級' THEN 1 ELSE 0 END) AS 初級,
    SUM(CASE WHEN クラス = '中級' THEN 1 ELSE 0 END) AS 中級,
    SUM(CASE WHEN クラス = '上級' THEN 1 ELSE 0 END) AS 上級
FROM(SELECT DISTINCT
	DATE_FORMAT(startTime,'%Y-%m') AS 日付,
	eventlog.userID AS ユーザー,
	CASE
	    WHEN users.level >= 4 THEN '上級'
	    WHEN users.level >= 2 THEN '中級'
	    ELSE '初級'
	END AS クラス
FROM eventlog
	INNER JOIN users ON users.userID = eventlog.userID)
	AS  クラス分け
GROUP BY 日付;
```
↓出力結果
```
日付	初級	中級	上級
2015-02	6	2	2
2015-03	10	8	1
2015-04	10	8	0
```
</br>

***

### 2-11_サブクエリで、平均や割合を求めよう 
今回は平均やそれ以上の割合を求めてみる。</br>
まず、平均レベルを調べる。
```sql
SELECT AVG(level) AS 平均レベル FROM users;

SELECT userID, name, level
FROM users
WHERE level >= 4;
```
↓出力結果
```
平均レベル
1.6300
userID	name	level
2	シュン	5
4	さゆり	4
```
</br>

次に算出した平均以上のユーザーを表示する。
```sql
SELECT AVG(level) AS 平均レベル FROM users;

SELECT userID, name, level
FROM users
WHERE level >= (SELECT AVG(level) AS 平均レベル FROM users);   -- 上記の平均を算出した式をWHEREに記述
```
↓出力結果
```
平均レベル
1.6300
userID	name	level
2	シュン	5
4	さゆり	4
6	サキ	3
8	けんぞう	3
12	しゅんじ	3
14	ソウタ	3
...
```
</br>

次に平均以上のユーザー数を算出する。
```sql
SELECT AVG(level) AS 平均レベル FROM users;

SELECT COUNT(userID) AS 平均以上のユーザー数
FROM users
WHERE level >= (SELECT AVG(level) AS 平均レベル FROM users);
```
↓出力結果
```
平均レベル
1.6300
平均以上のユーザー数
40
```
</br>

次に平均以上のユーザー数の割合を求める。</br>
まず全体の数を求め、それを元に算出する。
```sql
SELECT AVG(level) AS 平均レベル FROM users;

SELECT  
    COUNT(userID) AS 平均以上のユーザー数,                   
    (SELECT COUNT(*) FROM users ) AS 全体のユーザー数,               -- ※1
    COUNT(userID) / (SELECT COUNT(*) FROM users) *100 AS 割合
FROM users
WHERE level >= (SELECT AVG(level) AS 平均レベル FROM users);
```
↓出力結果
```
平均レベル
1.6300
平均以上のユーザー数	全体のユーザー数	割合
40	100	40.0000
```
**※1**：SELECT内に直接SQL文を代入する事ができる。</br>
</br>


