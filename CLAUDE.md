# 生涯学習新聞

生涯学習分野のデジタル新聞。静的HTMLのみで構成され、GitHub Pagesで公開している。

- 公開URL: https://djtobby.github.io/shogai-gakushu-shimbun/
- リポジトリ: DJTOBBY/shogai-gakushu-shimbun
- 姉妹紙: [手芸新聞](https://djtobby.github.io/shugei-shimbun/)（DJTOBBY/shugei-shimbun）— 本紙は手芸新聞と同じ構造・同じ発行手順

## 立て付け（最重要・毎号確認すること）

本紙は、**一般財団法人日本生涯学習協議会の理事長である山仲元が、個人として試作しているデジタル新聞**である。協議会の公式刊行物ではない。

この立て付けを守るため、以下を毎号必ず維持する。

- 奥付（12面）・`index.html`・`backnumbers.html`・`archive.html` の4箇所に「協議会の公式刊行物ではない」旨の免責文を残す
- 紙面の意見は編集部・編集長個人のものであり、協議会の見解を代表しないと明記する
- 発行人名は「山仲　元」。編集長コラムは「編集長」名義

**編集長コラムをAIが起草した場合は、本人の言葉に差し替えることを必ず提案すること。** 実名の媒体であり、文責は本人にある。

## ファイル構成

```
index.html          トップページ（最新号カード＋本紙紹介＋RSS購読案内）
backnumbers.html    号一覧
archive.html        記事横断検索＋編集長コラム連載一覧（各号のarticles配列を抽出したJSONを埋め込む）
feed.xml            RSS
sitemap.xml         サイトマップ
robots.txt          / manifest.json
issues/YYYY-MM-DD.html   各号の本体（1ファイル完結。全12面）
icons/              favicon.ico, apple-touch-icon.png, icon-192.png, icon-512.png, icon-master.png
og/YYYY-MM-DD.png   各号のOGP画像（1200×630）
notes/              ネタ帳・取材メモ（紙面ではない内部メモ）
```

## 新号を発行する手順（7点セット）

**この7つを全て更新すること。1つでも漏れるとリンク切れや情報の不整合になる。**

1. **`issues/YYYY-MM-DD.html`** — 前号をコピーし、全12面・`articles[]`配列・`renderHeadlineBand()`のitems・`ISSUE_DATE`・`ISSUE_FEATURE`・情報取得日を新規内容に書き換える。**使い回しは厳禁**
2. **`index.html`** — `.latest-callout` 内の号数・日付・見出し・リンク先・`og:image`
3. **`backnumbers.html`** — `<ul class="issue-list">` の先頭に新しい `<li>` を追加。`og:image` も更新
4. **`feed.xml`** — `<item>` を先頭に追加
5. **`archive.html`** — 新号のarticles配列を抽出してJSONデータに追記（下記コマンド参照）。`og:image` も更新
6. **`sitemap.xml`** — 新号のURLを追加（`lastmod`は発行日）
7. **`og/YYYY-MM-DD.png`** — 新号1面のOGP画像を生成（下記コマンド参照）

### archive.html 用のデータ抽出

```bash
node -e "
const fs=require('fs');
const html=fs.readFileSync('issues/YYYY-MM-DD.html','utf8');
const arr=new Function('return '+html.match(/const articles = (\[[\s\S]*?\n  \];)/)[1].replace(/;\$/,''))();
const out=arr.filter(a=>a.articleType!=='SHORTS').map(a=>({
  issueDate:'YYYY-MM-DD', issueLabel:'第N号', id:a.id, articleType:a.articleType,
  title:a.title, subtitle:a.subtitle||null, lead:a.lead, category:a.category, tags:a.tags,
  publishedAt:a.publishedAt
}));
console.log(JSON.stringify(out));
"
```
出力を `archive.html` の `const articles = [...]` の**末尾に追記**する（既存データは消さない）。

### OGP画像の生成

```bash
# 1. toolbarを隠した一時ファイルを作る
python3 -c "
s=open('issues/YYYY-MM-DD.html').read()
s=s.replace('<div class=\"toolbar no-print\">', '<div class=\"toolbar no-print\" style=\"display:none\">',1)
open('/tmp/og-src.html','w').write(s)"

# 2. 撮影（--user-data-dir は必須。省略するとユーザーの通常のChromeに干渉する）
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless \
  --user-data-dir=/tmp/chrome-isolated --disable-gpu \
  --screenshot="og/YYYY-MM-DD.png" --window-size=1200,630 --hide-scrollbars \
  --virtual-time-budget=3000 "file:///tmp/og-src.html"
```

**`--user-data-dir` を必ず付けること。** 2026年8月5日、これを省略して起動した結果、ユーザーの通常のChromeで印刷ダイアログが繰り返し開くという副作用が起きた。

（クラウド環境などChromeが無い場所で作業する場合は、OGP画像だけ後回しにして、他の6点を先に仕上げてよい。その場合は前号の画像を暫定的に参照させず、`og:image` は新号のパスのままにしてローカルで後から生成する。）

## 面構成（全12面）

| 面 | 内容 |
|---|---|
| 1面 | 総合（マストヘッド／見出し帯／一面トップ／サブニュース） |
| 2面 | オピニオン（編集長コラム／編集部論説／今週の特集） |
| 3面 | 分野別ニュース・小欄短信・新着記事一覧 |
| 4面 | 公民館・社会教育 |
| 5面 | リカレント・大学 |
| 6面 | 講座・教室 |
| 7面 | 学びの催事暦（表形式） |
| 8面 | 図書館 |
| 9面 | 博物館・文化 |
| 10面 | 特集・コラム・編集後記 |
| 11面 | 紙面索引 |
| 12面 | 奥付 |

## 記事データモデル

各号の `<script>` 内 `const articles = [...]` に記事オブジェクトを置く。`openArticleDialog()` が「続きを読む」で全文ダイアログを表示し、`?goto=記事ID` または `?goto=面ID` でディープリンクできる（archive.htmlはこれを使う）。

主なフィールド: `id, articleType, title, subtitle, lead, body[], category, tags[], author, authorRole, publishedAt, updatedAt, sourceName, sourceUrl, sourcePublishedAt, factOrOpinion, editorNote, keyPoints[], relatedArticles[]`

記事種別: `NEWS / BREAKING / EDITORIAL / CHIEF_EDITOR / FEATURE / INTERVIEW / MARKET / MATERIAL / TECHNIQUE / CULTURE / BUSINESS / WORLD / OPINION / SHORTS`

`displaySlot: 'auto-op-articles-p2'` を付けると2面に自動でコラム欄が増える。`boxSlot` を付けたSHORTSは3面の該当短信枠に入る。

## 落とし穴（過去に実際に起きた事故）

### 1. p2の3ボックスは本文が2箇所にある

2面の編集長コラム・編集部論説・今週の特集は、**同じ本文がHTMLとJSの2箇所に別々に存在する**。

- 表示用の生HTML（`<div class="op-box">` 内の `<div class="col-excerpt">`）
- `articles[]` 配列内の `col-chief-editor-001` / `editorial-001` / `feature-001`（ダイアログ用）

**片方だけ書き換えると、紙面表示と検索結果が食い違ったまま気づかない。** 新号を書いたら必ず両方を確認すること。検証コマンドは下記。

### 2. 記事の使い回し

前号をコピーして日付だけ変え、本文を書き換え忘れる事故が姉妹紙で実際に起きた。特に市場分析・論説・文化コラムのような常設コーナーで起きやすい。**レンダリング後のHTMLだけでなく、`articles[]` 配列の中身も号をまたいで機械的に突き合わせること。**

### 3. 裏の取れない数字を書かない

創刊号では、内閣府「生涯学習に関する世論調査」の年齢層別数値（60代55%、70歳以上42.5%）が二次情報でしか確認できず、調査年次を特定できなかった。そのため紙面4箇所に「調査年次は未確認」と明記した。

**確認できない数字・日付は書かない。書く場合は「未確認」と明記する。推測で埋めることは絶対にしない。**

## 発行前の検証

```bash
# JS構文チェック
python3 -c "
import re;s=open('issues/YYYY-MM-DD.html').read()
open('/tmp/c.js','w').write(re.search(r'<script>(.*)</script>',s,re.S).group(1))" && node --check /tmp/c.js

# p2の3ボックスがHTML側とarticles側で一致しているか
node -e "
const fs=require('fs');
const html=fs.readFileSync('issues/YYYY-MM-DD.html','utf8');
const arr=new Function('return '+html.match(/const articles = (\[[\s\S]*?\n  \];)/)[1].replace(/;\$/,''))();
const byId={}; arr.forEach(a=>byId[a.id]=a);
const norm=t=>t.replace(/<[^>]+>/g,'').replace(/\s+/g,'').replace(/[、。「」（）]/g,'');
[...html.matchAll(/<div class=\"col-excerpt\">([\s\S]*?)<\/div>\s*<button type=\"button\" class=\"read-more\" data-article-id=\"([^\"]+)\"/g)]
.forEach(([,inner,id])=>{
  const h=norm(inner), a=byId[id]; if(!a) return console.log('✗ 未定義',id);
  const t=norm(a.body.join(''));
  console.log((t.includes(h.slice(0,60))?'✓':'✗'), id, h.length, t.length);
});"
```

タグバランスの確認と、前号との本文重複チェックも行うこと。

## ネタ帳・取材メモ

`notes/` に地域別の取材メモがある（2026年8月5日、記者エージェント4体による取材）。

- `kaigai-kokusai-kikan.md` — ユネスコUIL、OECD、ILO、世界銀行、EU（12項目）
- `kaigai-europe.md` — 北欧、ドイツ、英国、EPALE（14項目）
- `kaigai-asia-pacific.md` — 韓国、シンガポール、中国、台湾、豪州（12項目）
- `kaigai-america-global-south.md` — 米国、カナダ、中南米、アフリカ（14項目）

各項目に「未確認」「二次情報」のフラグが立っている。**フラグの付いた情報を紙面に使うときは、必ず一次情報にあたり直すこと。**

### 記者エージェントの再取材

海外ネタを補充したいときは、サブエージェント（general-purpose）を地域別に配置する。プロンプトには必ず以下を含めること。

- 直近の情報を優先し、一次情報（機関の公式サイト）にあたる
- 発表日が確認できないものは「不明」と明記し、推測で埋めない
- 全文翻訳はせず要約にとどめる（著作権配慮）。引用は15語以内・1回まで
- 日本の制度との接点は「記者の見立て」として事実と区別する
- 成果物は `notes/` にMarkdownで書く

取材時に403やCAPTCHAで到達できなかったサイト: oecd.org、consilium.europa.eu、EPALE、gov.br/mec、unesco.org本体、survey.gov-online.go.jp（内閣府）。これらは検索結果の抜粋や公式PDFのミラーで代替し、その旨を明記すること。

## 主なネタ源

- [文部科学省 報道発表](https://www.mext.go.jp/b_menu/houdou/2026/index.html)
- [中央教育審議会 社会教育の在り方に関する特別部会](https://www.mext.go.jp/b_menu/shingi/chukyo/chukyo2/015/index.html)
- [文化庁 博物館総合サイト](https://museum.bunka.go.jp/)
- [マナパス（文科省・リカレント教育ポータル）](https://manapass.mext.go.jp/)
- [ユネスコ生涯学習研究所](https://www.uil.unesco.org/en) / [OECD Adult learning](https://www.oecd.org/en/topics/sub-issues/adult-learning.html)
- 各自治体の公民館・生涯学習ページ

## 編集方針

- 事実報道（NEWS等）と、意見・分析を含む記事（EDITORIAL・CHIEF_EDITOR・FEATURE・MARKET・BUSINESS・CULTURE）を、バッジと記事冒頭の注記で明確に区別する
- **1つの話題を複数面で角度違いに使い回さない。** 政策・現場・統計・施設・海外でジャンルを分散させる（姉妹紙で「同じ系統の記事ばかり」という指摘を受けた経緯がある）
- 全文転載はしない。公開情報を要約し、編集部の解説・見立てを加える形にする
- 出典は各記事末尾に必ず明記する
