# Tegaki Test 用 Ankiテンプレート

## フィールド

`Word`, `Reading`, `Audio`, `StrokeOrder`, `SourceGuid`

- `SourceGuid`: 元の `JPDB 25k Optimized.txt` のGUIDです。カードには表示せず、元ノートへ戻れるようにするための控えです。

## Front Template

```html
<div id="container">
  <div id="reading">{{Reading}}</div>
  <div id="audio">{{Audio}}</div>
  <div id="writebox">ここに書く</div>
</div>
```

## Back Template

```html
<div id="container">
  {{FrontSide}}
  <hr>
  <div id="word">{{Word}}</div>
  <div id="stroke-order">{{StrokeOrder}}</div>
  <div id="kvg-credit">Stroke order: KanjiVG © Ulrich Apel, CC BY-SA 3.0</div>
</div>
```

## Styling

```css
.card {
  margin: 0;
  background: #111;
  color: #f7f7f7;
  font-family: "UD Digi Kyokasho NK", "Yu Gothic", sans-serif;
  text-align: center;
}

#container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
  padding: 32px 16px;
}

#reading {
  font-size: 56px;
  line-height: 1.2;
}

#audio {
  font-size: 24px;
}

#writebox {
  width: min(82vw, 520px);
  min-height: 220px;
  border: 2px dashed #777;
  border-radius: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #777;
  font-size: 24px;
}

#word {
  font-size: 72px;
  line-height: 1.1;
}

#stroke-order {
  width: min(92vw, 760px);
}

.kvg-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
}

.kvg-kanji {
  width: 180px;
  height: 180px;
  padding: 12px;
  border-radius: 16px;
  background: #f7f7f7;
}

hr {
  width: min(92vw, 760px);
  border: 0;
  border-top: 1px solid #555;
}

#kvg-credit {
  margin-top: 8px;
  color: #777;
  font-size: 11px;
}
```

## インポート手順

### 前提条件

このTSVの音声フィールドは、元の `JPDB 25k Optimized.apkg` に入っている音声ファイルへ依存します。音声も確認したい場合は、先に元APKGをAnkiへ入れておいてください。

1. Ankiで `ファイル > インポート`。
2. `tegaki_test_20.tsv` を選ぶ。
3. ノートタイプが `jpdbfreq-tegaki-test`、デッキが `JPDB 25k Optimized::Tegaki Test` になることを確認。
4. フィールド順を `Word / Reading / Audio / StrokeOrder / SourceGuid` に合わせる。
5. インポート後、上のFront/Back/Stylingをカードテンプレートへ貼る。

## 注意

この試験版はKanjiVGのGitHub上のSVGを直接参照します。つまり、表示確認にはインターネット接続が必要です。全5446件へ進めるときは、SVGをAnkiメディアへローカル保存する形に切り替えるのが安全です。

KanjiVG: Copyright Ulrich Apel, CC BY-SA 3.0
