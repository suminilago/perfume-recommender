# 추천 창(GUI) 실행 안내

`run_recommender_gui.ipynb`는 `perfume_recommendation.ipynb`의 두 모델(콘텐츠 기반, Item-based KNN)을 불러와 **별도 창(tkinter)**에서 클릭으로 추천을 받아보는 노트북.

- 모델 코드는 `perfume_recommendation.ipynb`에서 클래스 정의만 그대로 불러옴 (`evaluation/evaluate.ipynb`와 같은 방식). 모델 노트북은 수정하지 않음
- tkinter는 파이썬 기본 포함이라 `requirements.txt` 외에 추가 설치 없음
- 전체 평점으로 학습한 추천 예시이며, 성능 수치는 `evaluation/evaluate.ipynb` 참고

## 실행

1. 최초 1회: `pip install -r requirements.txt`
2. 저장소 루트의 `data/` 폴더에 CSV 5개를 넣음 (`data/`는 `.gitignore` 대상이라 저장소에 포함되지 않음, 모델 노트북과 같은 파일)
3. 저장소 루트에서 `run_recommender_gui.ipynb`를 열고 모두 실행(Run All)
4. 새 창에서 [콘텐츠 기반 필터링] 또는 [협업 필터링]을 클릭. 창을 닫으면 셀 실행이 끝남
   - 창이 안 보이면 작업 표시줄에서 "향수 추천 시스템" 창을 찾기
   - 자기 PC에서 실행하는 Jupyter / VS Code에서만 창이 뜸 (Google Colab 등 원격 환경에서는 동작하지 않음)

## 창에서 할 수 있는 것

**콘텐츠 기반 필터링 (TF-IDF + 코사인 유사도)**
- **좋아하는 향수**(Fragrantica 24,063개 중 검색)와 **원하는 향 종류**(16가지) 중 하나만 골라도 되고 둘 다 골라도 됨. 둘 다 고르면 두 유사도의 평균으로 순위를 매김
- 향 종류 16가지는 Atrafshan 향 계열을 Fragrantica 어코드로 옮긴 것. 오리엔탈·푸제르·시프레(*)는 같은 어코드가 없어 비슷한 어코드 조합으로 근사
- **성별**은 여성·남성·유니섹스를 여러 개 체크 가능 (3개 모두 체크 = 전체)
- 결과 표의 '주요 향' 열에서 각 향수의 대표 어코드 3개를 확인 가능

**협업 필터링 (Item-based KNN)**
- 직접 평가하기: 향수를 검색해 1~10점을 주면 그 평점을 새 사용자로 추가해 KNN을 다시 학습하고 추천
- 기존 Atrafshan 사용자: 사용자를 고르면 평점 이력과 추천을 표시
- '근거 이웃 수'가 1이면 내 평점 하나만 보고 예측한 것이라 근거가 약함

**공통**
- 추천 결과 행을 **더블클릭**하면 향수 페이지가 기본 브라우저에서 열림
  - 콘텐츠 기반: Fragrantica 페이지
  - 협업 필터링: Fragrantica와 연결된(matched_exact) 향수는 Fragrantica 페이지, 연결되지 않은 향수는 Atrafshan 페이지

## 구현 메모

- 모델 설정값은 모델 노트북과 동일 (`brand_weight=0.2`, `k=20`, `min_common_users=2`, `shrinkage=10.0`, `min_neighbors=1`)
- 향 종류 점수: 고른 향 종류를 `accord_...` 토큰 문장으로 만들어 콘텐츠 모델이 학습한 TF-IDF로 변환한 뒤 향수 벡터와 코사인 유사도 계산 (`score_for_scents()`)
- 처음 만든 뒤 바꾼 부분은 노트북 안의 `# [수정]`, `# [추가]` 주석과 맨 위 '수정 이력' 표에 원래 코드·변경 내용·이유를 적어 둠
