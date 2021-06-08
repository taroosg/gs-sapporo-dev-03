# 認証処理の実装2（認証状態の確認）

todoリストの機能はログインしているときのみアクセスできるようにしたい！

現状では，ログアウトした状態でもURLを手打ちするとアクセスできてしまう．．．
- 登録画面，一覧画面などはログイン済ユーザのみ見られるようにする
- ログインをチェックし，ログインしていない状態ならログイン画面に移動

## 「ログインしている」とはなにか

以下の2つの状態を「ログインしていない」とみなす！！

- 「セッション変数に`session_id`を持っていない」
- 「idが最新ではない」はログインしていない状態


## 関数の定義

上記2条件を用いてログインの有無を判断する関数を作成する．

関数の役割は
1. ログインしていない場合はログインページへ強制送還．
2. ログインしている場合は`session_id`を更新してセッション変数に保存する．
    - セッション変数には常に最新の`session_id`が入っている状態にする．

複数のファイルでチェックを行うため，関数ファイルに記述しよう！

```php
// functions.php

function check_session_id()
{
  if (!isset($_SESSION["session_id"]) ||$_SESSION["session_id"] != session_id()) {
    header('Location:todo_login.php');
    exit();
  } else {
    session_regenerate_id(true);
    $_SESSION["session_id"] = session_id();
  }
}

```


## 関数の実行

あとは定義した関数を「ログインしていないときに動いてほしくないPHPファイル」で実行しよう．

セッションの機能を使用するため，`session_start();`を忘れないように注意！

下記ファイルで実行すると良き．
- `todo_input.php`
- `todo_read.php`
- `todo_edit.php`
- `todo_create.php`
- `todo_create.php`
- `todo_update.php`
- `todo_delete.php`

```php
// ログインしていないときに動いてほしくないPHPファイル

<?php
session_start();
include('functions.php');
check_session_id();

```


## 練習

メインのコンテンツは未ログインだとアクセスできないようにしよう！

- ログイン状態を確認する関数をつくろう！（functions.php）
- コンテンツのページで上記関数を実行しよう！

ログアウトした状態で各ページにアクセスできない状態（ログインページに移動）になればOK！