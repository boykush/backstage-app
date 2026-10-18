# backstage-app

boykush の catalog を出す Backstage の app。`@backstage/create-app` が作った雛形を build して `ghcr.io/boykush/backstage-app` に置き、infrastructure-as-code がそれをデプロイする。catalog の entity はこの repo ではなく boykush/github-management の `catalog/` にある。

## create-app との差分

公式 image（`ghcr.io/backstage/backstage`）は create-app の出力そのままで、plugin を足せない。この repo が雛形から変えているのは次だけで、それ以外は create-app のまま保つ。

- `packages/backend/src/index.ts` に `@backstage/plugin-catalog-backend-module-ai-model` を足し、AiResource kind を使えるようにしている。
- `package.json` の `resolutions` で `@yarnpkg/core` を `4.9.1` に固定している。無いと `yarn install` が `got-npm-11.8.2-*.patch` の ENOENT で落ちる（[backstage/backstage#35964](https://github.com/backstage/backstage/issues/35964)）。upstream が直したら外す。
- 雛形の `catalog-info.yaml` は置かない。

## 開発

```sh
mise install
yarn install
yarn start
```

yarn 本体は `.yarn/releases/` に同梱されている（`.yarnrc.yml` の `yarnPath`）。`yarn` を PATH に置くのは、`mise.toml` が有効にする corepack。

## 更新

依存は `yarn backstage-cli versions:bump` で上げる。create-app の雛形の変化は自動では入らないので、[upgrade-helper](https://backstage.github.io/upgrade-helper/) で版の間の差分を見て取り込む。
