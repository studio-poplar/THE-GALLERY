# BGM音源ファイルの配置について

`virtual-museum.html` は BGM として以下のパスを参照しています。

```
assets/audio/bgm-kyoto-garden.mp3
```

著作権上の理由(配布元の利用規約が「自身でのダウンロード・自社ホスティング」を前提としており、直リンクでの埋め込みを想定していないため)、音源ファイル本体はこのリポジトリに同梱していません。公開前に以下の手順でご用意ください。

## 選定した楽曲

- タイトル: "Traditional Japanese Instrumental Music (Kyoto Garden)"
- 制作者: Back_Drop
- 配布元: Pixabay Music
- URL: https://pixabay.com/music/world-traditional-japanese-instrumental-music-kyoto-garden-197608/
- ライセンス: Pixabay Content License
  - 商用利用: 可(改変・商用サイトへの組み込みを含め許諾されています)
  - クレジット表記: 任意(法的な義務はありません。footerに以下の任意クレジットを既に記載済みです)
    > Music: "Kyoto Garden" by Back_Drop (Pixabay)
  - ループ利用: 再生時間1分27秒。ループ再生を前提に作られたアンビエント楽曲のため、`<audio loop>`によるループ再生に適しています。

## 手順

1. 上記URLにアクセスし、Pixabayのダウンロードボタンからmp3ファイルを取得する(アカウント登録が必要な場合があります)。
2. ダウンロードしたファイルをこのフォルダに `bgm-kyoto-garden.mp3` というファイル名で配置する。
3. `virtual-museum.html` をブラウザで開き、ヘッダー右上の「SOUND ON/OFF」ボタンでミュート解除し、実際に再生されることを確認する。

## 別候補(同ライセンス、より長尺)

短い1曲のループでは物足りない場合、同じPixabayプラットフォーム上に "Japan Asia Music" by Tunetank(5:40、アンビエント)という候補もあります。個別のライセンス条件はPixabay Content Licenseに準拠しますが、採用する場合は上記と同様にダウンロードページで最終確認のうえご利用ください。
