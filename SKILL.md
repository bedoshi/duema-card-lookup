---
name: duema-card-lookup
description: Look up official Duel Masters (デュエル・マスターズ / デュエマ) card names and ability text from Takara Tomy's official card database (dm.takaratomy.co.jp). Use this skill whenever the user mentions "デュエル・マスターズ" or "デュエマ", asks about a specific card's effect/text, wants a decklist or combo explained, or provides a source (auto-generated video subtitles, chat logs, hand-written notes, etc.) that references Duel Masters card names — even if those names look garbled, phonetic, or only approximately right. Auto-generated Japanese captions frequently mangle Duel Masters card names (homophones, kanji substitution, dropped words), so always verify the name via search before trusting it, and never present a card's effect from memory or from the source text as-is — always confirm against the official database first.
---

# デュエル・マスターズ カード検索スキル

## なぜこのスキルが必要か

デュエル・マスターズのカード名は独特の読みや長いカタカナ語が多く、自動生成字幕(YouTube auto-caption)や聞き取りメモでは頻繁に誤変換される。例:

- 「プライマル・サーガ」と聞こえても実在するのは「プライマル・スクリーム」と「プライマル・サーガ」の両方で、効果が全く違う
- 「サガ」「サーガ」のような語尾の揺れ
- 「エムロマ」「エロマ」のような略称・愛称は正式カード名ではない
- 同名に見えても「龍神ヘヴィ」「極限龍神ヘヴィ」のように上位互換カードが別途存在する

これらを字幕や記憶だけで整理すると、存在しない効果を捏造してしまう(ハルシネーション)リスクが高い。必ず一次情報(公式サイト)に当たること。

## 手順

### 1. カード名の特定(Google検索)

ユーザーや参照ファイル(字幕・メモ等)から得たカード名は「聞き取り候補」として扱い、まずWebSearch(またはGoogle検索)で正式名称を特定する。

- クエリ例: `"<聞き取ったカード名>" デュエルマスターズ カード 効果`
- 聞き取りに自信がない場合は、周辺情報(文明・コスト・種族・「進化」「S・トリガー」等のキーワード、動画タイトル、デッキ名)も検索クエリに加えて絞り込む
- 候補が複数見つかった場合(同名で複数弾に収録されている、または似た名前の別カードがある)は、文脈(コスト・効果・デッキの色)から最も一致するものを選ぶか、ユーザーに確認する
- 検索結果から `dm.takaratomy.co.jp/card/detail/?id=xxxxx-xxx` の形式のURL、またはカードIDの手がかり(セット名+番号、例: DMC58 3/16)を掴む

### 2. 公式データベースからテキスト取得

カードIDやURLが分かったら、`mcp__workspace__web_fetch` で直接そのページを取得する。

```
https://dm.takaratomy.co.jp/card/detail/?id=<カードID>
```

カードIDが分からない場合は `site:dm.takaratomy.co.jp <カード名>` でWebSearchし、該当ページのURLを見つけてから取得する。

取得したページの `meta-description` に完全なカードテキストが入っているので、これを一次ソースとして扱う(本文中のテーブルにも同じ情報が重複して載っている)。特に以下を確認・記録する:

- 正式カード名・収録弾/型番(例: DMC58 3/16)
- カードの種類(クリーチャー/呪文/進化クリーチャー等)、文明、コスト、パワー、種族
- 特殊能力欄の全文(要約せず、公式テキストをそのまま引用する)
- カード画像のURL: 同じページの `meta-og:image`(または `meta-twitter:image`)に `https://dm.takaratomy.co.jp/wp-content/card/cardimage/<カードID>.jpg` の形式で入っている。本文中の `![<カード名>](...)` 画像リンクからも同じURLが取れる。これも一次ソースの一部として必ず控えておく

### 3. 出典を明記し、カード画像を添えて回答

ユーザーへの回答には、確認した公式ページのURLを Sources として必ず記載する(このセッションの引用ルールに従う)。カード名や効果を要約・言い換える場合も、元の公式テキストを併記するか、少なくとも一言一句変えずに引用した上で解説を加えること。

さらに、手順2で取得したカード画像のURLを回答に含める。Markdown形式(`![カード名](画像URL)`)で埋め込み、テキストを読むだけでなくカードの見た目も確認できるようにする。複数枚のカードを紹介する場合は、それぞれのカードの説明の直後(またはカード名の直下)に対応する画像を置き、どの画像がどのカードかが一目で分かるようにする。画像URLが取得できなかった場合は無理に埋め込まず、その旨を一言添えればよい。

注意: これは公式サイトの画像を直接参照するホットリンクであり、実行環境のネットワーク制限上、画像データそのものをダウンロードしてBase64埋め込みにしたり、ウィジェット等で再ホストしたりすることはできない。クライアントによっては画像プレビューの表示にクリックなど一手間必要になる場合があるが、これはクライアント側の外部画像プレビュー機能の仕様であり、この制約自体は許容してよい(無理に回避しようとしない)。

複数のカードが絡む話(コンボ、ループ、デッキ紹介など)を整理する場合は、各カードごとに上記1〜2を繰り返し、全カードのテキストを揃えてから手順やコンボの説明を組み立てる。想像や記憶で効果を補完しない。

### 4. 見つからない・確信が持てない場合

検索しても該当カードが特定できない、または字幕の聞き取りが不明瞭すぎる場合は、断定せずにユーザーに確認する。もっともらしい名前をでっち上げて回答しないこと。「〇〇という名前で検索しましたが該当カードが見つかりませんでした。正式名称や収録弾をご存知であれば教えてください」のように伝える。

## 注意点

- 同じカードでも再録(リメイク版・強化版)があるため、コスト・パワー・能力が完全一致するか必ず確認する
- 「メテオバーン」「G・リンク」「トライ・G・リンク」などのキーワード能力は、そのカード固有のテキストの一部としてそのまま引用する(独自解釈で言い換えない)
- ユーザーが過去の会話で誤った効果を前提に話していても、公式テキストと食い違う場合は指摘して訂正する
