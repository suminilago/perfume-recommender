# Perfume Recommendation System

## 1. 프로젝트 소개

사용자의 향수 선호 이력과 향수의 특성 정보를 활용하여 개인 맞춤형 향수를 추천하는 시스템.

두 가지 추천 방식을 구현하고 동일한 평가 조건에서 성능을 비교함.

- **Content-based Filtering:** 향수의 콘텐츠 특성을 이용한 추천
- **Item-based Collaborative Filtering:** 사용자 평점 이력을 이용한 아이템 기반 KNN 추천

## 2. 데이터셋

| 데이터셋 | 주요 역할 |
|---|---|
| Fragrantica | 향수 이름, 브랜드, 향수 특성 등 콘텐츠 정보 |
| Atrafshan | 사용자별 향수 평점과 향수 정보 |

서로 다른 데이터셋의 향수를 연결하기 위해 매핑 데이터를 사용함.

## 3. 추천 모델

### Content-based Filtering

향수의 특성 정보를 기반으로 사용자 선호 향수와 유사한 향수를 추천하는 방식.

### Item-based KNN

사용자 평점 이력을 바탕으로 향수 간 유사성을 활용하는 협업 필터링 방식.

구체적인 특징 벡터 구성, 유사도 계산, KNN 설정은 코드 및 평가 문서 참조.

## 4. 프로젝트 구조

- `perfume_recommendation.ipynb`: 추천 모델 구현
- `evaluation/evaluate.ipynb`: 모델 학습 및 성능 평가
- `evaluation/EVALUATION.md`: 평가 방법과 결과 분석
- `evaluation/results/model_comparison.csv`: 모델별 평가 결과
- `data/`: 전처리 데이터 및 향수 매핑 정보
- `requirements.txt`: 실행에 필요한 패키지

## 5. 실행 방법

1. 저장소 복제
2. Python 가상환경 생성 및 활성화
3. `pip install -r requirements.txt` 실행
4. 데이터 파일 경로 확인
5. `perfume_recommendation.ipynb` 실행
6. `evaluation/evaluate.ipynb` 실행

## 6. 모델 성능 비교

평가 방식: Top-10 추천 평가

| 모델 | 평가 사용자 | Hit Rate@10 | Precision@10 | Recall@10 |
|---|---:|---:|---:|---:|
| Content-based | 258 | 0.0620 | 0.0062 | 0.0620 |
| Item-based KNN | 258 | 0.0581 | 0.0058 | 0.0581 |

현재 실험에서는 Content-based 모델이 세 지표에서 소폭 높은 성능을 기록함.

단, 결과는 현재 후보군과 평가 사용자 선정 조건에 한정됨.

상세 평가 방법은 [EVALUATION.md](evaluation/EVALUATION.md) 참조.

## 7. 프로젝트 한계

- 사용자별 평점 데이터의 희소성
- 두 데이터셋 사이의 향수 ID 매핑 문제
- 평가 가능한 사용자 및 향수 후보의 제한
- 현재 평가 조건에 따른 결과 해석의 한계