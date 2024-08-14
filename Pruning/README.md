# Pruning
Pruning은 중요도가 낮은 파라미터를 제거하는 것이다.

다음과 같이 다양한 케이스가 존재한다.
- 단위: Structured(group) / Unstructured(fine grained)
- 정량적 기준: Magnitude(L2, L1), BN Scailing factor, Energy-based, ...
- 적용 기준: Network 전체를 기준(global), Layer 마다 내부 기준(local)
- 시점: 학습된 모델 / 초기화 시점

<p align="center">
  <img src="">
</p>

# Structured Pruning
Structured Pruning은 파라미터를 그룹(channel, filter, layer) 단위로 제거하는 것이다.
<p align="center">
  <img src="https://github.com/user-attachments/assets/2ca5bf2c-2323-4dec-8e7f-6506794a563c">
</p>

### Learning Efficient Convolutional Networks through Network Slimming [CV]
> #Structured(group) #BN scailing factor #Global #Trained Model

Weight의 BN의 scailing factor $\gamma$로 중요도를 판단한다.


### HRank: Filter Pruning using High-Rank Feature Map [CV]
> #Structured(group) #Feature map의 Rank(SVD) #Local #Trained Model [CV]

Weight가 아닌 Feture map output의 SVD Rank로 중요도를 판단한다. 

<p align="center">
  <img src="https://github.com/user-attachments/assets/2747f544-e58e-4b9d-9d82-4204251995e1">
</p>

위와 같이 다양한 모델, 서로 다른 데이터셋에 대해서도 feature map output의 SVD Rank의 차이가 없음을 실험적으로 확인하였다.

### Are Sixteen Heads Really Better than One? [NLP]
Mutli-Headed Attention(MHA)은 복합적인 의미들을 담아내기 위해 많은 수의 HEAD로 구성되어있지만 몇개가 제거되어도 성능의 감소가 없다는 것을 확인한 논문.

또한 서로 다른 데이터셋으로 실험해도 HEAD의 중요도는 비슷하다는 실험 결과가 있다.

- **Iterative Pruning of Attention Heads**<br>
여러 layer의 여러 Head가 존재할 때, 해당 논문에서는 모든 Head들의 **proxy importance score**를 구하고 정렬한 후에 p%의 HEAD를 제거하는 형태로 Pruning을 수행한다.<br>
이는 특정 Head가 존재할때와 그렇지 않을 때의 Loss 차이의 절대값을, 모든 데이터에 대해 계산을 한 후 평균값을 구한 것이다. 

# UnStructured Pruning
UnStructured Pruning은 파라미터를 각각 독립적으로 제거하는 것을 의미한다. 따라서 네트워크 내부의 행렬은 희소(Sparse)해진다는 특징이 있다.
<font color="gray">하지만 Sparse Computation에 최적화된 소프트웨어 또는 하드웨어가 아닌 이상 속도 향상이 되지는 않는다.</font>

### The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks [CV]
> #UnStructured(fine grained) #L1 norm #Global #Trained Model 

큰 신경망에는 복권 티켓(Lottery Ticker)과 같은 작은 서브네트워크가 존재하며, 이 **서브네트워크**가 초기화된 상태로 학습할 때 원래의 큰 네트워크와 유사한 성능을 낼 수 있다는 주장이다.

해당 논문에서는 다음의 Lottery Ticket을 찾는 방법을 통해 10% ~ 20%의 weight만으로 기존 네트워크와 동일한 성능을 갖는 서브네트워크를 찾는다. (아주 약간 Prune하고 재학습을 반복하는 Iterative Pruning을 수행)

1. 네트워크 임의 초기화 $f(x;\theta _0)$
2. 네트워크를 $j$번 학습하여 파라미터 $\theta _j$를 도출
3. 파라미터 $\theta _j$를 $p%$(보통 20%)만큼 pruning하여 mask **m**을 생성 
4. Mask 되지 않은 파라미터를 $\theta _0$로 되돌리고 이를 `winning ticket`이라 지칭 ($f(x;m \odot \theta)$)
5. Target sparsity에 도달할때 까지 2-4를 반복 $(1-p%)^n$ = 남은 파라미터 수 %

시간은 오래걸리지만 매우 좋은 성능을 보인 논문이다.

### Stabilizing the Lottery Ticket Hypothesis; Weight Rewinding [CV]
위의 LTH의 경우 데이터셋 또는 네트워크의 크기가 커졌을 때 불안정한 모습을 보이는 단점이 존재한다.<br>
이 방법은 LTH와 다르게 weight와 lr_scheduler를 처음으로 초기화 하는게 아닌 k번째 epoch으로 초기화하면 학습이 안정화되는 것을 보인 논문이다.

논문에서는 총 iteration의 0.1% ~ 7% 정도의 지점을 rewinding point로 언급한다.

### Comparing Rewinding And Find-tuning In Neural Network Pruning; Learning Rate Rewinding [CV]
위의 방법에서 Weight는 그대로 유지하고 LR_Scheduler만 특정 시점(k)로 rewinding 하는 전략이다.

### Linear Mode Connectivity and the Lottery Ticket Hypothesis [CV]
특정 initialization에 대해서, 학습 후에 도달하는(**수렴**되는) 공간이 유사한지 실험한 논문이다. 방법은 다음과 같다. <br>
특정 시점(0, k)에서 Seed를 변경하여 두 개의 Net을 학습시킨다. 이때 둘간의 Weight를 linear interploation하여 성능을 비교하고, 두 weight 공간 사이의 interpolated net들의 성능을 확인한다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/735228f3-3f5d-4aad-b498-df9be0a6cb21">
</p>
위의 그림에서 (위: epoch 0)인 경우 error가 크게 나옴을 확인할 수있다. (아래 : epoch k)의 경우 k가 증가할수록 instability는 점점 감소함을 확인할 수 있고, k가 적당히 작은 지점부터 loss가 상당히 많이 감소함을 확인할 수 있다.

결국 **"특정 시점부터 네트워크가 수렴하는 공간은 한 Flat 위에 있지 않은가"** 라는 해석을 제시한다.

### Deconstructing Lottery Tickets: Zeros, Signs, and the Supermask [CV]
LTH는 final weight의 L1 norm으로 masking을 수행한다.<br>
해당 논문에서는 inintial weight와 final weight을 함께 고려한다. (NLP의 moving average와 유사) <br>

<p align="center">
  <img src="https://github.com/user-attachments/assets/2f50bcb2-4f01-4967-8b57-15ab53195394">
</p>

위의 초기 가중치, 마지막 가중치를 기준으로한 그래프를 통해 다양한 해석을 할 수 있다.<br>
예를 들어 "movement"는 변위를 본것이라 해석할 수 있다.

### Movement Pruning: Adaptive Sparsity by Fine-Tuning [NLP]
Fine Tuning 과정에서 weight의 절대적인 크기 대신(Magnitude)에 0에서 멀어지는지 또는 가까워지는지를 보면서 Pruning을 수행하는 방법이다.

### Score-Based Pruning
Unstructured pruning의 대표적인 방법이다.

Score-Based Pruning은 weight와 동일한 크기의 score 행렬을 기준으로 v%는 1로 masking 그 외는 0으로 masking을 함으로써 pruning을 수행하는 것이다.

이때 Score는 Weight가 fine tuning 되면서 함께 학습되므로 self correction되는 효과가 존재한다.


# Structured - UnStructured
- Structured Pruning
    - 장점: 모델 사이즈 감소, 속도 향상
    - 단점: 성능 감소
- UnStructured Pruning
    - 잠점: 모델 사이즈 감소, 적은 성능 감소
    - 단점: 속도 향상 없음 (하드웨어)

# Pruning at initialization
추가로 Pruning 시점을 학습 이전에 수행하는 방법도 존재한다.<br>
이는 학습 이전에 "중요도"를 측정하여 수행하며, 시간 절약의 장점이 존재한다.

이때 다음과 같은 **중요도 계산 방법**이 존재한다.

- SNIP: Training 데이터 샘플, Forward해서 Gredient와 Weight의 곱의 절대값
- GraSP : Training 데이터 샘플, Forward해서 Hessian-gredient product와 Weight의 곱
- SynFlow : 전부 1로 된 가상 데이터를 Forward해서 Gradient와 Weight을 곱

### Zero-cost proxies for Lightweight NAS
해당 논문에서는 NAS를 통해 위의 중요도 계산방법을 실험한다. <br>
실험을 통해 중요도 측정 score와 학습 결과간의 상관 계수(Spearman 상관계수)가 높음을 확인했고, Synflow가 다양한 task에도 높은 상관 계수를 보였다.

Pruning at initialization은 학습이 필요 없는, 간접적으로 모델을 평가하는 기법으로써 활용이 될 가능성이 높다고 한다.

# 🔗 Reference
[Paper]<br>
*Learning Efficient Convolutional Networks through Network Slimming<br>
HRank: Filter Pruning using High-Rank Feature Map<br>
Are Sixteen Heads Really Better than One?<br>
The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks<br>
Stabilizing the Lottery Ticket Hypothesis; Weight Rewinding<br>
Comparing Rewinding And Find-tuning In Neural Network Pruning; Learning Rate Rewinding<br>
Linear Mode Connectivity and the Lottery Ticket Hypothesis<br>
Deconstructing Lottery Tickets: Zeros, Signs, and the Supermask<br>
Movement Pruning: Adaptive Sparsity by Fine-Tuning<br>
Zero-cost proxies for Lightweight NAS*
