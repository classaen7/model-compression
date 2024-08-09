# Matrix/Tensor Decomposition
하나의 고차원 텐서를 여러 개의 작은 텐서로 쪼개는 기법이다. 예로 CP decomposition과 Tucker decomposition가 있다.

이 방법을 통해 원래 데이터의 중요한 패턴이나 구조를 더 잘 이해하거나, 데이터를 압축하고 효율적으로 처리하기 위해 사용된다.

## Prior knowledge
- 대칭 행렬
- 정방 행렬
- 직교 행렬(orthogonal matrix)
  - 각 열 벡터들이 서로 independent하며 orthonormal 한 성질을 갖는 행렬.
- 행렬의 대각화(diagonalization)
  - 행렬을 대각행렬로 변환하는 과정으로 주로 행렬의 고유값과 고유벡터를 이용해 이루어짐.
- 가역 행렬
  - A의 고유벡터(eigenvector)가 각 열을 이루며 이들이 서로 선형 독립인 행렬
- 고유값(eigen value)과 고유벡터(eigen vector)
  - 고유벡터: 고유벡터는 행렬의 변환에서 방향이 변하지 않는 벡터
  - 고유값 : 주어진 행렬이 고유벡터를 얼마나 "늘리거나 줄이는지"를 나타내는 값
- 고유값 분해
  - 정방 행렬 중에서도 일부 행렬에 대해서 만 적용 가능한 대각화 방법
  - $$A = PDP^{-1}$$를 만족하는 대각행렬 D와 가역행렬 P를 찾는 과정

# SVD : 특이값 분해
고유값 분해와 유사하지만 특이값 분해는 행렬이 정방 행렬이 아니어도 적용가능한 행렬의 대각화 방법 ($AA^T, A^TA$는 정방행렬)<br>
어떤 랭크 r을 가지고 있는 m*n크기의 행렬 A를 $$A = U\Sigma V^{T}$$의 곱으로 인수분해 하는 것을 의미

<font align=center> $$A = U\Sigma V^{T} = \Sigma _{i=1}^{n} \sigma _i u_i v_i^T \qquad where\quad \sigma_1 \geq \sigma_2 \geq ... \geq \sigma_n$$ </font>

$$U$$ : $$AA^T$$를 고유값분해(eigendecomposition)해서 얻어진 직교행렬. 이는 A의 열벡터간의 내적을 의미함
→ 열 공간에서 어떻게 작용하는지를 설명 = 열 벡터들이 특정 방향으로 얼마나 확장되는지를 고유값에 저장

$$V$$ : $$A^TA$$를 고유값분해(eigendecomposition)해서 얻어진 직교행렬. 이는 A의 행벡터간의 내적을 의미함
→ 행 공간에서 어떻게 작용하는지를 설명 = 행 벡터들이 특정 방향으로 얼마나 확장되는지를 고유값에 저장

$$\Sigma$$ : $AA^T, A^TA$를 고유값분해해서 나오는 고유값(eigenvalue)들의 (제곱근)square root를 대각원소로 하는 m x n 직사각 대각행렬

## 이미지2

## low-rank approximation by SVD
$Rank(K)$를 가지는 $A_k$를 찾는것 = $A$와 찾으려는 $A_k$의 프로베니우스 노름을 최소화하는 $Rank(K)$를 찾을 때 SVD를 활용할 수 있음<br>
이때, 기존의 matrix보다 작은 matrix로 compression 하는 것이 목표이기 때문에, full rank보다 작은 k를 설정하게 된다.

## Image Compression
Singular value $\sigma$가 내림차순으로 정렬되어 있을 때 크게 기여하는 상위 몇개 만 사용하면 원본의 텐서를 잘 표현할 수 있다<br>
이는 위에서 찾은 최적의 $Rank(K)$로 행렬을 줄여도 원본 행렬을 잘 표현할 수 있음을 의미한다.

# CP : Canonical decomposition
- Vector Outer Products : 벡터의 외적

3-way structure → $i$차원 벡터 $a$와 $j$차원 벡터 $b$와 $k$차원 벡터 $c$가 외적 수행시 $(i,j,k)$ 크기의 텐서가 됨

- Alternating Least Square
**최적의 랭크**를 찾기 위한 ALS 알고리즘

Conv weight의 경우, 3차원 텐서이므로 3개의 컴포넌트가 필요하다. 이때 ALS 알고리즘은 2개의 컴포넌트를 고정하고 나머지 한개의 컴포넌트를 대상으로 Least Square 문제를 푸는 형태로 최적화를 수행한다.

하나의 컴포넌트를 최소화 하는 것은 Convex 문제에 속하지만, CP의 경우는 여러 개의 컴포넌트들의 곱으로 되어있기 때문에 사실 Non-Convex 문제이다. 

### 이미지

실제 계산은 행렬로 계산된다

mode 연산과 Khatri-Rao 연산은 언폴딩 해주는 연산임. 위의 이미지의 프로베니우스 노름을 최소화하면서 CP Decomposition이 수행됨


# Tucker Decomposition
Tucker decomposition 에서는 CP와 달리 대각 텐서가 아닌 좀 더 일반화된 코어 텐서를 사용

### 이미지 3

코어 텐서와 행렬 곱을 정의하기 위해 n-mode 연산이라는 것을 사용

n-mode 연산은 텐서의 여러 축과 매트릭스의 어떤 축을 계산할 건지에 대한 표기

예시) 모드-n 행렬은 코어 텐서의 $I_n$ 차원과 행렬 곱이 되어 모드-n 행렬의 차원인 $I_n$ 차원으로 바뀜

- Rank reduction with Tucker

SVD와 동일하게 동작함

각 rank 방향으로 tensor rank reduction을 수행함

손실은 발생하겠지만 기존 텐서 A를 근사하는 어떤 텐서를 계산할 수 있게 됨
