# 02. 분석 파이프라인

이 문서는 `notebooks/` 안의 세 노트북이 각각 무엇을 계산하고, 그 결과가 `slides/presentation.pdf`의 어느 슬라이드로 이어지는지를 셀 단위로 추적합니다. (셀 번호는 원본 통합노트북 `소스_코드_8팀_...ipynb` 기준입니다.)

## 전체 흐름

```
data/train_dataset.csv (38,984행)
        │
        ├─▶ notebooks/team_final/01_eda_and_hypotheses.ipynb        (원본 셀 0~40)
        │      전처리 → BMI_group(5구간) → 상관행렬 → 가설1·2·3 → BMI그룹 히트맵
        │
        ├─▶ notebooks/team_final/02_statistical_modeling_and_risk_score.ipynb  (원본 셀 41~63)
        │      전처리 재수행 → high_BMI(이진) → point-biserial 상관+p값
        │      → OLS 상호작용 회귀(HC3) → 4개 집단 표준화 위험지수 → 결론 파이차트
        │
        └─▶ notebooks/individual/member_exploration_alt_preprocessing.ipynb   (원본 셀 64~123)
               별도 전처리 기준(eyesight≤2.0) → WHtR 파생 → 4단계 연령군 → 6단계 BMI군
               → 동일 가설을 독자적으로 재검증 (최종 발표자료에는 미반영)
```

> team_final의 두 노트북은 서로 다른 변수 네이밍(`C_*` vs `COLOR_*`)과 서로 다른 전처리 결과(`train` vs `train_clean`)를 사용하지만, **둘 다 실제로 최종 발표자료 슬라이드 제작에 쓰였습니다.** 두 노트북이 같은 프로젝트의 서로 다른 기여자가 작성한 것으로 보이는 근거와 그로 인한 문제는 `03_issues_and_troubleshooting.md`의 Issue #3에서 다룹니다.

## 노트북 ↔ 발표자료 슬라이드 매핑

### `team_final/01_eda_and_hypotheses.ipynb`

| 원본 셀 | 계산 내용 | 대응 슬라이드 |
|---|---|---|
| 3, 8 | CSV 로드, 극단값 필터링 (`Gtp<500` 등) | "1-1. 데이터 수집 및 전처리" (38,984→38,740) |
| 9 | `BMI_group` 5구간 생성 (저체중/정상/과체중/비만2단계/고도비만) | 이후 모든 BMI별 슬라이드의 구간 기준 |
| 13~14 | 상관행렬(`train.corr()`) 계산 | "1-2. 흡연과 건강 데이터 간 상관관계 분석" |
| 15~17 | BMI 구간별 헤모글로빈 평균/박스플롯 | "3-1. 세부가설 1-1 헤모글로빈" |
| 18~23 | 연령대별 수축기·이완기 혈압 비교 | "3-2. 세부가설 1-2 혈압" |
| 24~27 | 연령대별 고혈압 비율·흡연율 | "3-2" 하단 "30·40·50대 고혈압 기준 이상 비율" |
| 28~29 | 흡연 유무별 HDL | "3-3. 세부가설 1-3 HDL" |
| 30~32 | 간 효소 평균·고위험군 비율·레이더 차트 | "4. 가설2 간 기능" 전체 |
| 33~38 | BMI×흡연 그룹별 중성지방/AST/ALT/허리둘레/HDL | "5. 가설3 고BMI 집단" |
| 40 | BMI 그룹별 흡연/비흡연 히트맵 (Gtp/ALT/AST/Waist) | "5-1, 5-2. BMI 그룹별 주요수치비교" 히트맵 |

### `team_final/02_statistical_modeling_and_risk_score.ipynb`

| 원본 셀 | 계산 내용 | 대응 슬라이드 |
|---|---|---|
| 45 | 전처리 재수행, `high_BMI`(이진), `age_group`(6구간) 생성 | — (내부 준비 단계) |
| 46 | point-biserial 상관계수 + p-value 산출 | "1-2" 슬라이드의 "|r|≥0.2/0.3/0.4" 유의미 범위 서술 |
| 48~53 | 헤모글로빈/혈압/HDL/간효소 OLS 상호작용 회귀(`smoking * high_BMI`) | 가설 1·2 각 슬라이드의 통계적 근거 |
| 54 | GTP·ALT·AST 상호작용 회귀 + BMI×흡연 선그래프 | "5. 가설3" 상호작용 시각화 |
| 56 | `group4`(저BMI/고BMI × 비흡연/흡연) 비중 파이차트 | "6-2. 타겟층 선정" 배경 |
| 59~60 | 4개 집단 표준화 위험 프로파일 바차트 | "6-2. 4개 집단의 표준화 위험 점수 비교" |
| 61~63 | BMI별 흡연 효과 크기, 고BMI 내부 비교, 통합 위험지수 | "6-2" 하단 근거 차트 |

### `notebooks/individual/member_exploration_alt_preprocessing.ipynb`

동일한 가설(혈관 지표, 간 기능, 고BMI×흡연)을 처음부터 독자적으로 재검증한 노트북입니다. `docs/03_issues_and_troubleshooting.md`의 Issue #4에서 설명하듯 전처리 기준이 team_final과 달라(`eyesight ≤ 2.0` vs `≤ 3.0`, 6단계 BMI 라벨, 4단계 연령군) 결과 수치가 발표자료와 정확히 일치하지 않습니다. 최종 제출본에는 사용되지 않았지만, 동일 가설을 다른 전처리 기준으로 재검증했다는 점에서 결과의 강건성(robustness)을 보여주는 참고 자료로 남겨둡니다.

## 사용된 통계 기법 요약

| 기법 | 위치 | 목적 |
|---|---|---|
| Point-biserial correlation | `02_statistical_modeling...ipynb` 셀 46 | 이진(smoking) × 연속형 변수 상관관계 + 유의확률 |
| OLS 회귀 (상호작용항, HC3 표준오차) | 셀 48~54 | `smoking × high_BMI` 상호작용 효과의 통계적 유의성 검증 |
| Z-score 표준화 위험지수 | 셀 60, 63 | ALT·GTP·허리둘레·HDL(역방향)을 하나의 위험 점수로 통합 |
| 단순 그룹 평균 비교 | `01_eda_and_hypotheses.ipynb` 전반 | 흡연/비흡연, BMI 구간별 지표 평균 시각화 |
