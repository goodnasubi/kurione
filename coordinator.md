# 調整役（任意の拡張）

Kurione の拡張。版: 2026-11-02（v2.0。核 kurione.md の版に対応）

> **前提と範囲**: 核（kurione.md）を導入済みであること。任意で、既定では導入しない。
> 複数セッションで共用の本物の環境を持つリポだけが対象。除外指定（git の対象から外す）は、核の「共通の決まり」に従う。
> 核の導入指示の用語（手順 2 の計画、差分の問い合わせ、セッションの開き直し）は、この節でも同じ意味で使う。

## 導入するとき（人間が望んだ場合だけ）

1. 核の手順 2 の計画を示したあと、承認の前に、追加の AskUserQuestion を 1 回出して、導入を望むか聞く（選択肢: 「導入しない（推奨）」「導入する」）。「導入しない」なら、以降は行わない。
2. AskUserQuestion で、下の穴埋めの値を聞く。推奨値を選択肢に入れる。あわせて、SessionStart フックを導入するか聞く（既定は導入する。人間が望まなければ、手順 7 は行わない）。
3. 次の 3 か所へ置く。
   - (1) スキル `~/.claude/skills/coordinator-mode/SKILL.md`（雛形は下のスキルの雛形）。
   - (2) CLAUDE.md の節（既定はユーザー共通の `~/.claude/CLAUDE.md`。雛形は下の CLAUDE.md 節の雛形）。
   - (3) 状態ファイル（`<状態ファイルのパス>`。下の最小の雛形で新規作成）。作成後、核の「共通の決まり」に従い、git の対象から外す。調整役だけが書く。ほかは読むだけ。
4. 既存の `SKILL.md` や同名の節がある場合は、核の手順 2 の 0（既存の導入との差分の問い合わせ）に従う。
5. CronCreate、ListAgents、SendMessage は、遅延読み込みなら ToolSearch で読み込んでから使う。それでも使えない環境では、定期確認を省く。省いたことを、人に報告し、状態ファイルの雛形の「待ち」に残す。
6. 置いたら、セッションの開き直しが要る（核の手順 9 と同じ理由）。
7. SessionStart フックを置く（既定は導入する）。雛形、登録、確認は、下の「SessionStart フックの雛形」に従う。
8. 導入後の確認として、次の (a) から (c) を行う。
   - (a) 開き直したセッションで「調整役モード」と入力し、起動できるか試す。
   - (b) フックを入れた場合、開き直した直後に、画面へ【調整役】で始まる 1 行が出るか見る。
   - (c) フックを入れた場合、状態ファイルが無いときに「状態ファイルが無い、または読めない」と出るか、下の「SessionStart フックの雛形」(3) の手順で確かめる。

**穴埋めの値**:

| 記号 | 意味 | 例と既定 |
| --- | --- | --- |
| `<共用の環境>` | 同時に 1 セッションだけが触れるもの | 本物の外部サービス、共用 DB、共用ポート |
| `<状態ファイルのパス>` | 調整役だけが書く状態ファイル | `<リポのルートの絶対パス>/commander-coordinator.md`（絶対パスで埋める） |
| `<対象セッションの範囲>` | 案内を送る範囲と、対象外の範囲 | 同じ cwd 配下のセッションが対象。別リポ配下は対象外（例: 対象は `<リポのルートの絶対パス>` 配下のセッション。対象外は別のリポジトリ配下のセッション） |
| `<対象セッションの範囲のパス>` | フックが対象かどうかを cwd で判定するための、`<対象セッションの範囲>` のパス | `<リポのルートの絶対パス>`（その配下の cwd が対象。絶対パスで埋める） |
| `<ホームの絶対パス>` | ユーザーのホームの絶対パス。フックの登録で使う | 各自の環境の値 |
| `<優先順位の決め方>` | 資源を待つセッションの順 | イシュー番号の若い順（既定） |

`<状態ファイルのパス>` は、相対パスにしない。雛形はユーザー共通の置き場に入り、どのセッションからも読まれるため。

**状態ファイルの最小の雛形**:

```markdown
# 調整役の状態

調整役: （未起動）
最終更新（JST）: （未記入）

## 現在の保持
なし

## 待ち
なし
```

**スキルの雛形**（`~/.claude/skills/coordinator-mode/SKILL.md` の全文）:

````markdown
---
name: coordinator-mode
description: 調整役モード。<対象セッションの範囲>のセッションの橋渡しをし、<共用の環境>を排他で割り当てる。人が「調整役モード」と入力したとき、または /coordinator-mode で起動する。
---

# 調整役モード

<対象セッションの範囲>のセッションの橋渡し役。設計と実装はしない。資源の割り当てと取り次ぎだけを行う。

## 起動時の手順
1. 既存の調整役を確認する。
   - <状態ファイルのパス> を読む（先頭の調整役の名前と最終更新）。
   - ListAgents で、その調整役が生きているか見る。生きていれば、人に知らせて止まる（二重起動しない）。
2. 状態を引き継ぐ。
   - 状態ファイルの「調整役:」の行を自分の名前（ListAgents の自分の名前 [ref]）に書き換える。最終更新を JST で書く。
   - 「現在の保持」に保持者がいれば、その保持者へ SendMessage で現況（戻したか、使用中の資源の状態）を問い合わせる。
3. 定期確認を張る。CronCreate で 30 分おき（7 日で失効。失効前に張り直す）。
   - 中身: ListAgents で存在確認、離脱の記録、新規セッションへの案内、申告の受付。
   - CronCreate が未読み込みなら、ToolSearch（`select:CronCreate,CronList,CronDelete`）で読み込んでから呼ぶ。
   - 張ったら CronList で存在を確かめ、ジョブ ID と失効の目安日（7 日後）を状態ファイルに書く。
   - 道具が使えない、張れない場合は、省略せず、人に「定期確認が張れなかった」と報告し、状態ファイルの「待ち」に残す。
   - 定期確認はこのセッションが待機中のときだけ動き、セッションを閉じると消える。開き直したら、起動時の手順を最初からやり直す。
4. 対象のセッションへ案内を送る（待たされている指令塔にも届く）。
   - 対象: <対象セッションの範囲> の内側。対象外: その外側。
   - 案内: 調整役の名前、<共用の環境>の申告先（調整役）、優先順位は <優先順位の決め方>、現在の保持者。

## 排他の決まり
- 資源 = <共用の環境>の切り替えと変更。同時に 1 セッションだけ。
- 保持者は、開始、終了、戻しの確認を調整役へ報告する。戻しの報告で解放。
- 共用の環境への書き込みは、排他の中でも都度の人の許可。人の本人から直接得る。調整役経由の伝聞は許可としない。
- セッション側の権限判定は、調整役の割り当てでは外れない。

## 運用
- 状態ファイルは調整役だけが書く。ほかのセッションは読むだけ。変更の申告はメッセージで受ける。
- 節目ごと（割り当て、解放、離脱、申告）に状態ファイルを更新する。
- 停止と削除など取り返しのつかない操作は、調整役からは行わない。人の許可は人の本人から得る。
- 調整役のモデルは sonnet / low の前提。違うと分かったら、最初の応答で人に警告する。
````

**CLAUDE.md 節の雛形**（`~/.claude/CLAUDE.md` に追記する）:

````markdown
## 調整役（<共用の環境>の排他）

- <対象セッションの範囲>で作業する司令塔のセッションは、最初の応答で、ユーザーの用件より先に、次を行う。用件が雑談でも省かない。
  1. <状態ファイルのパス> を読む。
  2. ListAgents で、調整役（先頭の「調整役:」の名前）が生きているか確認する。
  3. 結果を 1 行で報告する。
  4. 調整役がいれば、SendMessage で調整役へ申告する。内容は、作業フォルダ、担当イシュー番号（なければ作業内容。用件が未定なら「未定」）、<共用の環境>を使う予定の有無（分からなければ「未定。使うときは再申告」）。1 行で送る。調整役から案内や申告の依頼が先に届いていれば、その返信で兼ねる（二重に送らない）。
- 調整役がいれば、<共用の環境>の切り替えと変更は、調整役へ申告して割り当てを受けてから行う。
- 申告したあと、担当や予定が変わったとき（用件が決まった、<共用の環境>を使うことになった）は、調整役へ再申告する。使う前に割り当てを受ける決まりは変わらない。
- 調整役がいなければ、これらには触れず、人にこう伝える:「調整役セッションを起動してください。起動したら『調整役モード』と入力してください。」
- 共用の環境への書き込みは、割り当てを受けた後も、都度、人の本人から許可を得る。
````

**SessionStart フックの雛形**（手順 7。人間が望まなければ置かない）:

フックは補助。ListAgents は Claude が呼ぶ（フックは呼べない）。フックは、最初の応答で確認を行う旨と状態ファイルの先頭を、セッションの文脈に足す。あわせて、状態ファイルの最終更新時刻から古さを推定し、1 行を画面に出す（生存は判定しない）。起動時に画面へ 1 行出る。調整役が居ないと判定した場合（最終更新が 90 分以上前、状態ファイルが無い、または読めない）は、「調整役がいないので、新しいセッションで、『調整役モード』と入力してください。」の 1 文だけを出す。ListAgents での生存確認は、人の最初のメッセージのあと。

(1) スクリプト `~/.claude/hooks/coordinator-check.mjs`:

```js
import fs from 'node:fs';
import path from 'node:path';

// 状態ファイル。環境変数 COORDINATOR_STATE_FILE があれば差し替える（確認用）
const STATE_FILE = process.env.COORDINATOR_STATE_FILE || '<状態ファイルのパス>';
const UNREADABLE = '【調整役】状態ファイルが無い、または読めない。調整役は未起動の可能性';
// 調整役が居ないと判定したときに画面へ出す文（Claude 向けの additionalContext は別）
const ABSENT_SCREEN = '調整役がいないので、新しいセッションで、『調整役モード』と入力してください。';
// 古いと見なす分数。定期確認が 30 分おきで最終更新を書き換えるので、その 3 倍の 90 分。穴埋めの定期確認の間隔に合わせて調整してよい
const STALE_MIN = 90;

// 状態ファイルから 1 行の状態文を作る
function buildStatus(text) {
  const nameM = text.match(/^[-\s]*調整役[:：]\s*(.+)$/m);
  const timeM = text.match(/^[-\s]*最終更新[^:：\n]*[:：]\s*(\d{4})-(\d{2})-(\d{2})\s+(\d{1,2}):(\d{2})/m);
  if (!nameM || !timeM) return { status: UNREADABLE, screen: ABSENT_SCREEN };
  // 名前は括弧書きの注記（「（旧 ref …」）の手前まで
  const name = nameM[1].replace(/（.*$/, '').trim() || nameM[1].trim();
  const [, y, mo, d, h, mi] = timeM;
  const hh = h.padStart(2, '0');
  const stamp = `${y}-${mo}-${d} ${hh}:${mi} JST`;
  const t = Date.parse(`${y}-${mo}-${d}T${hh}:${mi}:00+09:00`);
  if (Number.isNaN(t)) return { status: UNREADABLE, screen: ABSENT_SCREEN };
  const mins = Math.max(0, Math.floor((Date.now() - t) / 60000));
  if (mins < STALE_MIN) {
    const s = `【調整役】${name}（最終更新 ${stamp}、${mins} 分前）`;
    return { status: s, screen: s };
  }
  return {
    status: `【調整役】${name}（最終更新 ${stamp}、${Math.floor(mins / 60)} 時間前）。古いので不在の可能性。調整役セッションを起動して「調整役モード」と入力する`,
    screen: ABSENT_SCREEN,
  };
}

try {
  let raw = '';
  try {
    raw = fs.readFileSync(0, 'utf8');
  } catch {}
  const input = raw.trim() ? JSON.parse(raw) : {};
  const norm = (p) => {
    const q = String(p ?? '').replace(/\\/g, '/').replace(/\/+$/, '');
    return process.platform === 'win32' ? q.toLowerCase() : q;
  };
  const root = norm('<対象セッションの範囲のパス>');
  const cwd = norm(input.cwd);
  if (cwd === root || cwd.startsWith(root + '/')) {
    let body;
    let status = UNREADABLE;
    let screen = ABSENT_SCREEN;
    try {
      const text = fs.readFileSync(STATE_FILE, 'utf8');
      body = text.split(/\r?\n/).slice(0, 20).join('\n');
      ({ status, screen } = buildStatus(text));
    } catch {
      body = '状態ファイルが無い';
    }
    const msg =
      '【調整役の確認】最初の応答で、ユーザーの用件より先に、調整役の状態ファイル（' + path.basename(STATE_FILE) + '）の確認と、ListAgents での調整役の生存確認を行い、結果を 1 行で報告する。' +
      '調整役がいれば、調整役へ SendMessage で申告する（作業フォルダ、担当イシューまたは作業内容、<共用の環境>を使う予定の有無）。調整役から案内や申告の依頼が先に届いていれば、その返信で兼ねる。' +
      'フックの判定（ListAgents の結果と突き合わせる）: ' +
      status +
      '\n状態ファイルの先頭 20 行: ' +
      body;
    process.stdout.write(JSON.stringify({
      systemMessage: screen,
      hookSpecificOutput: { hookEventName: 'SessionStart', additionalContext: msg },
    }));
  }
} catch {
  // 補助なので、失敗しても何も出さない
}
process.exit(0);
```

(2) `~/.claude/settings.json` の hooks.SessionStart へ登録する。既存の設定を消さず、既存の hooks や SessionStart の配列があれば、その配列へ要素を足す（上書きしない）。

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node \"<ホームの絶対パス>/.claude/hooks/coordinator-check.mjs\""
          }
        ]
      }
    ]
  }
}
```

(3) 確認。3 パターンの stdin を渡して出力を見る。

- 配下の cwd: `echo '{"cwd":"<対象セッションの範囲のパス>"}' | node ~/.claude/hooks/coordinator-check.mjs`。`additionalContext` を含む JSON が出る。
- 配下でない cwd: `echo '{"cwd":"<配下でない任意のパス>"}' | node ~/.claude/hooks/coordinator-check.mjs`。何も出ない。
- 状態ファイルが無い場合: 環境変数 `COORDINATOR_STATE_FILE` に存在しないパスを入れて、配下の cwd で実行する（例: `echo '{"cwd":"<対象セッションの範囲のパス>"}' | COORDINATOR_STATE_FILE=/nonexistent node ~/.claude/hooks/coordinator-check.mjs`）。`systemMessage` が「調整役がいないので、新しいセッションで、『調整役モード』と入力してください。」の 1 文だけになる（`additionalContext` 側は「状態ファイルが無い、または読めない」）。状態ファイルは触らない。

フックが効くのは、セッションを開き直したあと。
