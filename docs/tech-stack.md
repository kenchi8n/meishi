# Tech Stack

個人ポートフォリオサイト（Astro + Cloudflare Pages）の技術仕様。

## フレームワーク

| ツール                | バージョン | 用途                                           |
| --------------------- | ---------- | ---------------------------------------------- |
| Astro                 | 5.x        | メインフレームワーク（SSG + 必要箇所のみ SSR） |
| `@astrojs/cloudflare` | latest     | Cloudflare Pages 向け adapter                  |

UI island は使用しない。インタラクションは Astro コンポーネントと素の JS で完結させる。

## スタイリング

| ツール          | 用途                         |
| --------------- | ---------------------------- |
| Tailwind CSS v4 | ユーティリティファースト CSS |

## コンテンツ管理

| ツール                    | 用途                                                  |
| ------------------------- | ----------------------------------------------------- |
| Astro Content Collections | ブログ記事を `.md` / `.mdx` で管理                    |
| MDX                       | コンポーネントを埋め込める拡張 Markdown               |
| Satori (`@vercel/og`)     | ブログ記事ごとの OG 画像を Cloudflare Edge で動的生成 |

## Linter / Formatter

| ツール                      | 対象ファイル         | 用途                                               |
| --------------------------- | -------------------- | -------------------------------------------------- |
| oxlint                      | `.ts`, `.tsx`, `.js` | Lint（OXC プロジェクト製の高速 Rust ベース Linter）|
| oxfmt                       | `.ts`, `.tsx`, `.js` | Format（OXC プロジェクト製 Formatter）             |
| `astro check`               | `.astro`             | 型チェック（oxlint/oxfmt が `.astro` 未対応のため補完） |
| TypeScript (`strict: true`) | 全 TS ファイル       | 型安全性                                           |
| husky + lint-staged         | コミット時           | oxlint・oxfmt・astro check をコミット前に自動実行  |

## CI / デプロイ

```
GitHub → GitHub Actions → Cloudflare Pages（wrangler deploy）
```

- `main` ブランチへの push → **production** デプロイ
- Pull Request → **preview** デプロイ（PR に URL がコメントされる）
- Cloudflare Pages の GitHub 直接連携は使わず、GitHub Actions + `wrangler-action` で制御する

### GitHub Actions ワークフロー概要

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run build
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: pages deploy dist/ --project-name=me
```

## ディレクトリ構成（想定）

```
me/
├── .github/workflows/
│   └── deploy.yml
├── docs/                   # 設計ドキュメント
├── src/
│   ├── components/
│   ├── content/
│   │   └── blog/           # .md / .mdx ファイル
│   ├── layouts/
│   └── pages/
│       ├── blog/
│       │   └── [slug].astro
│       └── index.astro
├── astro.config.mjs
├── biome.json
├── tailwind.config.mjs     # v4 では最小限の設定
└── wrangler.toml
```

## 依存パッケージ一覧

### dependencies

```json
{
  "astro": "^5.x",
  "@astrojs/cloudflare": "latest",
  "@astrojs/mdx": "latest",
  "@astrojs/tailwind": "latest",
  "tailwindcss": "^4.x"
}
```

### devDependencies

```json
{
  "oxlint": "latest",
  "oxfmt": "latest",
  "husky": "latest",
  "lint-staged": "latest",
  "wrangler": "latest"
}
```
