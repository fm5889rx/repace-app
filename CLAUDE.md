# RePace

リワーク（復職支援）用の生活記録表をデジタル化する Web アプリ。要件は [docs/要件定義書.md](docs/要件定義書.md) を参照。

## 言語・コミュニケーション

- 開発者とのやり取り、コードコメント、コミットメッセージ、ドキュメントはすべて **日本語** で書く。
- 変数名・関数名・クラス名・DB のテーブル/カラム名は英語（Laravel / Next.js の慣習に従う）。
- ユーザー向けの文言（画面、バリデーションメッセージ、エラー）は日本語。
  - Laravel: `APP_LOCALE=ja` / `APP_FALLBACK_LOCALE=ja` / `APP_FAKER_LOCALE=ja_JP`、タイムゾーンは `Asia/Tokyo`。
  - 言語ファイルは `lang/ja/`（`laravel-lang/common` で導入）。フィールド名の表示は `lang/ja/validation.php` の `attributes` に追加する。
  - Next.js: `<html lang="ja">`。

## 構成

- バックエンド: PHP 8.5 / Laravel 13（リポジトリ直下。本番も PHP 8.5 に固定）
- フロントエンド: Next.js 16 / TypeScript / Tailwind CSS（`frontend/`）
- DB: 開発は MySQL 8.0、本番は TiDB Cloud Starter（テストは SQLite メモリ DB）

## 開発環境（Docker）

PHP / Composer / Artisan はホストに入れず、コンテナ内で実行する。

```bash
docker compose up -d --build
docker compose exec php php artisan migrate
docker compose exec php php artisan test
docker compose exec php composer <command>
```

| サービス | URL / ポート |
| --- | --- |
| Laravel | http://localhost:8000 |
| phpMyAdmin | http://localhost:8080 |
| MySQL | localhost:3306 |
| Next.js | http://localhost:3000（`cd frontend && npm run dev`） |

## 作業ルール

- **フォルダ名はすべて英語**にする（例: `docs/design/`）。ファイル名は日本語でもよい（`docs/要件定義書.md`、`docs/設計書.md` は課題で指定された名前）。
- コミット・プッシュは依頼があったときだけ行う。
  - 例外: `docs/仕様チェックリスト.md` の更新（とそれに伴うドキュメントの修正）は、更新のたびにコミットして、`main` へ直接 push する。
- 生成したコードは開発者が口頭で説明できるよう、意図を簡潔に説明する。
- 生成したテストメソッド名は test_ のプレフィックス以外は日本語にすることで実装すること。
- 実装コードの先頭にクラス全体の機能を、各メソッドの説明も個別にコメントで入れること。
- 行コメントは最小限で良い。2行以上にわたる場合は適切に改行すること。
- 本実装コードとテストコードの関係：テストコードに合わせて本実装コードを無理やり書き換えないように注意。
- 本実装コードを修正した場合は、必ずブラウザ等で実際の動きを確認してから、テストコードを書き換えること。
- テストコードは正常系よりも異常系を重視して実装すること。

## リクエストの流れとレイヤーマップ

本プロジェクトは Next.js（画面）+ Laravel（JSON API）の構成。Laravel 側は画面（Blade）を持たない。
1 つの HTTP リクエストは次の順に流れる。**新しいコードをどの層に置くべきかは、適切に判断**すること。

```
routes/api.php            … 入口。URL・ミドルウェア・Controller の対応（/api/v1）
→ Middleware              … 認証（Sanctum）・ログイン試行制限などの状態チェック（app/Http/Middleware/）
→ Controller              … 薄い受付役。1 メソッド = 1 Action 呼び出しが基本形（app/Http/Controllers/）
→ FormRequest             … バリデーション + 認可（app/Http/Requests/{Entity}/）
→ Action                  … 1 業務操作 = 1 クラス（app/UseCases/{Entity}/）
→ Service / Model         … 横断ロジック / Eloquent（app/Services/, app/Models/）
→ JsonResponse            … JSON で返す（エラーも JSON。設計書のエラー定義に従う）
```

### app/ ディレクトリの責務

| ディレクトリ | 責務 |
|---|---|
| `Http/Controllers/` | リクエスト受付とレスポンス。ビジネスロジックは持たず Action へ委譲 |
| `Http/Middleware/` | 認証・試行制限など |
| `Http/Requests/{Entity}/` | FormRequest。`rules()` のバリデーションと `authorize()` の認可 |
| `Http/Resources/` | レスポンスの JSON 整形（API Resource） |
| `UseCases/{Entity}/` | **1 業務操作 = 1 Action クラス**（例: `StoreAction` / `UpdateAction` / `ConfirmWeekAction`） |
| `Services/` | 複数 Entity にまたがる集計・判定ロジック（例: 睡眠時間の計算） |
| `Models/` | Eloquent モデル。リレーション・スコープ・キャストを集約 |
| `Policies/` | 認可ポリシー。「誰がこのリソースに何をできるか」 |
| `Enums/` | 状態・種別の PHP Enum |
| `Exceptions/` | ドメイン例外 |
| `Providers/` | サービスプロバイダ |

## 成果物の透明性と構成管理

- **開発ステータスの明示義務**: 提示するすべてのソースコードの上部または解説において、該当ファイルが `【新規作成】` されたものか、既存のファイルを `【変更】` されたものかを、ディレクトリパスと共に必ず明示すること。
- **残骸ファイルの物理パージ**: リファクタリング方針の切り替えや仕様変更によって使用しなくなった Action、Service、Provider などの残骸ファイルは、コンテナの目詰まり（`ReflectionException`）やオートロードマップの破壊を防ぐため、その場で確実に物理削除すること。

## アーキテクチャとデータレイヤー

- **生クエリ（Raw Query）の禁止**: データベース層へのアクセスは、`Eloquent ORM`、名前付きスコープ、または Query Builder の正規メソッドを介して行う。生文字列を直接注入する SQL の直書き（`DB::raw`、`whereRaw`、`DB::select` など）は、セキュリティ脆弱性と保守性低下を招くため禁止。`DB::transaction()` は生クエリではないので使ってよい（例: 日をまたぐ保存）。
- **Fat Controller の禁止**: コントローラー内へのビジネスロジック、外部 API SDK の直接呼び出し、複雑なデータ整形を書かない。コントローラーは Action を呼んでレスポンスを返すだけの薄い層にする。バリデーションと認可は FormRequest（必要に応じて Policy）で行い、すべてのユースケースは単一責務の Action クラスへ委譲する。

## 型安全とルーティング

- **厳格な型宣言**: すべてのメソッド、コンストラクタ引数、戻り値（`JsonResponse`、`Response` など）に型宣言を付け、ファイル先頭に `declare(strict_types=1);` を付ける。
- **Route Model Binding の名前の一致**: `routes/api.php` のワイルドカードのパラメータ名（例: `{day}`、`{id}`）と、Controller・Action 側の引数名は、大文字・小文字も含めて完全に一致させる。
  - 解決がねじれるクラス型を扱う場合は、プレーンな `string $id` で受け取り、内部で `findOrFail($id)` で取得する。
- **Enum キャスト**: DB のカラム（`status` など）が Enum クラスでキャストされている場合は、保存・更新時に生の文字列（`->value`）ではなく `Enum オブジェクトそのもの` を代入する。

## 外部サービス連携・モックテスト

- **実通信の防止**: テストの `setUp()` で必ず `Http::preventStrayRequests();` を実行する。モックしていないリクエストが発生した時点で例外を投げてテストを失敗させ、外部サービスへの実通信や API キーの漏洩を防ぐ。
- **グループによる分離**: 外部サービスや実通信に依存するテストには、PHPUnit 12 の属性 `#[Group('external')]` を付ける（PHPUnit 12 は PHPDoc の `@group` を読まない）。通常のテストは `--exclude-group external` で除外して実行する。`phpunit.xml` に `external` グループの設定が必要になったときに追加する（現時点では外部サービスに依存するテストはない）。

## 例外ハンドリング・認可・セキュリティ

- **UNIQUE 制約の競合（レースコンディション）**: UNIQUE 制約（例: `(user_id, log_date)`）の衝突で 500 エラーにしない。`Illuminate\Database\QueryException` は Action 内でキャッチしてドメイン例外に変換し、設計書のエラー定義どおりの JSON（409 または 422）で返す。Controller にキャッチ処理を書かない。
- **他人のデータへのアクセス**: 他人のデータは、取得・更新・削除のいずれも `404 Not Found` を返す（存在の有無を漏らさない。設計書 1.6 の「個人データ」・API 共通仕様）。
  - `403 Forbidden` は、ログイン済みで登録・ログインを呼んだ場合などに限る。
  - 確定済みの週への変更は `409 Conflict`。
- **ファイルの保管**: 提出用 PDF はブラウザの印刷機能で作るため、サーバーには保存しない。今後サーバーにファイルを保存する場合は、`public` ディスクを使わず、`private` ディスクに保存する（`Storage::disk('private')->put`）。URL の直打ちで他人に見られないようにするため。
