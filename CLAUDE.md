# roppo-viewer — 六法ビューア

最新の六法（憲法・民法・商法・刑法・民訴・刑訴）＋会社法を読みながら「全体のどこにいるか」（編→章→節→款→目→条）が常に見えるシングルページ Web アプリ。2026-08-09 作成。

## 構成

- **単体 HTML**: `index.html` のみ（ビルドなし・依存なし）
- **プレビュー**: launch.json `roppo-viewer`（port 8124, python3 http.server）
- **データ源**: e-Gov 法令 API v2（`https://laws.e-gov.go.jp/api/2`）から**ブラウザが直接取得**（CORS 許可済み `access-control-allow-origin: *`）。常に最新改正版が取れる
- 取得データは Cache API（`roppo-v1`）にキャッシュ。次回以降は即表示 + バックグラウンドで改正チェック（`/laws?law_id=` の `law_revision_id` 比較）→ 差分あれば自動再取得しトースト通知

## 機能

- 法令タブ 6 つ（`LAWS` 配列。追加はここに law_id を足すだけ）
- 左サイドバー: 編〜目の階層目次（条範囲付き・折りたたみ・現在位置ハイライト＆自動追従）
- パンくず: 現在読んでいる位置の 編›章›節›款›条（見出し）をスクロール連動で表示
- マップバー: 法令全体を編ごとの色分け帯で表示、赤マーカー=現在位置、クリックでジャンプ
- 検索窓: 数字（`709` / `第709条` / `398の22`）=条ジャンプ、文字列=全文検索（スニペット付き）
- 読書位置は法令ごとに localStorage 保存（`roppo.pos.*` = **スパイ番号**、ピクセルではない）
- URL ハッシュ `#minpo/709` で共有・ブックマーク可
- ライト/ダーク自動、`/` キーで検索フォーカス、モバイルはハンバーガーで目次

## 実装メモ（ハマりどころ）

- **rAF・smooth scroll・scrollIntoView は使わない**。フレーム駆動 API は非表示タブ/ヘッドレスで発火せず固まる。スクロール連動は scroll イベント直接 + 300ms ポーリング保険、ジャンプは `scrollTop` 直接代入、目次追従は手動スクロール計算
- **レイアウト再アンカー**: `revalidate()` が `scrollHeight` の変化（ウィンドウ幅・フォント変化等）を検知したら、直前に読んでいたスパイ番号へ `scrollTop` を再アンカーし目次も追従させる
- **法令切替の保存レース**: 切替時 `flushPos()` で旧法令の位置を同期保存。デバウンスタイマー発火時は `S.law.key === curLawKey` の時だけ保存
- e-Gov JSON は `{tag, attr, children}` ネスト。対応タグ: Part/Chapter/Section/Subsection/Division、Article(Caption/Title)、Paragraph(Caption/Num/Sentence)、Item/Subitem1/Subitem2、Column（全角空白結合）、Ruby/Rt、TableStruct/Table/TableRow/TableColumn、Preamble（憲法前文）、EnactStatement。削除条は `Num="38:84"` 形式の範囲
- 本則のみ表示（附則 SupplProvision は省略）

## 検証済み（2026-08-09）

6 法ロード、709/398の22/第236条ジャンプ、存在しない条のトースト、全文検索、憲法前文、刑訴の表 7 個、法令切替またぎの位置復元、リロード後復元、ダークモード。

## 運用

- **公開 URL**: https://natsunooda.github.io/roppo-viewer/ （GitHub Pages、main へ push すると自動反映）
- **ローカルで開く**: `開く.command` をダブルクリック（サーバー未起動なら port 8124 で起動してからデフォルトブラウザで開く）
- law_id 一覧: 憲法 321CONSTITUTION / 民法 129AC0000000089 / 商法 132AC0000000048 / 会社法 417AC0000000086 / 刑法 140AC0000000045 / 民訴 408AC0000000109 / 刑訴 323AC0000000131
- 法令追加は `index.html` の `LAWS` 配列に 1 行足すだけ（e-Gov の law_id を調べて key/tab/full を設定）
