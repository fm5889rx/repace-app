# RePace

リワーク（復職支援）で使われる「24時間の行動記録＋気分・疲労感の数値グラフ」の生活記録表をデジタル化する Web アプリ。スマホ・PC から選択式で入力でき、体調グラフの自動生成と、診察に提出できる A4 横向き PDF の出力までを一つで完結します。

- 詳細は [docs/要件定義書.md](docs/要件定義書.md) を参照
- バックエンド: PHP 8.5 / Laravel 13（リポジトリ直下）
- フロントエンド: Next.js 16 / TypeScript / Tailwind CSS（`frontend/`）
- インフラ: Docker（nginx / PHP-FPM / MySQL 8.0 / phpMyAdmin）

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

## ローカルでのスマホ検証（レスポンシブ）

デプロイ前に、開発機のローカル環境で、PC・Android・iOS の表示と操作を確認する手順です。

| 項目 | 内容 |
| --- | --- |
| 開発機 | iMac M4（Apple Silicon） |
| 検証するサーバー | Next.js（`http://localhost:3000`）。`/api` と `/sanctum` は Next.js が Laravel（8000 番）へ中継するため、スマホからは 3000 番だけにアクセスする |
| 事前の起動 | `docker compose up -d`（Laravel・DB）と、`cd frontend && npm run dev`（Next.js） |

> [!NOTE]
> 画面の幅が 768px（Tailwind の `md`）未満のときは、スマホ用のメイン画面（W-06）が表示されます。768px 以上（iPad など）では PC 用（W-03）です。スマホの確認には、iPhone や Pixel などの端末を選んでください。

### 1. Android エミュレータで検証する

1. Android Studio を開き、Device Manager（`Tools` ＞ `Device Manager`）から、エミュレータの端末（例: Pixel）を起動します。
2. エミュレータ内のブラウザ（Chrome）を開きます。
3. アドレスバーに `http://10.0.2.2:3000` を入力します。

> [!IMPORTANT]
> エミュレータ内の `localhost` は、開発機ではなく、エミュレータ自身を指します。開発機（Mac）には、Android の仕様で割り当てられた特別なアドレス `10.0.2.2` で接続します。

**初回だけ必要な設定**（`10.0.2.2` はこのアプリの許可リストにないため）

| 設定 | 内容 |
| --- | --- |
| `frontend/next.config.ts` | `allowedDevOrigins: ["10.0.2.2"]` を追加する（Next.js の開発サーバーは、`localhost` 以外からの開発用リソースへのアクセスを既定で拒否する） |
| `.env` | `SANCTUM_STATEFUL_DOMAINS` に `10.0.2.2:3000` を追加する（追加しないと、Cookie によるログインができない場合がある） |

設定したあとは、Next.js と Laravel（`docker compose restart`）を再起動してください。

### 2. iOS シミュレータで検証する

1. Mac の App Store から「Xcode」をインストールします（初回は、追加コンポーネントの導入を求められることがあります）。
2. Xcode のメニューから `Xcode` ＞ `Open Developer Tool` ＞ `Simulator` を選び、シミュレータを起動します。
3. 端末（例: iPhone）を選び、ホーム画面の Safari を開きます。
4. アドレスバーに `http://localhost:3000` を入力します。

iOS シミュレータは Mac のネットワークをそのまま使うため、`localhost` で開発機に接続できます（Android のような特別なアドレスは不要です）。

### 3. Mac の Safari で iOS シミュレータをデバッグする（Web インスペクタ）

シミュレータ内の JavaScript エラーや CSS を、Mac の Safari の開発者ツールで調べる方法です。

1. Mac の Safari を開き、「開発」メニューを表示します。
   - Safari の `設定`（古い版は `環境設定`）＞ `詳細` を開き、「Web 開発者向けの機能を表示」（古い版は「メニューバーに"開発"メニューを表示」）にチェックを入れます。
   - すでに「開発」メニューがメニューバーにある場合は、この手順は不要です。
2. シミュレータの Safari で、検証したいページ（`http://localhost:3000`）を開いたままにします。
3. Mac の Safari のメニューバーから、`開発` ＞ `Simulator — [端末名]` ＞ 検証したいページを選びます。
4. Web インスペクタ（開発者ツール）が開きます。要素の検査、スタイルの確認、コンソールのログやエラーの確認ができます。

> [!TIP]
> 「開発」メニューにシミュレータが出てこないときは、シミュレータの Safari で、ページを開いていることを確認してください。それでも出ない場合は、シミュレータの `設定` ＞ `Safari` ＞ `詳細` で「Web インスペクタ」がオンになっているかを確認します。

### シミュレータで確認できないこと

エミュレータ・シミュレータでは、画面の幅と基本的な操作は確認できますが、次の点は実機で確認します（[仕様チェックリスト](docs/仕様チェックリスト.md) の Q-50）。

- 指によるスワイプやドラムロールの操作感
- iPhone の Safari での印刷ダイアログと PDF の保存
