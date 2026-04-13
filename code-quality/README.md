# コード品質が高いとはどういう状態か
- コードの水準が一定以上に保たれている
- 複雑なロジックやHackが放置されず、使いやすいAPIとして整理されている。
- 機能の追加や変更がスムーズに行える



コード品質を高めることで、バグが減少し、結果的に開発スピードが大幅にアップする。

# 品質を保つためには
- 思想
  - DRY原則
  - 型ファースト
- ツール
  - Linter
  - JSDoc
  - Git Hooks


# DRY原則

DRYとは「Don't Repeat Yourself」の略である。同じロジックや値を複数箇所に書くのを避け、1か所を変更するだけでシステム全体の仕様変更に対応できるようにする原則だ。「コピペ禁止」と覚えるとよいと思う。

> [!WARNING]
> DRY原則による過剰な抽象化は時間を浪費する場合もある。適切な粒度が大切である。

## 悪い例
以下の例では、税率計算のロジックが2か所に散らばっている。変更作業は大変だし、完了を証明するのが大変である。

```ts
// 注文確認画面
const price1 = 1000;
const total1 = price1 * 1.1;
console.log(`合計金額は ${total1} 円です`);

// 領収書発行画面
const price2 = 5000;
const total2 = price2 * 1.1; 
console.log(`領収金額: ${total2} 円`);
```

## 良い例
税率と計算ロジックを1か所にまとめた。これで、税率が変わったときは `TAX_RATE` を変更するだけで済む。

```ts
const TAX_RATE = 1.1;

function calculateTotal(price: number): number {
  return price * TAX_RATE;
}

// 注文確認画面
console.log(`合計金額は ${calculateTotal(1000)} 円です`);

// 領収書発行画面
console.log(`領収金額: ${calculateTotal(5000)} 円`);
```

# 型ファースト

データ構造を基準に開発を進めるアプローチである。型定義もDRY原則と同様に「コピペ」を避け、一元管理することが重要である。正しく型を定義することで、エディタが早い段階で間違いを教えてくれるようになる。

型ファーストを実現するツールとして、ネイティブのTypeScriptの機能を使う方法や、`Zod`、`Valibot`、`TypeBox` などのライブラリを活用する方法がある。

## 悪い例
以下の例では、似たような型を関数ごとに何度も定義している。
もし、後から `User` 型に新しい項目（例：電話番号）を追加した場合、`CreateUserRequest` や `UpdateUserRequest` にも手動で追加しなければならず、更新漏れによるバグの原因になる。

```ts
type User = {
  id: string;
  name: string;
  email: string;
  age: number;
};

type CreateUserRequest = {
  name: string;
  email: string;
  age: number;
};

type UpdateUserRequest = {
  name?: string;
  email?: string;
  age?: number;
};

// ユーザーを作成する関数
function createUser(data: CreateUserRequest) {
  // ...
}

// ユーザーの情報を編集する関数
function editUser(data: UpdateUserRequest) {
  // ...
}
```

## 良い例
TypeScriptの標準機能（`Omit` や `Partial`）を使って、元の `User` 型から別の型を生成する。
これにより、大元の `User` 型を変更するだけで、派生するすべての型に自動的に変更が反映される。

```ts
type User = {
  id: string;
  name: string;
  email: string;
  age: number;
};

// User型から 'id' だけを除外した型を作成
type CreateUserRequest = Omit<User, 'id'>;

// CreateUserRequestのすべてのプロパティを任意（省略可能）にした型を作成
type UpdateUserRequest = Partial<CreateUserRequest>;

// ユーザーを作成する関数
function createUser(data: CreateUserRequest) {
  // ...
}

// ユーザーの情報を編集する関数
function editUser(data: UpdateUserRequest) {
  // ...
}
```

# Linter

Linterは、コードを解析し、コーディング規約に違反している箇所を指摘してくれるツールである。

## 書き方の統一 (Consistency)
人によって変数の宣言や関数の書き方が違うと、コードが読みにくくなる。

```ts
// 悪い例：命名規則（キャメルケースとスネークケース）、引用符、セミコロンの有無がバラバラ
const user_name = "Alice"   // スネークケース、ダブルクオート、セミコロンなし
const userAge = 25;         // キャメルケース、セミコロンあり
const is_active = true      // スネークケース
const role = 'admin';       // シングルクオート

// 良い例（Linter/Formatterで統一後）
const userName = "Alice";
const userAge = 25;
const isActive = true;
const role = "admin";
```

## アクセシビリティ (Accessibility)

### 悪い例
画像に代替テキストがなく、ボタンではない要素にクリックイベントをつけているため、スクリーンリーダーやキーボード操作に対応できない。
```ts
const Header = () => (
  <div>
    <img src="logo.png" /> 
    <div onClick={handleHome}>ホームへ戻る</div>
  </div>
);
```

### 良い例
正しいHTMLタグと属性を使用することで、正しい実装になる。
```ts
const Header = () => (
  <header>
    <img src="logo.png" alt="会社ロゴ" />
    <button type="button" onClick={handleHome}>
      ホームへ戻る
    </button>
  </header>
);
```

## 複雑度 (Complexity)
ネストが深すぎるコードは、条件が複雑で読むのが困難になる。

### 悪い例
`if` 文が何重にもなっており、処理を追うのが大変である。
```ts
function getStatusMessage(user) {
  if (user.isLogged) {
    if (user.hasSubscription) {
      if (user.isTrial) {
        return "無料トライアル中です";
      } else {
        return "有料会員です";
      }
    } else {
      return "未購読です";
    }
  } else {
    return "ログインしてください";
  }
}
```

### 良い例
Early Returnでコードがすっきりと読みやすくなる。
```ts
function getStatusMessage(user) {
  if (!user.isLogged) return "ログインしてください";
  if (!user.hasSubscription) return "未購読です";
  
  return user.isTrial ? "無料トライアル中です" : "有料会員です";
}
```

# JSDoc

関数や型に対してドキュメントを書く習慣をつけるべきである。エディタ上で関数にマウスを合わせるだけで説明が表示されるようになる。Exampleまで書ければ理想的だが、まずは「その関数がどんな意図で作られ、どんな機能を持っているか」が伝わるように書くことが大切だ。

## 関数に対するDocs
```ts
/
 * 指定されたユーザーに通知を送信します。
 * * @param userId - 送信先のユーザーID（UUID形式）
 * @param message - 送信する本文。140文字以内である必要があります
 * @returns 送信に成功した場合はtrue、失敗または制限中の場合はfalseを返します
 * * @example
 * ```ts
 * sendNotification("550e8400-e29b...", "こんにちは！");
 * ```
 */
function sendNotification(userId: string, message: string): boolean {
  // ...
}
```

## 型に対するDocs
```ts
/
 * APIレスポンスの基本構造
 */
type ApiResponse = {
  / ステータスコード（200なら成功） */
  status: number;
  
  / * クライアントへ表示するメッセージ。
   * エラー時はここに関連するエラー内容が格納されます。
   */
  message: string;

  / @deprecated 代わりに `updatedAt` を使用してください */
  timestamp: number;
};
```

## TypeScriptでのJSDoc
JavaScriptでは引数の型情報などをJSDocに記載する必要があるが、TypeScriptでは型定義からエディタが自動的に推論してくれるため、型に関する記述は省略して簡潔に書くことができる。

```ts
/
 * @param id ユーザーのID（型情報はTSが知っているため不要）
 * @param age ユーザーの年齢（型情報はTSが知っているため不要）
 */
function update(id: string, age: number) { 
  // ... 
}
```

# Git Hooks と Linter (Biome) の連携

Linterや型定義を整備しても、ルール違反のコードがGitのリポジトリに保存されてしまっては意味がない。コードをコミットするタイミングで自動的にチェックを走らせ、違反があればブロックする仕組みを構築する。ここでは、Biome と、Git Hooksを管理する Lefthook を用いた導入手順を解説する。

## 1. インストール
まずは必要なパッケージをインストールし、初期化を行う。

```bash
# Biome と Lefthook のインストール
bun add -D @biomejs/biome lefthook

# Biome の初期設定（biome.json が生成される）
bunx @biomejs/biome init

# Lefthook の初期設定（lefthook.yml が生成される）
bunx lefthook install
```

## 2. コマンドの登録

```json
// package.json の scripts に追加
{
  "scripts": {
    "ci": "biome check ."
  }
}
```
これにより、`bun run ci` を実行するだけで、プロジェクト全体のフォーマットとLintチェックが走るようになる。

## 3. Biomeのルール設定（Linterの章で触れた内容の適用）
自動生成された `biome.json` を編集し、先述した「アクセシビリティ」「複雑度」「書き方の統一」を検知できるようにする。

```json
// biome.json
{
  "$schema": "https://biomejs.dev/schemas/1.8.3/schema.json",
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "lineWidth": 80
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "a11y": {
        // 画像のalt属性を必須にする（アクセシビリティの悪い例を検知）
        "useAltText": "error",
        // divなどにonClickをつける場合、キーボード操作も必須にする
        "useKeyWithClickEvents": "error"
      },
      "complexity": {
        // ネストの深い不要なelseを禁止し、早期リターンを促す（複雑度の悪い例を検知）
        "noUselessElse": "error",
        // 複雑すぎるロジックを警告する
        "noExcessiveCognitiveComplexity": "error"
      },
      "style": {
        // varの禁止やconstの推奨など、書き方をモダンに統一する
        "noVar": "error"
      }
    }
  }
}
```

## 4. Lefthook でコミット時に自動チェック
最後に `lefthook.yml` を編集し、コミットする直前に `bun run ci` が自動的に実行されるように設定する。

```yaml
# lefthook.yml
pre-commit:
  commands:
    lint-and-format:
      # ステージングされたファイルだけでなく、全体をチェックする
      run: bun run ci
```
