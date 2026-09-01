# THE GALLERY — 引き継ぎ指示書

対象: `virtual-museum.html`(公開サイト) / `admin.html`(管理画面) / `data/*.json`(コンテンツデータ)
最終更新: 2026-08-16
用途: このドキュメントは、今後このサイトに手を加える担当(人間・AIどちらでも)が、既存の設計・デザインルールから逸脱しないための仕様書です。着手前に必ず全体に目を通してください。

> **要約**: Phase 1(分類フィルター・2種スライドショー表示・URL状態保持・モバイルスワイプ・一時停止)に加え、CMS化(Phase 2)の一部として、サーバーを持たない自前の管理画面(`admin.html`)を実装済み。GitHub Pagesで一般公開済み。BGM機能はクライアント判断により実装後に撤去し、現在は搭載していない。

---

## 1. プロジェクト概要

「THE GALLERY」は、日本絵画(浮世絵中心)をKen Burns風のズーム&パンで見せるヒーロースライドショーと、下部の作品グリッド、作者紹介セクションで構成されるバーチャル美術館サイト。

サイト本体(`virtual-museum.html`)はビルド不要の単一HTML/CSS/JSファイルだが、**コンテンツデータは`data/*.json`に分離**しており、ページ読み込み時に`fetch`で読み込む構成になっている(この分離が、後述する管理画面からの直接編集を可能にしている)。そのため、**`file://`で直接開くのではなく、必ずHTTP(S)経由で配信する必要がある**(ローカル確認時は`scripts/static-server.js`、本番はGitHub Pages)。

現状のセクション構成(上から順):

1. `header`(固定ヘッダー。表示モード切替・一時停止・分類フィルター開閉の3ボタン。ホバー時のみ表示)
2. `.filter-panel`(分類フィルターパネル。右側ドロワー)
3. `.hero`(絵画スライドショー、ビューポート100vh)
4. `.gallery-intro`(セクション見出し)
5. `.grid`(作品一覧グリッド。クリックで詳細モーダル表示)
6. `.artists-intro` / `.artists`(作者紹介セクション)
7. `.detail-modal`(作品詳細モーダル)
8. `footer`

---

## 2. 公開・リポジトリ情報

- **公開URL**: https://studio-poplar.github.io/THE-GALLERY/
- **リポジトリ**: `github.com/studio-poplar/THE-GALLERY`(GitHub Pages、`master`ブランチのルートから配信)
- **ローカルフォルダ**: `C:\Users\dito\Desktop\Claudcode\成果物\THE GALLERY\`
  (2026-08-16に `Claudcode\THE GALLERY` からこの場所へ移動された。以前のパスへの参照が残っていないか要注意。GitHub Desktop側もリポジトリの再登録が必要になった実績あり)
- **ローカル動作確認用サーバー**: `scripts/static-server.js`(Node.js、ポート5183)。`.claude/launch.json`の`the-gallery`設定から起動可能。`file://`直接オープンでは`fetch`が失敗するため、必ずこのサーバー(またはGitHub Pages)経由で確認すること。
- **デプロイ方法**: GitHub Desktop等で`master`へコミット→push。ビルドステップなし、pushすればそのまま数十秒〜1分で反映される。

---

## 3. 技術スタック / アーキテクチャ

- 依存ライブラリ: **なし**(vanilla JSのみ)
- サーバー/データベース: **なし**。静的ホスティング(GitHub Pages)のみ
- コンテンツデータ: `data/works.json` / `data/artists.json` / `data/site-settings.json` / `data/hero-layout.json` の4ファイル。`virtual-museum.html`・`admin.html`双方がこれらを`fetch`で読み込む
- 固定語彙(年代区分`ERAS`・流派技法`SCHOOLS`)は**JSONにせず、両ファイルのJS内にハードコード**している(管理画面から自由に追加・削除されて表記ゆれが起きるのを防ぐための意図的な設計)
- 画像: 外部URL参照(Wikimedia Commons等の`Special:FilePath`経由、5.5節参照)。作者の肖像画は管理画面からアップロードした場合はJSONにdata URLとして直接埋め込まれる(5.6節参照)
- 音声: **BGM機能自体を撤去済み**(6章参照)

---

## 4. デザイントークン

### カラー

| 変数 / 値 | 用途 |
|---|---|
| `--ink: #f4efe4` | 基本文字色(オフホワイト) |
| `--dim: rgba(244,239,228,.6)` | 補助文字色 |
| `--accent: #d7b56a` | アクセントカラー(真鍮/ゴールド) |

**この3変数だけで配色を組む**ルールは継続。ただし管理画面のサイト設定タブから`--ink`/`--accent`(`--dim`はinkから自動計算)を変更可能。また、スライド内の文字グループだけは`hero-layout.json`の`color`で個別に上書きできる(5.7節参照。未設定時はサイト共通色を継承)。

### タイポグラフィ

- 本文: `"Hiragino Kaku Gothic ProN","Yu Gothic",-apple-system,BlinkMacSystemFont,sans-serif`(管理画面から変更可)
- 見出し・作品タイトル等: `.serif`クラス(`Georgia, "Times New Roman", "Hiragino Mincho ProN", serif`)
- ラベル系は letter-spacing 広めの小さいオールキャップス風で統一(既存踏襲)

### 余白・グリッド

- セクション横パディングは基本6%(ヘッダーのみ5%)
- グリッドは`repeat(auto-fit, minmax(280px,1fr))` + `gap:2px`

### z-indexマップ

| z-index | 要素 |
|---|---|
| 70 | `.detail-modal`(作品詳細モーダル。バックドロップは69) |
| 60 | `.filter-panel`(バックドロップは59) |
| 50 | `header` |
| 10 | `.progress` |
| 7 | `.hero-empty` |
| 6 | `.el-group`(スライド内テキストグループ) |
| 3 | `.vignette` |
| 2 | `.slide.active` |

新しいUIをヒーロー内に足す場合はこの表に沿うこと。**来場者シルエット(`.visitors`)・スクロール誘導テキスト(`.scroll-cue`)は削除済み**(クライアント要望、6章参照)。

---

## 5. コンポーネント仕様

### 5.1 ヘッダー(`header`)

`#modeBtn`(表示モード切替)・`#pauseBtn`(一時停止)・`#menuBtn`(フィルター開閉ハンバーガー)の3ボタン。**BGMミュートボタン(`#muteBtn`)は撤去済み**。

ホバー可能なデバイスでは、ヘッダーはデフォルト非表示(`opacity:0`)で、ホバー時のみ表示される。表示モードを「CAPTIONS OFF」に切り替えた瞬間はカーソルが乗ったままでも強制的に隠す(`.force-hidden`クラス、mouseleaveで解除)。タッチ端末では常時表示。

### 5.2 ヒーロー・スライドショー(`.hero`)

- データはフィルター結果(`currentWorks`)を`renderHero()`で毎回作り直す
- 1枚8秒(`SLIDE_MS`)。Ken Burns(CSS `kenburns`)・進捗バー(`barfill`)のアニメーション時間と一致させること
- **スライド内テキストは1つのグループとして扱う**(重要な仕様変更): タイトル・作者/年・解説文・流派タグ・技法ラベルの5要素は、以前は個別に自由配置できたが、作品ごとに文字量が違うと重なる問題があったため、**`.el-group`という1つのコンテナにまとめ、内部は自動縦積み(margin-topによる通常のドキュメントフロー)**にした。管理画面から編集できるのはグループ全体の位置(x, y)・幅(width)・回転(rotation)・文字色(color)のみで、内部要素を個別に動かすことはできない仕様。この設計を「不便だから」と個別配置に戻す場合は、重なり問題への対策(要素ごとの高さを考慮した自動レイアウト計算など)を別途検討すること。
- 画像のみ/画像+解説モード: `.hero.mode-image-only`で`.el-group`と`.progress`ごと非表示(Ken Burnsは維持)。モバイルでは`.caption`のみ追加で非表示。
- 一時停止: `.hero.paused`でKen Burns・進捗バーの両アニメーションを`animation-play-state:paused`。`remainingMs`/`slideDeadline`で残り時間を保持し再開時に再スケジュール。
- タッチスワイプ: 40px以上の横移動で`goTo(i)`を呼ぶ(自動タイマーは`goTo`内で毎回リセット)。
- 絞り込み結果0件: `#heroEmpty`を表示しスライド生成をスキップ。

### 5.3 文字の視認性(scrim)

`.el-group`内の各要素(title/sub/caption/specs)には、絵画の上でも文字が読めるよう半透明の黒地+ドットパターン+`backdrop-filter:blur(6px)`+`text-shadow`を適用している(2026-08-16、視認性改善のため強化済み)。絵画が完全に隠れることのないよう、あくまで半透明を維持すること。

### 5.4 作品グリッド(`.grid`/`.card`)・詳細モーダル(`.detail-modal`)

グリッドのカードをクリックしても**ヒーローのスライドショーには一切影響しない**(以前は`goTo()`でジャンプしていたが、クライアント要望で変更)。代わりに`.detail-modal`が開き、拡大画像・タイトル・作者/年・解説文・技法/サイズ・収蔵美術館(いずれもヒーローでは省略されている情報を含むフルスペック)を表示する。閉じるボタン・背景クリック・Escキーで閉じる。

### 5.5 作者紹介セクション(`.artists-intro`/`.artists`)

`ARTISTS`データを元に、フィルター条件に関わらず全作者を常時表示。肖像は220×280pxの縦長ボックス、`object-position:top`で上寄せ(顔が見切れる問題への対策、2026-08-16)。肖像未設定の作者は頭文字のプレースホルダー表示。

### 5.6 分類フィルター(`.filter-panel`)

作者は`<select>`(単一選択)、年代・流派/技法はチェックボックスの「チップ」UI(複数選択可)。状態は`state`オブジェクトで一元管理し、変更のたびに`refresh()`と`syncURL()`を呼ぶ。

**モバイル対応の追加**(2026-08-16): チェック項目が多く縦に長くなる問題への対策として、パネル下部に`position:sticky`の「この条件で見る(◯点)」ボタンを常時固定表示。押すとパネルを閉じ、ページ先頭までスムーズスクロールする。

URLクエリパラメータ: `artist` / `era` / `school` / `mode`。~~`muted`~~(BGM撤去に伴い削除)。

### 5.7 画像調達ルール

- `source:'wikimedia'`は`Special:FilePath`経由(`commonsUrl()`)。それ以外(`loc`/`mfa-boston`/`rijksmuseum`/`lacma`)は`file`に直接URL。
- 新規追加時は必ず実在確認してから記入(推測禁止)。
- 現状は**Wikimedia等への直リンク運用を継続する方針**(クライアント判断。自社サーバー/CDNへの移行は見送り、hotlinkのまま一般公開中)。将来アクセスが増えて帯域面が気になった場合に再検討する位置づけ。
- ライセンス確認記録は`gallery_works_checklist.csv`に追記していく運用。

---

## 6. 既知の変更点・撤去した機能

- **BGM機能は実装後に撤去済み。** 当初Pixabayの楽曲("Kyoto Garden" by Back_Drop)を採用する予定で実装したが、代替候補曲がPixabay上で「AI generated/modified」と明記されているのを発見したことをきっかけに、クライアント判断でBGM機能自体を仕様から外した。`<audio>`要素・ミュートボタン・関連JS・`footerCredit`/`bgmFile`フィールドはすべて削除済み。**復活させる場合は本ドキュメントの旧版(git履歴)を参照し、自動再生ポリシー対応(デフォルトミュート・ユーザー操作起点の再生)を作り直すこと。**
- **来場者シルエット(`.visitors`)・スクロール誘導テキスト(`.scroll-cue`)を削除。** クライアント要望による単純な削除。
- **スライド内5要素の個別配置 → 1グループの自動縦積みに変更。** 5.2節参照。
- **グリッドカードクリックの挙動を変更。** ヒーローへのジャンプ→詳細モーダル表示に変更(5.4節)。

---

## 7. 管理画面(`admin.html`)

サーバー・データベースを持たない、**自前の簡易CMS**。4タブ構成(作品/作者/サイト設定/スライドレイアウト)。

### 7.1 アクセス

`https://studio-poplar.github.io/THE-GALLERY/admin.html`。簡易パスワードゲートあり(クライアント側で把握済み)。**これはサーバー側認証ではなく、ブラウザ内でのSHA-256ハッシュ比較のみ**(view-sourceで解析すれば突破できるレベル)。`robots.txt`と`<meta name="robots" content="noindex">`で検索エンジンからは除外している。

### 7.2 保存の仕組み(重要)

サーバーを持たないため、2つの保存方法がある。

1. **GitHubに直接保存(推奨)**: 右上「⚙ GitHub連携設定」で、GitHubユーザー名/リポジトリ名/ブランチ名/Personal Access Token(Contents: Read and write権限に絞ったfine-grainedトークン)を設定すると、各タブの「GitHubに保存」ボタンから**GitHub Contents API経由で直接コミット**できる。トークンはブラウザの`localStorage`にのみ保存され、api.github.com以外には送信されない。
2. **JSONダウンロード(フォールバック)**: 「JSONをダウンロード」で出力し、手動で`data/`内の同名ファイルを置き換えてコミット。GitHub連携未設定の場合、またはトラブル時のフォールバックとして残してある。

作者の肖像画像は、管理画面からファイルをアップロードすると自動的に縮小・圧縮され、**JSON内にdata URLとして直接埋め込まれる**(`portraitSource:'upload'`)。この場合、画像ファイル自体を別途配置する必要はない。

### 7.3 コンフリクトについて

GitHub連携での保存とGitHub Desktop等での手動pushを併用していると、`data/*.json`でgitのマージコンフリクトが発生することがある(実際に発生した実績あり)。解決時は、両方の変更内容を確認した上で**JSONとして正しい形に手動で統合する**こと(コンフリクトマーカー `<<<<<<<` / `=======` / `>>>>>>>` を残したままコミットしない)。

---

## 8. データスキーマ

### `data/works.json`

```js
{
  title, artist, year, yearNum, era, schools: [], medium, venue, source, file, caption
}
```
(フィールドの意味は既存の`WORKS`スキーマと同じ。詳細は`virtual-museum.html`冒頭のコメント、または`admin.html`の作品編集フォームを参照)

### `data/artists.json`

```js
{
  name, dates, portraitSource, portraitFile, bio, works, legacy
}
```

### `data/site-settings.json`

```js
{
  logoPrefix, logoAccent, colorInk, colorDim, colorAccent, fontStack,
  introEyebrow, introHeading, introBody,
  footerDisclaimer, footerCopy
}
```
(`footerCredit`・`bgmFile`はBGM撤去に伴い削除済み)

### `data/hero-layout.json`

```js
{ x, y, width, rotation, color }
```
スライド内テキストグループ1つ分の位置・サイズ・回転・文字色(`color`は`null`でサイト共通色を継承)。**以前は要素ごとの配置だったが、現在はこの1グループ分のみ**(6章参照)。

---

## 9. 新セクション追加時のチェックリスト

- [ ] 配色は`--ink`/`--dim`/`--accent`の3色のみか
- [ ] 見出しに`.serif`クラスを使っているか
- [ ] セクション横パディングが6%基調か
- [ ] ヒーロー内に要素を足す場合、z-indexマップ(4章)と矛盾しないか
- [ ] 画像を追加する場合、`source`/`file`が実在確認済みか。`era`/`schools`は固定語彙からか
- [ ] モバイル幅で崩れないか。フィルター・一時停止・表示モードの操作がタッチでも可能か
- [ ] `data/*.json`を直接編集した場合、管理画面(`admin.html`)側のフォームフィールドとスキーマが一致しているか

---

## 10. 未対応・未確定のまま残っている事項

- OGPの動的生成(フィルター条件付きURL共有時のプレビュー)は未対応
- 個々の作品への固有URL(パーマリンク)は未対応
- `prefers-reduced-motion`は最小限の対応のまま
- 画像はWikimedia等への直リンクを継続する方針(自社ホスティングへの移行は保留)

---

*このドキュメントは2026-08-16時点の実装を正としています。実装を変更した場合は、対応する章を必ず更新してください。*
