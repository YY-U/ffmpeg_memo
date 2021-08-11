# ffmpegメモ

## 動画ダウンロード and 編集手順
→動画の指定の部分を抜き出し
→それを連番画像変換
→連番画像を一括リサイズ

## ffmpeg基本構造
ffmpeg 入力オプション -i 入力ファイル名 出力オプション 出力ファイル名

## 動画ダウンロード
・通常
```
youtube-dl YouTubeの動画URL
```
・最高画質、音質でダウンロード(元動画が mp4 形式でダウンロードできる場合)
```
youtube-dl YouTubeの動画URL -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]"
```
### 参考
https://knooto.info/youtube-dl/#%E5%8B%95%E7%94%BB%E3%82%92%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BC%E3%83%89%E3%81%99%E3%82%8B

## 動画の指定の部分を抜き出し
・秒数指定(ex. 0:30から1分間分を抜き出し)
```
ffmpeg.exe -ss 0:30 -i input.mp4 -t 1:00 -vcodec copy -acodec copy output.mp4
```
・区間指定(ex. 0:30～1:30を抜き出し)
```
ffmpeg.exe -i input.mp4 -ss 0:30 -to 1:30 -vcodec copy -acodec copy output.mp4
```
・開始時間とフレーム数で切り出し(ex. 0秒から100フレーム切り出し)
```
ffmpeg.exe -ss 0 -i input.mp4 -vcodec copy -acodec copy -vframes 100 output.mp4
```

### 参考
https://moewe-net.com/uncategorized/ffmpeg-cut-movie
https://nico-lab.net/cutting_ffmpeg/


## フレームレート変換
```
ffmpeg -i input.mkv -vf fps=30 -vcodec utvideo output.mkv
```
オリジナルよりフレームレートが少なければ間引く、多ければ重複フレームをつくる。
動画時間は変わらない


## 動画 → 連番画像
・動画1秒あたりフレームレートの枚数分画像が生成
```
ffmpeg -i input.mp4 image%04d.png
```
```
ffmpeg -i input.mp4 _high_%04d.png
```

## 動画から1秒あたり60枚の連番ファイルをつくる場合
```
ffmpeg -i input.mp4 -r 60 image%04d.png
```
学習データの場合はcol_high_0000.pngからスタート

### 参考
https://qiita.com/cha84rakanal/items/e84fe4eb6fbe2ae13fd8


## 複数画像一括リサイズ方法
```
ffmpeg -i image%04d.png -s 480x270 image%04d.png
```
```
ffmpeg -i col_high_%04d.png -s 480x270 _high_%04d.png
```

## 一括名前変更
```
ffmpeg -i input%04d.jpg output%04d.png
```
```
ffmpeg -i %d.jpg output%04d.png
```
```
ffmpeg -i %d.png output%04d.png
```

## 連番画像を0(0000.png)からにする
```
ffmpeg -i video.mp4 -start_number 0 output%d.png
```
・0からではないpng連番画像を0からのjpg連番画像に変換する
(ex. 59からの連番画像を0からにする場合)
```
ffmpeg -start_number 0059 -i image%04d.png -start_number 0000 image%04d.png
```
```
ffmpeg -start_number 0120 -i col_high_%04d.png -start_number 0000 col_high_%04d.png
```
```
ffmpeg -start_number 0181 -i image%04d.png -start_number 0000 image%04d.png
```
```
ffmpeg -i video.mp4 -start_number 0 col_high_%04d.png
```

### 参考
https://kakashibata.hatenablog.jp/entry/2018/11/25/155437


## 連番画像 → 動画化
```
ffmpeg -i image%04d.png LR.mp4
```

```
ffmpeg -i col_high_%04d.png LR.mp4
```

```
ffmpeg -framerate 30 -start_number 101 -i image_%03d.png -vframes 600 -vcodec libx264 -pix_fmt yuv420p -r 60 out.mp4
```
一つ目の"-framerate 30fps"で連番画像のフレームレート指定
二つ目の"-framerate 60fps"で動画化する際のフレームレート指定
"-vframes <number>" "number"で指定したフレーム数だけ変換
 
```
ffmpeg -framerate 60 -start_number 0001 -i image%04d.png -vframes 600 -vcodec libx264 -pix_fmt yuv420p -r 60 out.mp4
```
```
ffmpeg -framerate 60 -start_number 0176 -i _high_%04d.png -vframes 411 -vcodec libx264 -pix_fmt yuv420p -r 30 out.mp4
```
