# はじめに
本稿では、ユーザが自分の好きなポピュラー楽曲を指定すると、それをヒップホップ風に編曲するシステムを実現することで、ヒップホップをより身近な存在にすることを最終目標とします。

また本稿では、編曲手法としてドラムなどの音素材を重畳する手法を提案します。音素材を重畳する際にテンポなどを調整するためにテンポの推定などが必要ですが、推定は誤差が生じます。そこで本稿ではテンポ間のズレに注目し、テンポのずれた音源を作成します。

# 提案手法
1. ポピュラー楽曲のテンポを推定します。
2. ポピュラー楽曲の歌唱部を抽出します。
3. ビート時刻を計測して、歌唱開始直前のビート時刻で分割します。
4. 歌唱部を伸縮することでテンポをドラム音源のものに合わせます。テンポのずれたものも用意します。
5. 歌唱部とドラム音源のタイミングが合うようにドラム音源の前に無音区間を追加します。追加する区間の長さは聴きながら指定します。
6. テンポを変更したポピュラー楽曲の歌唱部ヒップホップのドラム音源を合成します。
<img width="957" alt="スクリーンショット 2024-02-14 14 13 30" src="https://github.com/5419037RuiKitahara/hiphop/assets/72202763/7740a936-66aa-4f9e-ad4a-f8c503eb6702">

# ポピュラー楽曲の歌唱部分とヒップホップのドラム音源を合成しよう
## ポピュラー楽曲とドラム音源の用意
#### 1. ドラム音源、使用したい曲のオーディオファイルがない場合、フリー音源サイトから歌唱を含む曲をダウンロードする

音源サイト<a href="https://cymatics.fm/pages/free-download-vault">Cymatics</a>には、無料でダウンロードできるヒップホップのドラムループ音源が多くあるので色々聴いてみて、好みのドラム音源をダウンロードしましょう。

またポピュラー楽曲は使う音源にこだわりがなければ、<a href="https://dova-s.jp/#google_vignette">フリー音源サイト</a>で好みの歌唱を含む曲をダウンロードしましょう。使用したい歌唱を含む曲がある場合mp3、wavファイル形式のオーディオファイルを用意してください。

#### 2. Google Colaboratoryを開いて準備をする
 
まず初めに、自身のGoogleドライブに使用したい歌唱が含まれる曲、ヒップホップのドラム音源をアップロードしましょう。

新しいGoogle Colaboratoryノートブックを開きましょう。開いたら、以下のコードでGoogleドライブをマウントします。
```ruby
from google.colab import drive
drive.mount('/content/drive')
```

次に、使用するドラム音源のパスとファイル名にあるドラム音源のテンポを入力しましょう。
```ruby
filename_b = "" # @param {type:"string"}
tempo_b =  #@param {type:"integer"}
print(filename_b)
```
また、歌唱を含む曲のパスを入力しましょう。
```ruby
import os
!pip install pydub
import pydub
from pydub import AudioSegment


#@title 曲のパスを指定
filename = "" #@param {type:"string"}

#outname=パスのファイル名
outname = os.path.basename(filename)
outname = outname.replace(".mp3","")
outname = outname.replace(".wav","")

#ファイル名のwavファイルを作成
if '.mp3' in filename:
  sound = pydub.AudioSegment.from_mp3(filename)
  sound.export(outname+".wav", format="wav")
if '.wav' in filename:
  sound = pydub.AudioSegment.from_wav(filename)
  sound.export(outname+".wav", format="wav")
filename = outname+".wav"
filename_s =filename
print(filename)
print(outname)
```

os.path.basename()関数で、ファイル名のみを補完します。

outname.replace()関数で、.mp3、.wavを削除したファイル名を補完します。
pydub.AudioSegment.from_mp3()関数、pydub.AudioSegment.from_wav()関数で音声データを読み込み、export()関数で"ファイル名.wav"のシンプルなwavファイルを作成します。これで長いパス名を省略します。

## 編曲したい曲のテンポを推定

librosaを用いて、テンポの推定を行います。
この時、次の音源分離の際に普通の音源ファイル名のままだとエラーが発生するため、原曲の最初のビート時刻から始まる「new.wav」を作ります。

## 歌唱部分の抽出
Spleeterを用いて音源抽出を行います。

## 歌唱の最初の位置を検出

## 歌唱直前のビート時刻を計測し、分割

## テンポを変更

## ドラムの前に無音区間を追加
