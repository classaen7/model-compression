# TensorRT
TensorRT는 학습된 딥러닝 모델을 최적화하여 Nvidia GPU 상에서 Inference 속도를 크게 향상시키는 모델 최적화 엔진이다.<br>
pytorch, tensorflow, mxnet 등 다양한 framework를 지원한다.

# 5가지 모델 최적화 기법
![title](https://github.com/user-attachments/assets/f32743de-1d13-4ae2-a5fe-edd642494909)   <br>
학습된 모델이 들어오면 위와 같은 기법을 사용해서 모델 최적화를 수행한다.

# ONNX
TensorRT는 일반적으로 ONNX를 거쳐서 수행된다.

> **ONNX**<br>
ONNX(Open Neural Network Exchange)는 그래프의 공통된 표현(operator)를 제공하여 프레임워크 간의 변환을 가능하게 하여 서로 다른 딥러닝 프레임워크에서 생성된 모델이 상호 호환될 수 있도록 도와주는 프레임워크이다.<br>

# PyTorch → ONNX
Pytorch는 대부분의 layer에 대해 onnx export를 지원한다.<br>
ONNX로 변환하기 위해선 모델의 입력 데이터 크기를 넘겨주어야 한다.
<font color="gray"> 지원하지 않는 operator에 대한 숙지가 필요하다. </font>

``` py
# 모델에 대한 입력값
x = torch.randn(batch_size, 1, 224, 224, requires_grad=True)
torch_out = torch_model(x)

# 모델 변환
torch.onnx.export(torch_model,               # 실행될 모델
                  x,                         # 모델 입력값 (튜플 또는 여러 입력값들도 가능)
                  "super_resolution.onnx",   # 모델 저장 경로 (파일 또는 파일과 유사한 객체 모두 가능)
                  export_params=True,        # 모델 파일 안에 학습된 모델 가중치를 저장할지의 여부
                  opset_version=10,          # 모델을 변환할 때 사용할 ONNX 버전
                  do_constant_folding=True,  # 최적화시 상수폴딩을 사용할지의 여부
                  input_names = ['input'],   # 모델의 입력값을 가리키는 이름
                  output_names = ['output'], # 모델의 출력값을 가리키는 이름
                  dynamic_axes={'input' : {0 : 'batch_size'},    # 가변적인 길이를 가진 차원
                                'output' : {0 : 'batch_size'}})
```

# TensorRT 변환과정
![title](https://github.com/user-attachments/assets/dcaaafe5-7fcc-4ec7-88d7-f37e5c79c65e)   <br>
🔗[자세한 Step 확인하기](https://docs.nvidia.com/deeplearning/tensorrt/quick-start-guide/index.html)   

TensorRT는 위처럼 5단계의 변환 과정이 필요하다.<br>
<font color="gray">Select A Precision : float32, float16, int8을 설정 (int8에서는 calibration이 필요)</font><br>

# ONNX → TensorRT : CLI (trtexec)
별도의 코드 없이 CLI 툴을 사용하여 쉽게 변환할 수 있다. <br>

`trtexec`라는 커맨드를 시작으로 다양한 커맨드라인 인자를 넣어줌으로 다음과 같이 다양한 기능을 수행할 수 있다. <br>
```
trtexec --onnx=export/test_model.onnx --workspace=8000
        --maxBatch=16 --saveEngine=export/cli_test_model_float16.trt 
        --float16 --dumpProfile --verbose
```
CLI args는 `trtexec --help`를 통해 확인할 수있다.


# ONNX → TensorRT : API
`trtexec`에서 사용한 인자들을 코드 내에서 설정하여 세팅할 수 있다. <br>
ONNX에서 TensorRT의 변환 워크플로우는 다음과 같다.<br>
![title](https://github.com/user-attachments/assets/a2f086e4-72d5-4ea5-b54f-b7fbb82c9c2a) <br>

TensorRT는 builder를 통해 특정 프레임워크로 정의된 Network를 직렬화(Serialize)를 수행하여 Engine을 생성한다. <br>
생성된 Engine을 가지고 Runtime을 수행한다.

- 예시 코드
```py
# TensorRT 변환준비, 변환 관련 class 정의
import onnx
import tensorrt as trt

# 변환된 ONNX 모델 Load
with open("onnx_model.onnx", "rb") as f:
    onnx_data = f.read()

# 모델 입력 데이터 크기 설정
network_input_shape = (8, 3, 32, 32) # NCHW

# TensorRT 로거 세팅(디버깅 목적)
trt_logger = trt.Logger(trt.Logger.VERBOSE)

# Batch Size 크기 설정
explicit_batch = [
    1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
]

# builder: 모델 변환, network: 모델 정의, parser: ONNX 모델 parser 파싱된 모델을 network로 전달
with trt.Builder(trt_logger) as builder, builder.create_network(
    *explicit_batch
) as network, trt.OnnxParser(network, trt_logger) as parser:
    ......
```

```py
# 모델 변환 파라미터 설정, 모델 변환
with trt.Builder(trt_logger) as builder, builder.create_network(
    *explicit_batch
) as network, trt.OnnxParser(network, trt_logger) as parser:
    ......

with trt.Builder(trt_logger) as builder, builder.create_network(
        *explicit_batch)
as network, trt.OnnxParser(network, trt_logger) as parser: 
    parser.parse(onnx_data)
    network.get_input(0).shape = (self.batch_size,) + \ 
        network.get_input(0).shape[1:]
    builder.max_batch_size = 1

    config = builder.create_builder_config() 
    # GPU 최대 메모리 할당량 설정
    config.max_workspace_size = 8 << 30 # 8GiB GPU memory. 
    config.set_flag(trt.BuilderFlag.GPU_FALLBACK) 
    # FP16 변환 플래그 설정
    config.set_flag(trt.BuilderFlag.FP16) 

    profile = builder.create_optimization_profile() 
    # 모델 최적화 프로파일리 설정
    profile.set_shape( "images", network_input_shape, network_input_shape, network_input_shape )
    config.add_optimization_profile(profile)
    # Network와 config를 builder에게 전달하여 TensorRT 모델 변환
    # (모델 최적화 수행과 동시에 각 GPU에 최적화된 연산자를 이용하도록 변환됨)
    engine = builder.build_engine(network, config)
```

```py
# Int8 quantization
# int8 calibration을 위한 이미지 필욜
# cache_file이 없는 경우 imt_dir 내의 이미지를 통하여
# calibration을 수행하고, cache를 생성
int8_calibrator = Int8Calibrator(
        img_dir="calibration/image/path",
        img_extension="png",
        batch_size=32,
        preprocess_type="CIFAR10",
        cache_file="export/int8_cache.bin",
)

profile.set_shape(
        "images", 
        network_input_shape,
        network_input_shape,
        network_input_shape
)
# Config 내에 int 8 flag와 int8_calibrator를 설정
config.set_flag(trt.BuilderFlag.INT8)
config.int8_calibrator = int8_calibrator
config.set_calibration_profile(profile)
config.add_optimization_profile(profile)
engine = builder.build_engine(network, config)
```

# 🔗 Reference
[NVIDIA Developer](https://developer.nvidia.com/)  <br>
[NVIDIA TENSORRT DOCUMENTATION](https://docs.nvidia.com/deeplearning/tensorrt/quick-start-guide/index.html)   <br>
[PyTorch 모델을 ONNX으로 변환하고 ONNX 런타임에서 실행하기](https://tutorials.pytorch.kr/advanced/super_resolution_with_onnxruntime.html)   <br>
[trtexec github](https://github.com/NVIDIA/TensorRT/tree/main/samples/trtexec)   

