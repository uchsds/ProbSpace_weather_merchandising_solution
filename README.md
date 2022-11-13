# 【ProbSpace】コンビニ商品の売上予測【_place_solution】

## 目次<br>
[コンペ概要](#コンペ概要)<br>
[結果](#結果)<br>
[作成した特徴量](#作成した特徴量)<br>
分析1：[可読性の高いホワイトボックスモデルの作成(LB 0.890 (Private 0.728))](#可読性の高いホワイトボックスモデルの作成lb-0890-private-0728)<br>
分析2：[LightGBMで精度アップ(LB 0.869 (Private 0.693))](#lightgbmで精度アップlb-0869-private-0693)<br>
分析3：[他の手法でもモデル作成しブレンド(LB 0.864 (Private 0.693))](#他の手法でもモデル作成しブレンドlb-0864-private-0693)<br>
[その他(個人的備忘録)](#その他個人的備忘録)<br>
<br>
## コンペ概要<br>
コンペサイト：https://comp.probspace.com/competitions/weather_merchandising<br>
<br>
気象データをもとに各商品の日別売上個数を予測<br>
評価指標はPinball loss Error (q=(0.01,0.1,0.5,0.9,0.99))<br>
![image](https://user-images.githubusercontent.com/118031932/201456256-98df1a40-3232-4f8f-8e9a-096368add78c.png)<br>
<br>
<br>
## 結果<br>
PublicLB：1位(スコア：0.864)<br>
PrivateLB：1位(スコア：0.693)<br>
<br>
<br>
## 作成した特徴量<br>
分析1用<br>
・year(2018～2019年に設定([この期間にする理由は後述](#alcol13)))<br>
・month<br>
・day<br>
・week<br>
・rain_tomorrow(次の日の雨の値)<br>
分析2,3用<br>
・week_number(月の中で何週目か)<br>
・rain_yesterday(前の日の雨の値)<br>
<br>
<br>
## 可読性の高いホワイトボックスモデルの作成(LB 0.890 (Private 0.728))<br>
コンペでは不要かもしれませんが、できるだけ可読性の高いモデルを作成することを意識して、データをよく観察し、回帰分析をメインに分析しました。<br>
基本的に一つ一つ確認しながら分析しました。ですので、コードにグラフの描画が多めです。<br>
そのままアップロードすると重くて表示されなかったので、実行結果は消しています。<br>
詳細な中身を確認したい場合は、実行してください。<br>
<br>
### ・ice1～3
テストデータのhighestの範囲に限定し、highestに対して単回帰分析<br>
ice1のみ外れ値を除去<br>
<br>
### ・oden1～4
lowest＜10と10≦lowest＜20の2つの範囲で、lowestに対して単回帰モデルを作成<br>
7～9月は販売していないようなので、データから除去<br>
外れ値を除去<br>
外れ値は2倍の値になっているようなので、外れ値除去モデルの99%分位点の2倍の値を99%分位点に変更
![image](https://user-images.githubusercontent.com/118031932/201453985-13d566ea-f0ee-45f4-a7f2-7c2293e707ce.png)<br>
<br>
### ・hot1～3
lowest≦15とlowest＞15の2つの範囲で、lowestに対して単回帰モデルを作成(testデータはすべてlowest≦15)<br>
週依存があるように見えるが、weekをOne-hot Encordingして変数に入れるとLBの値が下がったので、ここでは使用しなかった<br>
![image](https://user-images.githubusercontent.com/118031932/201454060-907f0e03-cdab-4280-adb6-1c1b58043e71.png)<br>
trainデータの予測値を作成(後に使用)<br>
<br>
### ・dessert1～5
week2,6とそれ以外の場合に分け、それぞれの日別の平均値を採用<br>
trainデータの予測値を作成(後に使用)<br>
<br>
### ・drink1,4
highest≧15の範囲で、highest,rain,rain<sup>2</sup>の重回帰分析<br>
highest＜15の範囲で、rain,rain<sup>2</sup>の重回帰分析<br>
highest＜15は下限値が決まっているようなので、下限値以下の値は下限値に置き換え<br>
### ・drink2,3
highest≧15の範囲で、highest,rain,rain<sup>2</sup>の重回帰分析<br>
highest＜15の範囲で、rain,rain<sup>2</sup>の重回帰分析<br>
week4のときに外れ値が多いので、それぞれの重回帰分析を全データ使用とweek4以外使用の2通り作成し、testデータのweek4のみ全データ使用モデルを採用<br>
![image](https://user-images.githubusercontent.com/118031932/201454113-0bda407c-4d1c-4a16-a552-e56cc3db0e8a.png)<br>
highest＜15は下限値が決まっているようなので、下限値以下の値は下限値に置き換え<br>
### ・drink5,6
highest≦10の範囲で、highest,rain,rain<sup>2</sup>の重回帰分析<br>
highest＞10の範囲で、rain,rain<sup>2</sup>の重回帰分析<br>
highest＞10は上限値が決まっているようなので、上限値以上の値は上限値に置き換え<br>
<br>
### ・alcol1～3
week,rainの重回帰分析<br>
※yearを2018～2019年に設定することで、weekとalcolの関係が線形になり、線形重回帰で扱いやすくなる
![image](https://user-images.githubusercontent.com/118031932/201454164-6c3adf82-28ab-4cb8-b46b-42a783f7cc1a.png)<br>
alcol3のみ外れ値を除去<br>
<br>
### ・snack1～3
週別の平均値を採用<br>
trainデータの予測値を作成(後に使用)<br>
<br>
### ・bento1～4
週別の平均値を採用<br>
trainデータの予測値を作成(後に使用)<br>
<br>
### ・tild1,2
手動で下記のような分類を実施<br>
各クラスの平均値を採用<br>
trainデータの予測値を作成(後に使用)<br>
![image](https://user-images.githubusercontent.com/118031932/201453788-896c01a6-fa16-4b01-9ccb-e95970231420.png)<br>
<br>
### ・men1～6
手動で下記のような分類を実施<br>
各クラスの平均値を採用<br>
trainデータの予測値を作成(後に使用)<br>
![image](https://user-images.githubusercontent.com/118031932/201453866-a94f31d6-c5e9-42c7-8bbf-55fa7df5c4f3.png)<br>
<br>
<br>
## LightGBMで精度アップ(LB 0.869 (Private 0.693))<br>
hotの週依存が見えない部分が気になったのと、平均値を採用しているターゲットの予測精度に不安があったので、<br>
これらのターゲットに関して前の予測値も特徴量に入れてLightGBMを実施しました。<br>
やや可読性は落ちますが、精度アップのために実施しました。<br>
<br>
### ・dessert1～5、snack1～3、bento1～4、tild1,2、men1～6
highest, lowest, rain, month, day, week, week_number, rain_yesterday, rain_tomorrow, 各予測値　を使用してLightGBM<br>
dessertは大きい値の予測範囲が広くなっているため、前の予測との平均値を使用(逆に前の予測は全体的に予測範囲が狭い)<br>
<br>
### ・hot1～3
week,各予測値　を使用してLightGBM<br>
<br>
<br>
## 他の手法でもモデル作成しブレンド(LB 0.864 (Private 0.693))<br>
LightGBMと同じ特徴量でXGBoostとRandam Forestも試してみました。<br>
<br>
### それぞれのスコア<br>
LightGBM(max_depth=10)：0.869(Private 0.693)　※最終提出①<br>
LightGBM(max_depth=50)：0.874(Private 0.691)<br>
XG Boost：0.877(Private 0.710)<br>
Randam Forest：0.887(Private 0.732)<br>
<br>
### これらをブレンド(Averaging)したスコア<br>
LightGBM(max_depth=10) + LightGBM(max_depth=50) + XG Boost + Randam Forest：0.867(Private 0.692)<br>
LightGBM(max_depth=10) + XG Boost + Randam Forest：0.866(Private 0.693)<br>
LightGBM(max_depth=50) + XG Boost + Randam Forest：0.864(Private 0.693)　※最終提出②<br>
<br>
<br>
## その他(個人的備忘録)<br>
### ・なぜこのコンペに参加したか
今年の5月ぐらいから、統計やデータサイエンスの勉強を始めて、統計検定2級、データサイエンティスト検定、G検定を取得しました。<br>
もう少し実践的なことをしてみたいと思っていたところ、データ分析コンペというものがあることを知り、挑戦してみることにしました。<br>
初めての挑戦なので、取っつきやすいテーブルデータ分析が良いと思い、こちらのコンペに参加させていただきました。<br>
いろいろ勉強することが多くて大変かなあと思っていましたが、案外オンラインゲームをしているような感覚で楽しめて取り組めました。<br>
Pythonも初学者なので勉強しながら進めました。ですので、汚いコードになっていると思います。そこは、ご容赦ください。<br>
<br>
### ・どういう感じで進めていったか
最初、重要な特徴量を探して、カーネルトリックを使ったRidgeやLasso、SVRなどで非線形回帰していました。<br>
非線形回帰した後は、LightGBM・XGBoost・RandamForestを使って予測を作成し、それらをスタッキングしてある程度精度を伸ばして、3位ぐらいまでいきました。<br>
このときのスコアは0.965だったと思います。<br>
これ以上手がなかったので、放置していました。<br>
<br>
残り5日ぐらいのタイミングで久しぶりに確認してみると、5位ぐらいまで順位が落ちていて、0.9以下のスコアも出始めていたので、<br>
何か別の方法があるのだろうと思い、もう一度１からモデル作成を始めました。<br>
もう一度データをよく見て、解析範囲を絞った単回帰・重回帰などを取り入れ始めました。<br>
一気にスコアが上がったのは、odenの外れ値の傾向に気づいたところからだったと思います。<br>
<br>
再作成したモデルに関しても、締め切り前日ぐらいからモデルのブレンドを試しました。<br>
個々のモデルのスコアはあまりよくなかったですが、ブレンドするとスコアが上がりました。<br>
Privateの値見るとあまり意味はなかったようですが・・・
<br>
### ・気づき
EDAは大事だなあと感じたので、今後は最初からデータをよく理解し、精度の高いモデルを早めに仕上げたうえで、<br>
パラメーターのチューニングやスタッキングなどの時間をしっかり確保したいと思いました。<br>
またスタッキングが強力であることも体感したので、コンペ序盤は多様なモデル作成をするのが良いと思いました。<br>
<br>
今回この辺りに気づいたのが、締め切り直前だったので、ほとんど取り組めませんでした。少し心残りです。<br>
