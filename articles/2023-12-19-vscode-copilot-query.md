---
title: "VSCode GitHub Copilotでいい感じのクエリを作ってもらおう"
emoji: "🤞"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["VSCode", "Copilot", "SQL"]
published: false
publication_name: "layerx"
---

この記事は[Visual Studio Code Advent Calendar 2023](https://qiita.com/advent-calendar/2023/vscode) 19日目の記事です。

もうすぐクリスマスですね。我が家は3歳の息子がいるのですが、今更「ツリー買い忘れたけどなんか飾り付けほしいなぁ」と思ってダイソーでツリーが印刷された布を買い、布を壁に貼ってシールや工作物をペタペタ貼ることでそれっぽいものができました。ツリーのゴミも散らからないので、もし悩んでいる方がいればぜひお試しください

DALL-Eで出力してもらったイメージはこちらになります（こんなにちゃんとしていませんが）

![christmastree](/images/christmastree.png)

はじめまして、LayerX 機械学習チームの [@yakipudding](https://twitter.com/yakipudding) です。

突然ですが、私はSQLを書くのが好きではないです。業務での調査や分析によくSQLを使用するのですが、構文やJOIN、どの項目がどのテーブルにあるのかなどを都度確認しながら書くのが面倒で、億劫に感じます。

そんな時、「VSCodeのGitHub Copilot拡張機能を使ってクエリを生成すれば良いのでは？」と考え、試したところ良い結果が得られたので、今回はその方法をご紹介します。

（※VSCode以外の他のGitHub Copilotが対応しているエディタでも同じことはできます）

# 1. クエリ生成に必要なプロンプト

まずCopilot関係なしに、生成AIにクエリを生成してもらう上で必要なプロンプトについて考えました。

## 1-1. DDL

クエリ生成のプロンプトとして、DDLを提供することが考えられます。DDLを使用すると、テーブルのカラム、型、関係性などのデータ構造情報を提供することができます。

例えばプロジェクトタスク管理システムのDBが以下のように定義されたとして

（ちなみにこのサンプルDDLもCopilot Chatで生成してもらいました）

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE projects (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE project_task_links (
    id INT PRIMARY KEY,
    project_id INT,
    task_id INT,
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (task_id) REFERENCES tasks(id)
);

CREATE TABLE tasks (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    description TEXT,
    status VARCHAR(20),
    due_date DATE
);

CREATE TABLE sub_tasks (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    task_id INT,
    status VARCHAR(20),
    due_date DATE,
    FOREIGN KEY (task_id) REFERENCES tasks(id)
);

CREATE TABLE tags (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE task_tag_links (
    id INT PRIMARY KEY,
    task_id INT,
    tag_id INT,
    FOREIGN KEY (task_id) REFERENCES tasks(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);

CREATE TABLE sub_task_tag_links (
    id INT PRIMARY KEY,
    sub_task_id INT,
    tag_id INT,
    FOREIGN KEY (sub_task_id) REFERENCES sub_tasks(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);

CREATE TABLE task_user_links (
    id INT PRIMARY KEY,
    task_id INT,
    user_id INT,
    FOREIGN KEY (task_id) REFERENCES tasks(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE sub_task_user_links (
    id INT PRIMARY KEY,
    sub_task_id INT,
    user_id INT,
    FOREIGN KEY (sub_task_id) REFERENCES sub_tasks(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

このDDLの下に

```
/*
以下のSQLを作成して
ProjectAに紐づいた未消化タスク一覧
*/
```

とコメントするだけで

```sql
SELECT
    tasks.id,
    tasks.name,
    tasks.description,
    tasks.status,
    tasks.due_date
FROM
    tasks
INNER JOIN
    project_task_links
ON
    tasks.id = project_task_links.task_id
INNER JOIN
    projects
ON
    project_task_links.project_id = projects.id
WHERE
    projects.name = 'ProjectA'
AND
    tasks.status = '未消化';
```

と出力してくれました。条件など修正する必要がある可能性がありそうですが、それっぽいクエリを作ってくれました。

## 1-2. サンプルクエリ

規模の大きなシステムのデータベースになると、テーブル数が多かったり、データの持ち方が複雑になることがよくあります。

そのような状況でDDLの情報だけを用いてクエリを生成しようとすると、不要なテーブルをJOINしてしまったり、必要なテーブルのJOINをスキップしてしまうことがありました。

クエリの形がある程度決まっていてSELECTするカラムや条件を必要に応じて変更したい場合は、サンプルクエリを与えることでいい感じのクエリを作成することができます。

例えばサンプルクエリとして先ほど出力されたクエリ

```sql
/*
以下のSQLを作成して
ProjectAに紐づいた未消化タスク一覧
*/
SELECT
    tasks.id,
    tasks.name,
    tasks.description,
    tasks.status,
    tasks.due_date
FROM
    tasks
INNER JOIN
    project_task_links
ON
    tasks.id = project_task_links.task_id
INNER JOIN
    projects
ON
    project_task_links.project_id = projects.id
WHERE
    projects.name = 'ProjectA'
AND
    tasks.status = '未消化';
```

を与えた上で

```sql
/*
以下のSQLを作成して
userAが担当していて期限が過ぎているタスク一覧
*/
```

とコメントすると

```sql
SELECT
    tasks.id,
    tasks.name,
    tasks.description,
    tasks.status,
    tasks.due_date
FROM
    tasks
INNER JOIN
    user_task_links
ON
    tasks.id = user_task_links.task_id
INNER JOIN
    users
ON
    user_task_links.user_id = users.id
WHERE
    users.name = 'userA'
AND
    tasks.due_date < '2019-01-01';
```

というクエリを生成してくれました。日付の条件は修正する必要がありそうですが、いい感じにサンプルクエリと似たような構成でクエリを作ってくれました。

# 2. VSCode GitHub Copilot拡張ならではの活用法

これまでの試みでも十分いい感じのクエリを作成してくれますが、ChatGPT等でも可能なプロンプトの話になっていました。

以降はVSCode GitHub Copilot拡張ならではの活用法を紹介したいと思います。

## 2-1. 命令文のテンプレートを用意

プロンプトのうち命令文に該当する

```sql
/*
以下のSQLを作成して
userAが担当していて期限が過ぎているタスク一覧
*/
```

の部分ですが、実際は細かく条件や取得するカラムを指定しないと希望したクエリを生成してくれないこともあります。

細かく条件や取得するカラムを書こうとすると、以下のようになります。

```sql
/*
以下のSQLを作成してください
条件:
  task idが100,101,205
  user idが12345
取得項目:
  task id
  user id
  project name
  task name
  due date
*/
```

これを毎回1から書く場合、自然言語でクエリを書いているような気分になって、面倒になりますよね。

そこでおすすめなのが、クエリ作成用のプロジェクトを作成しておいて、DDLとサンプル命令文・クエリをテンプレートとして保存するやり方です。

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

...

/*
以下のSQLを作成してください
条件:
  task idが100,101,205
  user idが12345
取得項目:
  task id
  user id
  project name
  task name
  due date
*/
SELECT
    tasks.id,
    tasks.name,
    tasks.description,
...
```

クエリを生成する際は、サンプル命令文をコピーして、必要に応じて変更するだけです。差分の修正だけで済むため、簡単にプロンプトを作成し、すぐにクエリを生成できます。

```sql
/*
以下のSQLを作成してください
条件:
  user idが56789 <-条件だけ変更
  期限が昨日まで
取得項目:
  task id
  user id
  project name
  task name
  due date
*/
```

## 2-2. 誤った箇所のみクエリを修正する

生成されたクエリが間違っていたり、プロンプトの指示が守られていない場合もあります。

チャットの場合、何が間違っているのかを自然言語で伝える必要がありますが、伝えること自体が面倒になったりしますよね。

VSCode GitHub Copilotの場合、誤った部分だけを削除して、該当箇所のみ再生成を指示することができます。

```sql
/*
以下のSQLを作成して
userAが担当していて期限が過ぎているタスク一覧
*/
SELECT
    tasks.id,
    tasks.name,
    tasks.description,
    tasks.status,
    tasks.due_date
FROM
    tasks
INNER JOIN
    user_task_links
ON
    tasks.id = user_task_links.task_id
INNER JOIN
    users
ON
    user_task_links.user_id = users.id
WHERE
    users.name = 'userA'
AND
--    tasks.due_date < '2019-01-01'; <- コメントアウト
-- 期限が昨日より前 <- 追加の指示
tasks.due_date < DATE_SUB(CURDATE(), INTERVAL 1 DAY); <- 追加の生成
```

これがサクッとできるのが非常に便利だなと〜思っています。

# 最後に

VSCode GitHub Copilotでいい感じのクエリを作ってもらうTipを紹介しました。

ぜひVSCode GitHub Copilotを活用して快適なクエリ生成ライフをお過ごしください！