# backstage-app

boykush の catalog を出す Backstage の app。`@backstage/create-app` が作った雛形を build して `ghcr.io/boykush/backstage-app` に置き、infrastructure-as-code がそれをデプロイする。catalog の entity はこの repo ではなく boykush/github-management の `catalog/` にある。

## create-app との差分

公式 image（`ghcr.io/backstage/backstage`）は create-app の出力そのままで、plugin を足せない。この repo が雛形から変えているのは次だけで、それ以外は create-app のまま保つ。

- `packages/backend/src/index.ts` に `@backstage/plugin-catalog-backend-module-ai-model` を足し、AiResource kind を使えるようにしている。
- `packages/backend/src/index.ts` に `@backstage/plugin-catalog-backend-module-github` を足し、各 repo の `catalog-info.yaml` を GitHub Discovery で取り込めるようにしている。
- `package.json` の `resolutions` で `@yarnpkg/core` を `4.9.1` に固定している。無いと `yarn install` が `got-npm-11.8.2-*.patch` の ENOENT で落ちる（[backstage/backstage#35964](https://github.com/backstage/backstage/issues/35964)）。upstream が直したら外す。
- `packages/backend/Dockerfile` に Python と MkDocs（`techdocs-requirements.txt`）を足している。TechDocs を backend の中で build するため（cluster の Pod には Docker が無い）。
- `packages/app/package.json` に `backstage-plugin-techdocs-addon-mermaid` を足し、TechDocs の mermaid の図をブラウザで描くようにしている。`app.packages: all` が依存から見つけるので、`App.tsx` には書かない。
- 雛形の `catalog-info.yaml`（create-app が作る仮のもの）は置かない。root の `catalog-info.yaml` は、この repo を boykush の catalog に載せる本物で、github-management から移したもの。

## 開発

```sh
mise install
yarn install
yarn start
```

yarn 本体は `.yarn/releases/` に同梱されている（`.yarnrc.yml` の `yarnPath`）。`yarn` を PATH に置くのは、`mise.toml` が有効にする corepack。

## 更新

依存は `yarn backstage-cli versions:bump` で上げる。create-app の雛形の変化は自動では入らないので、[upgrade-helper](https://backstage.github.io/upgrade-helper/) で版の間の差分を見て取り込む。
