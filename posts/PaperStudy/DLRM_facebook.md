<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# DLRM (Deep Learning Recommnedation Model)
#### Ref: <a>https://arxiv.org/abs/1906.00091</a>

* 이 모델은 personalization, recommendation system을 위한 모델이다.
* 해당 시스템이 일반적인 딥러닝 시스템과 다른 점은 categorial feature(범주형 변수 <-> 연속형 변수)를 handling 해야한다는 점이다.


## 1. Introduction
### - 모델 디자인을 위해 반영된 관점
### 1. View of Recommendation System
* Content filtering : 분류된 product를 범주화 -> User의 preference로 match
* Collaborative filtering : 과거 user behavior를 기반으로 추천
* Neighborhood filtering : user들과 product들을 그룹화
* Latent factor method : Matrix factorization을 이용하여 implicit factor를 구한 뒤 user와 product를 characterize

### 2. View from predictive analytics
* Predictive analytics : 확률 분류/예측을 위해 statistical model을 이용

### - Data processing
* **Embedding** : 보통 범주형 데이터를 표현하는데 쓰는 Sparse representation(one-hot 또는 multi-hot vector)을 Dense representaion으로 변환
* **MLP** (MultiLayer Perceptron) : Dense featur processing을 위해 쓰는 DNN모델의 일종

* 최종적으로 또 다른 MLP(top MLP)로 post-processing을 하여 결과를 도출한다.

## 2. Model Design and Architecture
### - Components
1. Embedding : Embedding table을 이용하여 구한다. sparse vector(e)와 table(W)의 곱으로 dense vector로 변환. 결론적으로는 table의 utilization이 가장 중요.
$$
w_i^T = e_i^TW
$$

2. Matrix Factorization
$$minimize \sum r_{ij}-w_j^Tv_j; w:product, v:user$$
    
3. Factorization Machine : Classification
$$ \hat{y} = b + w^Tx + x^Tupper(VV^T)x $$

4. MLP
$$ \hat{y} = f_k(f_{k-1}(x))$$
$$ f_i(x) = W_i\sigma(x)+b_i$$
   * sigma : activation function
   * W: weight matrix
   * b: bias, threshold와 비슷한 개념. MLP 노드에서 activation이 얼마나 쉽게 일어나게 할 것인가를 결정

### - DLRM Architecture
* Categorial features: Embedding vector로 변환된 후 Matrix fatorization으로 일반화.
* Continuous features -> bottom MLP를 통해 위의 embedding vector와 같은 차원의 dense representaion으로 변환
* Factorization Machine을 통해 위의 두 feature간 second-order interaction을 계산. 모든 embedding vector와 dense feature쌍의 dot product를 구해야함.
* 위의 dot product를 top MLP를 통해 연결/후처리함.
* 그리고 sigmoid function으로 결과 도출


### - Prior model과의 비교
#### Prior models
* Wide and Deep, Deep and Cross, DeepFM, xDeepFM
* higher-order interaction 구성을 위해 특화된 네트워크(모델)를 설계함
* 그리고 이 네트워크와 MLP의 결과를 합한다.

#### DLRM
* Factorization machine을 통해 top MLP에서 dot product에 의해 생성된 cross-term만 고려하였기 때문에 demensionality를 크게 줄였다.
* Key difference : how these network threat embedded feature vectors and their cross-terms
* DLRM은 각 feature vector를 single category를 표현하는 single unit으로 해석하는 반면, Deep and Cross 같은 네트워크는 각 벡터를 서로간의 cross-term을 생성해야하는 새로운 단위로 취급한다.

## 3. Parallelism
* DLRM은 CNN, GAN, RNN모델 보다는 더 많은 파라미터를 가진다. -> MP(model parallelism)이 필요하다.

* Embedding : Embedding table등으로 인해 GB단위의 메모리가 필요. DP(daa parallelism)을 쓰기에는 모든 디바이스에 embedding을 replicate하는 것이 비효율적(거의 불가능)이다.

* MLP : 파라미터 수는 작지만 처리해야할 데이터가 많다 -> concurrent processing을 위해 DP가 필요

## 4. Data
* Random, synthetic, public dataset을 돌렸다.
* Random, synthetic : test from the system perspective
* Public : test on real data and measure accuracy

1. Random
    * Continuous : Uniform or Gaussian distribution
    * Categorial : 주어진 multi-hot vector에 non-zero element를 얼마나 둘 것이냐

2. Synthetic
    * Data의 privacy problem 해결을 위한 방식
    * System component test를 위해 original trace의 locality를 capture하여 비슷하게 만든다.

3. Public
    * Kaggle과 같은 opened real data를 가지고 함

## 5. Experiments
* 실행 시간중 대부분이 embedding lookup과 fully connected layer에 쓰임