# Kurione

<p align="center"><img src="assets/kurione-baton.png" width="240" alt="Kurione のマスコット（指揮棒を持つクリオネ）"></p>

Kurione: a commander workflow for Claude Code (orchestrator–subagent pattern)

## 名前の意味

Kurione はクリオネ。クリオネは透明で、中身が透けて見える。判断・依頼・状態メモ・報告が見える運用を目指す。

## マスコット

<p align="center">
<img src="assets/kurione-simple.png" width="200" alt="Kurione のマスコット（シンプル版）">
<img src="assets/kurione-baton.png" width="200" alt="Kurione のマスコット（指揮棒を持つクリオネ）">
<img src="assets/kurione-family.png" width="200" alt="Kurione のマスコット（家族）">
</p>

画像は AI（OpenAI の画像生成）で作成。

## 何をするか

- メインのセッションを「司令塔」とし、設計、実装、レビューなどをサブエージェントに任せる。
- 司令塔は、依頼文、判断、状態メモ（commander-state.md）、報告を残し、何が起きたかを後から追えるようにする。
- 人間の判断が必須のことだけ人に確認し、それ以外はマージと片付けまで進める。
- 導入先の Claude Code に、エージェント定義、スキル、CLAUDE.md を置く手順を手順書にまとめた。

## 手順書

前提: Claude Code（サブエージェントが使える環境）。私の運用が元。合わない所は直して使う。

- [核 kurione.md](./kurione.md): 司令塔方式の規則と導入の共通部分
- [付録 GitLab](./appendix-gitlab.md): MR の管理に GitLab（glab）を使う場合
- [付録 GitHub](./appendix-github.md): PR の管理に GitHub（gh）を使う場合
- [coordinator.md](./coordinator.md): 任意。調整役（複数セッションで共用資源を排他する拡張）

使い方: kurione.md と、使うホストの付録を Claude Code に読ませて、「導入して」と頼む。

## ライセンス

MIT License。詳細は [LICENSE](./LICENSE)。

画像（assets/）は、生成サービスの利用条件に従う。

## 版

v2.0（2026-11-02）
