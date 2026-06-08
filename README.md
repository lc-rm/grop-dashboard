# GROP（本社）採用分析ダッシュボード

Indeed応募〜書類通過〜内定の選考ファネルを分析する社内ダッシュボード。
シードグループ版と同型（GitHub Pages ＋ Firebaseログイン ＋ 単一HTML ＋ Chart.js）。

- 公開URL（予定）: https://lc-rm.github.io/grop-dashboard/
- ログイン: @link-core.co.jp の Google アカウント限定（Firebaseはシードと同じプロジェクトを流用）
- タブ: 概要 / **書類通過（主役）** / 媒体比較 / 職種別 / オフィス別 / 年代別 / 広告コスト

## データの流れ

```
GROPスプレッドシート（人事管理用 等）
  └─ Apps Script（AppsScript.gs）が 5分ごとに「非PII集計のみ」を JSON 化
       └─ GitHub（lc-rm/grop-dashboard）の data.json に自動 push
            └─ index.html が data.json を fetch して描画
```

★ 氏名・電話・メール・正確な年齢は **一切出力しない**（年齢は「○代」の帯のみ）。GitHub Pages は誰でも data.json を取得できるため。

## ファイル

| ファイル | 役割 |
|---|---|
| `index.html` | ダッシュボード本体（単一HTML） |
| `data.json` | 集計済み非PIIデータ（Apps Scriptが自動更新 / ローカルは convert.py で生成） |
| `convert.py` | ローカルのExcel → data.json 変換（開発・検証用） |
| `AppsScript.gs` | 本番のデータ自動更新（Googleスプレッドシート → GitHub） |

## ローカル開発

```bash
# Excelから data.json を再生成（任意のxlsxを引数指定可）
python3 convert.py ~/Downloads/★GROP様\(本社\)：indeed.xlsx data.json
# プレビュー（port8797）
python3 -m http.server 8797
# ログインゲートはブラウザのコンソールで擬似通過:  hideGate();_loaded=true;await load()
```

## 本番セットアップ（残作業）

### 1. GitHubリポジトリ
1. `lc-rm/grop-dashboard` を **Public** で作成
2. `index.html` / `data.json` / `README.md` をアップロード
3. Settings > Pages > Source = `main` / `(root)` で有効化

### 2. Apps Script 自動更新（会社アカウント r_murai で）
1. GitHubで fine-grained トークン作成（Contents: Read and write / 対象 `lc-rm/grop-dashboard`）
2. script.google.com に `AppsScript.gs` を貼り付け
3. プロジェクト設定 > スクリプトプロパティ に `GITHUB_TOKEN` = 作ったトークン を登録
4. 関数 `setupTrigger` を1回実行 → 5分ごとの自動 push が有効に

> Apps Scriptは GROPスプレッドシートを開ける会社アカウントで運用すること。
> `index.html` の更新は GitHub Web UI で再アップロード（自動pushは data.json のみ触る）。

## データ元シート（`★GROP様(本社)：indeed.xlsx` / スプレッドシート）

- **人事管理用**: 応募者レベル生データ（選考ステータス付き、氏名はPIIのため集計時に破棄）
- **書類通過**: Indeed月別ファネル（①Indeed全体と②⭐️AirWORKの2表が縦並び → 最初の表のみ採用）
- **媒体比較**: Indeed/エン/DODA/マイナビの媒体別サマリー
- **各数字データ**: 月別ファネル＋利用金額（月額固定¥400k期のみコスト記録あり）
