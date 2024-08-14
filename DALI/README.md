# DALI (Data Loading Library)
모델의 Inference 속도 향상을 위해 NVIDIA GPU를 사용하여 Data Preprocessing 속도를 높이는 기법이다.

# 배경
CPU와 GPU의 Capacity 차이가 점점 더 심화되고있다. <br>
CPU에서 수행되는 데이터 전처리의 복잡성은 점점 늘어나고 있지만 CPU는 GPU만큼의 성장 곡선을 보여주지 못하고 있다. <br>
DALI는 CPU의 성능으로 인해 GPU로 데이터를 넣어주는(Data Feeding) 구간에 bottleneck이 될 것이라고 한다.<br>
따라서 전처리의 연산들을 CPU가 아닌 GPU에서도 수행하고자 한다.

# DALI 구성

```py
from nvidia.dali import fn

@nvidia.dali.pipeline_def
def rn50_pipeline():
    encoded, label = fn.readers.file(file_root=image_dir)
    decoded = fn.decoders.image(encoded, ...)
    resized = fn.resize(decoded, size=[224, 224])
    normalized = fn.crop_mirror_normalize(resized, mean=mean, std=std)
    return normalized, label

pipe = rn50_pipeline(batch_size=128, num_threads=4, device_id=0)
pipe.build()
image, labels = pipe.run()
```
DALI는 pipeline을 생성한 후 객체를 `build()`하고 `run()`하면 일종의 iterator처럼 이미지를 생성하게 된다.

# GPU offloading
`decoded = fn.decoders.image(encoded, device="mixed")`
decoded 과정에서 device를 "mixed"로 설정하면 **GPU offloading**이 된다.

> **Offload**란 특정 작업이나 연산을 원래 수행하던 주체(예: CPU)에서 다른 장치나 시스템(예: GPU, 네트워크 장비)으로 넘겨서 처리하는 것을 의미한다.

<p align="center">
<img src="https://github.com/user-attachments/assets/5ccc692a-154e-49d8-bf66-4de4ef330b4e"</img>
</p>

바로 위 처럼 기존에 CPU가 처리했던 "초록색 "영역을 GPU가 수행하게 되는 것이다.

# DataLoader
```py
pipe = TrainPipe( . . . )
train_loader = DALIClassificationIterator(pipe)

for i, data in enumerate(train_data):
    x, y = data
    pred = model(x)
    loss = loss_func(pred, y)
    backward(loss, model)
```
Pytorch와의 호환을 위해 DataLoader를 대체 가능하게 하는 iterator를 제공한다. 따라서 for문 내부의 구조는 동일하다.


# 실험 및 결과
- 실험 환경
1. 102개의 클래스를 가지는 Flower dataset을 활용 (클래스당 40 ~ 258장)
2. DataLoaindg 비교: 기타 augmentation없이 resizing만 수행 (224, 224)
3. Training & Inference time 비교: Resnet18 pretrained, FC layer 교체

- 비교군
1. CPU 환경에서 전처리 수행
2. decode 과정은 cpu로, resize 이후 과정은 GPU로 수행
3. decode 과정부터 cpu, gpu 적절히 사용하는 **mixed**로 전처리 수행

3가지 비교를 통해 DALI(Mixed)가 가장 빠름을 확인하였다.

# 🔗 Reference
[NVIDIA DALI Documentation](https://docs.nvidia.com/deeplearning/dali/user-guide/docs/index.html)   
