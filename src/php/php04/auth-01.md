# 認証処理の実装1（ログインとログアウト）

## 必要なファイル

ログイン処理に`todo_login.php`と`todo_login_act.php`，ログアウト処理に`todo_logout.php`を使用する．

|file|役割|
|-|-|
|`todo_login.php`|ログイン情報（username, password）を入力して送信する．|
|`todo_login_act.php`|送信されたデータを受け取り，DB関連の処理を実行する．|
|`todo_logout.php`|セッション，ログイン情報を破棄する．|

## ログイン処理

### ログイン処理の流れ

1. ログインフォーム情報を入力して送信（`todo_login.php`）
2. 送信されたデータを受け取る（`todo_login_act.php`）
3. 受け取ったデータがDBにあるかどうかチェック（`todo_login_act.php`）
    - 成功時（DBにユーザのデータが存在した場合）
        - DBにログイン情報があればセッション変数に格納（`todo_login_act.php`）
        - セッション変数にログイン情報を保持して`todo_read.php`に移動
    - 失敗時（DBにユーザのデータが存在しなかった場合）
        - `todo_login.php`に戻るリンクを表示する（ログイン失敗）

ログインフォームに`action`，`method`，`name`を設定する．

データを`todo_login_act.php`に送信する．

```php
// todo_login.php

<form action="todo_login_act.php" method="POST">
  // ...
  <div>
    username: <input type="text" name="username">	// name属性
  </div>
  <div>
    password: <input type="text" name="password">	// name属性
  </div>
  <div>
    <button>Login</button>
  </div>
  // ...
</form>

```

データを受け取ったら，ユーザのテーブルに該当するデータが存在するかどうかを確認する．

```php
// todo_login_act.php

<?php
session_start();
include('functions.php');

$username = $_POST['username'];
$password = $_POST['password'];

$pdo = connect_to_db();

// username，password，is_deletedの3項目全てを満たすデータを抽出する．
$sql = 'SELECT * FROM users_table WHERE username=:username AND password=:password AND is_deleted=0';

$stmt = $pdo->prepare($sql);
$stmt->bindValue(':username', $username, PDO::PARAM_STR);
$stmt->bindValue(':password', $password, PDO::PARAM_STR);
$status = $stmt->execute();

```

データの有無で条件分岐する．
- データが存在しない場合はログイン画面へ移動するリンクを表示する．
- データが存在した場合はセッション変数にsession_idとユーザのデータを入れ，一覧画面に移動する．

```php
if ($status == false) {
  $error = $stmt->errorInfo();
  echo json_encode(["error_msg" => "{$error[2]}"]);
  exit();
} else {
  $val = $stmt->fetch(PDO::FETCH_ASSOC);
  if (!$val) {
    echo "<p>ログイン情報に誤りがあります</p>";
    echo "<a href=todo_login.php>ログイン</a>";
    exit();
  } else {
    $_SESSION = array();
    $_SESSION['session_id'] = session_id();
    $_SESSION['is_admin'] = $val['is_admin'];
    $_SESSION['username'] = $val['username'];
    header("Location:todo_read.php");
    exit();
  }
}

```

### 💡 Key Point

>`session_id`はこの後使用するので必ず保存しよう．
>
>ユーザデータはアプリケーションに必要なものを適宜選別してセッションに保存する．


## ログアウト

ログアウトの処理は前項のセッション終了のコードそのもの．

ユーザの痕跡をなくすことが肝要．

```php
// todo_logout.php

<?php
session_start();
$_SESSION = array();
if (isset($_COOKIE[session_name()])) {
  setcookie(session_name(), '', time() - 42000, '/');
}
session_destroy();
header('Location:todo_login.php');
exit();

```

## 練習

ログインとログアウトの処理を実装しよう！

1. ログインフォームを実装！（`todo_login.php`）
2. ログイン処理を実装！（`todo_login_act.php`）
3. ログアウト処理を実装！（`todo_logout.php`）

以下の動作が確認できればOK．
- 存在するユーザのusernameとpasswordを入力して一覧ページが表示される．
- 存在しないユーザのusernameとpasswordを入力してログインページへのリンクが表示される．

ユーザデータがない場合は適当なデータを入れておこう．