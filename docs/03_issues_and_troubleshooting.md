# 03. 이슈 및 트러블슈팅

이 문서는 세 노트북(`team_final/01`, `team_final/02`, `individual/member_exploration...`)을 셀 단위로 직접 읽고 실행 로그를 대조해서 찾아낸 이슈를 정리합니다. 순서는 **재현조건 → 증상 → 원인 → 해결방법 → 영향/검증**입니다.

셀 번호는 원본 통합노트북(`소스_코드_8팀_...ipynb`) 기준이며, 분리된 각 파일은 맨 앞에 안내 마크다운 셀이 하나 추가되어 있어 **파일 내 실제 위치는 "원본 셀 번호 + 1(또는 -40/-63 오프셋)"** 입니다. 각 이슈마다 두 좌표를 모두 표기했습니다.

---

<a id="issue-1"></a>
## Issue #1 — 중복 제거 단계가 최종 전처리 데이터에 반영되지 않음

**위치**: `notebooks/team_final/01_eda_and_hypotheses.ipynb` (파일 내 7~9번째 셀, 원본 셀 6~8)

**재현조건**
```python
# 원본 셀 6 (파일 내 7번째 셀) — 실제 코드 3줄
df.duplicated().sum()
train = df.drop_duplicates()
train.duplicated().sum()

# 원본 셀 8 (파일 내 9번째 셀)
train = df[                       # ⚠️ train이 아니라 df를 다시 참조
    (df['Gtp'] < 500) & (df['ALT'] < 500) & (df['AST'] < 500) &
    (df['LDL'] < 400) & (df['serum creatinine'] < 3.0) &
    (df['eyesight(left)'] <= 3.0) & (df['eyesight(right)'] <= 3.0)
].copy()
train.describe()
# 이상치 제거 완료
```

**증상**
셀 6에서 `train = df.drop_duplicates()`로 중복 5,517행을 명시적으로 제거해 놓고도, 바로 다음 코드 셀(셀 8)에서 `train`을 다시 `df`(중복이 포함된 원본)로부터 필터링해 덮어씁니다. 발표자료의 "전처리 전 38,984개 → 전처리 후 38,740개(제거 244개)"는 **중복 제거를 거치지 않은 상태**에서 극단값만 제거한 값과 정확히 일치합니다.

| 기준 | 행 수 |
|---|---|
| 원본 | 38,984 |
| 중복 제거만 (셀 6 결과) | 33,467 |
| 극단값 필터만, df 기준 (셀 8 결과 = 실제 사용된 값) | **38,740** |
| 중복 제거 + 극단값 필터 모두 적용 (의도했던 값으로 추정) | 33,256 |

**원인**
같은 변수명(`train`)을 여러 셀에서 재사용하면서, 셀 8이 셀 6의 결과(`train`)가 아니라 최초 원본(`df`)을 다시 참조했습니다. Pandas는 조용히 값을 덮어쓰므로 실행 중에는 문제가 드러나지 않고, `train.describe()`의 행 수만 보면 "전처리가 잘 됐다"고 착각하기 쉽습니다.

**해결방법**
```python
train = df.drop_duplicates()
train_clean = train[
    (train['Gtp'] < 500) & (train['ALT'] < 500) & (train['AST'] < 500) &
    (train['LDL'] < 400) & (train['serum creatinine'] < 3.0) &
    (train['eyesight(left)'] <= 3.0) & (train['eyesight(right)'] <= 3.0)
].copy()
```

**영향/검증**
`data/train_dataset.csv`로 직접 재현한 결과, 중복까지 제거하면 최종 표본은 33,256행이 되고 헤모글로빈·HDL·간효소 평균값은 소수점 첫째~둘째 자리 수준에서만 달라집니다(결론에 영향 없음). 다만 "38,740개"라는 발표자료 수치 자체는 중복 미제거 상태라는 점을 후속 보고서에는 명시하는 것을 권장합니다.

---

<a id="issue-2"></a>
## Issue #2 — 저장된 실행 결과와 현재 코드가 불일치 (Stale Output)

**위치**: `notebooks/team_final/02_statistical_modeling_and_risk_score.ipynb` (파일 내 10번째 셀, 원본 셀 50)

**재현조건**
해당 셀을 재실행하지 않고 저장된 `.ipynb`의 출력만 그대로 열람.

**증상**
셀의 **현재 소스 코드**는 다음과 같이 boxplot을 그립니다.
```python
#가설1
#세부가설1-3_HDL은 흡연자의 경우 안 좋은 데이터를 갖고 있을 것이다
plt.figure(figsize=(6,5))
train_clean.boxplot(column='HDL', by='smoking')
...
#평균비교막대
```
그런데 저장된 출력은 `display_data → display_data → error → display_data` 순서로 총 4개이며, 그중 에러는 다음과 같습니다.
```
AttributeError: Rectangle.set() got an unexpected keyword argument 'figsize'
...
----> bars = plt.bar(hdl_mean.index, hdl_mean.values, color=[COLOR_NONSMOKER, COLOR_SMOKER], figsize=(6,5))
```
현재 셀에는 `plt.bar(...)` 코드 자체가 없습니다 — 마지막 줄의 주석 `#평균비교막대`("평균 비교 막대그래프"를 의미)만 흔적으로 남아 있어, **막대그래프 버전을 작성 → 에러 발생 → boxplot 버전으로 교체 → 재실행 없이 저장**된 것으로 추정됩니다.

**원인**
1. 근본 원인: `figsize`는 `plt.figure(figsize=...)`의 인자이지 `plt.bar()`의 인자가 아닙니다. `plt.bar()`에 전달된 미지원 키워드는 내부적으로 `matplotlib.patches.Rectangle.set()`으로 그대로 전달되어 `AttributeError`가 발생합니다.
2. 표면적 원인: 에러 이후 코드를 boxplot 방식으로 고쳤지만 셀을 다시 실행하지 않고 저장해, 오래된 에러 출력이 새 코드 위에 그대로 남았습니다.

**해결방법**
```python
# 잘못된 코드
bars = plt.bar(x, y, color=[...], figsize=(6,5))

# 올바른 코드
plt.figure(figsize=(6,5))
bars = plt.bar(x, y, color=[...])
```
제출 전 **커널 재시작 → 전체 셀 재실행**(Restart & Run All)으로 소스와 출력을 동기화해야 합니다. 실제로 전체 노트북의 `execution_count`가 모두 `null`인 것으로 보아, 이 노트북은 제출 시점에 한 번도 "위에서부터 끝까지" 정상 실행된 상태로 저장되지 않았을 가능성이 있습니다.

**영향/검증**
`docs/02_analysis_pipeline.md`의 매핑표 기준으로 이 셀은 "3-3. 세부가설 1-3 HDL" 슬라이드의 근거 코드입니다. boxplot 자체는 정상 작동하는 코드이므로 최종 발표자료 수치에는 영향이 없습니다.

---

<a id="issue-3"></a>
## Issue #3 — team_final 내부에서도 서로 다른 네이밍·전처리가 혼재

**위치**: `team_final/01_eda_and_hypotheses.ipynb` vs `team_final/02_statistical_modeling_and_risk_score.ipynb`

**재현조건**
두 파일을 나란히 열어 색상 팔레트 정의와 전처리 변수 정의를 비교.

**증상**

| 구분 | `01_eda_and_hypotheses.ipynb` (원본 셀 2) | `02_statistical_modeling...ipynb` (원본 셀 45) |
|---|---|---|
| 색상 변수명 | `C_SMOKER`, `C_NONSMOKER`, `C_HIGH_BMI`, `C_LOW_BMI`, `C_TEXT`, `C_GRID` | `COLOR_SMOKER`, `COLOR_NONSMOKER`, `COLOR_HIGH_BMI`, `COLOR_LOW_BMI`, `COLOR_TEXT`, `COLOR_GRID` |
| 색상 값 | `#A63A50` 등 동일 | `#A63A50` 등 **값은 완전히 동일** |
| 데이터프레임 변수 | `train` (원본 셀 8) | `train_clean` (원본 셀 45, CSV를 처음부터 다시 읽음) |
| BMI 구간 | `BMI_group`, 5구간: 저체중/정상/과체중/**비만2단계**/**고도비만** (원본 셀 9) | `high_BMI` 이진 변수만 사용 (BMI≥25) |
| 연령대 구간 | 없음 (`age_group`은 20/30/40/50/60/70대+, 셀 18) | `age_group`, 6구간: 20~70대+ (원본 셀 45, 경계값 `[15,29,39,49,59,69,120]`) |

두 노트북은 **완전히 동일한 색상 팔레트(hex 값)를 각자 독립적으로 하드코딩**했고, `train`/`train_clean`이라는 이름은 같지만 서로 다른 셀에서 독립적으로 새로 정의됩니다. 즉 우연히 정합적으로 보일 뿐, 실제로는 두 사람이 노션에 공유된 "색상 팔레트 표"만 보고 각자 코드를 작성한 것으로 보입니다.

**원인**
각 팀원이 개인 Colab 사본에서 작업 후, 최종 제출 시 셀 단위로 이어붙이기만 하고 변수·네임스페이스를 통합 정리하지 않았습니다. 두 노트북을 하나로 합쳐서 위에서부터 순서대로 실행하면 문제 없이 동작하지만(각자 독립적으로 데이터를 다시 읽어오므로), 팔레트 값을 한쪽만 수정하면 다른 쪽은 반영되지 않는 등 유지보수 시 혼란의 소지가 있습니다.

**해결방법**
1. 공통 설정을 `notebooks/_shared/config.py`로 분리 — 색상 팔레트, BMI/연령 구간 기준을 상수로 정의하고 두 노트북이 import해서 사용
2. 공통 전처리를 `notebooks/_shared/preprocessing.py`의 `load_and_clean(path)` 함수로 통일 (Issue #1의 중복 제거 버그도 이 함수 안에서 한 번에 고칠 수 있음)
3. `01`과 `02`처럼 번호를 붙여 실행 순서를 명시한 지금의 구조를 유지하되, 새 팀원이 셋째 노트북을 추가할 때는 반드시 `_shared` 모듈을 사용하도록 컨벤션 문서화

---

<a id="issue-4"></a>
## Issue #4 — team_final과 individual 노트북의 전처리 기준 drift

**위치**: `individual/member_exploration_alt_preprocessing.ipynb` (파일 내 8~10번째 셀, 원본 셀 71~73) vs `team_final/01` 원본 셀 8~9

**재현조건**
동일한 `data/train_dataset.csv`를 두 노트북에서 각각 전처리.

**증상**

| 기준 | team_final | individual |
|---|---|---|
| 시력 필터 | `eyesight(left/right) ≤ 3.0` | `eyesight(left/right) ≤ 2.0` |
| BMI 구간 수 | 5구간 (저체중/정상/과체중/비만2단계/고도비만) | 6구간 (저체중/정상/과체중/**1단계 비만**/**2단계 비만**/**3단계 비만**) |
| 연령 구간 | 6구간, 10년 단위 (20대~70대+) | 4구간 (청년/중년/장년/노년, 경계값 35/50/64) |
| 파생 변수 | BMI만 | BMI + `WHtR`(허리둘레/키) 추가 |

시력 필터 기준이 다르기 때문에(`≤2.0`이 `≤3.0`보다 더 많은 행을 제외) individual 노트북의 최종 표본 크기는 team_final과 다르며, BMI·연령 구간 라벨도 달라 두 노트북의 그래프를 나란히 비교하면 카테고리 개수 자체가 맞지 않습니다.

**원인**
individual 노트북은 team_final과 별개로 처음부터 작성된 개인 탐색 버전으로, 팀 공용 전처리 기준이 확정되기 전(또는 확정과 무관하게) 개인적으로 다른 기준을 적용한 것으로 보입니다.

**해결방법 / 권장사항**
- individual 노트북은 **결과를 team_final과 직접 비교하는 용도로 사용하지 않기** — 결론 재현용이 아니라 "다른 기준으로도 같은 방향의 결과가 나오는지"를 보는 강건성(robustness) 참고 자료로만 취급
- 향후 유사 검증이 필요하면 `_shared/config.py`의 공통 기준을 individual 노트북에서도 import해서 사용하고, 의도적으로 다른 기준을 쓸 경우 노트북 상단에 "team_final과 다른 점" 표를 명시

---

<a id="issue-5"></a>
## Issue #5 — Colab 전용 명령어로 인해 로컬 환경에서 실행 불가

**위치**: `team_final/02` 원본 셀 41, `individual/...` 원본 셀 65 (두 곳에서 각각 독립적으로 폰트 설치)

**재현조건**
로컬 Jupyter/VSCode 등 Colab이 아닌 환경에서 노트북을 처음부터 실행.

**증상**
```python
!apt-get update -qq
!apt-get install -y fonts-nanum -qq      # team_final/02, 원본 셀 41
!sudo apt-get install -y fonts-nanum     # individual, 원본 셀 65 (동일 작업을 별도로 반복)
```
`apt-get`은 Linux(Colab) 전용이라 Windows/Mac 로컬 환경에서 실패합니다. 또한 CSV 경로가 `/content/train_dataset.csv`(Colab 세션 전용 경로)로 하드코딩되어 있어 로컬에서는 `FileNotFoundError`가 발생합니다.

**해결방법**
- 데이터 경로를 상대경로로 통일: `pd.read_csv('data/train_dataset.csv')`
- 한글 폰트는 `koreanize-matplotlib` 패키지 하나로 통일 (`pip install koreanize-matplotlib` 후 `import koreanize_matplotlib`)
- 두 노트북에서 반복되는 폰트 설치 코드는 Issue #3에서 제안한 `_shared/` 공통 모듈로 이동

---

## 요약 표

| # | 이슈 | 위치 | 최종 상태 |
|---|---|---|---|
| 1 | 중복 제거 미반영 | team_final/01 | 원인 규명 완료, 수정 코드 제시. 결론 영향 없음(수치 소폭 변경) |
| 2 | 출력-소스 불일치(stale output) | team_final/02 | 원인 규명 완료. 재실행으로 해결 가능 |
| 3 | 팀 내 네이밍/전처리 중복 | team_final/01 vs 02 | 공용 모듈화 권장안 제시 |
| 4 | team_final ↔ individual 기준 drift | individual | 용도 분리(강건성 참고자료)로 정리 |
| 5 | Colab 전용 명령 의존 | team_final/02, individual | 대체 코드 제시 |
