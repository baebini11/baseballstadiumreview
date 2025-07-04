# 야구장 Google 리뷰 분석

20160463 배원빈

> 참고

코드: MIT

데이터·보고서: CC BY-NC-SA 4.0

## 목차

1. [프로젝트 준비](#프로젝트-준비)
2. [과정](#과정)
3. [결과 및 통계](#결과-및-통계)
4. [결론](#결론)

---

## 프로젝트 준비

### 🔍 분석 대상

| 국가    | 구장                          |
| ------- | ----------------------------- |
| 🇰🇷 한국 | **사직구장**                  |
| 🇺🇸 미국 | **텍사스 글로브 라이프 파크** |
| 🇺🇸 미국 | **보스턴 펜웨이 파크**        |

### 🛠️ 분석 방법

- **Colab**
  - 한–영 리뷰 토크나이즈 & 전처리
- **Voyant Tools**
  - 빈도·문맥 시각화
- **임베딩 벡터**
  - Sentence-BERT로 문장 유사도 측정
- **언론 기사 크로스체크**
  - 리뷰-미디어 간 여론 일치도 확인

### ❓ 핵심 질문

1. 평점이 높은/낮은 리뷰에서 반복되는 표현은 무엇인가?
2. 구글 리뷰와 대중매체 보도는 얼마나 비슷하거나 다른가?

---

## 과정

1. **데이터 수집**
   - 각 구장의 Google 리뷰 크롤링 → _평점만_ 있는 글 제외, **200 문장** 추출
2. **전처리**
   - `sent_tokenize` 로 문장 분리
   - 불용어 제거·토큰화 (한글 & 영어)
3. **임베딩 & 유사도 분석**
   - `sentence-transformers/all-MiniLM-L6-v2` 모델 사용
   - Cosine similarity 로 표현 간 거리 계산
4. **시각화**
   - Voyant 워드클라우드 & KWIC
   - Matplotlib 막대그래프
5. **교차 검증**
   - 샘플 문장 수작업 검토 → 임베딩 결과 신뢰도 확인

> **Why Voyant + Colab?**  
> 두 가지 툴을 병행해 **빈도 기반**·**의미 기반** 시각을 모두 확보하기 위함입니다.

---

## 결과 및 통계

### 1) 사직구장

| 평점  | 핵심 키워드    | 인사이트                      |
| ----- | -------------- | ----------------------------- |
| ★★★★★ | `장소`, `응원` | 팬 경험·응원 문화에 높은 만족 |
| ★☆☆☆☆ | `낙후`, `위생` | 노후 시설·청결 불만           |

### 2) 텍사스 글로브 라이프 파크

| 평점  | 핵심 키워드                 | 인사이트               |
| ----- | --------------------------- | ---------------------- |
| ★★★★★ | `경관`, `좌석`              | 시야 및 관람 환경 호평 |
| ★☆☆☆☆ | `음식`, `티켓 가격`, `시설` | F&B·가격·편의성 불만   |

> **주의사항**
>
> - `good` → 대부분 _not good_ 구문에서 등장 → 분석 제외
> - `like` → 비교 접속사(“~와 같은”)가 많아 제거

### 3) 보스턴 펜웨이 파크

| 평점  | 핵심 키워드                 | 인사이트                   |
| ----- | --------------------------- | -------------------------- |
| ★★★★★ | `역사`, `가이드`, `경기장`  | 전통·투어 프로그램 호평    |
| ★☆☆☆☆ | `팬/스태프`, `시설`, `불편` | 관람객 매너·노후 시설 불만 |

### 🧩 임베딩 유사도 검증

- 추출 키워드 ↔ 유사 표현 비교 시 **의미 일치율** 높음 → 분석 신뢰도 확보
- 영어 리뷰는 직접적 불만 대신 **완곡 표현** 경향

### 📰 언론 기사 비교

| 구장   | 기사 내용                        | 리뷰와의 상관성                   |
| ------ | -------------------------------- | --------------------------------- |
| 사직   | 시설 노후·개선 필요              | **일치**                          |
| 텍사스 | 2024-06-02 외부 음식 반입 허용   | 음식·가격 불만 해소 시도          |
| 펜웨이 | ‘그린 몬스터’·정치적 현수막 논란 | 벽·팬 갈등에 대한 **호불호** 반영 |

---

## 결론

1. **평점별 어휘 편향이 뚜렷** — 높은 평점에는 긍정어, 낮은 평점에는 부정어가 집중됨.
2. **임베딩 벡터 분석**이 실제 문맥과 높은 상관성을 보여 의미 기반 통계의 유효성을 확인.
3. **구글 리뷰 ↔ 언론 보도** 키워드·톤이 대체로 일치 → 리뷰 데이터는 여론 지표로 활용 가능.
4. 야구장 평가는 **시설**뿐 아니라 **경험 가치(가족·연인과의 추억 등)** 가 크게 작용함.

---

### 데이터 전처리 & 임베딩 예시

```python
from pathlib import Path
from nltk.tokenize import sent_tokenize
from sentence_transformers import SentenceTransformer, util

# 1️⃣ 리뷰 로드 (파일 → 리스트)
#    └─ data/
#        ├─ sajik.txt
#        ├─ globe_life.txt
#        └─ fenway.txt
reviews_dir = Path("data")
reviews: list[str] = []

for txt in reviews_dir.glob("*.txt"):
    reviews += (txt.read_text(encoding="utf-8").splitlines())

# 2️⃣ 문장 단위로 분리
sentences = [
    sent.strip()
    for review in reviews
    for sent in sent_tokenize(review)
    if sent.strip()
]

# 3️⃣ 문장 임베딩 & 유사도 측정
model = SentenceTransformer("all-MiniLM-L6-v2")
embeddings = model.encode(sentences, convert_to_tensor=True)

sim = util.cos_sim(embeddings[0], embeddings[1]).item()
print(f"문장 0–1 유사도: {sim:.4f}")
```
