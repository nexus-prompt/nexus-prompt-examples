# nexus-prompt-examples

Nexus Prompt（Chrome拡張）で作成したプロンプトのサンプル集と、
`nexus-prompt-cli` を用いたローカル/CIでの整形・Lint手順の例です。

## クイックスタート

1. エクスポート
   - Chrome拡張 Nexus Prompt から zip をエクスポートして、このリポジトリに展開します。

2. インストール
   - `devDependencies` に CLI を追加します。
   ```bash
   npm i -D nexus-prompt-cli
   ```

3. ローカルでの利用
   - 整形: `npx --no-install nexus-prompt fmt .`
   - 整形(確認のみ): `npx --no-install nexus-prompt fmt --check .`
   - Lint: `npx --no-install nexus-prompt lint .`

   任意で `package.json` にスクリプトを追加すると便利です:
   ```json
   {
     "scripts": {
       "fmt": "nexus-prompt fmt .",
       "fmt:check": "nexus-prompt fmt --check .",
       "lint": "nexus-prompt lint ."
     }
   }
   ```

## リポジトリ構成

- `app-prompts.json`: エクスポートしたプロンプトの一覧と順序を管理
- `*.md`: 各プロンプトの本文（エクスポートされたMarkdown）
- `.github/workflows/`: CI 設定（任意）

## CI 設定例（.github/workflows/prompt-ci.yaml）

### 1) fmt の差分が出たら CI を失敗させる

```yaml
name: Nexus Prompt CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  workflow_dispatch: {}

jobs:
  ci:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install
        run: npm ci

      - name: Format (check)
        run: npx --no-install nexus-prompt fmt --check .

      - name: Lint
        run: npx --no-install nexus-prompt lint .
```

### 2) 自動コミットする

```yaml
name: Nexus Prompt CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  workflow_dispatch: {}

jobs:
  ci:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install
        run: npm ci

      - name: Format
        run: npx --no-install nexus-prompt fmt .

      - name: Lint
        run: npx --no-install nexus-prompt lint .

      - name: Commit changes
        run: |
          if [ -n "$(git status --porcelain)" ]; then
            git config user.name "github-actions"
            git config user.email "github-actions@users.noreply.github.com"
            git add -A
            git commit -m "chore: format prompts"
            git push
          fi
```

## 動作要件

- Node.js 20 以上を推奨
- npm が利用可能であること

## 備考

- CLI の詳細は `npx --no-install nexus-prompt --help` を参照してください。
