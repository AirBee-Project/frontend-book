# Gitについて学ぼう

まずは自分のローカル環境で、ファイルの歴史を記録する「Git」の使い方を学びます。

## リポジトリを作成しよう
コマンドラインを開いて、今回の作業用フォルダを作成し、その中に移動します。
```bash
mkdir my-first-web
cd my-first-web

```


## `git init`

Gitを使うには、まず「このフォルダを記録してね」という合図を送る必要がある。これを **リポジトリの初期化** と言います。

以下のコマンドを実行してみましょう。

```bash
git init

```

> **解説**
> フォルダの中に非表示の `.git` という特別なフォルダが作られます。これがGitの実体です。

## `git status`

Gitは常にフォルダ内の様子を見守っています。今どういう状態なのかを教えてくれるのが `git status` です。

まずは、自己紹介ページとなる `index.html` を作ります。

**`index.html`** 

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>私の自己紹介</title>
</head>
<body>
    <h1>こんにちは！</h1>
    <p>Gitハンズオンへようこそ。これからWEBサイトを育てていきます！</p>
</body>
</html>

```

ファイルを作ったら、状態を確認します。

```bash
git status

```

## `git add`

Gitでは、いきなり変更を保存するのではなく、まずは「今回の記録に含めるファイル」をチョイスするステップがあります。これを **ステージング** と言います。

さっき作った `index.html` を準備状態にしましょう。

```bash
git add index.html

```

もう一度 `git status` で確認してみます。

```bash
git status

```


## `git commit`

準備した変更を、メッセージ付きで正式に記録します。この記録の単位を **コミット** と呼びます。ゲームでいう「セーブポイント」を作るイメージです。

```bash
git commit -m "Initial commit: 自己紹介ページのベースを作成"

```

もう一度 `git status` を打つと、`nothing to commit, working tree clean`と言われます。無事にセーブできました。


## `git branch`

ブランチとは、歴史を枝分かれさせて、他のメンバーの作業を邪魔せずに自分の新機能開発や実験ができる仕組みです。

### 1. ブランチの一覧を確認する

現在あるブランチを確認します。

```bash
git branch

```

最初は `main`（または `master`）という名前のブランチにいることがわかります。頭に `*` がついているのが、今自分がいるブランチです。

### 2. 新しいブランチを作成する

WEBサイトのデザインを整えるために、新しく `feature-style` という名前のブランチを作ってみましょう。

```bash
git branch feature-style

```

### 3. ブランチを切り替える

作ったブランチに移動します。

```bash
git switch feature-style

```

もう一度 `git branch` を叩くと、`* feature-style` に切り替わっているはずです。

### 4. 別世界でファイルを変更してコミットする

このブランチのまま、`index.html` を少しおしゃれに書き換えてみましょう。`<body>` タグに背景色をつけてみます。

**`index.html` （変更後）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>私の自己紹介</title>
</head>
<body style="background-color: #f0f8ff; font-family: sans-serif;">
    <h1>こんにちは！</h1>
    <p>Gitハンズオンへようこそ。これからWEBサイトを育てていきます！</p>
    <p>スタイルを追加しました！</p>
</body>
</html>

```

変更したら、さっき習った `add` と `commit` で保存します。

```bash
git add index.html
git commit -m "Add: 背景色とフォントのスタイルを追加"

```

### 5. グラフとして表示してみよう

これまでの歴史がどう枝分かれしているか、コマンドライン上でグラフとして見てみましょう。

```bash
git log --oneline --graph --all

```