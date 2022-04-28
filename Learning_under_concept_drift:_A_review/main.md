
# Concept Drift の概要（2019年のサーベイ論文の要約から）

---
# concept drift とは
学習データが日々追加されモデルを学習更新していく場合、ある時点で状況が変化し、予測精度が悪化する可能性があります。この状況変化が **concept drift** です。

実務ではこれを対処するため、concept drift を検知し、直近のデータのみを学習するなどして、精度の劣化に対処することになります。この検知は cocept drift detection と呼ばれ、cocept drift の研究分野では中心的な課題テーマとなっています。

この記事では、concept drift のサーベイ論文である以下の文献に基づいて、簡単に concept drift について整理していきたいと思います。

### 参考にしたサーベイ論文

```
[1] Lu, Jie, et al. "Learning under concept drift: A review." IEEE Transactions on Knowledge and Data Engineering 31.12 (2018): 2346-2363.

URL: https://ieeexplore.ieee.org/abstract/document/8496795/
引用数: 391 @2022/04/24
```

---
# Concept Drift とは

[1]では、以下の場合に、時点$t$にてconcept drift が発生した、と定義しています。

$$
P_t(X,y) \neq P_{t+1}(X,y)
$$

これは、説明変数ベクトル$X$と目的変数$y$の同時確率分布$P$が、時点$t$で変化したことを表しています。
この条件式から、$X$から$y$を予測するモデルが変化したと想像できます。しかし、この条件式はモデルの変化だけを表しているわけではありません。それは以下の同時確率分布の展開から確認できます。

$$
P_t(X,y) = P_t(X) \times P_t(y|X)
$$

上の式の右辺、$P_t(y|X)$が$X$から$y$を予測するモデルです。
つまり、concept drift はモデルの変化だけでなく、$P_t(X)$が変化しても発生します。$P_t(X)$は説明変数の分布が変化しているので、例えば、異なる母集団からデータを抽出した場合は変化します。現実では、$P_t(y|X)$と$P_t(X)$は同時に変化し、concept drift を引き起こすこともあるでしょう。

この厳密な concept drift の定義は、以降の concept drift detection の手法を理解するのに必ずしも必要ではありません。しかし、予測の精度悪化の要因を分解して理解しておくことは重要です。(後、ちょっとカッコいい)


# concept drift detection の処理プロセス

concept drift の検知は以下の4つの処理ステージをつなぎ合わせたものです。

- **Stage1**: 
    concept drift が発生したか検証したい時点$t$を決め、過去データと未来データに分離する。（データを前後に分ける）
- **Stage2**(必須ではない): 
    過去データと未来データを別々に学習して、2つの予測モデルを得る。
- **Stage3**: 
    2つの予測モデル、あるいは過去データと未来データを比較し、その非類似度を計算する。
- **Stage4**: 
    非類似度が有意かを検定し、concept drift が発生したかを判定する。
  
この4つの分類を[1]は以下の図にまとめました。

![fig5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/429933/791f4380-36ca-0708-ba3a-b62de65fc29d.png)


[1]はこの4つの処理は共通しているとした上で、concept drift detection の手法を以下の3つに分類しました。

- **Error Rate-Based Drift Detection**:
    予測誤差を計測していき、時点$t$で誤差が統計的に有意な閾値を上回った場合に concept drift を検知する。
- **Data Distribution-Based Drift Detection**:
    過去データと未来データの分布を比較し、その非類似度を計算します。非類似度が統計的に有意であれば concept drift を検知する。
- **Multiple Hypothesis Test Drift Detection**:
    複数のconcept drift detectionの手法を組み合わせて、concept driftの検出精度や効率を改善する。

以降では、Error Rate-Based Drift Detection と Data Distribution-Based Drift Detection の代表的な手法を一つずつ紹介します。

## DDM: Error Rate-Based Drift Detectionの代表的な手法

DDM(Drfit Detection Method)は、初期の concept drift detection の手法であり、その後、多くの手法のベースとなりました。

この手法の処理については、以下の図を作成しました。まずは以下を確認してください。

![DDM.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/429933/8120367b-1c62-5377-0c6b-7111184f9fec.png)


DDMは$t$を順番に処理して、予測モデルの誤差率$p_t$を更新していきます。この際、過去の更新から最も小さかった誤差率を$p_{min}$として保持しておきます。そして、時点$t$で更新された誤差率$p_t$が、最小誤差率$p_{min}$から最小標準偏差の3倍以上も大きかった場合、時点$t$で concept drift が発生したと判断します。

これがDDMの処理概要ですが、もう少し補足しておきます。
DDMでは、$p_t$が$p_{min}$から最小標準偏差の2倍以上も大きかった場合も警告としてフラグ管理します。ここでは、この警告が発生した時点を$t_{2}$とします。その後、DDMがconcept driftが発生したと判断した時点を$t_3$とします。
この場合、concept drift は$t_2$から徐々に始まり、$t_3$で確認されたと考えることもできます。よって、実務では時点$t_3$以降は、$t_2$から$t_3$のデータで学習した予測モデルに切り替えることを検討することになります。

## RD: Data Distribution-Based Drift Detection の代表手法
Data Distribution-Based Drift Detectionでは、過去データと未来データの比較してその非類似度を計算します。非類似度が高ければconcept driftが発生した、と見なします。この時、2つのデータの分離方法にもいくつかのパターンがあり、データごとに判断すべきでしょう。

![fig8.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/429933/db8ef0f8-f3f0-17ed-558a-b910f41d643a.png)


RD(Relativized Discrepancy)は、過去のデータセットの分布($P_1$)と未来のデータセットの分布($P_2$)を比較し、その距離を互いの偏差の総和($TV$:Total Variation)を以下のように計算します。なお、式中の$sup$は集合の上限値を選択する演算です。

$$
TV(P_1, P_2) = 2 {sup}_{X} |P_1(x)-P_2(x)|
$$

この2つの分布の距離$TV$が有意に大きくなった場合、concept driftが発生した、と判定します。
この有意な判定方法については複雑なので、原著([リンク](https://dl.acm.org/doi/pdf/10.5555/1316689.1316707))をご確認ください。


# concept drift を分析する上で注目すべきポイント

[1]はconcept drift detectionの結果を分析する上で、注目すべき3つのポイントを上げています。

- **When**: The time of Concept Drift Occurs.
    concept driftが発生した時点$t$の検出です。多くのconcept drift detection手法がこれを直接の検出対象としています。また、concept driftの移行期間$[t_1, t_2]$の検出も重要です。
- **How**: The Severity of Concept Drift. 
    concept drift の深刻度を分析することです。今まで紹介してきた手法の誤差や分布間距離の大きさは発生したconcept driftの深刻度を表しているとも言えます。
- **Where**: The Drift Regions of Concept Drift.
    例えば時点$t$でconcept driftが発生し、$y$(={1 or 0})の判別モデルが$f_t(x)$から$f_{t+1}(x)$に変ったとします。この時、$f_t(x)$だったら1にラベリングされたが$f_{t+1}(x)$なら0にラベリングされてしまうサンプルが出現します。このようにモデルが変わったことで予測が大きく異ったサンプルは、どのようなconcept drift が発生したのかを分析する上で重要です。

上記の3点を丁寧に分析することで、どのようなconcept driftが発生したのかを分析できるかもしれません。


# concept drift detection　と予測モデルの変更/更新

concept drift を検知した後に、どのように既存の学習モデルを更新するか、その戦略についてまとめます。[1]は以下の更新戦略をあげています。

- 単純な再トレーニング(simple retraining)
- アンサンブル再トレーニング(ensemble retraining)
- モデル調整(model adjusting)

## 単純な再トレーニング 

concept drift を検知した時点で、新しいデータで既存のモデルを再学習して更新するアプローチです。

再トレーニングするデータを選ぶために、window strategy がしばし用いられます。ここでの window はデータの期間$[t_1, t_2]$を表します。window strategy の目的は再学習すべき適切な window を選ぶことです。

例えば、Paried Learners [2] では、このwindow strategyに従った再トレーニングを提案しています。
stable learner と reactive learning の2つの学習モデルを用意し、予測には stable learner を用います。stable learner が誤分類し、一方でreactive learner が正しく分類したインスタンスが増えれば、reactive learner を新しいstable learner に変更します。

window strategy の課題として、windowの長さ($t_2 - t_1$)の適切な設定があります。
短いほど直近の傾向に対応しやすいですが、過学習の危険性も高まります。この課題に取り組んだのが ADMIN [3] で、適切なwindowの選択するアルゴリズムです。ADMINでは分析者が事前にwindowの長さを設定する必要はなく、windowの取りうるすべてのカットを検証し、sub-window間の変化率に応じて、sub-windowの長さを最適化します。 

## アンサンブル再トレーニングの例

Dynamic Weighted Majority (DWM) [5] は重み付き投票ルールを concept drift に対応したアンサンブル手法の一つです。
DWMではアンサンブル分類器が誤分類すると、それに対応するために新しい分類器を追加します。また各予測モデルが誤分類した場合にその重みを減らし、その重みが一定以下になるとアンサンブルから削除します。

DWMの他にも、予測誤差率に応じて重みの調整幅を最適化するなど、様々な拡張や提案が行われています。

## モデル再調整の例

同じアルゴリズムのモデルを新しいデータで再トレーニングするのではなく、アルゴリズムの内部にconcept driftを検知するプロセスを統合するアプローチもあります。

DELM [4] はELMアルゴリズムを拡張し、隠れ層のノード数を調整することで、concept driftに対応します。DELMでは分類誤り率が増加すると、その変化を学習するためにノードが追加されるように設計されています。

この他の研究でも、忘却パラメータの導入など、様々なELMアルゴリズム拡張が提案されている。


# concept drift 検知アルゴリズムの評価方法

[1]は以下の3つのタイプの評価指標をあげています。

- holdout
    検証に用いる時点$t$のデータインスタンスが属しているconcept driftが教師ラベルとして既知であるなら、アルゴリズムが正確に集合分けをする性能を検証できます。
- prequential
    ストリーミング処理を意識した評価方法で、誤差指標として prequential error ($\sum_{t}f(\hat{y}_t, y_t)$)を用います。
- controlled permutation
    複数のテストセットを局所分布を保持するように並び替えます。この時、並び替えた結果が時間順になっている良いと考えます。これは時間的に近いインスタンスは同じconcept driftに属していることを前提とした評価方法です。

この他にも、concept driftの発生を示す教師ラベルがない場合に使われる、2つの異なるモデルによる予測誤差を比較する指標もあります。例えば、学習するデータのwindowが異なる2つのモデルの誤差を比較することで、concept driftの発生を推定することができます。

# 合成データセット

concept drift の研究では、複数の論文で用いられる検証用の合成データが公開されています。そのデータについては、論文[1]のTable3に整理されていますので、原著をご確認ください。

# 研究用に公開されている現実データ

同様に、研究用に公開された現実データもあります。論文[1]のTable4に整理されていますので、原著をご確認ください。

# 他の研究分野における concept drift の問題

## Imbalance（不均衡）データの分野では...

不均衡データ問題とは、例えば「ガンの検査問題では、学習データの99%は陰性なので、対策せずに学習するとすべてのサンプルを陰性と予測してしまう」といった不都合が発生します。
concept drift はストリーミングデータを学習しますので、学習の初期ではデータ量が不十分で不均衡データを学習してしまう可能性があります。

これに対応するために、不均衡データ問題とconcept driftの両方に対応するための研究がいくつかなされています。

### Big data mining の分野では...

例えば、秒単位でストリーミングデータを処理し、concept drift を検知しなければならない場合、Big data mining の研究が発展させてきた分散サーバー処理に対応した検知アルゴリズムが必要になります。

この複合的な課題についても、いくつかの研究が解決法を提案しています。

### 強化学習や半教師学習の分野では...

強化学習では、既存のデータを学習するだけでなく、次にどのデータを収集するか、も含めて最適化します。また、半教師学習では、モデルで予測したラベルを別の十分に独立したモデルで再学習します。

これらの学習は十分な教師ラベルがない状況へのアプローチですが、そのような状況はストリーミングデータでは一般的です。特に、concept driftが発生した場合は、過去のデータを学習する価値は薄まりますので、学習できる教師ラベルがさらに少なくなります。

この複合的な課題についても、いくつかの研究が解決法を提案しています。

## Decision Rulesの分野では...

そもそも、decision rules（例えば決定木）のようなデータ分割モデルは、concept drift の検知モデルとして応用しやすいです。そのため、この2つの分野を複合させた応用研究が多くあります。

---
# さいごに

以上で、サーベイ論文[1]による concept drift のレビューを要約してみました。
原著ではより詳しく、具体的な研究論文を引いていますので、気になるパートがあれば、原著をご確認くださいませ。

また、私の記述内容に誤りがあった場合はコメント欄にてご指摘いただけますと助かります。

それでは、この記事が何かのお役に立てば幸いでございます。


---
# References

```
[1] Lu, Jie, et al. "Learning under concept drift: A review." IEEE Transactions on Knowledge and Data Engineering 31.12 (2018): 2346-2363.

[2] S. H. Bach and M. Maloof, “Paired learners for concept drift,” in Proc. 8th Int. Conf. Data Mining, 2008, pp. 23–32.

[3] A. Bifet and R. Gavald a, “Learning from time-changing data with adaptive windowing,” in Proc. SIAM Int. Conf. Data Mining, vol. 7, 2007, Art. no. 2007.

[4]　S. Xu and J. Wang, “Dynamic extreme learning machine for data stream classification,” Neurocomputing, vol. 238, pp. 433–449, 2017.

[5] J. Z. Kolter and M. A. Maloof, “Dynamic weighted majority: An ensemble method for drifting concepts,” J. Mach. Learn. Res., 2007, pp. 2755–2790.

