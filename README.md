# SSAFY AI 챌린지

## 🧑‍💻 팀 정보

**팀명:** 서울1반 4조  
**팀원:** 강태인, 나성현, 백우성, 정환승  
**소속:** SSAFY 서울 1반

---

## 📘 프로젝트 개요

### 🎯 목표

- **Visual-Language Model (VLM)** 의 개념과 구조 이해
- **Image + Text 데이터 전처리, 실험, 튜닝 과정** 체험
- 대형 AI 모델 개발 전 과정을 직접 수행하며  
  **모델 성능 개선 및 실험 관리 역량 강화**

### 🚀 진행 전략

1. **Baseline 모델 분석 및 실행**
2. 개선 가능한 부분 도출 후 다양한 모델 실험
3. **Hyperparameter Tuning** 및 **Data Augmentation** 적용
4. **입력 데이터 고도화**: SOM 기법, Task Type 지정, OCR, Object Box, 해상도 개선
5. **Validation 전략 및 Rule-based Ensemble**을 통한 모델 안정화
6. 최종 결과 제출 → **Public Score 32위(제출 수 1위) 달성**

---

## 🧩 베이스라인 및 문제 정의

### 문제 인식

- 사람이 봐도 헷갈리는 시각적 질문 문제들 존재
- 모델의 추론 성능이 부족하거나 입력 이해가 제한됨  
  → “모델이 문제일까, 데이터가 문제일까?”에 대한 탐구 진행

---

## 🔍 모델 실험 및 비교

### 모델 라인업

| 모델  | 목표  | 비고  |
| --- | --- | --- |
| Qwen2.5 VL 3B | Baseline 3B | Baseline |
| Qwen3 VL 8B | 모델 크기 개선 | New Baseline Model로 채택 |
| Llava OneVision 1.5 VL 8B | 새로운 모델 발굴 | 성능이 어정쩡함 |
| Varco Vision 1.7B | 한국어 특화 모델 발굴 | 한국어 특화 but 모델이 너무 compact함. |

### Qwen2.5 → Qwen3 개선점

- 학습 데이터 다양성 및 도메인 확장
- 멀티모달 처리력 강화 (OCR, 문서, 도표 인식)
- 프롬프트 순응도 및 안정성 향상
- 한국어 포함 다국어 대응 성능 개선

---

## 🧮 데이터 확장 (Data Augmentation)

### English & Korean VQA 데이터 증강

- **한국어/영문 데이터 병합 구축**
- **Image Data Augmentation**: Online 적용
- **OCR 텍스트**를 추가하여 모델의 시각적 인식 보조

### Image Resolution 실험

| Image Size | 결과  |
| --- | --- |
| 384 | 손실 낮지만 성능 보통 |
| 448 | Public 기준 최고 성능 |
| 1024 | 손실 증가 |

→ **최종 해상도: 448 통일**

---

## 💡 입력 정보 확장 (Hint Engineering)

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

## ✍️ 프롬프트 엔지니어링 (Prompt Engineering)

### 목표

- 멀티모달 VQA에서 **추론 유도 + 답안 안정성 향상**

### 구성

| 구분  | 내용  |
| --- | --- |
| Baseline | 질문 + (a/b/c/d) → 답만 출력 |
| Reason Prompt(한글) | 관찰 → 근거 → 결론 흐름 유도 |
| Reason Prompt(한영혼합) | 영문 키워드(Answer only, Final answer 등) 혼합 |

### 결과 (Qwen3-8B 기준)

- **Validation Loss**: 4.29 → 3.11 → 2.46
- **Train Loss**: 1.05 → 0.76 → 0.60
- 영문 지시어 혼합 → 모델의 지침 이해 및 수렴 안정화 향상  
  → Public 기준 **Reason(한글)** 프롬프트가 최고 성능

---

## ⚙️ 하이퍼파라미터 튜닝 (Parameter Tuning)

### 목표

- 해상도 및 학습률 조정으로 **Val Loss 안정화** 및 효율 달성

### 관찰

- **image_size=448**, **lr=1e-4 + cosine + warmup 5~8%** 조합이 가장 안정적
- lr < 3e-5 → underfit / lr ≥ 3e-4 → 초기 진동
- baseline 대비 val/loss 감소 + VRAM 효율 개선

---

## 🧾 검증 및 앙상블 전략

### Validation Strategy

- Train : Valid = 9 : 1 (3498 : 389)
- **Task type(유사 문항)**이 균등 분포되도록 분할  
  → k-fold 미사용에 따른 일반화 저하 방지

### Decision Ensemble

- 서로 다른 설정의 **5개 모델** 결과를 **다수결 투표(≥3표)**
- 동률 시 사전 지정 우선순위 모델로 타이브레이크
- **성능 향상:** 단일 모델(0.93~0.94) → 앙상블(0.95113, +1~2%)

---

## 📊 실험 관리 (Experiment Management)

### 툴: **Wandb - Weights & Biases (W&B)**

#### 장점

- 실험 로그/하이퍼/버전 일원화
- `train/valid loss`, LR, GPU 메모리 등 실시간 시각화
- 코드·환경·모델 아티팩트 자동 기록 → 재현성 확보
- Run 비교 및 그룹화로 빠른 의사결정
- 협업 공유 및 리포트 관리 용이
- Kaggle 연동 (offline → sync 지원)

---

## 📚 학습 및 기술 습득

### 주요 학습 내용

- **VLM (Visual-Language Model)** 구조와 개념
- **LoRA**, **Set of Mark(SOM)** 기법
- **Hyperparameter Tuning** 및 **Optuna** 라이브러리 활용
- **PyTorch 실습**, **W&B 실험 관리**, **효율적 코드 관리**

---

## 🏁 결과 및 성과

- **SSAFY AI 챌린지 Public 32위, 제출 수 1위**
- Reason 프롬프트 및 Hint 기반 입력 개선으로  
  **VQA 문제 정확도 및 안정성 향상**
- 모델 실험 → 검증 → 앙상블까지 일관된 개선 루프 확립

---

## 🙏 결론 및 소감

> 이번 프로젝트를 통해 시각-언어 통합 모델의 학습 과정과  
> 다양한 실험 설계 및 검증 전략을 체험하며  
> “모델의 성능은 데이터, 입력 설계, 실험 관리가 함께 만들어진다”는  
> 중요한 교훈을 얻었습니다."
