# 향수 추천 시스템 데이터 전처리

## 1. 사용할 컬럼 선택

콘텐츠 기반 모델은 `fra_cleaned.csv`를 기준으로 만들고, `fra_perfumes.csv`에서 표시용 이름과 설명을 보완함.

| 원본 컬럼 | 처리 후 컬럼 | 용도 |
| --- | --- | --- |
| `url` | `perfume_id`, `url` | 향수 식별 및 원본 확인 |
| `Perfume`, `Brand` | `perfume_name`, `brand` | 향수 이름·브랜드, 데이터셋 연결 |
| `Top`, `Middle`, `Base` | `top_notes`, `middle_notes`, `base_notes` | 향료 특징 |
| `mainaccord1~5` | `main_accords` | 전체적인 향의 특징 |
| 노트 + 어코드 | `content_text` | TF-IDF 입력 텍스트 |
| `Gender`, `Year`, `Country` | `gender`, `release_year`, `country` | 필터링·표시·매칭 보조 정보 |
| `Rating Value`, `Rating Count` | `avg_rating`, `rating_count` | 전체 평균 평점·평가 수 |
| `Name`, `Description` (`fra_perfumes.csv`) | `raw_display_name`, `description` | 원본 이름·설명 표시 보완 |

평균 평점, 브랜드, 설명은 `content_text`에 넣지 않음. 기본 유사도는 노트와 어코드로 계산하도록 준비함.

협업 필터링은 `atrafshan_data.json`에서 다음 정보를 사용함.

| 원본 정보 | 처리 후 컬럼 | 용도 |
| --- | --- | --- |
| 사용자 ID | `user_id` | 평가한 사용자 식별 |
| 향수 ID | `perfume_id` | 평가받은 향수 식별 |
| `user_vote_on_perfume.votes`의 `رایحه` | `scent_rating` | 해당 사용자의 향 평점 |
| 향수 이름·브랜드·출시 연도 | `perfume_name`, `brand`, `release_year` | Fragrantica와 연결 |
| 댓글 ID·답글 여부 | `comment_id`, `is_comment_answer` | 평가 출처 확인 |

지속력·확산력·병 디자인 평점과 리뷰 텍스트는 보존했지만, 기본 KNN용 평점에 합치지 않음. `users_vote`의 전체 평균 평점도 개인 평점으로 사용하지 않음.

## 2. 결측치·중복 처리

| 처리 대상 | 적용한 방법 | 이유 |
| --- | --- | --- |
| 사용자 평점 결측 | KNN용 파일에서 제외하고 전체 리뷰 파일에 보존 | 실제 평가하지 않은 점수를 만들지 않기 위해 |
| 사용자 향 평점 `0` | 기본 KNN용 파일에서 제외하고 보존 | 실제 최저점인지 미평가 표시인지 확인되지 않았기 때문 |
| 사용자·향수 ID 결측 | KNN용 파일에서 제외 | 누가 어떤 향수를 평가했는지 연결할 수 없기 때문 |
| 일부 메타데이터 결측 | 빈 값으로 유지 | 근거 없이 연도나 향 특징을 추정하지 않기 위해 |
| 노트·어코드의 빈 값 | 특징 텍스트에서 제외 | 빈 값을 향 특징으로 학습하지 않도록 처리 |
| 반복되는 향 특징 | 같은 종류의 중복 토큰 제거 | 같은 특징이 불필요하게 반복되지 않도록 처리 |
| 보완용 Fragrantica 중복 ID | 평가 수가 많은 행 선택, 동률이면 앞선 행 선택 | 이름·설명 보완에 사용할 대표 행 결정 |
| 같은 사용자–향수의 반복 평점 | 같은 점수는 첫 행 유지, 충돌 점수는 별도 검토 대상으로 구성 | 중복 반영과 임의 평균 방지 |

이번 데이터에서 유효 사용자–향수 평점 간 충돌이 발견되지 않음.

향 평점 처리 결과:

- 원본에서 1~10점인 평가: 5,573건
- 그중 사용자 ID가 없는 평가: 8건 제외
- 최종 협업 필터링용 평가: 5,565건
- 의미가 불확실한 0점: 1,309건은 기본 평점 파일에서 제외하고 전체 리뷰 파일에 보존

텍스트는 소문자화하고 공백·구두점을 정리함. 여러 단어로 된 향 표현은 하나의 특징으로 유지함. 예를 들어 `pink pepper`는 `pink_pepper`로 변환함.

노트와 어코드는 구분함. 예를 들어 `note_rose note_musk accord_floral accord_woody`에서 `note_rose`는 장미 향료 정보이고, `accord_floral`은 꽃향 계열 정보를 뜻함.

## 3. 두 데이터셋의 연결 가능 여부

두 사이트의 향수 ID는 서로 다르므로 ID 숫자가 같다는 이유만으로 연결하지 않음.

- Fragrantica 향수 ID: `fr:숫자`
- Atrafshan 향수 ID: `atr:숫자`

연결은 다음 기준으로 진행함.

1. 브랜드와 향수 이름의 대소문자·공백·구두점 등을 통일함.
2. 향수 이름 앞에 동일한 브랜드명이 붙어 있으면 해당 부분만 제거해 비교함.
3. 브랜드와 이름이 유일하게 일치하는지 확인함.
4. 양쪽에 출시 연도가 있으면 충돌 여부를 확인함.
5. `EDT`, `EDP`, `Intense`, `Elixir` 등 제품 구분 표현은 유지함.

| 연결 결과 | Atrafshan 향수 수 | 의미 |
| --- | --- | --- |
| `matched_exact` | 882개 | 정해둔 이름·브랜드 비교 규칙에 따라 자동 연결 |
| `review_required` | 30개 | 연도 충돌·중복 등으로 추가 검토 필요 |
| `unmatched` | 457개 | 자동 연결 조건에 맞는 향수를 찾지 못함 |
| **합계** | **1,369개** | Atrafshan 전체 향수 |

882개는 규칙상 일치한 결과이며, 사람이 제품 페이지 전체를 대조한 결과로 간주하지 않음. 미연결 향수에는 표기 차이 또는 기준으로 사용한 Fragrantica 24,063개 카탈로그 외부에 있을 가능성이 포함됨.

연결된 향수 중 유효한 사용자 평점이 있는 향수는 682개이며, 해당 평가 데이터는 총 3,530건으로 집계함.

## 4. 모델별 최종 데이터 형태

| 모델·용도 | 파일 | 데이터 단위 | 핵심 입력 |
| --- | --- | --- | --- |
| 콘텐츠 기반: TF-IDF + 코사인 유사도 | `perfumes_content.csv` | 향수 하나당 한 행 | `perfume_id`, `content_text` |
| 협업 필터링: Item-based KNN | `user_ratings.csv` | 사용자–향수 평가 하나당 한 행 | `user_id`, `perfume_id`, `scent_rating` |
| 두 방식의 결과를 같은 향수 기준으로 연결 | `user_ratings_linked.csv` | 연결 가능한 사용자–향수 평가 하나당 한 행 | 평점 정보 + `fragrantica_id` |

콘텐츠 기반 모델은 이후 `content_text`를 TF-IDF 벡터로 변환하고 코사인 유사도를 계산하도록 함.

협업 필터링은 이후 사용자–향수 평점 행렬을 구성하도록 함. 현재 CSV는 행렬 변환 전의 긴 형태로 유지함. 미평가 칸은 임의의 평점으로 채우지 않음.

## 5. 최종 CSV 6개

| 파일 | 행 수 | 의미와 사용 목적 |
| --- | --- | --- |
| `perfumes_content.csv` | 24,063행 | Fragrantica 향수별 특징 데이터. 정리된 노트·어코드와 TF-IDF용 `content_text`가 들어 있는 콘텐츠 기반 모델의 핵심 파일 |
| `user_ratings.csv` | 5,565행 | 사용자 ID와 향수 ID가 있고, 향 평점이 1~10점인 평가 데이터. Item-based KNN의 핵심 파일 |
| `perfume_mapping.csv` | 1,369행 | Atrafshan 향수별 Fragrantica 연결 결과. 연결된 ID와 `match_status`, 판단 이유를 확인할 수 있으며 미연결 행도 포함 |
| `user_ratings_linked.csv` | 3,530행 | `user_ratings.csv` 중 Fragrantica와 자동 연결된 평가만 모은 파일. `fragrantica_id`로 향수 특징 데이터와 연결 가능 |
| `atrafshan_perfumes.csv` | 1,369행 | Atrafshan 향수의 이름·브랜드·출시 연도·향 계열 등을 향수별로 정리한 메타데이터 파일 |
| `atrafshan_reviews_preserved.csv` | 36,434행 | 전체 리뷰와 항목별 원본 평점, 처리 상태를 보존한 파일. 평점 결측·0점·ID 결측 행도 포함하므로 전체를 그대로 KNN 학습에 사용하면 안 됨 |
