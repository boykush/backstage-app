# catalog-remote-mcp

boykush の catalog——repo をまたいで作用する関係——を引くための remote MCP サーバーへの参照を1つ持つだけの plugin。[Agent Plugins](https://agent-plugins.org/specification) の形で書く。名前の `remote` は他の plugin に合わせたもので、手元で Backstage を立てる経路は用意していない。

サーバー名は `catalog`。tool は `mcp__catalog__*` として見える（`catalog.query-catalog-entities` / `catalog.get-catalog-entity` / `catalog.get-catalog-model-description` / `catalog.validate-entity`。名前に plugin の `catalog.` が付く）。読み取りだけで、entity を登録・削除する action は配信側で出していない。

## 配る中身

`mcp.json` の、`https://catalog-mcp.boykush.com/mcp` への参照だけ。実体はいくつかの repo に分かれている。

- **中身** は各 repo の root の `catalog-info.yaml` と、どの repo にも属さない entity を持つ [github-management](https://github.com/boykush/github-management) の `catalog/`。Backstage が GitHub Discovery で default branch から読むので、引けるのは **main に push 済みのもの**
- **MCP を出す仕組み** はこの repo の build（`@backstage/plugin-mcp-actions-backend`）
- **動く場所** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/backstage/`
- **外向きの hostname と path** は infrastructure-as-code の `applications/agentgateway/config.yaml`。URL が変わるならここが先に変わり、`mcp.json` が後を追う
- **認証** は infrastructure-as-code の `terraform/access.tf`。前段の Cloudflare Access が token を出し、agentgateway がそれを確かめる。初回だけ `/mcp` から GitHub でログインする

## 何のために引くか

repo の中だけを見て「この変更はここで閉じている」と判断しないため。全 repo に配られる設定、決定の適用範囲、MCP サーバーの提供元と利用先が entity と関係として入っているので、**着手前に「誰に届くか」を聞ける**。entity の書き方の決まりは github-management の AGENTS.md が持つ。

## 使う

apm で受け取るなら、[ai-plugins](https://github.com/boykush/ai-plugins) がこの plugin から作る package に依存する。apm は Agent Plugins の形を Claude と Codex へは展開しないので、その変換を ai-plugins が持つ。

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/catalog-remote-mcp
```

```bash
apm install
```

Claude は `.mcp.json`、Codex は `.codex/config.toml` に入る。どちらも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

## 引っかかり所

- クラスタは毎晩 01:37〜06:37 JST に落ちる。その間は Cloudflare が 530 を返す。catalog を見たいだけなら Backstage の UI（`https://backstage.boykush.com`、Cloudflare Access のワンタイム PIN）か、各 repo の `catalog-info.yaml` を直接読む
- `~/.claude/settings.json` の `enabledMcpjsonServers` に `catalog` は入っていないので、**Claude では初回に承認プロンプトが出る**。出したくなければ [dotfiles](https://github.com/boykush/dotfiles) 側に足す
