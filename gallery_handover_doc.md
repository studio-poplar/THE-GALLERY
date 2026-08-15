# THE GALLERY — 引き継ぎ指示書
対象ファイル: `virtual-museum.html`
最終更新: 2026-08-15(Phase 1機能追加を反映)
用途: このドキュメントは、今後 `virtual-museum.html` に新しいセクションを追加していく際に、既存のデザイン・実装ルールから逸脱しないための仕様書です。新しいセクションを作る担当(人間・AIどちらでも)は、着手前に必ずこのドキュメント全体に目を通してください。

> **2026-08-15更新の要約**: 依頼書(製作依頼書)に基づき、①分類フィルター(作者・年代・流派技法) ②スライドショー2パターン(画像のみ/画像+解説) ③BGM(デフォルトミュート) ④URL状態保持 ⑤モバイルスワイプ ⑥一時停止ボタン を実装(Phase 1)。掲載作品も浮世絵中心の日本絵画22点に総入れ替え。CMS化(依頼書5章)はPhase 2として未着手。

---

## 1. プロジェクト概要

「THE GALLERY」は、名画をKen Burns風のズーム&パンで見せるヒーロースライドショーと、下部の作品グリッドで構成されたバーチャル美術館サイトのデモです。フレームワークは使わず、素のHTML/CSS/JS 1ファイルで完結しています(ビルド不要、そのままブラウザで開けば動く)。

現状のセクション構成(上から順):

1. `header`(固定ヘッダー、常時最前面。分類フィルターの開閉・表示モード切替・BGMミュート・一時停止の各ボタンを含む)
2. `.filter-panel`(分類フィルターパネル。ヘッダーのハンバーガーから開閉する右側ドロワー)
3. `.hero`(絵画スライドショー、ビューポート100vh)
4. `.gallery-intro`(セクション見出し)
5. `.grid`(作品一覧グリッド、スクロールでフェードイン)
6. `footer`

今後はこの5番と6番の間、または6番の前に新セクションを追加していく想定です。

---

## 2. 技術スタック / 依存関係

- 依存ライブラリ: **なし**(vanilla JSのみ。GSAPも未使用)
- 画像: すべて外部URL参照(Wikimedia Commons経由。ダウンロード・同梱はしていない)
- 音声: `<audio>`要素で参照。ファイル本体(`assets/audio/bgm-kyoto-garden.mp3`)は同梱していないため、公開前にライセンスに沿って取得し配置すること(4.7節参照)
- ビルドツール: なし。単一HTMLファイルとして完結させること
- 対象ブラウザ: モダンブラウザ前提(`aspect-ratio`, `IntersectionObserver`, CSS `clamp()`, `URLSearchParams`, `history.replaceState` などを使用)

**新セクションでGSAP ScrollTriggerのような「スクロール連動の作り込み」をしたい場合**は、このプロジェクトの姉妹デモ(`architecture-scroll-demo.html` 等)で使っている以下のCDNを追加してよい。ただし増やすほど読み込みが重くなるため、本当に必要なセクションのみに留めること。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
```

現状のセクション(グリッドのフェードイン)は`IntersectionObserver`だけで実現しており、単純な「スクロールで出現」程度ならGSAPを追加せずこちらを流用するほうが軽量。

---

## 3. デザイントークン

### カラー

| 変数 / 値 | 用途 |
|---|---|
| `--ink: #f4efe4` | 基本文字色(オフホワイト) |
| `--dim: rgba(244,239,228,.6)` | 補助文字色(サブテキスト、キャプション) |
| `--accent: #d7b56a` | アクセントカラー(真鍮/ゴールド。ロゴの一部、進捗バー、番号ラベル、フィルターのチップ選択状態などに使用) |
| 背景 `#0a0a0b` 系 | ページ全体の地色。セクションごとに `#050505`〜`#0a0a0b` の範囲で微調整可 |

新セクションでも **この3変数(`--ink` / `--dim` / `--accent`)だけで配色を組む**こと。新しい色を増やす場合は必ずこのドキュメントに追記してから使う。

### タイポグラフィ

- 本文・UI文字: `"Hiragino Kaku Gothic ProN","Yu Gothic",-apple-system,BlinkMacSystemFont,sans-serif`(bodyのデフォルト)
- 見出し・作品タイトルなど「美術館らしさ」を出したい箇所: `.serif` クラス(`Georgia, "Times New Roman", "Hiragino Mincho ProN", serif`)
  - 使用箇所: ロゴ、作品タイトル(`.meta-top .title`)、セクション見出し(`h2`)、カードのタイトル(`.cinfo .t`)、フィルターパネルの見出し(`.filter-panel h3`)
- ラベル系(venue名、specs、eyebrow、hero-btn、chipなど)は **letter-spacing を広めに取った小さいオールキャップス風**の扱いで統一している(`.62rem`〜`.72rem`、`letter-spacing: .08em`〜`.24em`)。新しいラベルを増やす際もこのトーンに合わせる。

### 余白・グリッド

- セクション横パディングは基本 **6%**(`.gallery-intro`, `.meta-bottom` 等)。ヘッダーのみ `5%`。
- セクション上下パディングの目安: 導入文セクションは `120px 上 / 40px 下`、footerは `80px 上 / 60px 下`。新セクションもこの桁感(80〜120px単位)に合わせ、詰まりすぎないようにする。
- グリッドは `repeat(auto-fit, minmax(280px,1fr))` + `gap:2px` の「隙間をほぼ潰した美術館的レイアウト」。カード間の余白を広げたい場合は意図的にデザイン変更として扱う(今のスタイルは"ミニマルな展示壁"を意識している)。

### z-index マップ(重要: ヒーロー内で新規要素を足すときに必ず確認)

| z-index | 要素 |
|---|---|
| 60 | `.filter-panel`(分類フィルターのドロワー。開いている間のみ表示。バックドロップは59) |
| 50 | `header`(ハンバーガー・一時停止/BGM/表示モードの各ボタンを含む`.header-actions`。header自体は`pointer-events:none`で、クリック可能な要素にだけ個別に`auto`を付与する既存ルールを踏襲) |
| 10 | `.progress`(進捗バー) |
| 7 | `.hero-empty`(フィルター結果0件時のメッセージ) |
| 6 | `.meta-top` / `.meta-bottom` / `.scroll-cue` |
| 5 | `.visitors`(来場者シルエット) |
| 3 | `.vignette` |
| 2 | `.slide.active` |

ヒーロー内に新しいUIを足す場合は、この表に沿って適切なz-indexを割り振ること(vignetteより手前・visitorsと衝突しない位置、が基本ルール)。一時停止ボタンなど新しい操作要素を追加する際は、ハンバーガーボタンと同様に「`pointer-events:auto`を個別付与し、この表に行を追加する」を必ず行うこと。

---

## 4. コンポーネント仕様

### 4.1 ヘッダー(`header`)

固定表示、`pointer-events:none` をheader自体にかけ、クリック可能な要素にだけ `pointer-events:auto` を個別に付与している。`.header-actions`内に表示モード切替(`#modeBtn`)・BGMミュート(`#muteBtn`)・一時停止(`#pauseBtn`)・ハンバーガー(`#menuBtn`)の4ボタンを配置。すべて実クラス`.hero-btn`(ハンバーガーのみ`.menu-btn`)を使い、キーボードフォーカス可能な`<button>`要素にしてある(引き継ぎ指示書6章で指摘されていたキーボード操作対応の第一歩)。

ハンバーガー(`#menuBtn`)クリックで`.filter-panel`の開閉をトグルする(現時点でのナビゲーションの役割はこれのみ。セクション間ジャンプ用のオーバーレイメニューは引き続き未実装)。

### 4.2 ヒーロー・スライドショー(`.hero`)

- データは `WORKS` 配列(4.4節参照)から**分類フィルターで絞り込んだ結果(`currentWorks`)**をJSで動的にDOM生成している。フィルターが変わるたびに`renderHero()`が全スライド・進捗バーを作り直す。HTMLを直接編集して作品を増やさないこと。
- 1枚あたりの表示時間: `SLIDE_MS = 8000`(8秒)。Ken Burnsのアニメーション時間(`kenburns 8s`、CSS側)、進捗バーの`barfill 8s`アニメーションと必ず一致させる。ここを変更する場合は **JSの`SLIDE_MS`とCSSの2箇所のアニメーション時間**を揃えて直すこと。
- スライド切り替えは `goTo(i)` 関数に集約。Ken Burnsを毎回リスタートさせるため `animation:none` → 強制リフロー(`void bg.offsetWidth`) → `animation:''` という定石パターンを使っている。
- 進捗バー(`.progress .bar`)は、以前はCSS `transition`で実装していたが、**一時停止機能のために`@keyframes barfill`によるCSSアニメーションへ変更した**(`animation-play-state`はtransitionには効かないため)。同様の「一時停止可能な進捗表現」を作る場合はtransitionではなくアニメーションを使うこと。
- **一時停止(`#pauseBtn`)**: `.hero`に`.paused`クラスを付与し、`.hero.paused .slide.active .bg` と `.hero.paused .progress .bar.filling i` の両方に`animation-play-state:paused`をかけることで、Ken Burnsと進捗バーを同時に止める。JS側では`setTimeout`の残り時間を計算して保持し、再開時に残り時間で再スケジュールする(`remainingMs`/`slideDeadline`)。
- **タッチスワイプ**: `.hero`に`touchstart`/`touchend`を追加。40px以上の横移動でしきい値判定し、既存の`goTo(i)`を呼ぶ(自動タイマーは`goTo`内で毎回リセットされるため、スワイプ後は自動送りタイマーも自然にリセットされる)。
- **画像のみ/画像+解説モード**: `.hero`に`.mode-image-only`クラスを付与すると`.meta-top`/`.meta-bottom`ごと非表示になる(Ken Burns・進捗バー・来場者シルエット・vignetteは維持)。「解説」の実体は`.meta-top`内の`.caption`(作品ごとの`caption`フィールド、2〜3文の日本語解説)。モバイル幅(640px以下)では`.caption`自体を非表示にし、タイトル/作者/年のみ残す。
- **絞り込み結果が0件の場合**: `#heroEmpty`(z-index:7)を表示し、スライド生成・タイマーはスキップする。新しいフィルター軸を追加する際もこの空状態のハンドリングを維持すること。

### 4.3 来場者シルエット(`.visitors`)

SVGをJSで動的生成(`personSVG()`関数)。すべてのスライドに対して固定表示(スライドごとに変えていない)。今後スライドごとに人数や配置を変えたい場合は、`WORKS`の各要素に `visitorCount` のようなフィールドを追加し、`personSVG`の呼び出し数をそれに応じて変える形で拡張できる。

### 4.4 作品データスキーマ(`WORKS`配列)

```js
{
  title:   '作品タイトル(日本語+英語表記)',
  artist:  '作者名(英語表記 + 日本語カッコ書き。例: "Katsushika Hokusai (葛飾北斎)")',
  year:    '制作年 表示用文字列(例: "c. 1830–1832")',
  yearNum: '制作年 フィルター/ソート用の数値(代表年)',
  era:     '年代区分。固定語彙: genroku(元禄期) | horeki-temmei(宝暦・天明期) | kasei(化政期) | bakumatsu(幕末期) | other(その他の時代)。ERAS配列で定義',
  schools: '流派・技法。固定語彙の配列(複数可): ukiyo-e(浮世絵) | kano(狩野派) | rinpa(琳派) | yamato-e-tosa(大和絵・土佐派) | nanga(文人画・南画)。SCHOOLS配列で定義',
  medium:  '技法 · サイズ  ※ " · " (半角スペース+中点+半角スペース) で区切る。JS側でsplit(" · ")して2行表示に分解しているため区切り文字を変えないこと',
  venue:   '収蔵美術館名(英語・大文字表記)',
  source:  '画像調達元。"wikimedia" | "loc" | "mfa-boston" | "rijksmuseum" | "lacma" のいずれか',
  file:    'sourceが"wikimedia"の場合はCommonsのファイル名(File:を除く)。それ以外のsourceの場合は画像への直接URL',
  caption: '作品解説文(2〜3文、日本語)。表示モード「画像+解説」で.meta-top内に表示される',
}
```

`era`・`schools`は自由記述にせず、必ず`ERAS`/`SCHOOLS`定数(JS冒頭)で定義した固定語彙から選ぶこと(表記ゆれ防止。依頼書4-2節)。新しい年代区分・流派を追加する場合は、この2つの定数配列に追記し、このドキュメントの表(4.4節)も更新すること。

作者名から生成する`artistSlug()`はURLクエリ用のID(例: `katsushika-hokusai`)を作る。マクロン等の発音区別符号(ō, ūなど)はNFD正規化で除去してから使っているため、新しい作者を追加する際も英語表記部分(カッコの前)がそのままスラグの元になる。

### 4.5 画像調達ルール(重要)

- `source:'wikimedia'`の作品は、**Wikimedia Commonsの `Special:FilePath` リダイレクトURL**経由で参照する(`commonsUrl()`関数、従来通り)。
- `source`がそれ以外(`loc` / `mfa-boston` / `rijksmuseum` / `lacma`)の場合は、`file`フィールドに画像への直接URLをそのまま持たせる(`imageUrl()`関数が`source`で分岐する)。現時点で掲載している22点はすべて`wikimedia`経由。
- 新しい作品を追加する際は、必ず実際にファイル/オブジェクトページの存在を確認してから記入すること(存在しないファイル名・URLを推測で入れない)。この確認はWikimedia Commonsに限らず、LOC/MFA Boston/Rijksmuseum/LACMAいずれのsourceでも同様。
- 掲載できるのは著作権保護期間が満了した作品、またはライセンス上再配布可能な作品のみ。実在の現代アーティストの作品や、ライセンス不明な画像は使用しない。対象時代は依頼書4-1節の通り「著作権に触れない範囲(浮世絵中心の江戸期以前)」に限定する方針。
- 本番運用に移行する際は、Wikimedia直リンクではなく自社サーバー/CDNに画像をホスティングし直すこと(hotlinkは帯域・可用性をWikimedia側に依存するため、デモ・プロトタイプ用の割り切り)。footerに免責文言が入っているので、実運用に切り替える際はこの文言も見直すこと。
- 作品追加時のライセンス確認記録は、同梱の `gallery_works_checklist.csv` に追記していく運用とする(依頼書4-5節、9章参照)。

### 4.6 作品グリッド(`.grid` / `.card`)

`IntersectionObserver`で `.card` が画面に入ったら `.in` クラスを付与 → CSSの `transition` でフェードイン+上方向スライドイン、という素朴な実装。カードクリックでヒーローの該当スライド(絞り込み後のインデックス)にジャンプする機能付き(`goTo(i)` + トップへスムーズスクロール)。フィルターが変わるたびに`renderGrid()`で作り直すため、`io.observe()`も再描画のたびに新しいカードへ再登録している。

### 4.7 分類フィルター(`.filter-panel`)

- 作者は`<select>`(単一選択、`WORKS`から動的生成)、年代・流派/技法はチェックボックス+ラベルの「チップ」UI(複数選択可、`ERAS`/`SCHOOLS`固定語彙から生成)。
- フィルター状態は`state`オブジェクト(JS冒頭)で一元管理し、変更のたびに`refresh()`(ヒーロー/グリッド/カウント再描画)と`syncURL()`(URLクエリへの反映)を呼ぶ。
- URLクエリパラメータ: `artist`(スラグ、デフォルト`all`時は省略) / `era`(カンマ区切り、全選択時は省略) / `school`(カンマ区切り、全選択時は省略) / `mode`(`image-only`の時のみ付与) / `muted`(`0`の時のみ付与)。ページ読み込み時に`restoreStateFromURL()`で復元し、`history.replaceState`でURLを都度更新する(依頼書4-3節)。

### 4.8 BGM(`#bgm`)

- `<audio id="bgm" loop preload="none">`要素。デフォルトは`muted=true`かつ再生停止。ヘッダーの`#muteBtn`をユーザーが押した時点で初めて`bgm.play()`を呼ぶ(ブラウザの自動再生ポリシーに準拠。依頼書4-3節)。
- スライド切り替え(`goTo`/8秒タイマー)とは完全に独立したロジックで、スライドが切り替わってもBGMの再生・ループには一切干渉しない。
- 音源ファイルは著作権上の理由でリポジトリに同梱していない。`assets/audio/bgm-kyoto-garden.mp3`というパスを参照しているが、実ファイルは公開前に用意すること(詳細は同フォルダの`README-bgm.md`参照)。

---

## 5. 新セクション追加時のチェックリスト

新しいセクションを実装する担当は、公開前に以下を確認すること。

- [ ] 配色は `--ink` / `--dim` / `--accent` の3色のみで構成されているか
- [ ] 見出しに `.serif` クラスを使っているか(和文でも英文でも、美術館らしい格調を出す箇所は明朝/セリフ系)
- [ ] セクション横パディングが `6%` 基調になっているか
- [ ] ヒーロー内に要素を足す場合、z-indexマップ(3章)と矛盾しないか。新しい操作ボタンは`pointer-events:auto`個別付与+z-indexマップへの追記を忘れていないか
- [ ] 画像を新規追加する場合、`source`/`file`のペアが実在確認済みか(4.5節)。`era`/`schools`は固定語彙から選んでいるか(4.4節)
- [ ] スクロールアニメーションを追加する場合、軽量な内容は`IntersectionObserver`、映画的なスクラブ演出が必要な場合のみGSAP ScrollTriggerを追加しているか(2章参照)
- [ ] モバイル幅(375px程度)で崩れないか確認したか。分類フィルター・一時停止・ミュート・表示モードの各操作がタッチでも問題なく行えるか
- [ ] URLクエリで状態を共有した場合、再読み込みで同じ表示に復元されるか(4.7節)

---

## 6. 既知の未対応事項 / 申し送り

- **CMS化は未着手(Phase 2)。** 依頼書5章の通り、分類項目(フィールド)の追加・削除やデザイン変更をノーコードで行えるようにするには、データと表示ロジックを分離したアーキテクチャへの移行が必要。着手にはCMS製品選定・ホスティング/予算/納期・「デザイン変更」の範囲など、依頼書6章の未確定事項の回答が前提。
- **BGM音源ファイルは同梱していない。** ライセンス上の理由(Pixabayの利用規約でも自身でのダウンロード・自社ホスティングが前提)から、公開前に`assets/audio/`配下に配置する必要がある(`README-bgm.md`参照)。
- **ハンバーガーメニューは分類フィルターの開閉専用。** セクション間を移動するナビゲーション機能はまだない。セクションが増えたら、別途ナビゲーション設計が必要。
- **`prefers-reduced-motion`は最小限の対応。** Ken Burnsと進捗バーのアニメーションをCSSメディアクエリで無効化しているが、一時停止ボタンとは独立した仕組みであり、どこまで作り込むかは依頼書6章で未確定のまま。
- **OGPの動的生成は未対応。** フィルター条件付きURLを共有した際、SNSでのプレビュー(タイトル・説明文)はページ全体の`<title>`のまま変化しない。依頼書6章で引き続き未確定。
- **個々の作品への固有URL(パーマリンク)化は未対応。** フィルター条件の共有(4.7節)とは別の論点として残っている。
- **画像の実読み込みは開発環境側で最終確認できていない。** Wikimedia Commons上でファイルページの存在(実在性)はWebFetchで確認済みだが、22点すべての実ブラウザでの表示確認は別途行うこと。
- **本番の画像/BGMホスティングは未対応。** 4.5節・4.8節の通り、現状はWikimedia直リンク+BGM別配置のデモ実装。

---

## 7. 参考: スクロール演出を追加したい場合のテンプレート

映画的な「スクロール量に応じて何かが動く/切り替わる」セクションを追加したい場合、姉妹プロジェクトで検証済みの以下のパターンを使うこと(実装時のハマりどころも含めて検証済み)。

```html
<div class="wrapper" style="position:relative; height:300vh;">
  <div class="scene" style="position:sticky; top:0; height:100vh; overflow:hidden;">
    <!-- ここに演出したい要素 -->
  </div>
</div>
```

```js
gsap.registerPlugin(ScrollTrigger);
const progressState = { value: 0 };
gsap.to(progressState, {
  value: 1,
  ease: 'none',
  scrollTrigger: {
    trigger: '.wrapper',
    start: 'top top',
    end: 'bottom bottom',
    scrub: 0.6,
    onUpdate: (self) => { /* self.progress (0〜1) を使って演出を更新 */ }
  }
});
```

**注意点(過去にハマった箇所)**:

- `body`に `overflow-x:hidden` を指定すると、仕様上 `overflow-y` が暗黙に `auto` になり、`position:sticky` とスクロール位置がズレるバグが起きる。横方向のはみ出し防止には必ず `overflow-x:clip` を使うこと(このファイルでも既に`clip`にしてある)。
- Three.jsでオブジェクトをスクロール量に応じて移動・拡大したい場合、`transform-origin`をCSSで`0 0`と書いても、GSAPがアニメーションを制御すると内部的に`50% 50%`基準へ上書きされることがある。GSAPで動かす要素は `gsap.set(el, { transformOrigin: "0px 0px", ... })` のようにGSAP側のプロパティとして明示的に指定すること。

---

*このドキュメントは `virtual-museum.html` の現状(2026-08-15時点)を正としています。実装を変更した場合は、対応する章を必ず更新してください。*
