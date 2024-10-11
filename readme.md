# LaTeX-OCR Wrapperツール

## 概要
NDL OCRのWrapperツール(またはEasyOCR Wrapper)により出力されたレイアウト情報を読み取り、数式領域の文字認識を一括で行います。

数式認識はLaTeX-OCR(https://github.com/lukas-blecher/LaTeX-OCR)を用います。


## ソフトウェア構成
本ツールは以下のようなスクリプト群からなります。
* proc_latexocr.py
    * NDL OCRのWapperプログラム(/docer/run_wrapper.sh)の結果を受け取り、数式領域に対して文字認識をかけてLaTeXフォーマットへ変換します。
    * 処理結果はJSONファイルに格納されます。
* read_ndlocr_xml.py
    * NDL OCRの出力JSONファイルを読み込み、LaTeX-OCRの結果を同じフォーマットのJSONへ格納します。
    * proc_easyocr.pyの中で呼び出されます。


## セットアップ方法

git cloneでコードをダウンロードし、シェルスクリプト(dockerbuild.sh)でDockerコンテナを生成します。

```
$ git clone https://github.com/takmin/latexocr_wrapper.git
$ cd latexocr_wrapper
$ sh ./docker/dockerbuild.sh
```


## 使い方

docker/run_docker.shを用いることで、Dockerコンテナの立ち上げ、数式認識の実行、コンテナの解放という流れを入力フォルダの中の画像群に対し一度に行うことができます。
NDL OCRで処理した結果を入力として、数式ブロックに対して更にLaTeX-OCRで数式認識を行います。

```
sh ./docker/run_docker.sh <input_dir> <json_dir> <output_dir>
```
* input_dir
    * 画像の格納されたディレクトリ。
* json_dir
    * NDL OCR Wrapperから出力された認識結果jsonが格納されたディレクトリです。
* output_dir
    * LaTeX-OCRの出力先ディレクトリを指定します。ここに<json_dir>内のjsonファイルに図および表内のOCR結果を加えたjsonファイルが出力されます。


### proc_latexocr.py
docker/run_docker.shは、Dockerコンテナを起動後、コンテナ内部でこのスクリプトを起動しています。
```
python3 proc_latexocr.py <input_dir> <json_dir> <output_dir>
```
* input_dir
    * 画像の格納されたディレクトリ。
* json_dir
    * NDL OCR Wrapperから出力された認識結果jsonが格納されたディレクトリです。
* output_dir
    * EasyOCRの出力先ディレクトリを指定します。ここに<json_dir>内のjsonファイルに図および表内のOCR結果を加えたjsonファイルが出力されます。



## JSONファイルのフォーマット
本プログラムが出力するJSONファイルは以下のようなフォーマットとなります。

```
{
  “width”:  1332,
  “height”:  1776,
  "LINES": [
    {
        "type": 本文,
        "x": 20,
        "y": 140,
        "width": 460,
        "height": 10,
        "confidence": 0.9,
        "string": "こんにちは、はろー、さようなら" 
    }
  ],
  "BLOCKS": [
    {
        "type":  “テキスト”,
        "x": 20,
        "y": 20,
        "width": 460,
        "height": 500,
        "confidence": 0.8,
        "LINES": [
           # 上記LINESと同じ構造
         ]
    },
    {
       "type":  “表組”,
        "x": 20,
        "y": 20,
        "width": 460,
        "height": 500,
        "confidence": 0.8,
        "LINES": [
         ]
    },
    {
       “type”:  “図版”,
        "x": 20,
        "y": 20,
        "width": 460,
        "height": 500,
        "confidence": 0.8,
        "LINES": [
           #図の中の文字列
         ]
    },
    {
　　 　"type":  “数式”,
        "x": 20,
        "y": 20,
        "width": 460,
        "height": 500,
        "confidence": 0.8,
        "LINES": [
           #LaTeXの数式表現文字列
         ]
    }
  ]
}  

```
このようにレイアウトのブロックを抽出して"BLOCKS"内に配列として格納し、そのブロックのカテゴリを"type"で指定しています。
また、ブロック内に含まれる文字列は"LINES"という配列に格納されます。この"LINES"の要素も"type"属性を持ちますが、この時EasyOCRで認識された文字列には"EasyOCR"というtypeがつけられます。


## 処理の流れ

proc_latexocr.pyはNDL OCR Wrapper（またはEasyOCR Wrapper）の出力を読み取って、そこか数式領域を抽出して文字認識を行い、LaTeX形式で格納します。LaTeX-OCRは文字列検出の機能はないため、その部分はproc_ndlocrの結果を利用します。
1. 画像ファイルをロード
2. 対応するjsonファイルをロード
3. jsonから数式BLOCKを抽出
4. 全LINEを抽出し、数式BLOCKとのオーバーラップを判定
5. オーバーラップしたLINEに高さ方向にマージンを取った領域で画像を切り取り
6. 切り取った画像に対してLaTeX-OCRをかける
7. 認識結果をjsonへ再格納


## 変更履歴
* 2024.10.11 First Version
    * 作成者：株式会社ビジョン＆ITラボ　皆川卓也
