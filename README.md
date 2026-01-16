# ZIT(Z image turbo)모델 LoRA 훈련 및 사용 - img2img 수채화 만들기
* 생성모델로 원본 이미지의 형태를 유지하기는 어려움.
* IP Adapter를 사용할 경우 가능 NST와 비슷한 결과를 내는 것이 가능함.
* LoRA를 훈련을 하더라도 프롬프트를 추가하는 것이 더 좋은 결과를 나타냄.
  * ex)동일한 프롬프트일시 LoRA를 튜닝한 것이 원하는 이미지 스타일과 비슷하게 나옴
 
## 문제점
* 입을 닫고 있지만 입이 약갈 열린다.
* 밑의 사진과 같이 특정 부분이 전혀 다른 이미지가 생성되는 경우가 있다.

## 개선
* 훈련된 LoRA를 사용하더라도 프롬프트를 통해서 더 좋은 결과를 내는 것이 가능하다.

Z image turbo : [모델](https://github.com/Tongyi-MAI/Z-Image)
AI Toolkit : [프로젝트 설정](https://github.com/ostris/ai-toolkit/tree/main/config/examples)
VideoX-Fun : [이미지 컨트롤](https://github.com/aigc-apps/VideoX-Fun)

## 훈련데이터 생성
ZIT모델을 사용하여 수채화 백그라운드 이미지 생성
Generating_learning_data.python 실행시 300장의 이미지 생성
.txt파일은 생성된 이미지의 프롬프트를 사용.
* 동일 모델을 사용하여 데이터를 구축하여 .txt파일을 삭제하여 훈련.







## LoRA 훈련 세팅 yaml 기능 설명
### 1. 프로젝트 및 경로 설정 (General)
| 항목 | 설정값 | 설명 |
| :--- | :--- | :--- |
| **Job 유형** | `extension` | 기존 모델의 기능을 확장하는 작업 |
| **작업 이름** | `zimage_spring_watercolor_style` | 결과물 식별을 위한 고유 명칭 |
| **트리거 워드** | `<springwc>` | 생성 시 스타일을 불러오는 핵심 키워드 |
| **학습/DB 경로** | "경로" | 학습 결과 및 로그 저장 위치 |

### 2. 모델 및 아키텍처 (Model & Arch)
* **베이스 모델**: `Tongyi-MAI/Z-Image-Turbo` (알리바바 개발 고속 생성 모델) - 허깅페이스 다운로드() or 모델 경로
* **아키텍처**: `zimage:turbo` (Flow-matching 기반 터보 구조)
* **보조 어댑터**: `zimage_turbo_training_adapter_v2.safetensors` 활용
* **최적화**: `low_vram: true` 및 `bf16` 연산 적용으로 효율성 극대화

### 3. LoRA 네트워크 상세 (Network)
스타일을 원본 모델에 주입하기 위한 신경망 설정입니다.
* **Type**: `lora`
* **Rank (Linear/Conv)**: `16 / 8` (학습할 정보량의 밀도)
* **Alpha (Linear/Conv)**: `16 / 8` (학습 가중치의 강도)
* **데이터 정밀도**: `bf16` (학습 효율 및 메모리 절약)

### 4. 데이터셋 및 학습 파라미터 (Training)
| 항목 | 설정값 | 비고 |
| :--- | :--- | :--- |
| **데이터 경로** | `.../zimage_spring_watercolor_style` | 수채화 학습 이미지 폴더 |
| **해상도** | `512 x 512` | 스타일 전이에 최적화된 해상도 |
| **총 학습 단계** | `3000 Steps` | 이미지 약 300장 규모에 적합한 반복 횟수 |
| **학습률 (LR)** | `0.0001` | 가중치 업데이트 속도 |
| **최적화 도구** | `adamw8bit` | 8비트 최적화로 VRAM 부담 완화 |
| **학습 대상** | `UNet: ON / Text Encoder: OFF` | **스타일 전이**를 위해 이미지 구조만 학습 |

### 5. 검증 및 샘플링 (Validation)
학습 품질을 모니터링하기 위한 설정입니다.
* **샘플링 주기**: `200 Steps` 마다 테스트 이미지 생성
* **샘플러**: `flowmatch` (Turbo 전용 스케줄러)
* **추론 설정**: `Steps: 8` / `Guidance Scale: 1` (적은 단계로 고품질 생성)
* **부정 프롬프트**: "" (실사 느낌 배제용)

---
**Note:** 본 설정은 수채화 특유의 질감을 살리기 위해 `content_or_style: "style"` 모드가 활성화되어 있습니다.






## LoRA 훈련
```bush
conda activate 가상환경
cd ai-toolkit
pip install -r requirements.txt
python run.py config\examples\z_image_lora.yaml
```
