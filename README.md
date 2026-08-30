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

## スキルの編集後

```sh
$ npx skills add ./skills-src/shirei-dispatch
```
