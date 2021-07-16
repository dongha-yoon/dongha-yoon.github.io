<script type="text/javascript"
        src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS_CHTML"></script>

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
inlineMath: [['$','$'], ['\\(','\\)']],
processEscapes: true},
jax: ["input/TeX","input/MathML","input/AsciiMath","output/CommonHTML"],
extensions: ["tex2jax.js","mml2jax.js","asciimath2jax.js","MathMenu.js","MathZoom.js","AssistiveMML.js", "[Contrib]/a11y/accessibility-menu.js"],
TeX: {
extensions: ["AMSmath.js","AMSsymbols.js","noErrors.js","noUndefined.js"],
equationNumbers: {
autoNumber: "AMS"
}
}
});
</script>

# DCN (Deep & Cross Network)

#### Ref: <https://arxiv.org/abs/1708.05123>

* DLRM (<https://arxiv.org/abs/1906.00091>)에서 비교 대상으로 많이 언급된 모델이라 읽어보게 되었다.

* 많은 prediction modeld에서 성공의 열쇠는 feature engineering이다. 이 프로세스는 보통 manual feature engineering이나 소모적인 searching을 통해 해결을 하였는데, DNN (Deep Neural Network)를 통해 feature interaction을 자동적으로 학습할 수 있게 되었다.

* 그러나 DNN 또한 interaction을 명확하게 만들지 못하고, 모든 cross feature에 대하여 효율적이지 않았다.

* 그래서 DCN은 기존 DNN에서 parallel한 cross network를 설계하여 bounded-degree 내에서 feature interaction을 효율적으로 학습할 수 있도록 했다.

## 1. Introduction

* Advertising industry에서 많이 쓰이는 payment model중 하나가 CPC(Cost-per-click)이기 때문에 CTR(Click-through rate) prediction을 얼마나 정확히 하냐가 publisher의 수입에 큰 영향을 미쳤다.

* Web-scale recommender system data is discrete and categorial (sparse feature)

* Cross network는 feature crossing을 명시적이고 자동적으로 적용한다.

* DNN도 feature간 복잡한 interaction을 잡아낼 수 있지만 cross network에 비해 몇 배의 parameter가 더 필요하고, cross feature를 명시적인 형태로 보여줄 수 없고, 몇가지 형태의 feature interaction을 효율적으로 학습할 수가 없다.

### Related Work

* Dataset의 차원의 크기가 극적으로 증가함에 따라 task-specific feature engineering을 피하기 위해 embedding과 neural network에 기반한 방법들이 제안되었다.

* FM (Factorization Machine)
  * Sparse feature를 low-dimensional dense vector로 변환
  * 벡터 내적으로 freature interaction을 학습
  * 구조 자체가 얕기 때문에 표현력에 있어서 한계가 있다.
  * FM을 higher-order로 확장하는 방법을 시도해봤지만 cost가 undesirable

* DNN
  * Embedding vector와 비선형 activation function을 통해 high-degree feature interaction을 학습할 수 있다

* Residual network
* Deep crossing
* Wide-and-deep
  * Implicit하고 highly nonlinear한 DNN의 문제를 해결하기 위해 효과적이고, 명확한 bounded-degree interaction을 학습할 수 있도록 설계된 모델
  * 그러나 이 모델의 성공 여부가 cross feature의 적절한 선택에 달려 있다.
    * 그러나 아직 이 선택에 대한 명확하고 효율적인 해답이 없다.
    * Performance가 unstable하다는 의미 같음.

## 2. DCN

```txt
        +-----------------------+
        |   Combination Layer   |
        +-----------------------+
            /               \
    +-----------+       +-----------+
    |   Cross   |       |    Deep   |
    |  Network  |       |  Network  |
    +-----------+       +-----------+
           |                   |
    +-------------------------------+
    |  Embedding and Stacking Layer |
    +-------------------------------+
```

### - Embedding and Stacking Layer

* Embedding: One-hot vector로 표현되는 sparse feature를 embedding vector로 변환

$$
\textbf x_{embed,i} = W_{embed,i}\textbf x_i
$$

* Stacking: Embedding 벡터와 dense 벡터를 하나의 벡터로 연결하여 network에 feed

$$
\textbf x_0 = [\textbf x_{embed,1}^T,...,\textbf x_{embed,k}^T,\textbf x_{dense}^T]
$$

### - Cross Network

$$
 \textbf x_{l+1} = \textbf x_0\textbf x_l^T\textbf w_l + \textbf b_l + \textbf x_l
$$

* Layer depth를 따라서 cross feature의 차원이 하나씩 증가한다.
* \# of parameters = $d \times L_c \times 2$,
  * $d$ : input dimension, $L_c$ : \# of cross layers

* Parameter 수가 input dimension의 크기와 linear하기 때문에 cross network전체의 complexity 역시 input dimension에 linear하다.

* 따라서 기존 DNN에서 cross network를 추가한 방식(DCN)의 complexity가 기존 DNN과 같은 레벨이다.

* 위와 같은 장점을 가질 수 있는 이유는 $\textbf x_0\textbf x_l^T$의 rank-one property를 이용하면 matrix 전체를 computre, store할 필요가 없기 때문이다.

### - Deep Network

$$
\textbf h_{l+1}= \text{ReLu}(W_l\textbf h_l + \textbf b_l)
$$

* Activation function으로 ReLu를 사용하였다.
  * 기존에 많이 쓰이던 sigmoid 함수는 일정 값을 기준으로 0인지 1인지 구분하므로 binary classification에 적절한데, layer가 많아질수록 gradient가 0으로 수렴하게 되는 gradient vanishing 문제가 생기게 된다.
  * 이러한 문제를 해결하기 위해 hidden layer에서는 0 이하의 값은 0, 0 이상은 값 그대로 리턴하는 ReLU 함수를 적용하고, 마지막 output layer에서만 sigmoid를 적용하여 모델의 정확도를 높일 수 있다고 한다.
  * <https://medium.com/@kmkgabia/ml-sigmoid-%EB%8C%80%EC%8B%A0-relu-%EC%83%81%ED%99%A9%EC%97%90-%EB%A7%9E%EB%8A%94-%ED%99%9C%EC%84%B1%ED%99%94-%ED%95%A8%EC%88%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-c65f620ad6fd>

* \# of parameters = $d \times m + m + (m^2 + m) \times  (L_d-1)$,
  * $d$ : input dimension, $L_d$ : \# of deep layers, $m$ : layer size (assume all layer has equal size)

### - Combination Layer

* Cross 와 deep network 각각에서 나온 최종 결과를 연결하여 logit layer에서 처리를 하여 output 산출

$$
p=\sigma([\textbf x_{L_1}^T, \textbf h_{L_2}^T]\textbf w_{logits})
$$

## 3. Cross Network Analysis

### - Polynomial Approximation

* Weierstrass approximation theorem <br> : Any function under certain smoothness assumption can be approximated by a polynomial to an arbitrary accuracy

* Multivariate polynomial을 polynomial로 reduce 해서 뭐시기뭐시기...


### - Generalization to FMs

* FM model : Feature $x_i$는 weight vector $v_i$와 연관되어 있다.
  * Weght of croess term $x_ix_j = \langle v_i,v_j \rangle$

* DCN : $x_i$는 스칼라 $\lbrace w_k^{(i)}\rbrace_{k=0}^l$과 연관되어 있다.
  * Weght of croess term $x_ix_j = \lbrace w_k^{(i)}\rbrace_{k=0}^l \times \lbrace w_k^{(j)}\rbrace_{k=0}^l$

* Parameter sharing은 model을 더 효율적이게 할 뿐 아니라 unseen feature interaction을 일반화 하여 모델이 노이즈에 더 강하게 해준다.

* FM은 얕은 구조를 가지고 있기 때문에 degree 2까지의 cross term을 represent할 수 있다.
  * 그래서 DLRM에서는 FM을 그대로 쓴것 같다.

* DCN은 모든 cross term을 construct할 수 있으며 higher-order FM과는 다르게 파라미터 수가 input dimension에 따라 linear하게 변한다는 장점이 있다.


### - Efficient Projection
