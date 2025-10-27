# SSAFY AI 챌린지 개발 보고서

## 팀 정보

**팀명:** 서울1반 4조  
**팀원:** 정환승, 강태인, 나성현, 백우성\
**소속:** SSAFY  서울 1반

---

## 프로젝트 개요

### Overview
- 이미지로 묻고, AI로 답하다 - 이미지 기반 질의응답 모델 개발 AI 챌린지

### 대회개요
- VQA(Visual Quesion Answering)모델은 인간의 시각적 사고에 가장 가까운 형태의 AI 기술입니다. 이번 AI 챌린지는 이미지와 텍스트를 함께 이해하는 멀티모달 인공지능을 직접 개발하고 평가하는 대회로 실제 환경에서 활용 가능한 AI 모델을 개발하며 인공지능 기술을 체험하게 됩니다.

### 목표

- **Visual-Language Model (VLM)**모델링을 통해 멀티모달 AI의 개념과 작동 원리 이해
- **Image + Text 데이터 전처리, 실험, 튜닝 과정** 체험  
- 대형 AI 모델 개발 전 과정을 직접 수행하며  
  **모델 성능 개선 및 실험 관리 역량 강화**

### 진행 전략

1. **Baseline 모델 분석 및 실행**
2. 개선 가능한 부분 도출 후 다양한 모델을 적용해보고, 방법론 적용
3. **Hyperparameter Tuning**과 **Data Augmentation**을 적용해가며 모델의 학습 상태를 지속적으로 확인하고, 성능 향상을 위한 실험을 진행
4. **입력 데이터 고도화**: SOM 기법, Task Type 지정, OCR, Object Box, 해상도 개선
5. **Validation 전략 및 Rule-based Ensemble**을 통한 모델 안정화  

---

## 베이스라인 및 문제 정의

### 문제 인식

- 사람이 봐도 헷갈리는 시각적 질문 문제들 존재
- 모델의 추론 성능이 부족하거나 입력 이해가 제한됨  
  → “모델이 문제일까, 데이터가 문제일까?”에 대한 탐구 진행

---

## 모델 실험 및 비교

### 모델 라인업

| 모델                        | 목표           | 비고                          |
| ------------------------- | ------------ | --------------------------- |
| Qwen2.5 VL 3B             | Baseline 3B  | 기존 Baseline                    |
| Qwen3 VL 8B               | 모델 크기 개선     | New Baseline Model로 채택      |
| Llava OneVision 1.5 VL 8B | 새로운 모델 발굴    | 성능이 기대에 미치지 못함                    |
| Varco Vision 1.7B         | 한국어 특화 모델 발굴 | 한국어 특화 but 모델이 너무 compact함. |

### Qwen2.5 → Qwen3 개선점

- 학습 데이터 다양성 및 도메인 확장  
- 멀티모달 처리력 강화 (OCR, 문서, 도표 인식)  
- 프롬프트 순응도 및 안정성 향상  
- 한국어 포함 다국어 대응 성능 개선  

---

## 데이터 확장 (Data Augmentation)

### English & Korean VQA 데이터 증강

- **한국어/영문 VQA 데이터 병합 및 정제**
- **Image Data Augmentation** (resize, flip, color jitter, random crop 등) Online 적용
- **OCR 텍스트**를 추가하여 모델의 시각적 인식 보조  


### Image Resolution 실험

| Image Size | 결과              |
| ---------- | --------------- |
| 384        | 손실 낮지만 성능 보통    |
| 448        | Public 기준 최고 성능 |
| 1024       | 손실 증가           |

→ **최종 해상도: 448 통일**

---

## 입력 정보 확장 (Hint Engineering)

### Task Type + Hint

- 질문 유형별 하드코딩 (예: 비교, 색상, 위치 등)
- 각 유형에 맞는 **Prompt Hint** 추가  
  → 모델이 문제 성격을 명확히 이해하도록 유도

### Set of Mark (SOM)

- 객체를 정확히 구분하지 못하는 문제 완화로 시각 단서 강화

### Object Detection Bounding Box

- **Object Detection**과 결합하여 Bounding Box 마킹 제공으로 시각 단서 강화

### OCR text

- **OCR text Hint** 추가로 text 단서 강화  

---

## 프롬프트 엔지니어링 (Prompt Engineering)

### 목적

- 멀티모달 VQA에서 **추론 유도 + 답안 안정성 향상**
- 모델이 **관찰 → 근거 → 결론**의 추론 흐름을 따르도록 유도  

### 구성

| 유형                   | 설명                                        |
| -------------------- | ----------------------------------------- |
| Baseline             | 단순 질문 + 선택지(a/b/c/d) -> 답 출력                               |
| Reason Prompt (한글)   | 논리적 서술을 유도하되, 최종 출력은 a/b/c/d              |
| Reason Prompt (한영혼합) | “Final answer”, “Answer only” 등 영어 지시어를 혼합 |

### 성능 변화 (Qwen3-8B 기준)

| 구분           | Validation Loss | Train Loss |
| ------------ | --------------- | ---------- |
| Baseline     | 4.29            | 1.05       |
| Reason(한글)   | 3.11            | 0.76       |
| Reason(한영혼합) | **2.46**        | **0.60**   |

> 영어 지시어 혼합 시, 모델의 지침 이해도가 향상되고 수렴 안정화 효과가 확인됨.
---

## 하이퍼파라미터 튜닝 (Parameter Tuning)

### 목표

- 해상도 및 학습률 조정으로 **Val Loss 안정화** 및 효율 달성

### 주요 설정

| 항목         | 값 / 방법                                          | 설명                        |
| ---------- | ----------------------------------------------- | ------------------------- |
| 모델         | Qwen3-VL-8B                                     | Qwen2.5 대비 향상된 멀티모달 이해 성능 |
| 이미지 해상도    | 448                                             | Public 기준 최고 성능           |
| 학습률(lr)    | 1e-4 (cosine or linear scheduler + warmup 5~8%) | 안정적 수렴                    |
| Batch size | GPU VRAM 고려 자동 설정                               | 효율적 학습                    |
| Optimizer  | AdamW                                           | weight decay 적용           |

### 관찰

- **image_size=448**, **lr=1e-4 + cosine + warmup 5~8%** 조합이 가장 안정적  
- lr < 3e-5 → underfit / lr ≥ 3e-4 → 초기 진동  
- baseline 대비 val/loss 안정적 수렴 + VRAM 효율 개선  

---

## 검증 및 앙상블 전략

### Validation Strategy

- Train : Valid = 9 : 1 (3498 : 389)  
- **Task type(유사 문항)**이 균등 분포되도록 분할  
  → k-fold 미사용에 따른 일반화 저하 방지

### Decision Ensemble

- 서로 다른 설정의 **5개 모델** 결과를 **다수결 투표(≥3표)**  
- 동률 시 사전 지정 우선순위 모델로 타이브레이크  
- **성능 향상:** 단일 모델(0.93~0.94) → 앙상블(0.95113, +1~2%)  

---

## 실험 관리 (Experiment Management)

### 툴: **Wandb - Weights & Biases (W&B)**

#### 장점

- 실험 로그/하이퍼/버전 일원화  
- `train/valid loss`, LR, GPU 메모리 등 실시간 시각화  
- 코드·환경·모델 아티팩트 자동 기록 → 재현성 확보  
- Run 비교 및 그룹화로 빠른 의사결정  
- 협업 공유 및 리포트 관리 용이  
- Kaggle 연동 (offline → sync 지원)  

---

## 학습 및 기술 습득

### 주요 학습 내용

- **VLM (Visual-Language Model)** 구조와 개념  
- **LoRA**, **Set of Mark(SOM)** 기법  
- **Hyperparameter Tuning** 및 **Optuna** 라이브러리 활용  
- **PyTorch 실습**, **W&B 실험 관리**, **효율적 코드 관리**  

---

## 결과 및 성과

- **SSAFY AI 챌린지 Public 32위, 제출 수는 1위(열심히 실험했다)**
- Reason 프롬프트 및 Hint 기반 입력 개선으로  
  **VQA 문제 정확도 및 안정성 향상**
- 모델 실험 → 검증 → 앙상블까지 일관된 개선 루프 확립  

---

## 결론 및 소감

> 5일도 안되는 짧은 시간동안 진행한 AI 과제임에도 불구하고 시각-언어 통합 모델의 학습 과정과  
> 다양한 실험 설계 및 검증 전략을 체험할 수 있었습니다. 덕분에 VQL 과제에 대한 감을 얻을 수 있었습니다.
> 
> 아쉬웠던 점은 기간이 너무 짧았고, 활용할 수 있는 컴퓨팅 환경이 제한적이었기 때문에 Set of Mark 등 
> 원하는 결과물을 내기위해 계속 디벨롭해보지 못했다는 점 입니다. 
> 때문에 추후 다른 프로젝트를 진행하게 되었을 때, 이 경험을 반영해 더 좋은 결과물을 내보고 싶다는 생각이 들었습니다. 
