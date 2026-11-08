# catalog-remote-mcp

この repo の Backstage が出す catalog——boykush の repo と、repo をまたぐ関係——を引くための remote MCP サーバーへの参照と、その使い方の skill を持つ plugin。[Agent Plugins](https://agent-plugins.org/specification) の形で書き、apm の package にして配るのは [ai-plugins](https://github.com/boykush/ai-plugins)。名前の `remote` は ai-plugins の他の plugin に合わせたもので、手元で Backstage を立てる経路は用意していない。

置き場が `plugins/` でないのは、そこがこの repo の yarn workspace（Backstage の plugin）だから。

サーバー名は `catalog`。tool は `mcp__catalog__*` として見え、名前に Backstage の plugin の `catalog.` が付く。読み取りだけで、entity を登録・削除する action は配信側で出していない。

## 配る中身

- `mcp.json`: `catalog` サーバーへの参照
- skill `catalog-review`。PR の変更が catalog の上でほかの repo に及ぶかを確かめ、対になる PR と catalog とのずれを報告する手順。boykush/workflows の ai-review が、判定に含めない報告として使う

サーバーの実体は次のように分かれている。

- **中身** は各 repo の root の `catalog-info.yaml` と、[github-management](https://github.com/boykush/github-management) の `catalog/`。Backstage が GitHub Discovery で default branch から読むので、引けるのは **main に push 済みのもの**
- **動く場所** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/backstage/`。この repo が build した image を、read-only の action に絞った MCP として出す
- **外向きの hostname と path** は infrastructure-as-code の `applications/agentgateway/config.yaml`。URL が変わるならここが先に変わり、`mcp.json` が後を追う
- **認証** は infrastructure-as-code の `terraform/access.tf`。前段の Cloudflare Access が token を出し、agentgateway がそれを確かめる。初回だけ `/mcp` から GitHub でログインする。CI（ai-review）は別の hostname から run の OIDC token で入り、その許可も agentgateway の設定が持つ

## 何のために引くか

repo の中だけを見て「この変更はここで閉じている」と判断しないため。全 repo に配られる設定、決定の適用範囲、MCP サーバーの提供元と利用先が entity と関係として入っているので、**着手前に「誰に届くか」を聞ける**。

## 引っかかり所

- クラスタは毎晩 01:37〜06:37 JST に落ちる。その間は Cloudflare が 530 を返す。catalog を見たいだけなら Backstage の UI（`https://backstage.boykush.com`、Cloudflare Access のワンタイム PIN）か、各 repo の `catalog-info.yaml` を直接読む
- `~/.claude/settings.json` の `enabledMcpjsonServers` に `catalog` は入っていないので、**Claude では初回に承認プロンプトが出る**。出したくなければ [dotfiles](https://github.com/boykush/dotfiles) 側に足す
