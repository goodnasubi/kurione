# Kurione

Kurione: a commander workflow for Claude Code (orchestrator–subagent pattern)

## 名前の意味

Kurione はクリオネ。クリオネは透明で、中身が透けて見える。判断・依頼・状態メモ・報告が見える運用を目指す。

## 何をするか

- メインのセッションを「司令塔」とし、設計・実装・レビューなどをサブエージェントに任せる。
- 司令塔は、依頼文・判断・状態メモ（commander-state.md）・報告を残し、何が起きたかを後から追えるようにする。
- 人間の判断が必須のことだけ人に確認し、それ以外はマージと片付けまで進める。
- 導入先の Claude Code に、エージェント定義・スキル・CLAUDE.md を置く手順を手順書にまとめた。

## 手順書

- [GitLab 版](./commander-workflow-gitlab.md): MR の管理に GitLab（glab）を使う場合
- [GitHub 版](./commander-workflow-github.md): MR（PR）の管理に GitHub を使う場合

## 版

v1.20
