# Quickstart
기본적인 라이브러리에는 두 타입의 모델이 정의되어 있다.
최상위 객체는 단순히 Torch의 모듈과 같은 방식으로 사용 가능하다.

1) CNN-like QCNN model

``` 
import torch
from qnn.models import CNNLikeQCNN  

model = CNNLikeQCNN(...)
x = torch.randn(2, 1, 28, 28)
logits = model(x)
print(logits.shape)
```

2) quantum native QCNN model
```
from qnn.component import QModel

```
