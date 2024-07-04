# 24_yagai_jisshu
##### 2024年度・前期「生物学野外実習」理学部海洋自然科学科
##### https://tiglon.jim.u-ryukyu.ac.jp/portal/Public/Syllabus/SyllabusSearchStart.aspx?lct_year=2024&lct_cd=401194001&je_cd=1
##### @琉大熱生研・西表研究施設

##### 2024年7月5日・琉大周辺で見られるチョウの種多様性の評価

##### 目標：Rパッケージ `vegan` を使って、多様性指数を計算する
##### キーワード：多様性指数・アルファ多様性・ベータ多様性・ガンマ多様性・累積種数曲線・Chao指数

##### 2024年6月3日作成
##### 注）付け焼き刃のため、何かとんでもない間違いをしているかもしれません。すみません。

##### 1. R環境の導入とveganのインストール
今回は`webR`を利用します。

1.1. WebRのデモ版 https://webr.r-wasm.org/latest/ をブラウザで開く。

1.2. `vegan`をインストールする。
```
library(vegan)
```

`vegan`を読み込めず、ダウンロードするか聞かれるので、`Yes`
```
Failed to load package "vegan". Do you want to try downloading it from the webR binary repo? 

1: Yes
2: No

Selection: Yes
Downloading webR package: permute
Downloading webR package: lattice
Downloading webR package: MASS
Downloading webR package: cluster
Downloading webR package: nlme
Downloading webR package: Matrix
Downloading webR package: mgcv
Downloading webR package: vegan
Loading required package: permute
Loading required package: lattice
This is vegan 2.6-6.1
```

これで、`vegan`を使えるようになっているはず。ブラウザを一回閉じると、インストールし直さないといけないので注意。

##### 2. データの読み込み
2.1. webRにデータをアップロードする。
`home > web_user`を選択した状態で、`Upload file`をクリックし、アップロードしたいファイルを選ぶ。

2.2. Rにデータを読み込む。
```
lepi.table <- read.table("demo.txt", header = TRUE)
```
データが `lepi.table` に格納されているはず。

```
head(lepi.table)
```
```
> head(lepi.table)
  環境 区画 観察者 ジャコウアゲハ ベニモンアゲハ ナミアゲハ
1 道路    A     あ              0              2          0
2 道路    A     い              1              0          0
3 道路    B     う              0              0          5
4 道路    C     え              1              0          0
5 圃場    D     あ              0              0          0
6 圃場    E     い              0              0          0
```

1列目が`環境`、2列目が`区画`、3列目が`観察者`、その後にデータが並んでいるはず。

このデータの部分だけをさらに別の`lepi.data`に格納する。

```
# 列数の確認
ncol(lepi.table)
```

```
> ncol(lepi.table)
[1] 79
```

```
# 4列目から最後の列（ここでは79列目）を lepi.data に格納する
lepi.data <- lepi.table[,4:79]
```

2.2. 地点ごとの種数を計算する
```
# データを格納する indices を作る
indices <- lepi.table[,c("環境","区画","観察者")]

# アルファ多様性として、それぞれの地点の種数を計算する
indices$alpha <- specnumber(lepi.data)

# ついでにシンプソンの多様度指数を計算する
indices$simpson <- diversity(lepi.data, "simpson")
```

```
> indices
     環境 区画 観察者 alpha   simpson
1    道路    A     あ     8 0.8264463
2    道路    A     い     9 0.8581315
3    道路    B     う    13 0.8947055
4    道路    C     え    12 0.8934911
5    圃場    D     あ     7 0.7857143
6    圃場    E     い     7 0.7776000
7    圃場    E     う     8 0.7534626
8    圃場    F     え     9 0.8266667
9  実験林    G     あ    11 0.8777778
10 実験林    H     い     5 0.7343750
11 実験林    I     う     5 0.6944444
12 実験林    I     え     7 0.7900000
```

2.3. 環境ごとの種数を計算する
```
specnumber(lepi.data, indices$環境)
```

```
> specnumber(lepi.data, indices$環境)
  圃場 実験林   道路 
    17     16     28 
```

```
#道路
specnumber(colSums(lepi.data[1:4,]))
#圃場
specnumber(colSums(lepi.data[5:8,]))
#実験林
specnumber(colSums(lepi.data[9:12,]))
```


3.1. 環境ごとの推定種数（Chao指数）を計算する
```
specpool(lepi.data, indices$環境)
```

```
> specpool(lepi.data, indices$環境)
       Species   chao  chao.se jack1 jack1.se    jack2     boot  boot.se n
圃場        17  66.50 29.97229  26.0 5.267827 32.00000 20.80078  2.185570 4
実験林      16  65.50 29.96112  25.0 6.538348 31.00000 19.79688  3.133387 4
道路        28 118.75 75.53745  44.5 9.968701 54.83333 35.08594  4.257397 4
```

3.2. 環境ごとの累積種数グラフを描画する。

色パレットの`wesanderson`をインストール

```
library(wesanderson)
```
```
> library(wesanderson)
Failed to load package "wesanderson". Do you want to try downloading it from the webR binary repo? 

1: Yes
2: No

Selection: Yes
Downloading webR package: wesanderson
```

```
#道路
plot(specaccum(lepi.data[1:4,]), col=wes_palette("Darjeeling1")[1], ylim=c(0,30))
par(new=T)
#圃場
plot(specaccum(lepi.data[5:8,]), col=wes_palette("Darjeeling1")[2], ylim=c(0,30), ann=F)
par(new=T)
#実験林
plot(specaccum(lepi.data[9:12,]), col=wes_palette("Darjeeling1")[3], ylim=c(0,30), ann=F)
```

`Save plot`で図をダウンロード

3.3. 全体の種数、推定種数を計算する
```
specnumber(colSums(lepi.data))
```

```
> specnumber(colSums(lepi.data))
[1] 42
```

```
specpool(lepi.data)
```

```
> specpool(lepi.data)
    Species     chao  chao.se    jack1 jack1.se    jack2     boot   boot.se   n
All      42 64.04167 12.27606 63.08333 8.405934 73.91667 51.38597   4.356911 12
```

3.4. 全体の累積種数グラフを描画する。
```
plot(specaccum(lepi.data), col=wes_palette("Darjeeling1")[4])
```

`Save plot`で図をダウンロード

4.1. それぞれの環境のベータ多様性（地点間の違い）を計算する

今回は「ガンマ多様性 = アルファ多様性 + ベータ多様性」とする。

```
#道路
specnumber(colSums(lepi.data[1:4,])) - mean(indices$alpha[1:4])

#圃場
specnumber(colSums(lepi.data[5:8,])) - mean(indices$alpha[5:8])

#実験林
specnumber(colSums(lepi.data[9:12,])) - mean(indices$alpha[9:12])
```

```
> #道路
specnumber(colSums(lepi.data[1:4,])) - mean(indices$alpha[1:4])

#圃場
specnumber(colSums(lepi.data[5:8,])) - mean(indices$alpha[5:8])

#実験林
specnumber(colSums(lepi.data[9:12,])) - mean(indices$alpha[9:12])
[1] 17.5
[1] 9.25
[1] 9
```

4.2. 全体のベータ多様性（環境間の違い）を計算する
```
specnumber(colSums(lepi.data)) - mean(specnumber(lepi.data, indices$環境))
```

```
> specnumber(colSums(lepi.data)) - mean(specnumber(lepi.data, indices$環境))
[1] 21.66667
```






