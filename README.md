<p align="center">
  <img src=".github/assets/weqLogo.png" alt="Web Engineer Quiz" width="240">
</p>

# web-engineer-quiz-infra

[![Deploy to Lightsail](https://github.com/idw-coder/web-engineer-quiz-infra/actions/workflows/deploy.yml/badge.svg)](https://github.com/idw-coder/web-engineer-quiz-infra/actions/workflows/deploy.yml)
[![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/idw-coder/web-engineer-quiz-infra)

![Docker Compose](https://img.shields.io/badge/Docker%20Compose-2496ED?logo=docker&logoColor=white)
![MySQL 8.4](https://img.shields.io/badge/MySQL-8.4-4479A1?logo=mysql&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?logo=nextdotjs&logoColor=white)
![AWS Lightsail](https://img.shields.io/badge/AWS%20Lightsail-FF9900?logo=amazonwebservices&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?logo=githubactions&logoColor=white)

「ウェブエンジニア問題集」の本番サーバー構成（Docker Compose）を置くリポジトリ。

アプリ本体（Next.js）は [idw-coder/next-basic](https://github.com/idw-coder/next-basic) にある。このリポジトリには、アプリを動かすための compose ファイル・DB の初期データ・デプロイ用ワークフローだけを置く。

旧名は `express-mysql-docker`。もとは Express の API を置いていたが、API は Next.js の Route Handler に移り、Express は本番の compose から外した（2026-08-12）。

## 本番の構成

Lightsail の `/var/www/app` にこのリポジトリを clone して、`docker-compose.prod.yml` で次の2つのコンテナを動かしている。

| サービス | イメージ | 内容 |
| --- | --- | --- |
| `mysql` | `mysql:8.4.0` | DB。データは `mysql_data` ボリュームに保存 |
| `nextjs` | `ghcr.io/idw-coder/next-basic:latest` | アプリ本体。ポート 3000 |

## デプロイの流れ

デプロイは2本のワークフローで行う。

- **アプリを更新したとき**：next-basic に push すると、そちらのワークフローがイメージをビルドして GHCR に push する。そのあとサーバーで `nextjs` コンテナだけを入れ替える
- **このリポジトリを更新したとき**：main に push すると `.github/workflows/deploy.yml` が動き、サーバーで `git pull` と `docker-compose -f docker-compose.prod.yml up -d` を実行する

## ファイル

```
.
├── docker-compose.yml        # ローカル開発用（mysql と mailserver）
├── docker-compose.prod.yml   # 本番用（mysql と nextjs）
├── .github/workflows/
│   └── deploy.yml            # 本番サーバーへのデプロイ
└── db/
    ├── QUIZ_TAGS_SCHEMA.sql  # クイズのタグのテーブル定義
    └── seed-*.sql            # クイズの初期データ
```

`backend/`（旧 Express）と `next-basic/` は `.gitignore` に入っていて、このリポジトリでは管理していない。

## 環境変数

`.env` はどれも gitignore 済み。値はサーバー上にだけ置く。

- `db/.env`：MySQL コンテナが読む（`MYSQL_ROOT_PASSWORD` など）
- `nextjs/.env`：`nextjs` コンテナが読む。DB の認証情報・認証まわりの秘密鍵・Stripe のキーを置く

`DB_HOST` / `DB_PORT` / `DB_NAME` / `DB_CONNECTION_LIMIT` は `docker-compose.prod.yml` の `environment:` に書いてあるので、`nextjs/.env` には入れない。`NEXT_PUBLIC_*` はビルド時にイメージへ入るので、ここには不要。

## ローカル開発

```bash
docker compose up
```

MySQL が 3306 番、SMTP コンテナ（`boky/postfix`）が 1025 番で立ち上がる。Next.js は next-basic 側で `npm run dev` を実行する。
