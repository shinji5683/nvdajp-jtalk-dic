# nvdajp-jtalk-dic

NVDA 日本語版（nvdajp）専用 JTalk 拡張辞書（NAIST-JDIC + bep-eng.dic + 独自拡張）のビルドレシピおよびバイナリリリース。

この辞書は Open JTalk 系の日本語形態素解析辞書 [NAIST-JDIC](http://naist-jdic.sourceforge.jp/) をベースに、英単語の読み（`bep-eng.dic` 由来の `nvdajp-eng-dic`）、短漢字読み（`nvdajp-tankan-dic`）、および読み・アクセント推定の改善や点訳表記フィールドのための独自エントリ（`nvdajp-custom-dic`）を統合したものです。

## 概要と目的

- **nvdajp の JTalk 音声合成専用**: `bep-eng.dic`（約4万語）を含むため、長年親しまれてきた自然な英単語発音（カタカナ英語読み）を提供します。
- **バイナリ配布（prebuilt）**: CI や手元開発環境での毎回の MeCab コンパイル・辞書ビルドをスキップし、検証済みの辞書バイナリ（`jtalk-dic-v*.zip`）を高速に取得・展開できるようにします。
- **libkuraji-jtalk-dic との関係**: 点訳エンジン [libkuraji](https://github.com/nishimotz/libkuraji) 向けの `libkuraji-jtalk-dic` は BSD 3-Clause のため `bep-eng.dic`（GPL）を含みません。本リポジトリは GPL-2.0-or-later であり、nvdajp（GPL-2.0）専用の辞書アセットです。

## リポジトリの中身

- `src/nvdajp-jtalk-dic/`: 辞書ビルドスクリプト（`make_jdic.py` ほか）、NAIST-JDIC ソース・nvdajp 拡張データ、および `bep-eng.dic`
- `src/mecab-src/`: 辞書コンパイラ `mecab-dict-index` をビルドするための MeCab ソース（Open JTalk フォーク、Windows/MSVC 専用）
- ビルド済みの辞書バイナリ一式（`sys.dic`、`matrix.bin`、`char.bin`、`unk.dic`、`dicrc`、`DIC_VERSION`）は GitHub Releases に `jtalk-dic-<tag>.zip` として公開されます。

## ビルド方法

### 1. mecab-dict-index のビルド（Windows + MSVC が必要）

```console
cd src/mecab-src/src
nmake /f Makefile.mak MACHINE=x64 mecab.lib mecab-dict-index.exe
```

### 2. 辞書のビルド（英語読み辞書の切り替え対応: `bep` / `cmudict`）

ユーザーが好みに応じて「標準カタカナ英語読み（`bep-eng.dic`）」と「英語発音ベースの読み（`CMUdict` + `arpabet_to_kana.py`）」を選択・切り替えできるよう、`--eng-dic {bep,cmudict}` オプション（または環境変数 `NVDAJP_ENG_DIC`）に対応しています。

```console
# 標準カタカナ英語読み（bep-eng.dic、既定）
python src/nvdajp-jtalk-dic/make_jdic.py --eng-dic bep --mecab-dict-index src/mecab-src/src/mecab-dict-index.exe --outdir build/dic

# 英語発音ベースの読み（CMUdict + ARPAbet 変換）
python src/nvdajp-jtalk-dic/make_jdic.py --eng-dic cmudict --mecab-dict-index src/mecab-src/src/mecab-dict-index.exe --outdir build/dic-cmudict
```

生成される出力ディレクトリに `sys.dic`、`matrix.bin`、`char.bin`、`unk.dic`、`dicrc`、`DIC_VERSION` が出力されます。

CSV ソースの検証のみを行う場合（MeCab コンパイル不要）:
```console
python src/nvdajp-jtalk-dic/make_jdic.py --eng-dic bep --validate-only
python src/nvdajp-jtalk-dic/make_jdic.py --eng-dic cmudict --validate-only
```

## ユーザー辞書（独自語彙の追加）

```console
python src/nvdajp-jtalk-dic/build_userdic.py \
  --mecab-dict-index src/mecab-src/src/mecab-dict-index.exe \
  --dic-dir build/dic \
  --csv examples/userdic/sample.csv \
  --outfile build/jtusr.dic
```

## ライセンス

GNU General Public License version 2 or later (GPL-2.0-or-later). 詳細は [LICENSE](LICENSE) を参照してください。

## CI / リリース

- `.github/workflows/validate.yml`: CSV 検証のみを Ubuntu で実行（push / PR ごと）。
- `.github/workflows/build-dic.yml`: Windows ランナー上で mecab-dict-index のビルドから辞書生成までを検証。
- `.github/workflows/release-dic.yml`: `v*` タグを push したときに実行。辞書一式（`jtalk-dic-<tag>.zip` と `.sha256`）および `mecab-dict-index-<tag>.exe`（と `.sha256`）を GitHub Release に自動添付します。
