# Beyond accuracy: evaluating recommender systems by coverage and serendipity

この論文はレコメンデーションの評価指標の一つであるcoverageとserendipityについて、その計算方法を提案したものです。

通常の精度だけを評価すると一部の人気itemばかりにレコメンドが偏ります。
それらは、すでにuserが知っているitemである可能性が高く、レコメンドシステムとして実用的な推薦を行なっているとは言えません。
そこで網羅性（coverage）や意外性（serendipity）についても評価する必要があります。
これらの概念は以下の先行研究に基づいたものです。

```
Herlocker J., Konstan J., Terveen L. and Riedl J. 2004.
Evaluating collaborative filtering recommender systems.
ACM Transactions on Information Systems, 22(1), pp. 5–53.
```





## 論文情報

https://dl.acm.org/citation.cfm?doid=1864708.1864761

現時点（2019/9/30）で被引用数は434です。

---

## ABSTRACT

精度以外のレコメンデーションシステムの実用性を測る指標として、coverageやserendipityがあります。

この研究でははじめに、coverageとserendipityの指標値について議論し、そのトレードオフの関係にも言及します。


## 1. INTRODUCTION

省略します。

## 2. COVERAGE

coverage には２つのコンセプトがあります。

 1. システムが推薦するitemの割合
 2. userに対して推薦するitemの割合

この論文では1をprediction coverage, 2をcatalogue coverage と呼びます。

$$
PredictionCoverage = \frac{|I_p|}{|I|}
$$

- $I_p$ :　システムがレコメンドしうるitemの集合
- $I$ : itemの全集合

単なる価格フィルタ表示システムであれば、5000円以下のitemは全て $I_p$ に含まれます。

$$
WeightedPredictionCoverage = \frac{\sum_{i \in I_p}r(i)}{\sum_{j \in I}r(j)}
$$

- $r(i)$ : item $i$ が提案された時の有用性。この有用性については別に定義が必要です。

$$
CatalogCoverage = \frac{|U_{j=1...N}I_{L}^{j}|}{|I|}
$$

- $I_{L}^{j}$ : 計測期間中に $j$ 回目に推薦した top-n item リスト $L$ に含まれるitem集合。  
- $N$ : 観測されたレコメンデーションの回数

つまり、catalog coverage は一度でも表示されたitemの和集合とitemの全集合の比率です。

$$
WeightedCatalogCoverage = \frac{|U_{j=1...N}I_{L}^{j} \bigcap B^j|}{|U_{j=1...N}B^j|}
$$

- $B^j$ : 有効であると定義されたitem集合。定義は別に必要になります。

## 3. SERENDIPITY

推薦の意外性を計測する上で最もシンプルなのが、原理的な推薦モデル結果との違いを比較するです。

$PM$ (Primitive Model) を原始的な推薦の結果セットとし、 $RS$ (Recommender System)を計測したいレコメンデーションシステムによる推薦結果セットとする。

その意外性は以下のように定義できる。

$$
UNEXP = \frac{RS}{PM}
$$

上記だけでは意外性しか評価しておらず、レコメンデーションの有用性の指標となってはいない。
そこで関数 $u(RS_i)$ を導入する。
$RS_i$ はレコメンデーションシステムによる $i$ 番目の結果。
ユーザーが推薦結果に評価を行いそれが有用と判断されれば $u(RS_i)=1$ と定義する。
有用でないと判断すると $u(RS_i)=0$ となります。
この有用性を考慮すると以下のような指標が定義できます。

$$
SRDP=\frac{\sum_{i=1}^{N}u(RS_i)}{N}
$$

ここで、serendipity(意外性)とnovelty(新規性)の違いについて確認しておきます。
普段アクション映画ばかりを見ているuserに、彼女が知らないアクション映画を推薦することがnoveltyです。
普段アクション映画ばかりを見ているuserに、気に入られるドキュメンタリー映画を推薦することがserendipityです。

つまり、ユーザーが普段利用しているitemと類似性の低いitemを推薦しつつ、userの満足度を高めることがserendipity(意外性)です。

## 4. TRADE OFFS

accuracy(精度), serendipity(意外性), coverage(網羅性)には密接な相関性があります。

一般的には以下のような傾向があります。

- accuracyを高めると、catalog coverage が減少する。
- catalog coverage が上昇しても serendipityが上昇するとは限らない。
- serendipityが上昇すると、catalog coverageは必ず上昇する。
- accuracy と catalog coverage の両方を改善すると、serendipityが減少する。
- serendipityを改善するとaccuracyが悪化する。

serendipityを改善は、accuracyの悪化を引き起こすためuserのシステム利用離脱のリスクを引き起こします。
このリスクを抑制するためには、以下のことが重要となるでしょう。

1. userになぜこのitemが推薦されているのかを説明する
2. 推薦リストの並び替えや複数のリストを平行運用する


## 5. CONCLUSION

省略します
