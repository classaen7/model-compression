# Weight factorization & Weight Sharing

Weight Factorization은 큰 크기의 가중치 행렬을 더 작은 두 개의 행렬로 분해하여 표현하는 방법이다.

Weight Sharing은 동일한 가중치를 여러 층(layer)에서 공유함으로써 모델의 복잡성을 줄이고, 파라미터 수를 감소시키는 방법이다.

SOTA 모델들은 점점 크기가 커짐에 따라 모델의 크기를 키워서 distil하는 것이 common practice이지만 하드웨어의 메모리 제한이나, training의 communication overhead가 존재한다.

### ALBERT : A Lite BERT for Self-supervised Learning of Language Representations
해당 논문은 BERT을 대상으로 Weight factorization와 Weight Sharing를 적용한 논문이다.

이는 모델을 작게만든다기 보다는, 모델을 효율적으로 더 크게 만들기 위한 방법을 제시한다.
이를 위해 다음과 같은 방법을 제시한다.

- **Cross-layer parameter sharing**

parameter를 sharing 함으로 (BETR-large)18배의 파라미터를 줄일 수 있었다고 한다. (하지만 모델의 실제 size가 줄어드는 것은 아니다. 실제 속도는 Factored Embedding을 포함해서도 1.7배의 향상이다.) <br>

weight sharing은 **network parameter stabilizing**에 효과가 있다.
<p align="center">
<img src="https://github.com/user-attachments/assets/07fc0556-15c1-4c6d-887e-7dea66509c11"/>
</p>
각 레이어의 입력과 출력의 embedding이 BETR-large에 비해 수렴함을 확인할 수 있다.

- **Sentence Order Prediction(SOP)**

Next Sentence Prediction(NSP)는 Masked Language Modeling(MLM)에 비해 너무 쉬웠고, NSP는 topic prediction, coherence prediction을 위함이었지만 MLM loss와 ovelap됐다.

SOP는 inter-sentence coherence에 더 초점을 맞춘다. 또한 SOP loss는 NSP loss task까지 잘 수행한다는 실험을 보인다. (NSP는 SOP task에 도움을 주지 못함)

- **Factored Embedding Parameterization**

WordPiece Embedding : context-independent representations을 학습함

HiddenLayer Embedding : context-dependent representations을 학습함

BERT의 목적은 context-dependent representations을 위함이지만, WordPiece Embedding이 Hiddenlayer Embedding과 Coupling 되는 경우가 존재함. 

<p align="center">
<img src="https://github.com/user-attachments/assets/9ce6aea6-e00e-48e3-9fe4-65884a278f45"/>
</p>

따라서 WordPiece Embedding을 대상으로 Factorization을 적용하여 기존 크기 `O(V * H)`를 `O(V * E + E * H)`로 변경하여 파라미터 수를 크게 줄임


