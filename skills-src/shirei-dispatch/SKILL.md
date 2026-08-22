---
name: shirei-dispatch
description: Herdr上の別のエージェントセッションに指示を送り、結果を安全に回収する。「別セッションに指示を出して」「他のエージェントに聞いて」「〜に中継して」等で使用。HERDR_ENV=1が必要。
---

# shirei-dispatch

このスキルは、このリポジトリ(shirei)固有の「他エージェントへの安全な配送手順」を扱う。Herdrの一般的な操作概念(pane/workspace/tab、`herdr agent` コマンド体系など)は、先に `herdr` スキルを読むこと。

## 他エージェントへの指示配送手順

1. `herdr agent list` で対象を探す。ターゲット指定には `pane_id` かユニークなagent名だけを使う。`agent` フィールド(kind、例: `codex`)をそのまま `agent prompt` 等のターゲットに渡すと `agent_not_found` になる(実際に確認済み)。
2. `herdr agent prompt <pane_id> "<指示>" --wait --timeout <N>` で送信する。この呼び出し自体が `timeout` エラーを返しても、指示は届いて実行が始まっていることがある(実際に発生した)。`timeout` エラーを「失敗」と即断しない。
3. 必要なら `herdr agent wait <pane_id> --until idle --timeout <M>` で追加で待つ。この `agent wait` 呼び出しも `timeout` を返すことがある(実際に発生した)。
4. 完了判定は常に `herdr agent get <pane_id>` が返す `agent_status`(`idle` または `done`)を正とする。`agent prompt` や `agent wait` のタイムアウトエラーではなく、この `agent_status` の値だけを信頼する。
5. `herdr agent read <pane_id> --source recent-unwrapped --lines 300` で出力を回収し、ターミナルの生ログからユーザーへの回答部分だけを抜粋して提示する。
6. ファイル共有フォールバックは使わない。指示先エージェントは別コンテナで動いている可能性があるため、`herdr` スキルに記載されている「一時ディレクトリにMarkdownを書かせて読み返す」手法はこのリポジトリでは使わない。出力が途切れている場合は次のいずれかで対応する。
    - `--lines` を増やして再読する
    - 対象エージェントに「続きを表示して」と再度プロンプトを送る
7. Claude Codeのサンドボックス下では、`herdr` コマンド呼び出しにサンドボックス無効化(`dangerouslyDisableSandbox`)が必要になることがある。これはHerdrデーモンとUnix socket経由で通信するためで、`Operation not permitted` が出たらこのオプションを付けて再試行する。

## 安全なherdrコマンド一覧(正本)

<!-- shirei-dispatch:safe-herdr-commands:start -->
以下は読み取り専用コマンドで、自動許可している。このリストを変更する場合は `.claude/settings.json` 等も同時に更新すること。

- `herdr status`
- `herdr agent list`
- `herdr agent get <target>`
- `herdr agent read <target> ...`
- `herdr agent explain <target> ...`
- `herdr agent wait <target> ...`
- `herdr pane list [--workspace ...]`
- `herdr pane current [...]`
- `herdr pane read <pane_id> ...`
- `herdr workspace list`
- `herdr tab list [...]`
<!-- shirei-dispatch:safe-herdr-commands:end -->

上記以外のコマンドは、状態を変更するかフォーカス/画面を動かすものであり、自動許可しない。必ず通常の確認フローを経ること。該当例:

- `agent prompt` / `agent send-keys` / `agent start` / `agent focus` / `agent attach`
- `pane split` / `pane run` / `pane send-text` / `pane send-keys` / `pane close` / `pane move` / `pane focus` / `pane resize` / `pane zoom`
- `workspace create` / `workspace close`
- `tab create` / `tab close`
