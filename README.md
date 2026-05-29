# GRKD-Tegaki-Deck

日本語を「見る」だけでなく、実際に手で書いて覚えるための Anki デッキ補助リポジトリです。

Front では読み・音声・短い文脈ヒントを見て、自分の手で漢字を書きます。Back では答えと KanjiVG の筆順アニメーションを見て、書き方を確認します。

## 最新リリース

```text
v0.1.0 — 1段 Preview
```

まず使う場合は、GitHub Releases から `.apkg` をダウンロードして Anki に読み込んでください。

```text
GRKD-Tegaki v0.1.0.apkg
```

このリリースは **1段だけ** です。2段以降はまだ制作中です。

## 何が入っているか

```text
GRKD-Tegaki.txt
grkd_tegaki_1dan_animation_template.md
grkd_tegaki_shodo_desk_template.md
logo_black.svg
logo_white.svg
release/
archive/
```

| パス | 内容 |
|---|---|
| `GRKD-Tegaki.txt` | Anki インポート用のデッキバックアップです。 |
| `grkd_tegaki_shodo_desk_template.md` | 現在おすすめのカードデザインです。Light/Dark 対応。 |
| `grkd_tegaki_1dan_animation_template.md` | 旧版のシンプルな筆順アニメーションテンプレートです。 |
| `logo_black.svg` / `logo_white.svg` | Light/Dark 用のロゴです。現在のテンプレートは GitHub 上のSVGを読み込みます。 |
| `release/` | Anki に入れる更新用 TSV など、実際に使う成果物です。 |
| `archive/` | 作業途中の TSV、レビュー用 TSV、実験ファイルです。普通は読まなくて大丈夫です。 |

## 基本の使い方

### 1. デッキを入れる

一番かんたんな方法は、GitHub Releases から `.apkg` を入れることです。

```text
GRKD-Tegaki v0.1.0.apkg
```

手動で組みたい場合だけ、Anki で `GRKD-Tegaki.txt` をインポートします。

`GRKD-Tegaki.txt` は作業用の元データです。普通は `.apkg` 版を使ってください。

### 2. 1段のヒントを更新する

`release/` にある更新用 TSV をインポートします。

```text
release/GRKD-Tegaki_1dan_context_hint_update_SAMPLE_SHAPE.tsv
```

インポート時はここだけ注意してください。

```text
既存ノートを更新する: ON
既存ノートをスキップ: OFF
追加: 0
更新: 2997
```

追加が出たら、一度止めてください。GUID が合っていない可能性があります。

### 3. カードテンプレートを貼る

`grkd_tegaki_shodo_desk_template.md` の Front / Back / Styling を Anki のカードテンプレートへ貼ります。

1段だけ変えたい場合は、元のノートタイプを直接編集せず、先にノートタイプを複製してください。元ノートタイプを直接変えると、2段・3段・4段にも見た目が広がります。

### 4. ロゴについて

現在のテンプレートは、公開済みGitHub上のSVGロゴを直接読み込みます。

```text
https://raw.githubusercontent.com/GoRakuDo/GRKD-Tegaki-Deck/main/logo_black.svg
https://raw.githubusercontent.com/GoRakuDo/GRKD-Tegaki-Deck/main/logo_white.svg
```

オフラインでもロゴを出したい場合だけ、`logo_black.svg` と `logo_white.svg` を Anki メディアへ入れて、テンプレート内のURLをローカル名へ戻してください。

## デザイン

現在のおすすめは「静かな書道机」デザインです。

派手な学習アプリではなく、読む、思い出す、書く、裏で確認する、という流れを邪魔しない見た目を目指しています。

- Light / Dark 対応
- Anki Desktop / スマホを意識した縦積みレイアウト
- 読みと答えは `UD Digi Kyokasho NK` を優先
- UI文字は `Gen Interface JP` を優先
- 色は OKLCH ベース

## KanjiVG について

このテンプレートは、漢字の筆順表示に KanjiVG を使います。

```text
KanjiVG © Ulrich Apel
License: CC BY-SA 3.0
https://github.com/KanjiVG/kanjivg
https://kanjivg.tagaini.net/
```

テンプレート内の JavaScript は、KanjiVG の SVG を読み込んで筆順アニメーションとして表示します。現時点では GitHub の raw URL から読み込むため、表示にはネット接続が必要です。

オフラインでも安定させたい場合は、将来的に KanjiVG SVG を Anki メディアへ入れるローカル版に切り替える予定です。

## ライセンス

このリポジトリは現時点では **No License** です。

つまり、明示的な利用許諾はまだ出していません。閲覧・個人利用の範囲で置いています。再配布、商用利用、改変版の公開を考えている場合は、ライセンスを決めるまで待ってください。

KanjiVG 由来の筆順データについては、KanjiVG 側の CC BY-SA 3.0 が適用されます。詳しくは `NOTICE.md` を見てください。

## 注意

- このリポジトリは個人学習用の作業物です。
- Anki へ入れる前に、必ず自分のコレクションをバックアップしてください。
- テンプレートや TSV は、まず数枚で表示確認してから本番に使ってください。
- Issue や提案は歓迎ですが、ゆっくり対応します。
