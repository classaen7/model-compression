# AutoML 배경
좋은 성능을 내는 모델의 Configuration을 찾기 위해선 반복적인 작업이 필요하다.

보통 이러한 반복적인 작업은 사람에 의해 수행되는데 AutoML은 사람을 그 과정에서 빼내는 것을 목표로 한다.

AutoML(HPO : Hyperparameter Optimizer)는 loss를 최소화해주는 하이퍼파라미터를 찾는다.

이는 진정한 End to End 러닝을 의미한다.

# 딥러닝 Configuration 특징
DL의 하이퍼파라미터는 Categorial / Continuous / Integer 타입이 존재한다.

또한 **Conditional**한 특징이 존재한다.


# 모델 경량화 관점
경량화에 대해 두가지 관점이 존재

1. 기존의 모델을 경량화
2. 탐색을 통한 경량 모델을 찾기

이때 AutoML은 2번의 예시로 볼 수 있다.

## AutoML PipeLine
1. Configuration λ 설정 
2. Train with λ (학습이 잘 되는지는 Black box에 해당)
3. Evaluate Objective f(λ) (목적 함수는 상황에 맞게 요청됨 - 분류, 회귀 등)
4. Blackbox optimization: Objective 함수를 Maximize


## Bayesian Optimization
1. Update **Surrogate Function** ( f(λ)를 예측하는 regression model )
2. Update **Acquisition Function** ( Surrogate Function을 사용해서 다음 시도해 볼 λ를 결정하는 model )

Gaussian Process Regression은 불확실성을 모델링 할 수 있다.
    예를 들어 2개의 점을 알고 있을 때, 그 사이의 값을 불확실성을 가지고 표현할 수 있다.
        이러한 특성의 이유로 Gaussian Process Regression Model은 대표적인 **Surrogate Model**이다.

**Acquisition Function**는 Exploration과 Exploitation의 trade off를 통해 다음의 λ를 결정한다.

## Tree-structured Parzen Estimator
Bayesian Optimization은 $N^3$의 시간복잡도를 갖으며 Conditional 관계가 섞여 있을 때 사용하기 어려움

- TPE를 통한 다음 step λ 계산 방법
1. 현재까지의 observation을 good(25%), bad(75%)로 구분 (특정 quantile(inverse CDF)로 구분)
2. KDE(Kernel density estimation)을 통해 분포 p(g), p(b)를 각각 추정
3. p(g)/p(b)은 EI(Expected Improvement, acquisition function 중 하나)에 비례하므로 높은 값을 가지는 λ를 다음 step으로 설정


# Reference
[Algorithms for Hyper-Parameter Optimization](https://papers.nips.cc/paper/2011/file/86e8f7ab32cfd12577bc2619bc635690-Paper.pdf)
