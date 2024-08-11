# Quantization
물리학에서 양자화(量子化, 영어: quantization)란 좁은 의미에서 거시적으로 연속적인 양을 어떤 기본 단위(양자)의 정수배로 측정하는 양으로 재해석하는 것을 뜻한다. [[위키백과]](https://ko.wikipedia.org/wiki/%EC%96%91%EC%9E%90%ED%99%94_(%EB%AC%BC%EB%A6%AC%ED%95%99))

## Fixed Point & Floating Point

- **Fixed Point : 고정 소수점**<br>
![title](https://private-user-images.githubusercontent.com/79098475/356880012-0def6b5e-b7e9-4eb3-b863-d66fa1ca17d9.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjMzOTI3NjgsIm5iZiI6MTcyMzM5MjQ2OCwicGF0aCI6Ii83OTA5ODQ3NS8zNTY4ODAwMTItMGRlZjZiNWUtYjdlOS00ZWIzLWI4NjMtZDY2ZmExY2ExN2Q5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODExVDE2MDc0OFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWRhYzYwNzE1YjlmZDQ1ZGI1MWNjOWFkYzYxN2JiYmNkMDY1YzJkNWE4MDU3ZmY2MWE1OWIyMWRhNjgwN2M3ZTAmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.OxloFsJZLPm7rv0di_7Uq-E6-e-mS5LVBChBr8sQpQQ)   

Fixed Point는 소수점 위치가 고정되어있다. 이는 숫자를 표현할 때 소수점이 항상 일정한 위치에 있다는 뜻이다. 

고정 소수점은 `m bit`의 **Signed Integer**와 `n bit`의 **Fractional Part**로 구성되어있다. 이때 `n bit`로 표현되는 소수부분은 $1/2^{-n}$의 해상도를 갖는다.

따라서 담을 수 있는 수의 개수가 동일하다. 이는 등간격으로 수가 표현되어 항상 동일한 해상도를 갖음을 의미한다.

- **Floating Point : 부동 소수점**<br>
![title](https://private-user-images.githubusercontent.com/79098475/356880019-182b73d5-6363-45c2-adfc-42ca6d329e09.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjMzOTI3NjgsIm5iZiI6MTcyMzM5MjQ2OCwicGF0aCI6Ii83OTA5ODQ3NS8zNTY4ODAwMTktMTgyYjczZDUtNjM2My00NWMyLWFkZmMtNDJjYTZkMzI5ZTA5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODExVDE2MDc0OFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTA4ZDk0ODdhOWQwZGNiMTdhNTJmZjU4MzljZjliYWU1NzdmMjg5YmQxNWI0MDUzYmYxOGIyMWYxMGYxZDc4NTImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.oZYBxUm-GteyiqjnD3upZ8tneYz9Qd5toylRK9Lb9_E)   

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
![title](https://private-user-images.githubusercontent.com/79098475/356888926-4c9b86c2-3ac5-4edb-b5d8-c479f82cf342.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjMzOTEzMTYsIm5iZiI6MTcyMzM5MTAxNiwicGF0aCI6Ii83OTA5ODQ3NS8zNTY4ODg5MjYtNGM5Yjg2YzItM2FjNS00ZWRiLWI1ZDgtYzQ3OWY4MmNmMzQyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODExVDE1NDMzNlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWUxZjA5NjkxOTIyYTE2YWZmOTgyNTRmM2I3MzA4NWRjNmUxYzQwMjU2NDBlYjc0NzJkNmZiZDQyYWU1NzllZDQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.OoE7dEwQjgAx7BkrQIzQnnJfJjre_71E8F1mFQjJ584)   

이때 구하는 값인 $Y_{q,i,j}$로 식을 정리하면 다음과 같다.

![title](https://private-user-images.githubusercontent.com/79098475/356889140-ce8ad934-3878-4f79-b94f-036ac29e2093.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjMzOTEzMTYsIm5iZiI6MTcyMzM5MTAxNiwicGF0aCI6Ii83OTA5ODQ3NS8zNTY4ODkxNDAtY2U4YWQ5MzQtMzg3OC00Zjc5LWI5NGYtMDM2YWMyOWUyMDkzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODExVDE1NDMzNlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWIxMzg2MTJiYTM1OWNkZDQ0M2U0NGE5NTI3OTQ5ZDIwZDBlNWU0YTUyYTQ4YjY1ZTc3YjIxZTE0ODdlMDE5ZjImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.rsQsu_RPP8ilzSI3j313YcF9-Qv7G97YOokyz0llMIs)   
이때 단순 연산횟수는 양자화 이전의 matrix multiplication보다 많아지지만 Integer type의 연산이므로 전력 소모도 적고 대체로 빠르게 수행할 수 있다.

## Activation Quantization
ReLU에 Quantization을 적용하는 유도식을 확인하겠다.

![title](https://private-user-images.githubusercontent.com/79098475/356891121-56761bd5-84bb-4ac0-914a-0200fe70f234.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MjMzOTMyMjUsIm5iZiI6MTcyMzM5MjkyNSwicGF0aCI6Ii83OTA5ODQ3NS8zNTY4OTExMjEtNTY3NjFiZDUtODRiYi00YWMwLTkxNGEtMDIwMGZlNzBmMjM0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDA4MTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwODExVDE2MTUyNVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTg4OWQ5NDFmY2E5MjJhY2I2YzE3YzdlMGZiMDViOWMyZTRiMDMyMWE5MDRlZjgwNmE3YTQ3NGI4ODE2YWU0NTcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.Ff8un4h6d2zVffJlh4tVgMFC__K5zfBz_yRx7xvXbno)

Quantization mapping과 유사하게 쉽게 유도할 수 있다.

