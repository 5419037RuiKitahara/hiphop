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
<img width="948" alt="スクリーンショット 2024-02-15 12 59 31" src="https://github.com/5419037RuiKitahara/hiphop/assets/72202763/e6d77e2a-3958-4842-bbfd-ca928dc19438">


# ポピュラー楽曲の歌唱部分とヒップホップのドラム音源を合成しよう
## ポピュラー楽曲とドラム音源の用意
#### 1. ドラム音源、使用したい曲のオーディオファイルがない場合、フリー音源サイトから歌唱を含む曲をダウンロードする

音源サイト<a href="https://cymatics.fm/pages/free-download-vault">Cymatics</a>には、無料でダウンロードできるヒップホップのドラムループ音源が多くあるので色々聴いてみて、好みのドラム音源をダウンロードしましょう。

またポピュラー楽曲は使う音源にこだわりがなければ、<a href="https://dova-s.jp/#google_vignette">フリー音源サイト</a>で好みの歌唱を含む曲をダウンロードしましょう。使用したい歌唱を含む曲がある場合mp3、wavファイル形式のオーディオファイルを用意してください。

#### 2. Google Colaboratoryを開いて準備をする
 
まず初めに、自身のGoogleドライブに使用したい歌唱が含まれる曲、ヒップホップのドラム音源をアップロードしましょう。

新しいGoogle Colaboratoryノートブックを開きましょう。開いたら、以下のコードでGoogleドライブをマウントします。
```py
from google.colab import drive
drive.mount('/content/drive')
```

次に、使用するドラム音源のパスとファイル名にあるドラム音源のテンポを入力しましょう。
```py
filename_b = "" # @param {type:"string"}
tempo_b =  #@param {type:"integer"}
print(filename_b)
```
また、歌唱を含む曲のパスを入力しましょう。
```py
import os
!pip install pydub
import pydub
from pydub import AudioSegment


#曲のパスを指定
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

librosaを用いて、テンポ、ビート時刻の推定を行います。
```py
import librosa
def tempo_estimate(filename):
  y, sr = librosa.load(filename)
  tempo, beat_frames = librosa.beat.beat_track(y=y, sr=sr)
  return tempo
```
```py
def beat_estimate(filename):
  y, sr = librosa.load(filename)
  tempo, beat_frames = librosa.beat.beat_track(y=y, sr=sr)
  beat_times = librosa.frames_to_time(beat_frames, sr=sr)
  return beat_times
```
この時、次の音源分離の際に普通の音源ファイル名のままだとエラーが発生するため、原曲の最初のビート時刻から始まる「new.wav」を作ります。
```py
tempo = tempo_estimate(filename)
beat_times = beat_estimate(filename_s)
if tempo>170:
  tempo =tempo/2
sound = AudioSegment.from_file(filename)
sound1 = sound[beat_times[0]:]
sound1.export("new.wav", format="wav")
filename = "new.wav"
print(tempo)
```
## 歌唱部分の抽出
Spleeterを用いて音源抽出を行います。
```py

!pip install spleeter

def sound_separation(filename):
  #Spleeterを使用し歌声部分とその他を分離
  !spleeter separate -h
  !spleeter separate -o output/ {filename}
```
このままだとファイル名がnewのままなので名前を変えます。
```py
sound_separation("new.wav")
os.rename("/content/output/new", "/content/output/"+outname)
```
## 歌唱の最初の位置を検出
```py
!pip install inaSpeechSegmenter
!pip install pydub
from inaSpeechSegmenter import Segmenter

def vocal_detection(filename):
  seg = Segmenter(vad_engine='smn', detect_gender=False)
  # 区間検出実行（たったこれだけでOK）
  segmentation = seg(filename)
  speech_segment_index = 0
  start_time = 0
  for segment in segmentation:
    segment_label = segment[0]
    i = 0
    if (segment_label == 'speech'):  # 音声区間
        start_time = segment[1] * 1000
        print(start_time)
        break
  return start_time
```
音声区間検出ライブラリinaSpeechSegmenterを用いて音声区間の最初の位置を検出します。
```py
start = vocal_detection("/content/output/"+outname+"/vocals.wav")
```
## 歌唱直前のビート時刻を計測し、分割
beat_framesをlibrosa.frames_to_time()関数で時間に直します。
```py
#歌声直前のビート時刻計測
beat_times = beat_estimate(filename_s)
beat_start = 0
i = 0
for b in beat_times:
  b = beat_times*1000
  if b[0] > start:
    beat_start = b[0]
  if b[i]<=start :
    beat_start = b[i]
  i+=1
print(beat_start)

# 歌声の直前までの無音を消去


sound = AudioSegment.from_file(voice_name)
sound1 = sound[beat_start:] #歌声部分[開始,終了]
sound1.export("voice1.wav", format="wav")
voice_name_c="voice1.wav"
```

ビート時刻から歌声直前のビート時刻を探し、歌唱をその位置から開始するようにします。
## テンポを変更
```py
# テンポを変更

import soundfile as sf

def tempo_change(filename):
  y, sr = librosa.load(filename)
  tempo=tempo_estimate(filename_s)
  y_new  = librosa.effects.time_stretch(y, rate=tempo_b/tempo)
  sf.write("tc.wav", y_new, sr, subtype="PCM_24")
```
## ドラムの前に無音区間を追加
```py
#ドラムパターンに無音部分を追加
def silent_plus(duration,filename):
  a = AudioSegment.silent(duration=duration)
  b = AudioSegment.from_file(filename)
  b = b*30
  c = a + b
  c.export("drum.wav",format="wav")
```
## ドラムと歌唱を合成
```py
silent_plus(0,filename_b)
# 歌唱とドラムを合成
sound1 = AudioSegment.from_file("drum.wav") #ドラム
sound2 = AudioSegment.from_file("tc.wav")#歌声
output_last =sound1.overlay(sound2,position=0)
# save the result
output_last.export(outname+"_mixed.wav", format="wav")
```

# おわりに
librosaのテンポが合っていないと、歌唱とドラムのタイミングを合わせるのは難しいと思うので、<a href="https://vocalremover.org/ja/key-bpm-finder">外部webサイト</a>で実際のテンポを調べると合わせやすいです。
