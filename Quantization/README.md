# Quantization
물리학에서 양자화(量子化, 영어: quantization)란 좁은 의미에서 거시적으로 연속적인 양을 어떤 기본 단위(양자)의 정수배로 측정하는 양으로 재해석하는 것을 뜻한다. [[위키백과]](https://ko.wikipedia.org/wiki/%EC%96%91%EC%9E%90%ED%99%94_(%EB%AC%BC%EB%A6%AC%ED%95%99))

## Fixed Point & Floating Point

- **Fixed Point : 고정 소수점**<br>
<p align="center">
  <img src="https://github.com/user-attachments/assets/0def6b5e-b7e9-4eb3-b863-d66fa1ca17d9">
</p>

Fixed Point는 소수점 위치가 고정되어있다. 이는 숫자를 표현할 때 소수점이 항상 일정한 위치에 있다는 뜻이다. 

고정 소수점은 `m bit`의 **Signed Integer**와 `n bit`의 **Fractional Part**로 구성되어있다. 이때 `n bit`로 표현되는 소수부분은 $1/2^{-n}$의 해상도를 갖는다.

따라서 담을 수 있는 수의 개수가 동일하다. 이는 등간격으로 수가 표현되어 항상 동일한 해상도를 갖음을 의미한다.

- **Floating Point : 부동 소수점**<br>
<p align="center">
  <img src="https://github.com/user-attachments/assets/182b73d5-6363-45c2-adfc-42ca6d329e09">
</p>  

IEEE에서 정의하는 대표적인 32-bit 실수표현 방식이다.

하지만 고정소수점 표현과 달리 가변적으로 수를 표현하기 때문에 비해 더 넓은 범위와 더 높은 정밀도를 제공한다. 

**sign** `1 bit`,  **exponent** `q bit`, **mantissa** `p bit`로 구성된다.

Fixed Point에 비해 수를 표현하는 방식이 어려운데, 식으로 확인해보면 다음과 같다.

<p align="center">$$(-1)^{s} \times 2^{(e_0e_1...e_{q-1})_2-2^{q-1}} \times (1.d_0d_1...d_{p-1})_2$$</p>

수식을 잘 살펴보면 **exponent**는 $-2^{q-1} ~ 2^{q-1}$범위의 정수부분을 표현할 수 있고, **mantissa**는 1.xxx 형태의 수를 갖는 $1~2$ 범위의 값을 **exponent**에 곱해서 소수 범위의 값을 표현할 수 있다.


## (Affine)Quantization Mapping
$x \in [\alpha, \beta]$인 floating point value를 n-bit integer인 $x_q \in [\alpha_q, \beta_q]$로 매핑하는것을 의미한다. 

양자화는 크게 정규화 → 스케일링 → CLIP의 순서로 수행된다. 

이때 다음과 같은 수식으로 양자화(Quantization)화 역양자화(De-Quantization)를 수행할 수 있다. 

- **Quantization Process**<br>
<p align="center"> $x_q = round(\frac{1}{s} x + z)$</p>

- **De-Quantization**<br>
<p align="center">$x = s(x_q - z)$</p>

이때 $s = \frac{\beta-\alpha}{\beta_q-\alpha_q}$, $z = round(\frac{\beta\alpha_q-\alpha\beta_q}{\beta - \alpha})$이며 각각 scale과 shift를 담당한다. (Affine 변환으로 볼 수 있다.)
이때 zero-pad나 ReLU처럼 DL에서는 0.0이 양자화 이후에도 정확히 0을 유지해하므로 다음과 같은 식이 도출되며 $d$는 정수여야하는 constraint가 추가된다.

$x_q = round(\frac{1}{c}0 - d) = round(-d) = -round(d) = -d$


# Neural Network Quantization
딥러닝에서 양자화란 floating-point type을 integer-point type혹은 fixed-point type으로 변환하는 것을 의미한다. 

주로 Weight Quantization과 Activation Quantization 2가지 측면에서 이루어진다.

이를 통해 모델의 사이즈는 감소하며 MAC(Multiply-Accumulate) unit을 통하여 연산을 하는 하드웨어입장에서 에너지 소비가 감소하고, 한번에 읽어오는 크기가 늘어나므로 bandwidth가 증가(FP32→int8의 경우 4배 증가)한다.

NN의 Layer 연산은 matrix 연산과 activation 연산의 연속으로 이루어져있다. 

따라서 NN의 Quantization을 이해하기 위해 Matrix Multiplication → Activation → Layer Fusion 순으로 알아보겠다.

## Matrix Multiplication Quantization
Convolution 연산을 의미하는 matrix multiplication 식에 위의 Quantization mapping을 적용하면 다음과 같은 식이 유도된다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4c9b86c2-3ac5-4edb-b5d8-c479f82cf342">
</p>   

이때 구하는 값인 $Y_{q,i,j}$로 식을 정리하면 다음과 같다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/ce8ad934-3878-4f79-b94f-036ac29e2093">
</p>
  
이때 단순 연산횟수는 양자화 이전의 matrix multiplication보다 많아지지만 Integer type의 연산이므로 전력 소모도 적고 대체로 빠르게 수행할 수 있다.

## Activation Quantization
ReLU에 Quantization을 적용하는 유도식을 확인하겠다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/56761bd5-84bb-4ac0-914a-0200fe70f234">
</p>

기본 Quantization mapping과 유사하게 쉽게 유도할 수 있다.

# Layer Fusion
**Inference시점에**, Conv layer 뒤에 추가되는 Activation layer와 batchnorm layer 등을 fuse하여 하나의 Conv layer로 표현할 수 있다. <br>
이때 layer 갯수가 줄어들어 추론 속도가 향상되며 중간의 Quant, De-Qaunt 과정이 줄어들게 된다.

## Convolution + BatchNorm Fusion
BatchNorm은 다음과 같은 과정을 갖는다.
<p align="center">
  <img src="https://github.com/user-attachments/assets/44a4fb9e-997c-41f0-80e3-6e5c9c0834d7" >
</p>
Inference 시에 BatchNorm(Freezed BatchNorm)은 EMA와 같은 기법을 통해 저장해 놓은 Train Dataset의 $\mu$와 $\sigma$를 통해 `normalize`를 수행하고 이후 $\gamma$와 $\beta$를 통해 `re-normalize`를 수행한다.

이 두번의 Affine mapping은 다음과 같이 하나의 Affine mapping으로 묶어서 표현할 수 있다.

<p align="center">$y = \frac{\gamma}{\sqrt{Var[x]+\epsilon}}*x + (\beta - \frac{\gamma E[x]}{\sqrt{Var[x]+\epsilon}}) $</p>

실제 논문에서는 `re-normalize`만을 $\gamma$와 $\beta$로 표현한다. 즉 입력 $x$에 대하여 하나의 Affine mapping만 수행하고, 이는 대각행렬, Feature map, bias의 Matrix 연산으로 표현할 수 있다.

$\hat{F} _{C,i,j} = \gamma _C F_{C,i,j} + \beta _C$

이때 $F_C$는 이전 Conv를 통과한 output channel = c의 Feature map이다. 
Channel wise한 연산이므로 1x1 Conv로도 표현 가능하다.

- **Vector unwrapping**

실제 Convolution 연산은 unrolling을 통한 matrix multiplication으로 수행된다.

이를 Convolution과 BatchNorm을 조합하기 위해서 사용하면 다음과 같다.

먼저 입력으로 들어오는 Feature map을 Convolution kernel size에 해당하는 크기의 vector로 unwrapping한다. 

이는 커널을 sparse matrix로 표현하고, 입력을 vector로 펼치는 것으로 계산할 수 있다.

이를 통해 column vector를 만들 수 있으며 BatchNorm과 결합하면 다음과 같은 최종 식이 유도 된다.

<p align="center">
$\hat{f} _{i,j} = W_{BN} *( W_{conv}*f_{i,j} + b_{conv}) + b_{BN}$
$\hat{f}_{i,j} = W_{fused}*f_{i,j} + b_{fused}$ <br>
$W_fused = W_{BN} * W_{conv}$ <br>
$b_{fused} = W_{BN} * b_{conv} + b_{BN}$

</p>

## Fused Quantization : Conv-Activation Fusion
위에서 Convolution과 BatchNorm을 하나의 Convolution으로 Fusion하는 것을 확인하였다. 


각 layer마다 range가 다르기 때문에 하나의 layer를 지날때마다 다음과 같은 구조가 반복된다.
Quant for layer_1 -> layer_1 -> De-Quant -> Quant for layer_2 -> ...
즉 각 layer마다 quantized params$(s,z)$가 필요하다. 
이때 Convolution과 Activation이 fuse되면 scale을 바꾸기 위해 필요한 De-Quant - Quant 과정이 사라지게 된다.

$Y'_{i,j} = ReLU(Y_{i,j}, 0, 0, 1)$을 전개하면 다음과 같다. ($Y$는 중간 output, $Y'$는 최종 output)

<p align="center">
  <img src="https://github.com/user-attachments/assets/6e1a28d3-10d3-4b83-a9f7-14f78a3e5835" </img>
</p>

Fuse된 결과의 quantization에 중간 Conv layer($Y$)의 quantization 파라미터인 $s_Y, z_Y$가 없음을 확인할 수 있다. 이는 중간의 Quant, De-Quant 과정이 생략된 것을 의미한다.



## 🔗 Reference
[부스트코스 : 모델 경량화](https://www.boostcourse.org/ai302/joinLectures/374476)   
[Floating Point & Fixed Point image](https://www.researchgate.net/figure/Representation-of-the-floating-point-and-fixed-point-formats_fig1_225139564)   


