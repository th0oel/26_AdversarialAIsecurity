# 자율운항선박 이미지 분류 모델 적대적 AI 검증

선박 이미지 분류 AI(CNN, MobileNetV2)가 FGSM 적대적 공격에 얼마나 취약한지 측정하고, 실험 결과가 원자료와 일치하는지 자동으로 검증하는 연구입니다.

## 개요

- 테스트 이미지 781장으로 CNN·MobileNetV2의 기준(Clean) 성능 측정
- FGSM 공격(ε = 0, 0.01, 0.03, 0.05) 적용 후 정확도와 공격 성공률(ASR) 비교
- 3×3 가우시안·평균 필터 전처리 방어의 효과와 한계 비교
- manifest·모델 해시·CSV·JSON·문서 수치를 감사 스크립트로 교차 검증

## 핵심 결과

### Clean 성능

| 모델 | 정답 수 | 정확도 |
|---|---:|---:|
| CNN | 504 / 781 | 64.5% |
| MobileNetV2 | 613 / 781 | 78.5% |

### FGSM 공격 결과

| ε | CNN 정확도 | CNN ASR | MobileNetV2 정확도 | MobileNetV2 ASR |
|---:|---:|---:|---:|---:|
| 0.00 | 64.53% | 0.00% | 78.49% | 0.00% |
| 0.01 | 41.61% | 35.52% | 12.42% | 84.18% |
| 0.03 | 24.33% | 62.30% | 13.19% | 83.20% |
| 0.05 | 20.23% | 68.85% | 16.13% | 79.45% |

- **ASR**: 공격 전에는 맞혔지만 공격 후 틀린 이미지의 비율입니다. 분모는 공격 전에 정답을 맞힌 이미지 수입니다.
- MobileNetV2는 Clean 정확도가 더 높지만 ε=0.01에서 ASR이 84%로, **높은 정확도가 공격 강건성을 보장하지 않습니다.**

### 전처리 방어

- 가우시안·평균 필터는 기존 FGSM 이미지의 정확도를 일부 회복시켰습니다.
- 전처리까지 고려한 **방어 인지 FGSM**에는 효과가 크게 떨어졌습니다.
- 정상 이미지에서 CNN 정확도는 낮아지고 MobileNetV2 정확도는 높아졌습니다.

## 실험 설정

- 입력 범위: `[0, 1]` (`rescale=1./255`)
- 공격: Untargeted FGSM, 1 step
- 공격 식: `x_adv = clip(x + ε · sign(∇x L), 0, 1)`
- 교란 제약: `L∞ ≤ ε + 1e-6`

## 실행 방법

```bash
pip install -r requirements.txt
export PYTHONPATH=src

python -m pytest -q
python scripts/audit_research_evidence.py
python scripts/audit_paper_claims.py
```

Windows에서는 `export` 대신 `set PYTHONPATH=src`를 사용합니다.
모든 검사를 통과하면 exit code 0, 누락·변조·불일치가 있으면 exit code 1로 종료됩니다.

## 저장소 구성

| 경로 | 역할 |
|---|---|
| `configs/` | 클래스·테스트 manifest·실험 설정 |
| `src/adversarial_ai/` | 공격·평가·감사 패키지 |
| `scripts/` | 실험 및 감사 실행 스크립트 |
| `tests/` | 단위·변조 탐지 테스트 |
| `results/` | Clean·FGSM·방어 실험 결과 |
| `docs/` | 실험 계약·결과·재현성 문서 |

## 한계

- 두 모델의 입력 해상도가 다릅니다(CNN 128×128, MobileNetV2 224×224).
- 외부 환경에서 다시 실행했을 때 MobileNetV2 예측 16건이 달랐고, 원인은 아직 밝혀지지 않았습니다.
- BIM·PGD 등 다른 공격과 적대적 학습은 아직 다루지 않았습니다.
- 분류 성능 저하가 실제 운항 피해로 이어지는지는 검증하지 않았습니다.
