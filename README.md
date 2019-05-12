# hw4

ISA525700 Computer Vision for Visual Effects<br/>Assignment 4<br/>Team 19
===

## Abstract
本文以ORB作為影像特徵辨識與尋找配對最佳化，輔以SIFT與SURF兩種模型為比較方法，以影像之辨識、旋轉、照明光度與縮放比例等為基礎條件下，進行評估效能，再以SIFT作為 features detector 進行後續無限縮放電腦特效。

Keyword: ORB, SIFT, SURF

## Table of Contents
1. [Introduction](#Introduction)
2. [Method](#Method)
3. [Take a sequence of moving-forward images in NTHU campus](#Take-a-sequence-of-moving-forward-images-in-NTHU-campus)
4. [Show feature extraction and matching results between two images](#Show-feature-extraction-and-matching-results-between-two-images)
5. [implement different feature extrators and compare](#implement-different-feature-extrators-and-compare)
6. [Perform image alignment and generate infinite zooming effect](#Perform-image-alignment-and-generate-infinite-zooming-effect)
7. [Exploit creativity to add some image processing to enhance effect](#Exploit-creativity-to-add-some-image-processing-to-enhance-effect)
8. [Video editor](#Video-editor)
9. [Conclusion](#Conclusion)
10. [Reference](#Reference)

## Introduction
本文第一項實驗係以Open CV在圖片執行特徵影像辨識(feature detectors)，並於其它圖片(Image)中，找到最佳配對(matching)影像特徵；第二項實驗係延續以第一項實驗的有最佳影像特徵配對結果的SIFT detector，用以產生無限縮放電腦視覺效果。本實驗在影像之辨識、旋轉、照明光度與縮放比例等為基礎條件下，進行評估效能。

本實驗提出可視化呈現成果，顯示辨識到影像特徵與其它影像最佳配對，可解決程式上的問題。ORB是目前電腦視覺中，較先進的detector，與SIFT與SURF是否比較不同結果。

我們在NTHU校內拍攝一系列前後移動的影像，在雲端採用GPU運算執行ORB模型，顯示兩個影像間的特徵辨識與配對結果，並且執行影像對齊，與無限縮放視覺效果。

並輔以SIFT與SURF兩種模型為比較方法，比較電腦視覺呈現結果。同時也採用後製軟體作影像處理，以增強效視覺效果，例如模糊或著色等。



## Method
### ORB



- ORB是計算SIFT和SURF很好的替代方案，因為SIFT和SURF已申請專利，且Cost與matching是主要專利。

- ORB基本上是FAST關鍵辨識和Brief descriptor的結合，具備許多改善，以增強效能。首先，它使用FAST搜尋關鍵點，然後應用Harris角點測量來搜尋其中的前N個點。它還使用金字塔方式來產生多尺度特徵。但有一個問題是，FAST不計算方向。那麼影像旋轉不變性方面呢？作者提出了以下改善。

- 計算貼片強度與加權質心，位於中心的角落。從該角點到質心的矢量方向給出方向。為了改善旋轉不變性，用x和y計算矩，其應該在半徑為r的圓形區域中，其中r是此貼片的大小。

- ORB使用簡易descriptor，但Brief在輪換方面表現不佳。因此，ORB所做的是根據關鍵點的方向「引導」Brief。對於位置（xi，yi）處的n個二進制測試的任何影像特徵集，定義2×n矩陣，其包含這些像素的坐標。然後使用貼片的方向θ，找到其旋轉矩陣並旋轉S以獲得轉向（旋轉）版本Sθ。

- ORB將角度離散為2π/ 30（12度）增量，並構建預先計算的簡要模式的查詢表。只要關鍵點方向θ在視圖之間是相同的，將使用正確的點集Sθ來計算其descriptor。

- BRIEF具有一個重要特性，即每個位特徵具有較大的方差，平均值接近0.5。但是一旦它沿著關鍵點方向定向，它就會失去這個屬性並變得更加分散。高差異使得特徵更具辨別力，因為它對輸入有不同的響應。另一個理想的特性是使測試不相關，因為每次測試都會對結果產生影響。為了解決所有這些問題，ORB在所有可能的二進制測試中執行貪婪式搜索，以找到具有高方差和意味著近似0.5的那些，以及不相關的。結果稱為rBRIEF。

- 對於描述符匹配(descriptor matching)，採用改進傳統LSH的多探測LSH。使得ORB比SURF快的多，SIFT和ORB描述符比SURF更好。結論為ORB是用於全景拼接等的低功率設備的不錯選擇。


### SIFT與SURF


- 在SIFT模型中，Lowe此採用高斯差分近似高斯的發普拉斯轉換，尋找尺度空間。SURF更進一步，用Box Filter趨近LoG。如下圖表示，這種近似法最大優點為借助積分影像可輕易運算出Box Filter的卷積，同時亦可針對不同尺度平行完成。此外SURF仰賴Hessian矩陣的尺度和位置的行列式。
 
 ![](https://i.imgur.com/CdstdX2.png)


- 對於方向分配，SURF在水平與垂直方向係採用小波響應，對於大小在6s範圍內的鄰近區域，足夠高斯量使用它。然後繪製下圖中的空間中，透過計算角度60度的滑動定向窗口內所友響應總和來估計主導定向。有趣的是，小波響應可以容易使用積分影像找到任何影像縮放。對於不需要旋轉影像處理，無須找到影像定位，加速影像處理過程。SURF提供Upright-SURF與U-SURF兩項功能，可以提高速度，並且可以達到±15。OpenCV則支援兩者，取決於旗標(flag)與直立(upright)。如果為0，則計算方向。如果為1，則不計算方向並且速度更快。, 

 ![](https://i.imgur.com/QVioVzy.png)


## Take a sequence of moving-forward images in NTHU campus.
![](https://i.imgur.com/1yjPcO1.gif)
    
## Show feature extraction and matching results between two images
### result
![](https://i.imgur.com/7oe0gw0.jpg)

![](https://i.imgur.com/ZPHhIIY.jpg)
### Analyze
可以發現會被偵測到的特徵點是不是在畫面每個位置機會都一樣，ORB detector會有特別喜歡的位置，在偵測校園場景特徵的時候，ORB 喜歡深處的樹木特徵，而在偵測娃娃與仙人掌雕刻時，對仙人長雕刻的幾乎沒有抓取任何特徵(此結果在顯示100條特徵線的時候仍然如此)。


## implement different feature extrators and compare
因本次作業目標是align照片已達成infinty zoom in 效果，所以在比較ORB、SIFT、SURF時選用連續照片的其中一張進行各種轉換。
### result
#### 特徵點分佈圖
ORB:
![](https://i.imgur.com/JF0hTt3.jpg)

SIFT:
![](https://i.imgur.com/YLIO7hQ.jpg)

SURF:
![](https://i.imgur.com/sRSWeha.jpg)

#### translation
|ORB|![](https://i.imgur.com/IIOciY0.jpg)
|---|---|

|SIFT|![](https://i.imgur.com/nwoafFu.jpg)
|---|---|

|SURF|![](https://i.imgur.com/3qZa1Tz.jpg)
|---|---|


#### blur
|ORB|![](https://i.imgur.com/MJGHRTs.jpg)
|---|---|

|SIFT|![](https://i.imgur.com/BhR4KLh.jpg)
|---|---|

|SURF|![](https://i.imgur.com/6iE4HuX.jpg)
|---|---|

#### illumination
|ORB|![](https://i.imgur.com/kAKWYAv.jpg)|
|---|---|

|SIFT|![](https://i.imgur.com/F8I7vj9.jpg)|
|---|---|

|SURF|![](https://i.imgur.com/MMMg2jd.jpg)|
|---|---|

#### scale
|ORB|![](https://i.imgur.com/2Ge7iGX.jpg)
|---|---|

|SIFT|![](https://i.imgur.com/GnoviTb.jpg)
|---|---|

|SURF|![](https://i.imgur.com/8mVHJTP.jpg)
|---|---|

#### rotation
|ORB|![](https://i.imgur.com/X9ikQzu.jpg)
|---|---|

|SIFT|![](https://i.imgur.com/oXCX96g.jpg)
|---|---|

|SURF|![](https://i.imgur.com/tLu1LJD.jpg)
|---|---|

#### afine
|ORB|![](https://i.imgur.com/r9tJXs2.jpg)
|---|---|

|SIFT|![](https://i.imgur.com/4V9NewR.jpg)
|---|---|

|SURF|![](https://i.imgur.com/4N3OVs2.jpg)
|---|---|

### Analyze
#### 性能
排名以最少錯誤比對線為較優，如果錯誤比對線數量相似則以特徵點分布均勻判斷為較優秀，之所以如此排名是因為目標找到最適合製作本次作業要求 infinity zoom in 的detector。  

|method|translation|blur|illumination|scale|rotation|afine|
|---|---|---|---|---|---|---|
|ORB |2nd|3rd|3rd|3rd|3rd|2nd|
|SIFT|1st|1st|2nd|1st|1st|1st|
|SURF|1st|2nd|1st|2nd|2nd|3rd|

#### 時間
樣本較少，所以僅提供大致時間尺度。

|ORB|SIFT|SURF|
|--|--|--|
|100ms|500ms|1000ms|


#### 特徵點數量
以連續照片偵測到的特徵點進行平均

|ORB|SIFT|SURF|
|--|--|--|
|500|33134|52969|

每張照片ORB只有500個特徵點，推測是已經進行過收斂。
### Discussion
- 單存從特徵點的數量上SURF>SIFT>ORB
- 如果考量到需要即時運算的話ORB>SURF>SIFT
- 在我們選用的照片上除了光線變化SURF比對較優秀，其他在不考慮時間的情況下選用SIFT都比較好
- 照片的特徵點不管選用何種方法右側的特徵點比對到的數量都非常少，黑板預期會是很好的特徵點分布位置，結果事實並非如此，這可能導致使用程式進行align遇到問題
## Perform image alignment and generate infinite zooming effect
[![Watch the video](https://i.imgur.com/HM0jYHb.png)](https://youtu.be/sDeTAt3p93k)
圖片大致上依照以下方式進行
- 首先會先抽取每張照片的sift descriptor。 
- 接著會比較兩張照片的descriptor找出matching point。比較方式為，計算第一張照片的所有discriptor與第二張照片所有的discriptor的距離。如果第一近的兩點距離比第二近的兩點距離乘上0.7倍還小，那就是為matching point。
- 接這利用這些matching point計算出transformation matrix H。換言之，找出矩陣H使得$(x_2, y_2)=(x_1,y_1)*H$。其中$(x_1, y_1)$為第一張照片的點，對應到第二張照片 $(x_2,y_2)$ 的點。計算方式為，利用最小平方差以及前一步驟算出的matching point，找出最適合的H。
- 最後再利用Ｈ將image2進行transformation轉成符合image1的位置。
- 下圖為alignment結果，圖1為離相機的圖，圖2為較遠的圖，故最後的拼接圖，及為將圖2做完transform貼到圖1上

| 圖1 | 圖2 | 拼接圖|
| ------ | ----------- |----|
| ![](https://i.imgur.com/8Mbd01A.png)|![](https://i.imgur.com/scQZYM8.png)|![](https://i.imgur.com/K4D8zuY.png)|
| ![](https://i.imgur.com/GWh4m6j.png)| ![](https://i.imgur.com/lQJHN7V.png)| ![](https://i.imgur.com/K8IMKbj.png)|
|![](https://i.imgur.com/SkjQGLo.png) | ![](https://i.imgur.com/vZsQ4g8.png)| ![](https://i.imgur.com/0pg2ayM.png)|

- 影片的部分，上方看到的版本，是兩兩相疊以後，放到IMOVIE加上zoomin效果疊加而成。

- 另外有嘗試過recursive的疊圖，也就是假設有三張image img1, img2, img3，會先將img3疊加到img2，生成新的img2，再將新生成的img2疊加到img1。但該種方式會影響到image本身的feature point，故使用sift抽出的feature來做alignment的效果會極差。下圖為極端的例子之ㄧ
![](https://i.imgur.com/yjv7pto.png)


- 此外我們有發現如果有重複pattern太多的場景，會造成matching 的point不太準確，而這種outliers也理所當然的會造成alignment的不準確，如下圖
![](https://i.imgur.com/BUjcsME.jpg)

| SIFT | SIFT + RANSAC |
| ------ | ----------- |
|![](https://i.imgur.com/9zd1eLj.jpg)|![](https://i.imgur.com/R1PQ3UA.jpg)



- 其解決辦法方式可分為以下兩種:
    - 盡量找有明顯特徵的環境拍攝
    - 使用RANSAC來過濾掉outlier
        > RANSAC概略步驟如下
        > - 隨機從計算出的matching points取樣
        > - 計算出可以fit這些點的model(下圖中的紅色虛線)
        > - 定義一個margin(蝦途中兩條實線範圍)，看看有多少點(inlier，下圖中的藍色點)是在這個model的margin範圍裡面，並以此當坐騎分數
        > ![](https://i.imgur.com/0uyFWCI.png)
        >如此重複上述步驟，找出分數最高的model






## Exploit creativity to add some image processing to enhance effect

該部分基於上方影片，加上blur的效果，使觀看者會有更像看到infinite zoom in。其中以下影片並未新增任何新的素材，其全部重複使用上方素材加上模糊效果。

[![Watch the video](https://i.imgur.com/HM0jYHb.png)](https://youtu.be/--AQbj-amc0)

## Video editor
我們在NTHU校園取景，利用影片尺寸縮放與透明度方法，創造出前進場景效果，影片中的足球也產生3D視覺效果。
![](https://i.imgur.com/wlwbOKu.gif)


另一個在NTHU校園走廊取景，相同利用影片尺寸縮放與透明度方法，創造出後退場景效果。
![](https://i.imgur.com/Mek5Eba.gif)


## Conclusion


- SIFT和SURF抓出的key points較ORB多，但SIFT和SURF執行時間最久。算是一種品質和時間的trade off

- 因為第一部分實驗結果顯示，SIFT抽取feature執行時間雖然較久，但品質最優，因此本次影片使用sift來製作。

- 有重複pattern的場景，會導致feature matching變得較困難，outlier會較多。

- 如果要對有重複pattern的場景進行alignment，可以使用像是RANSAC之類的方法來去除outlier。

## Reference
- https://docs.opencv.org/3.1.0/d1/d89/tutorial_py_orb.html
- https://docs.opencv.org/3.4.0/df/dd2/tutorial_py_surf_intro.html
