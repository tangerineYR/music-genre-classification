
# Historical 60-second Dataset Evaluation

**평가 일자:** 2026년 10월 23일  
**평가 프로토콜:** `portfolio_refactoring`

2025년 프로젝트 당시 생성한 60초 WAV 데이터셋을 대상으로 **Single-source `Cover_Timbre` 방식과 Feature-aware Source Mapping 방식**의 음악 장르 분류 성능을 비교했습니다.

두 실험 조건에는 동일한 곡과 레이블, 동일한 균형 표본, 동일한 10개 교차검증 Fold를 적용했습니다. 전처리 과정의 데이터 누수를 방지하기 위해 `MinMaxScaler`는 각 Fold의 훈련 데이터에서만 학습했습니다.

본 문서의 결과는 2026년에 수정된 평가 절차를 통해 측정한 값이며, 2025년 당시의 모델 성능으로 소급하여 해석하지 않습니다.

## Dataset

2025년에 생성한 음원 데이터셋의 전체 인벤토리에는 **5,300개 WAV 파일과 590곡**이 포함되어 있습니다.

전체 파일의 디코딩 및 유효성을 검사한 결과, 필수 음원 유형 4종을 모두 보유하고 검증을 통과한 곡은 **587곡**이었으며, **3곡은 필수 음원 누락으로 제외**했습니다.

| 장르 | 발견된 곡 | 검증 통과 | 제외 | 최종 평가 |
|---|---:|---:|---:|---:|
| Ballade | 217 | 216 | 1 | 39 |
| Dance | 128 | 127 | 1 | 39 |
| Hiphop | 39 | 39 | 0 | 39 |
| RnB | 39 | 39 | 0 | 39 |
| Rock | 39 | 39 | 0 | 39 |
| Trot | 128 | 127 | 1 | 39 |
| **합계** | **590** | **587** | **3** | **234** |

**평가에서 제외된 곡과 사유**

- `Similar_Ballade_00056`: `Cover_Genre`, `Cover_Rhythm` 누락
- `Similar_Dance_00087`: `Cover_Genre` 누락
- `Similar_Trot_00610`: `Cover_Genre` 누락

전체 5,300개 WAV 파일은 **16,000Hz Mono** 형식으로 확인됐습니다. 음원 길이는 약 60초이며, 최소 59.9999375초에서 최대 60.0000625초 범위에 분포했습니다.

샘플 단위 반올림으로 발생하는 미세한 길이 차이를 허용하기 위해 **±10ms의 길이 허용 오차**를 적용했습니다. 이 범위 안의 음원은 별도의 패딩이나 구간 재선택 없이 사용했습니다.

**데이터 품질 검사 결과**

| 검증 항목 | 결과 |
|---|---:|
| 디코딩 실패 파일 | 0개 |
| 길이 조건 위반 파일 | 0개 |
| 빈 음원 파일 | 0개 |
| 비유한 값(NaN·Inf)이 포함된 파일 | 0개 |
| 동일 곡 내부의 중복 음원 그룹에 속한 파일 | 77개 |
| 서로 다른 곡 사이의 중복 음원 파일 | 0개 |

동일 곡 내부에서 발견된 중복 음원은 데이터 감사 결과에 기록하되, 별개의 학습 표본으로 취급하지 않았습니다. 서로 다른 곡 사이에서 동일한 디코딩 음원이 발견되지는 않았습니다.

### Source Family Distribution

| 음원 유형 | 파일 수 |
|---|---:|
| `Cover_Arrangement` | 586 |
| `Cover_Genre` | 587 |
| `Cover_Instrument` | 1,180 |
| `Cover_Rhythm` | 589 |
| `Cover_Tempo` | 1,180 |
| `Cover_Timbre` | 1,178 |
| **합계** | **5,300** |

`Cover_Tempo`와 `Cover_Arrangement`는 데이터 인벤토리에 포함되지만, 이번 96차원 특징 추출 실험에서는 사용하지 않았습니다. 현재 데이터셋에는 원본 음원 유형인 `Org`가 포함되어 있지 않습니다.

특징 추출에 필요한 각 Source Family에서 여러 Variant가 존재할 경우, **데이터셋 상대 경로를 기준으로 사전순 첫 번째 파일**을 선택했습니다. 성능 지표를 참고하여 유리한 Variant를 선택하지 않았습니다.

장르·Source Family·Variant별 세부 파일 수는 [dataset_inventory.csv](dataset_inventory.csv)에서 확인할 수 있습니다.

## Protocol & Validation

두 실험 조건에서는 총 **587곡에 대해 각각 96차원의 스케일링되지 않은 특징 벡터**를 추출했습니다.

특징 벡터의 유한값 여부와 특성명 중복을 검사했으며, Single-source와 Feature-aware 데이터셋에 동일한 곡 ID와 레이블이 같은 순서로 포함되어 있는지 검증했습니다.

이후 장르별 최소 표본 수인 39곡을 기준으로 결정론적 언더샘플링(Deterministic Undersampling)을 수행하여, **6개 장르 × 39곡 = 총 234곡**의 균형 평가 표본을 구성했습니다.

**교차검증 설정**

| 항목 | 설정 |
|---|---|
| 평가 표본 | 234곡 |
| 장르 수 | 6개 |
| 장르별 곡 수 | 39곡 |
| 교차검증 | Stratified 10-Fold |
| 데이터 셔플 | `shuffle=True` |
| 난수 시드 | `random_state=42` |
| 스케일링 | `MinMaxScaler` |
| 스케일러 학습 범위 | 각 Fold의 훈련 데이터만 |
| 비교 방식 | 동일한 곡·레이블·Fold를 사용하는 Paired Comparison |

검증 Fold의 표본 수는 **24곡인 Fold가 4개, 23곡인 Fold가 6개**입니다. 각 평가 곡은 정확히 한 번 검증 Fold에 포함되며, 훈련 Fold와 검증 Fold 사이에 동일한 곡이 중복되지 않도록 분할했습니다.

**GMM 설정**

| 파라미터 | 값 |
|---|---|
| `n_components` | 7 |
| `covariance_type` | `diag` |
| `max_iter` | 400 |
| `random_state` | 42 |
| `init_params` | `kmeans` |

GMM 설정은 기존 프로젝트에서 사용한 값을 유지했습니다. 이번 재평가에서는 새로운 하이퍼파라미터 탐색이나 음원 Variant 성능 탐색을 수행하지 않았습니다.

## Measured Results

두 실험 조건을 동일한 10개 Fold에서 평가한 결과입니다.

| 실험 조건 | 평균 Accuracy ± 표준편차 | 평균 Macro F1 ± 표준편차 | 미수렴 Fold |
|---|---:|---:|---:|
| Single-source `Cover_Timbre` | 0.294746 ± 0.102069 | 0.270577 ± 0.093847 | 0 / 10 |
| Feature-aware Mapping | 0.303080 ± 0.071768 | 0.290179 ± 0.065378 | 0 / 10 |

**Feature-aware − Single-source 차이**

- **Accuracy:** +0.008333 (**+0.83%p**)
- **Macro F1:** +0.019602

Accuracy와 Macro F1은 0~1 범위의 값이며, 표준편차는 10개 Fold의 결과를 대상으로 `ddof=0`으로 계산했습니다.

모든 Fold에서 각 장르별 GMM은 정상 수렴했습니다.

### Fold Results

| Fold | Single-source Accuracy | Feature-aware Accuracy | Single-source Macro F1 | Feature-aware Macro F1 |
|---|---:|---:|---:|---:|
| 1 | 0.500000 | 0.291667 | 0.464286 | 0.287458 |
| 2 | 0.166667 | 0.333333 | 0.165079 | 0.311111 |
| 3 | 0.208333 | 0.375000 | 0.164286 | 0.359007 |
| 4 | 0.333333 | 0.291667 | 0.288360 | 0.288781 |
| 5 | 0.434783 | 0.391304 | 0.411447 | 0.368254 |
| 6 | 0.173913 | 0.260870 | 0.198413 | 0.280952 |
| 7 | 0.304348 | 0.260870 | 0.276150 | 0.220238 |
| 8 | 0.260870 | 0.347826 | 0.222222 | 0.298413 |
| 9 | 0.304348 | 0.347826 | 0.263034 | 0.348942 |
| 10 | 0.260870 | 0.130435 | 0.252493 | 0.138636 |

Fold별 Accuracy를 비교하면 Feature-aware 방식은 5개 Fold에서 더 높은 점수를 기록했지만, 나머지 5개 Fold에서는 Single-source 방식보다 낮았습니다.

따라서 평균 Accuracy가 소폭 높았다는 사실만으로 Feature-aware 방식이 일관되게 우수하다고 결론 내릴 수는 없습니다.

### Confusion Matrices

![Single-source와 Feature-aware 방식의 혼동행렬](confusion_matrix.png)

두 혼동행렬은 동일한 색상 척도를 적용했으며, 각 셀에는 예측 건수가 표시됩니다. 행은 실제 장르, 열은 예측 장르를 의미합니다.

**Single-source 방식에서 많이 발생한 오분류**

- Dance → Hiphop: 15곡
- Rock → Hiphop: 11곡
- RnB → Ballade: 11곡

**Feature-aware 방식에서 많이 발생한 오분류**

- Ballade → RnB: 14곡
- Rock → Hiphop: 10곡
- RnB → Trot: 10곡

혼동행렬은 모델이 어떤 장르를 혼동하는지 확인하기 위한 자료입니다. 개별 오분류 원인을 확인하려면 음원 및 특징 분포에 대한 추가 분석이 필요합니다.

## Interpretation & Limitations

Feature-aware Mapping은 균형화된 2025년 음원 표본에서 Single-source 방식보다 평균 Accuracy와 Macro F1이 소폭 높았습니다.

평균 Accuracy 차이는 **+0.83%p**였지만, 각 조건의 Fold 간 표준편차는 각각 약 10.21%p와 7.18%p로 관찰됐습니다. 또한 10개 Fold 중 Accuracy가 개선된 경우와 감소한 경우가 각각 5개였습니다.

따라서 이번 결과는 Feature-aware 방식의 **일관되고 안정적인 성능 개선을 입증하지 못합니다.** 통계적 유의성이나 인과적 효과도 주장하지 않습니다.

이번 평가에 사용한 음원은 2025년에 샘플링된 60초 구간입니다. 서로 다른 Cover Source가 정확히 동일한 시점에서 추출됐는지는 확인할 수 없으므로, 결과를 음원 변형 유형 자체의 인과적 효과로 해석하지 않습니다.

또한 필수 음원이 모두 존재하는 곡만 선정하고 장르별 표본 수를 39곡으로 균형화했으므로, 이번 수치는 AI Hub 전체 데이터셋의 대표 성능이나 별도 팀 모델의 성능을 의미하지 않습니다.

## Reproducibility

실험 설정, 장르별 표본 수, Fold별 검증 곡 ID, 혼동행렬, 조건별 평가 지표, 입력 파일 해시, 의존성 버전 및 모델 파라미터는 [cv_metrics.json](cv_metrics.json)에 기록했습니다.

[fold_metrics.csv](fold_metrics.csv)에는 Fold별 Accuracy·Macro F1·GMM 수렴 여부를, [feature_source_ablation.csv](feature_source_ablation.csv)에는 조건별 평균과 표준편차를 저장했습니다.

공개된 [dataset_inventory.csv](dataset_inventory.csv)는 장르·음원 유형·Variant별 집계 결과입니다. 개별 음원 경로와 SHA-256을 포함하는 전체 감사 기록 및 특징 추출 원본 파일은 로컬 산출물로 관리하며 저장소에 포함하지 않습니다.

전체 재실행에는 프로젝트 당시 생성한 60초 WAV 데이터셋이 필요합니다. 해당 데이터가 없는 환경에서는 코드 테스트와 저장된 결과의 확인은 가능하지만, 실제 음원 특징 추출 및 모델 성능의 독립 재현은 수행할 수 없습니다.
