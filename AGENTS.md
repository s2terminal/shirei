# shirei

このリポジトリは、他のエージェントセッションに指示を配送・中継するための司令塔リポジトリです。

- 他のエージェントセッション（Claude Code / Codex / OpenCode など）に指示を送って結果を受け取りたい場合は `shirei-dispatch` スキルを使ってください。
- ほかのエージェントはHerdr上で動いています。Herdrそのものの一般的な操作（pane/workspace/tabの作成やレイアウト操作など）は `herdr` スキルを参照してください。

## TODOリスト
`shirei-backlog/ ディレクトリにTODOリストがあります。1ファイル1タスクの形式で格納されています。

```
$ tree -L 1 shirei-backlog
shirei-backlog
├── doing
├── done
└── todo
```

todo状態のタスクを実行して、doing状態に移動させてください。
実行した結果は、todoファイルに追記する形でまとめてください。
許可なくdone状態にしないでください。
