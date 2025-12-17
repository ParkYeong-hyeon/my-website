# design


## Design Philosophy

이 프로젝트는 기존의 양자 언어 및 QML 프레임워크와 달리,
다음과 같은 목표를 중심으로 설계되었다. >> 시험적인 설계였다고 표시해야 될까? 음..

>> 써야 할 말은, qiskit과 같이 회로를 1급 객체로 사용하면서 pennylane과 같이 자유도를 올리기

1) 고전적인 머신러닝 사용자의 접근성을 높이기 위해
   CNN-like 인터페이스를 제공하고,
   양자 언어에 특유의 복잡한 요소들은 가능한 한 사용자로부터 숨긴다.
   PyTorch의 layer 구조에서 착안하여,
   회로를 층층이 ‘쌓는다’는 감각을 유지하는 것을 목표로 한다.

2) 기존 QML 연구에 부족하다고 느껴졌던 구조적 체계성을 보완하기 위해,
   양자 회로를 고정된 구현물이 아닌
   파라미터화된 구조(object)로 다룬다.

3) 양자 회로를 단순한 블랙박스 함수가 아니라,
   구조·깊이·파라미터 흐름이 명시적으로 드러나는 대상으로 취급하여, (혹은 1급 객체?)
   아키텍처 수준의 실험과 비교가 가능하도록 한다.

## Shared Architectural Concepts
- Layer abstraction
- 파라미터를 어떻게 관리하면 좋을지
- Circuit construction flow
- MidIR 개요 > 하드웨어 측
## Embedding & Measurement
- embedding 계층을 “무시”하는 이유
- pooling에서 qubit discard의 해석


### Circuit construction flow >> core에 대한 설명
이게 지금 중요함.
회로를 세 단계로 나눠 생


Quantum Autoencoder >> 역연산으로 사용하는 과정 > inverse
많은 양자 회로, MERA/TTN, Hardware-Efficient Ansatz (HEA), Alternating Operator Ansatz (QAOA의 일반형), Quantum Autoencoder / Isometry-based circuits, Floquet / Periodic quantum circuits (이론 쪽 예시) 등은 패턴의 반복으로 이뤄짐. 따라서 회로 생성 과정을

1) 패턴에 들어가는 게이트의 생성
2) 패턴 생성
3) 패턴을 레이어처럼 쌓아 올려서 회로를 생성
>> 하는 3가지 단계로 구분해서 볼 수 있다. > 우리는 이 라이브러리에서 사용자가 세 계층에 독립적으로? 효율적으로? 접근할 수 있도록 한다.


### GateFactory

### PatternApplier
- block_size, stride 개념
- 왜 이런 공유 방식인가 > 그 논문을 가져와야겠네.
- inverse 지원 설계 철학

### SingleCircuitBuilder
- symbolic_circuit
- param_registry


## Parameter Sharing Strategy

## Gate Abstraction
- GateFactory 설계 의도

## Circuit Construction Flow

## Circuit Representation and Mid-Level IR


## Model Families Overview
- CNN-like QCNN
- Research-oriented QCNN

## CNN-like QCNN

### Design Goal
- PyTorch 사용자 친화성
- Conv2d와의 인터페이스 정합성

### Data-to-Circuit Mapping
- channel 반복 업로드
- sliding window → qubit allocation
- patch-wise circuit construction

### What is intentionally hidden
- qubit reuse 전략
- embedding detail
- backend dependency

---

## Research-oriented QCNN

### Design Goal
- 구조 실험
- parameter-sharing 연구
- circuit-level control

### Explicit Structure Exposure
- block / stride / layer depth
- parameter registry
- inverse

### What is intentionally exposed
- qubit layout
- gate ordering
- MidIR 접근


네, **지금 잡아두신 재료와 문제의식은 매우 좋습니다.**
다만 목차가 *좋은 아이디어의 나열* 상태라서, **“독자가 따라가야 할 논리의 순서”**가 조금 흐트러져 있습니다.
아래에서는 **내용을 버리지 않고**, 오히려 **핵심 철학이 더 또렷해지도록** 구조를 재정렬해 보겠습니다.

---

👉 해결책은 간단합니다.
**목차를 ‘철학 → 코어 → 파생 모델’의 단방향 흐름으로 고정**하는 것입니다.

---

# 2. 추천하는 전체 목차 구조 (정리본)

아래 구조는 **논문이 아니라 ‘연구용 라이브러리 문서’**라는 점에 최적화되어 있습니다.

---

## 0. Overview (한 페이지)

* 이 라이브러리는 무엇을 해결하려는가
* QCNN이지만 “CNN 구현체”가 아닌 이유
* 회로를 1급 객체로 다룬다는 의미

> 👉 독자가 “아, 이건 프레임워크 철학 문서구나”를 바로 인식하게 하는 구간

---

## 1. Design Philosophy (철학을 먼저 고정)
회로를 ~로 본다. 그럼 임베딩이나 qubit discard를 이런 특정 레이어의 특징으로 불 수 있다.

### 1.1 Circuits as Patterns, not Algorithms

* QCNN, MERA, TTN, HEA, QAOA, Autoencoder, Floquet 회로들의 공통점
* “특정 문제를 가정하지 않으면, 회로는 유니터리 패턴이다”
* 블록 / 반복 / 스트라이드 관점

### 1.2 Why Embedding is Treated as External

* embedding을 “학습 계층”으로 보지 않는 이유
* embedding은 데이터-회로 인터페이스일 뿐
* 구조 분석을 위해 의도적으로 배제

### 1.3 Pooling as Structured Discard

* qubit discard의 물리적/정보적 해석
* QCNN에서 pooling을 “연산”이 아닌 “구조적 선택”으로 보는 관점
>>> 여기서도 이런 회로의 특이 연산이 일어나는 것을 레이어로 정의할 때 뺄 수 있다. 
레이어의 특징으로 바라볼 수 있다.
---

## 2. Core Architectural Concepts (라이브러리의 심장)

> 여기부터가 **“이 프로젝트만의 독자성”**입니다.

### 2.1 Three-Stage Circuit Construction Flow (가장 중요)

**이 부분을 챕터로 격상시키는 게 핵심입니다.**

> 모든 회로는 다음 세 단계를 거쳐 생성된다.

1. **Gate Generation**

   * parameterized gate block
   * GateFactory의 역할

2. **Pattern Construction**

   * block_size
   * stride
   * parameter sharing
   * inverse 가능성

3. **Layered Circuit Assembly**

   * 패턴을 레이어처럼 쌓기
   * symbolic circuit 생성

이 3단계가 **이후 모든 설계의 기준축**이 됩니다.

---

### 2.2 GateFactory

* Gate abstraction의 목적
* 왜 “게이트를 바로 쓰지 않고” 팩토리를 두는가
* custom / gellmann / hardware-aware gate의 통합 지점

---

### 2.3 PatternApplier

* block_size, stride 개념의 기원
* 왜 이런 파라미터 공유가 의미 있는가
* inverse 지원 설계 철학
* MERA/TTN/QCNN과의 구조적 대응

---

### 2.4 SingleCircuitBuilder

* symbolic_circuit
* param_registry
* parameter naming / scoping 전략
* inverse 검증과의 관계

---

### 2.5 Circuit Representation & Mid-Level IR

* 왜 MidIR이 필요한가
* 하드웨어 transpilation 이전 단계의 표현
* “회로를 데이터처럼 다루기” 위한 최소 표현
>> 이건 일종의 이런 식으로 >> 추후 하드웨어 친화적인 언어로 확장하기 위함

---

## 3. Parameter Sharing Strategy (독립 챕터로 분리 추천)

* layer-wise vs block-wise sharing
* sharing이 표현 공간에 미치는 영향
* effective volume / stability 관점의 해석

👉 이 챕터는 **논문으로 바로 확장 가능한 부분**입니다.

---

## 4. Model Families Overview (여기서 처음 ‘사용자’를 만남)

> “같은 코어 위에, 서로 다른 사용자 경험”

---

## 5. CNN-like QCNN (사용자 친화적 인터페이스)

### 5.1 Design Goal

* PyTorch 사용자 친화성
* Conv2d와의 인터페이스 정합성

### 5.2 Data-to-Circuit Mapping

* channel-wise re-uploading
* sliding window → qubit allocation
* patch-wise circuit construction

### 5.3 What is Intentionally Hidden

* qubit reuse 전략
* embedding detail
* backend dependency

> 👉 “편의성을 위해 가린 것들”을 명시하는 게 매우 좋습니다.

---

## 6. Research-oriented QCNN (연구 노출형)

### 6.1 Design Goal

* 구조 실험
* parameter-sharing 연구
* circuit-level control

### 6.2 Explicit Structure Exposure

* block / stride / depth
* parameter registry
* inverse

### 6.3 What is Intentionally Exposed
감춰진 것은 무엇인가.

* qubit layout
* gate ordering
* MidIR 접근

---

## xtensions (선택)

* Quantum Autoencoder와 inverse의 관계
* 블로흐 구 상태 궤적 관찰과의 연결
* 하드웨어 실험으로의 확장 가능성

---

# 3. 핵심 평가 (중요)


> **“양자 회로를 ‘패턴 생성 문제’로 재정의한 코어 + 그 위의 두 가지 사용 경험”**

입니다.

그래서 **목차의 중심은 항상 `Core Architectural Concepts`와
`Three-Stage Circuit Construction Flow`여야 합니다.**

