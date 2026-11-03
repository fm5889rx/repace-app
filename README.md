# RePace

リワーク（復職支援）で使われる「24時間の行動記録＋気分・疲労感の数値グラフ」の生活記録表をデジタル化する Web アプリ。スマホ・PC から選択式で入力でき、体調グラフの自動生成と、診察に提出できる A4 横向き PDF の出力までを一つで完結します。

- 詳細は [docs/要件定義書.md](docs/要件定義書.md) を参照
- バックエンド: PHP 8.5 / Laravel 13（リポジトリ直下）
- フロントエンド: Next.js 16 / TypeScript / Tailwind CSS（`frontend/`）
- インフラ: Docker（nginx / PHP-FPM / MySQL 8.4 / phpMyAdmin）

## 開発環境

```bash
docker compose up -d --build
docker compose exec php php artisan key:generate
docker compose exec php php artisan migrate
```

| サービス | URL / ポート |
| --- | --- |
| Laravel (nginx) | http://localhost:8000 |
| phpMyAdmin | http://localhost:8080 |
| MySQL（ホストから接続） | localhost:3306 |
| Next.js | http://localhost:3000 |

```bash
cd frontend
npm install
npm run dev
```

ポートは `.env` の `NGINX_PORT` / `PHPMYADMIN_PORT` / `FORWARD_DB_PORT` で変更できます。
