# 생체 신호 기반 흡연 여부 비교 시각화 프로젝트

> 내일배움캠프 QA/QC 5기 · 8조 88하조
> 팀장: 이지아 · 팀원: 박재경, 안영진, 유지수, 이근주

건강검진 생체 신호 데이터를 활용해 흡연자와 비흡연자의 주요 건강 지표 차이를 EDA·통계 검정·시각화로 분석한 팀 프로젝트입니다. 이 저장소는 발표 이후 팀원들의 흩어진 산출물(기획서·발표자료·개별 실험 노트북)을 역추적해 재구성했습니다.

## 핵심 결과 요약

전처리 데이터 기준 흡연자 vs 비흡연자 평균 비교:

| 지표 | 비흡연자 | 흡연자 | 차이 |
|---|---|---|---|
| 헤모글로빈 (g/dL) | 14.14 | 15.45 | +1.31 |
| HDL | 59.3 | 53.8 | −5.5 |
| AST (U/L) | 25.0 | 27.4 | +2.3 |
| ALT (U/L) | 24.4 | 30.7 | +6.3 |
| γ-GTP (U/L) | 30.3 | 53.3 | +23.0 (차이 가장 큼) |

고BMI 집단에서 흡연 시 γ-GTP 차이가 최대 62%까지 벌어지는 등, **고BMI × 흡연의 결합 효과가 단일 요인보다 크게 나타남**을 확인했습니다. 자세한 결론과 한계는 `slides/presentation.pdf` 및 `docs/02_analysis_pipeline.md`를 참고하세요.

## 폴더 구조

```
.
├── README.md
├── requirements.txt
├── data/
│   └── train_dataset.csv                  # 원본 데이터 (Kaggle Smoker Status Prediction)
├── notebooks/
│   ├── team_final/                        # 팀 공식 통합 노트북 — 실제 발표자료 제작에 쓰인 코드
│   │   ├── 01_eda_and_hypotheses.ipynb            # 전처리·상관관계·가설1~3·BMI그룹 히트맵
│   │   └── 02_statistical_modeling_and_risk_score.ipynb  # 상관분석 p값·OLS 상호작용·위험지수
│   └── individual/                        # 팀원별 실험 노트북 — 최종 발표에는 미반영
│       └── member_exploration_alt_preprocessing.ipynb
├── docs/
│   ├── 01_project_plan.md                 # 기획서 정리
│   ├── 02_analysis_pipeline.md            # 노트북 ↔ 발표자료 슬라이드 매핑, 파이프라인 설명
│   ├── 03_issues_and_troubleshooting.md   # ★ 재현조건→원인→해결까지 추적한 이슈 5건
│   └── 04_tutor_feedback.md               # 튜터 서면 피드백 + 팀 회고(KPT)
└── slides/
    └── presentation.pdf                   # 발표자료
```

## 어디서부터 볼까요

- **프로젝트를 처음 보신다면** → `docs/01_project_plan.md` (기획서) → `slides/presentation.pdf` (발표자료) 순으로
- **분석 코드가 발표자료와 어떻게 연결되는지 궁금하다면** → `docs/02_analysis_pipeline.md`
- **노트북 실행 중 에러가 나거나 수치가 안 맞는다면** → `docs/03_issues_and_troubleshooting.md`에서 동일 증상이 이미 정리되어 있는지 먼저 확인
- **팀 회고나 튜터 피드백** → `docs/04_tutor_feedback.md`

## 데이터셋

- 파일: `data/train_dataset.csv`
- 출처: [Kaggle – Smoker Status Prediction Dataset](https://www.kaggle.com/datasets/gauravduttakiit/smoker-status-prediction)
- 원본 규모: 38,984행 × 23열. `team_final` 노트북 기준 최종 분석 표본은 33,256~38,740행 (정확한 수치와 그 이유는 `docs/03_issues_and_troubleshooting.md` Issue #1 참고)

## 실행 방법

```bash
pip install -r requirements.txt
jupyter notebook notebooks/team_final/01_eda_and_hypotheses.ipynb
```

노트북은 원래 Google Colab에서 작성되어 Colab 전용 명령과 `/content/` 경로를 포함합니다. 로컬 실행 시 주의사항은 `docs/03_issues_and_troubleshooting.md` Issue #5를 참고하세요.

## 팀 역할 분담

| 역할 | 담당 |
|---|---|
| 문제 정의 및 분석 기획 | 이지아, 안영진 |
| 데이터 전처리 | 박재경, 유지수 |
| EDA 및 시각화 | 박재경, 이근주 |
| 인사이트 정리 및 결과 해석 | 이근주, 유지수 |
| 발표 자료 작성·발표·녹화 | 이지아, 안영진 |
| 문서 정리 및 일정 관리 | 이지아 |

전체 기획 배경은 `docs/01_project_plan.md`에 정리되어 있습니다.
