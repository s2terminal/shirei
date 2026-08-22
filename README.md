# shirei

他のエージェントセッション(Claude Code / Codex / OpenCode など)に指示を送るための司令(shirei)リポジトリ。

## clone後のインストール手順

use [mise](https://mise.jdx.dev/)

```sh
$ mise install
$ npx skills experimental_install
```

Herdr環境内(環境変数 `HERDR_ENV=1` が設定された状態)でClaude Codeを起動する。

## 編集後

```sh
$ npx skills add ./skills-src/shirei-dispatch
```
