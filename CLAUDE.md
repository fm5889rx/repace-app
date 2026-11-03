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
- DB: MySQL 8.4（テストは SQLite メモリ DB）

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
- 生成したコードは開発者が口頭で説明できるよう、意図を簡潔に説明する。
