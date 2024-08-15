# Knowledge Distillation
Knowledge Distillation은 큰 모델(teacher)에서 작은 모델(student)로 지식을 이전하는 기법을 의미한다.

이때 Teacher의 정보를 아래의 그림처럼 어디서 빼내는가에 따라 분야가 나뉜다.

<div align="center">
    <img src="https://github.com/user-attachments/assets/9adba450-2114-4bec-9b8a-4de02c764a9c"/>
</div>

- Input, Hidden, Output Layer: Relation-Based Knowledge
- Hidden Layer: Feature-Based Knowledge
- Output Layer: Response-Based Knowledge


# Response-Based Knowledge Distillation
teacher model의 Final prediction(last output layer)을 활용하는 KD 기법이다. <br>
last output은 보통 hinton loss를 사용하는 것이 특징이다. 
> Hinton loss를 사용하면 "near-zero" 확률들이 모델의 일반화 capabilities와 테스트 셋에서 잘 동작하는 것을 반영한다. <br>
이는 Soft Prob 에는 Semantic한 정보가 담겨있기 때문에 일반화 capabilities를 더 잘 반영한다는 뜻이다.

# Feature-Based Knowledge Distillation
teacher model의 layer의 중간 표현(intermediat representations)을 student가 학습하도록 하는 기법이다.

<div align="center">
<img src="https://github.com/user-attachments/assets/14022e44-b0c8-42e3-98b7-322d94c08a92"/>
</div>

loss는 매핑되는 teacher와 student의 중간 layer를 transformation하여 비교한다.

Feature-Based KD 연구의 주된 방향은 Feature distillation 과정에서 유용한 정보는 가져오고 중요하지 않은 정보는 가져오지 않도록 Transformation function과 distance 위치 등을 조정하는 것이다.

중간 결과를 가져오기 때문에 network 구조에 크게 의존적인 기법이다.

# Relation-Based Knowledge Distillation
다른 레이어나, Data Sample, Feature map output들  간의 **관계**를 정의하여 Knowledge distillation을 수행하는 기법이다.

<div align="center">
<img src="https://github.com/user-attachments/assets/8692ee25-455e-475b-9192-e35373628c20"/>
</div>

위의 이미지 처럼 벡터 공간내의 샘플 이미지들의 관계를 유사하게 학습을 하는 것이다.
(e.g. Conventional KD, Relational KD)

### Self-training with Noisy Student improves ImagesNet classification
해당 논문은 작은 student를 만드는것보다 성능이 높은 모델을 만드는 데에 초점을 맞춘다.

다음의 과정으로 당시 ImageNet SOTA를 기록했다.
1. Label 이미지로 teacher 모델 학습
2. teacher 모델을 사용해서 pseudo label 진행
3. label+pseudo label이미지로 student 모델 학습

student를 다음 iteration의 teacher로 사용하고 student의 크기는 유지하거나 점점 키웠다 (EfficientNet 사용)

# KD in NLP
KD는 NLP에서 가장 핫한 연구 방향이다.

<div align="center">
<img src="https://github.com/user-attachments/assets/74bffac4-20af-400e-95a8-4bafbe2a7dd5"/>
</div>

위와 같이 NLP에서 KD는 teacher의 어떤 정보를 사용할지, network를 어떻게 줄일지에 대한 방법이 존재한다.

### DistillBERT
BERT 모델의 경량화 버전으로, Knowledge Distillation 기법을 활용하여 원래 BERT의 성능을 대부분 유지하면서도 모델 크기와 추론 속도를 크게 개선한 모델이다.

**Triple Loss**
- Masked Language Modeling Loss(CE-loss)
- Distillation(Hinton) Loss: teacher, student의 softmax prob의 KL Div
- Cosine embedding loss: teachre, studnet의 히든 state 벡터 간의 코사인 유사도 (Feature-Based)


**Student architecture 구성**
- token-type embeddings 제거
- pooler 제거
- 레이어 개수 절반
- hidden dim 유지 (computation에 큰 영향을 끼치지 않아서)

**Student initialization**
- 구조는 BERT가 12개이고, DistillBERT가 6개일 때 Teacher 레이어를 2개씩 구분했을 때 Out을 가지고 옴 
(즉, student의 i번째 layer는 teacher의 2*i번째 layer)

논문에서는 각 로스의 중요도와 초기화가 실험적으로 굉장히 중요하다고 말함.


### TinyBERT

<div align="center">
<img src="https://github.com/user-attachments/assets/b599bdaf-7cd7-4303-94d9-a471793747b7"/>
</div>

본 논문은 BERT의 다양한 layer에서 값을 뽑아 KD를 수행하는 방법을 제안하며 높은 성능을 보인다.

논문에서 제안하는 loss는 다음과 같다. <br>
- Embedding layer distillation loss (MSE)
- transformer distillation method(Attention based + Hidden state)
- Prediction-layer Distillation(hinton loss)


