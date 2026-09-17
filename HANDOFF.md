# クラスコ社内ポータル — 引き継ぎドキュメント

このファイルを最初に読めば、これまでの経緯と決定事項が分かります。
（Claude Codeで作業を始めるときは「HANDOFF.md を読んで続きをやりたい」と伝えてください）

---

## 1. 概要
- Googleサイト（`sites.google.com/crasco.jp/portal`）に **HTMLを埋め込んで**構築した社員専用ポータル
- 目的: 各種業務ツール・情報への「**入口（ハブ）**」。あちこち見に行かずここからまとめてアクセスできる

## 2. アクセス / 公開範囲
- **Googleサイト**: 公開範囲を「クラスコ」グループに限定（一般公開なし・Googleログイン必須）
  - 将来グループ会社ごとにページを分ける可能性あり → アクセスは会社ごとのGoogleグループで指定する方針
- **GASウェブアプリ**: 埋め込みiframeのサードパーティCookie制限を回避するため「**全員（匿名）**」で公開
  - そのため機密情報は出さない設計にしている（来客は「来客あり」に伏せ、誕生日は非表示）

## 3. ファイル構成（このリポジトリ）
- `index.html` … ポータル本体（現行の最終版。※`crasco-portal-embed.html` をリネーム）
- `calendar.html` … 全社カレンダー（**保留中**。JSONP化が残課題）
- `gas/topics.gs` … 注目トピック用GAS（`clasp clone` で取得推奨）
- `gas/calendar.gs` … カレンダー用GAS（同上）
- `HANDOFF.md` … このファイル

## 4. デザイン方針
- 素のHTML/CSS ＋ **インラインSVGアイコン**、フォント = `Zen Kaku Gothic New`
- 配色: primary = インディゴ `#4f46e5` / surface `#f6f7f9` / text `#1f2937`（派生は color-mix）
- body背景は透明、`.portal` をグレーの角丸パネルにしている（Sites埋め込みの余白対策）
- Tailwind / Font Awesome は不使用（自己完結・表示安定のため）

## 5. 業務ツール（リンク集）
- **AI BOT**（強調カード）: https://sites.google.com/crasco.jp/crasco-lm … 各部門の業務Botの入口
- 勤怠管理: https://s3.kingtime.jp/independent/recorder/personal/ … KING OF TIME
- 稟議・申請: https://sites.google.com/crasco.jp/workflow/ホーム … ワークフロー
- 社内QA: https://notebooklm.google.com/notebook/cb6c5052-6351-44ab-8333-6c207ed34d6c … NotebookLM
- Google Chat / Googleカレンダー
- 書庫（共有ドライブ）: https://drive.google.com/drive/folders/1_34bBvlW6BqQwuhGnS8pungLtL5lDuaw
- 社内マニュアル・規程: **準備中**（実ファイルURL未設定）
- 各ボタンには「何を開くか」の説明を1行添える方針（役員フィードバック）

## 6. 注目トピック（実データ稼働中）
- 仕組み: **スプレッドシート → GAS(JSONP) → ポータルが最大3件表示**
- GAS(/exec): https://script.google.com/macros/s/AKfycbxo-PThnNg5PjQayANcfmyaxIVSSQ8S9IzOhvJeGtp7YQCJV5bpjOmktCLOBm7Ot2r3Cw/exec
- シート列: `会社 / id / 掲載 / カテゴリ / 発信元 / タイトル / 本文 / 投稿日 / リンクURL / 画像URL`
- **会社キーは半角小文字「crasco」で統一**（不一致だと表示されない。過去に「クラスコ」で詰まった）
- 掲載=TRUE を、投稿日の新しい順に3件表示。0件時は「注目トピックはありません。」
- カテゴリ: 会長のお言葉 / 有益情報 / お知らせ / 注意（色分け）
- **連携はJSONP必須**（`fetch`はSites埋め込みでCORSに阻まれるため使えない）
- カードは画像あり=サムネ、なし=テキストのみ。本文は3行クランプ、リンクありは「詳しく見る」、なしは長文で「続きを読む」

## 7. 全社カレンダー（保留中）
- 3カレンダーをGASで合算:
  - 社休・行事: `c_8558f637315d30e754bb72b52bdfc2aa6af6a2f7f7bd804bf2666592d7fa9325@group.calendar.google.com`（通常表示）
  - 本社来客予定: `c_79401efe9b32280dd3657597e21adc6b9f8297536625d1d9a8f7bc26d256f091@group.calendar.google.com`（タイトルを「来客あり」に伏せる）
  - 従業員誕生日: `c_acb28fa32b7cd720c4cff8c4ee852d99ef46a7daf7b021845692a3b675d2ddf3@group.calendar.google.com`（**非表示**。配列から外すだけ）
- GAS単体では `/exec` で正しいJSONを確認済み・国内(JST)利用・件数上限なし（7日以内は全件）
- **残課題**: 埋め込みiframe内の`fetch`がリダイレクト+CORSで失敗 → **topics.gsと同じJSONP(callback)方式に変えれば解決見込み**

## 8. 既知の運用課題 → 移行の理由
- Sitesの「コード埋め込み」は更新のたびに手貼りが必要で保守が重い
- 対策: HTMLを**自ホスト（GitHub Pages / S3）**に置き、Sitesは**URL埋め込み**に変更 → `push`で即反映
- GASは**clasp**でgit管理＋コマンドで push / deploy

## 9. 次のタスク（優先順）
1. リポジトリ化 → GitHub Pages公開 → SitesをURL埋め込みに切替
2. GASをclasp管理化（topics.gs / calendar.gs を clone）
3. カレンダーのJSONP化 → 埋め込み再開
4. （任意）ツールのリンクもスプレッドシート化して「コードを触らない運用」へ
5. 社内マニュアルの実ファイル差し替え
6. 総務へ運用引き継ぎ（トピック投稿手順のマニュアル作成）→ 社内広報 → 運用開始
