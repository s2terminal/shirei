# shirei

他のエージェントセッション(Claude Code / Codex / OpenCode など)に指示を送るための司令(shirei)リポジトリ。

## clone後のインストール手順

use [mise](https://mise.jdx.dev/)

```sh
$ mise install
$ npx skills experimental_install
```

Herdr環境内(環境変数 `HERDR_ENV=1` が設定された状態)でClaude Codeを起動する。

### TODOリストの作成

```sh
$ mkdir shirei-backlog
$ path_to_todo="path/to/todo"
$ mv shirei-backlog $path_to_todo/
$ ln -s $path_to_todo/shirei-backlog shirei-backlog
```

```sh
$ mkdir shirei-backlog/todo
$ mkdir shirei-backlog/doing
$ mkdir shirei-backlog/done
```

### Claude Code のローカル設定

`shirei-backlog` はシンボリックリンクであり、リンク先の絶対パスは環境ごとに異なる。
リンク先を作業ディレクトリ外に置いた場合、Claude Code は `shirei-backlog` 配下のファイル読み書きやサンドボックスの書き込みで確認プロンプトを出す。

共有設定の `.claude/settings.json` には、環境に依存しない許可を既に含めている。
環境依存の絶対パスは共有設定に書けないため、各自 `.claude/settings.local.json` に記述する。
`<リンク先の絶対パス>` は `readlink -f shirei-backlog` で確認できる。

```json
{
  "permissions": {
    "allow": [
      "Read(//<リンク先の絶対パス>/**)",
      "Edit(//<リンク先の絶対パス>/**)",
      "Write(//<リンク先の絶対パス>/**)"
    ],
    "additionalDirectories": [
      "/<リンク先の絶対パス>"
    ]
  },
  "sandbox": {
    "filesystem": {
      "allowWrite": [
        "/<リンク先の絶対パス>"
      ]
    }
  }
}
```

`Read` `Edit` `Write` の許可ルールでは、絶対パスを表すために先頭を `//` にする。
`additionalDirectories` と `sandbox.filesystem.allowWrite` では、通常の絶対パス表記（先頭 `/`）にする。

### Google Antigravity の設定

Antigravity はリポジトリ内の権限設定ファイルを読まない。
権限はユーザー単位の設定（CLIは `~/.gemini/antigravity-cli/settings.json`、IDEは Settings > General > Permission Settings、プロジェクト単位の上書きは Settings > Projects）に記述する。

`.claude/settings.json` と同等の許可ルールを `antigravity/settings.json` に置いている。
CLIの場合は、次のコマンドで既存の設定にマージする。

```sh
$ mkdir -p ~/.gemini/antigravity-cli
$ f=~/.gemini/antigravity-cli/settings.json
$ [ -f "$f" ] || echo '{}' > "$f"
$ jq -s '.[0] * {permissions: {allow: ((.[0].permissions.allow // []) + .[1].permissions.allow | unique)}}' "$f" antigravity/settings.json > "$f.tmp" && mv "$f.tmp" "$f"
```

IDEの場合は、`antigravity/settings.json` の `allow` の各行を Permission Settings の Allow List に追加する。

Claude Code の設定との対応は次のとおり。

- `Bash(...)` は `command(...)` に対応する。`command(...)` は先頭トークンの前方一致で、ワイルドカードを使う場合は `command(regex:...)` にする（トークンごとに完全一致で評価される）。
- `WebFetch(domain:...)` は `read_url(...)` に対応する。
- `sandbox.excludedCommands` に相当する設定はない。`command(...)` で許可したコマンドは、サンドボックスの内外を問わず確認なしで実行される。
- `defaultMode: acceptEdits` は、Antigravity の Default プリセット（ワークスペース内のファイル編集を許可）に相当する。
- `Skill(...)` に相当する権限ルールはない。スキルは `npx skills experimental_install` で導入される `.agents/skills/` から読み込まれる。

`shirei-backlog` のリンク先を作業ディレクトリ外に置いた場合は、ユーザー設定の `allow` に次を追加する。
`write_file` で許可したパスは読み取りも許可され、サンドボックスにも読み書き可能でマウントされる。

```json
"write_file(/<リンク先の絶対パス>)"
```

## スキルの編集後

```sh
$ npx skills add ./skills-src/shirei-dispatch
```
