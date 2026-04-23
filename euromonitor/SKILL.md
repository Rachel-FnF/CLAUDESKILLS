---
name: euromonitor
description: 유로모니터 Passport Stats API를 사용해 글로벌 시장 규모, 기업 점유율, 브랜드 점유율 데이터를 조회하고 분석합니다. "글로벌 시장 규모", "해외 점유율", "중국 스포츠웨어 Top", "패션 시장 성장률", "Sportswear 급성장 카테고리" 같은 글로벌 시장 데이터가 필요할 때 사용하세요.
disable-model-invocation: false
allowed-tools: Bash(py *), Bash(python *), Read, Grep, Glob, WebSearch
---

# 유로모니터 Passport Stats API 스킬

F&F 구독의 유로모니터 Passport Stats API를 호출하여 **글로벌 시장 규모 / 기업 점유율 / 브랜드 점유율** 데이터를 수집·분석합니다. 국내 금융 데이터(DART, 네이버 금융)로는 얻을 수 없는 **글로벌 벤치마크 데이터**가 필요할 때 사용하세요.

## 언제 사용하는지

- "글로벌/해외 시장 규모" "국가별 시장" "CAGR" 같은 질문
- "중국 Sportswear Top 10" 같은 해외 점유율 조회
- F&F/MLB/Discovery 관련 **글로벌 경쟁사 포지셔닝**
- "Outdoor/Performance/Sports-inspired 성장" 같은 하위 카테고리 트렌드
- 신흥 시장(인도, 인도네시아, 베트남 등) 패션 규모

DART·네이버 금융으로 해결되는 질문(개별 기업 주가·재무)은 다른 스킬을 쓰세요.

## 제공 데이터

| 서비스 | 조회 가능 데이터 |
|--------|----------------|
| **Market Size** | 국가×카테고리별 시장 규모, 성장률, 10년+ 역사 데이터 |
| **Company Share** | 시장 내 기업별 점유율/순위 (NBO 기준) |
| **Brand Share** | 시장 내 브랜드별 점유율 (ShareTypeId=5) |
| **Catalog** | 카테고리/국가/데이터타입 ID 레퍼런스 |
| **Extractor Job** | 대용량 비동기 추출 (파일 다운로드) |

F&F 구독 산업: **Apparel and Footwear (CLF)** · **Beauty and Personal Care (CT)** · **Personal Accessories (PLG)**

## 프로젝트 위치

- **Collector**: `C:\Users\AC1145\rachel.py\market-research-agent\backend\collectors\euromonitor_collector.py`
- **상수**: `C:\Users\AC1145\rachel.py\market-research-agent\backend\collectors\euromonitor_constants.py`
- **카테고리 레퍼런스**: `C:\Users\AC1145\rachel.py\market-research-agent\backend\reference\euromonitor\`
- **인증**: backend/.env (`EUROMONITOR_API_KEY`, `EUROMONITOR_USERNAME`, `EUROMONITOR_PASSWORD`)
- **Python venv**: `backend/venv/Scripts/python.exe` 사용

## 기본 사용 패턴

### 1. Python 스크립트로 조회 (권장)

```python
import sys
sys.path.insert(0, 'C:/Users/AC1145/rachel.py/market-research-agent/backend')
from collectors.euromonitor_collector import get_collector
from collectors.euromonitor_constants import Geo, Apparel, Beauty

col = get_collector()

# ── Market Size ──
data = col.get_market_size(
    geography_ids=[Geo.KOREA, Geo.CHINA],
    industry_code='CLF',
    category_ids=[Apparel.SPORTSWEAR],
)

# ── Company Share (기업 점유율) ──
# unit_type=2 → USD Value, share_type_id=1 → NBO (National Historical Owner)
data = col.get_company_share(
    geography_ids=[Geo.CHINA],
    industry_code='CLF',
    category_ids=[Apparel.SPORTSWEAR],
    unit_type=2,
    share_type_id=1,
    limit=50,
)

# ── Brand Share (브랜드 점유율) — ShareTypeId=5 가 NBO 브랜드 기본 ──
data = col.get_brand_share(
    geography_ids=[Geo.CHINA],
    industry_code='CLF',
    category_ids=[Apparel.SPORTSWEAR],
    unit_type=2,
    share_type_id=5,
    limit=50,
)
```

### 2. 한 줄 실행 (간단 질의)

```bash
cd "C:/Users/AC1145/rachel.py/market-research-agent/backend"
"venv/Scripts/python.exe" -c "import sys; sys.stdout.reconfigure(encoding='utf-8'); from collectors.euromonitor_collector import get_collector; from collectors.euromonitor_constants import Geo, Apparel; col = get_collector(); print(col.get_market_size(geography_ids=[Geo.KOREA], industry_code='CLF', category_ids=[Apparel.SPORTSWEAR]))"
```

## 핵심 상수 (자주 쓰는 ID)

### 국가 (Geo)
- `Geo.KOREA` (280), `Geo.CHINA` (195), `Geo.USA` (380), `Geo.JAPAN` (276)
- `Geo.HONG_KONG` (259), `Geo.TAIWAN` (373), `Geo.VIETNAM` (387), `Geo.INDIA` (265)
- 지역: `Geo.ASIA_PACIFIC` (105), `Geo.WORLD` (389)
- 그룹: `Geo.FNF_CORE_MARKETS`, `Geo.GREATER_CHINA`, `Geo.KEY_SEA`

### 산업 코드 (IndustryCode)
- **CLF** = Apparel and Footwear (패션)
- **CT** = Beauty and Personal Care (뷰티)
- **PLG** = Personal Accessories

### 카테고리 (Apparel)
- `Apparel.TOTAL` (18477) - Apparel and Footwear 전체
- `Apparel.SPORTSWEAR` (17620) - Sportswear
- `Apparel.OUTDOOR_APPAREL` (18960) - Discovery Expedition 관련
- `Apparel.SPORTS_INSPIRED_APPAREL` (19956) - MLB lifestyle
- `Apparel.PERFORMANCE_APPAREL` (17621)
- `Apparel.HATS_CAPS` (19777) - MLB 모자
- `Apparel.CHILDRENSWEAR` (18478) - MLB Kids
- 전체 목록: `reference/euromonitor/README.md`

### UnitType (Company/Brand Share만)
- `1` = Rank (순위)
- `2` = USD Value (매출액)

### ShareTypeId
- `1` = NBO (National Historical Owner) — Company Share 기본
- `5` = NBO Brand — Brand Share 기본

## 응답 데이터 구조

### Market Size
```json
{
  "total": 4,
  "marketSizes": [{
    "geographyName": "China",
    "categoryName": "Sportswear",
    "industry": "Apparel and Footwear",
    "dataType": "Retail/Off-trade Value RSP",
    "unitName": "USD",
    "unitMultiplier": 1000000,
    "data": [{"year": 2020, "value": 43167.03}, ...]
  }]
}
```

**실제 값 계산**: `value * unitMultiplier`. 위 예시는 2020년 중국 Sportswear = 43167 × 1,000,000 = **$43.2B USD**

### Company Share / Brand Share
```json
{
  "total": 34,
  "companyShares": [{
    "companyName": "Anta (China) Co Ltd",
    "dataType": "Retail/Off-trade Value RSP",
    "shareTypeName": "National - Historical Owner (NBO)",
    "data": [{"year": 2025, "value": 11606.00}, ...]  // USD Million
  }]
}
```

## 분석 패턴 예시

### A. 국가별 CAGR 계산
```python
d = {p['year']: p['value'] for p in record['data']}
cagr = ((d[2025] / d[2020]) ** (1/5) - 1) * 100
```

### B. 시장점유율 % 계산
```python
# CompanyShare 응답은 USD 매출액. 전체 시장 대비 %로 변환
company_sales = 11606  # Anta 2025
market_total = 60496   # China Sportswear 2025 전체 (Market Size에서 조회)
share_pct = company_sales / market_total * 100  # = 19.2%
```

### C. Top N 기업 추출
```python
companies = [(r['companyName'], r['data'][-1]['value']) for r in rows if r['companyName'] not in ('Total', 'Others')]
companies.sort(key=lambda x: x[1], reverse=True)
top10 = companies[:10]
```

## 헬퍼 함수

```python
from collectors.euromonitor_constants import lookup_geo, lookup_category

# 이름으로 ID 찾기
lookup_geo("Vietnam")                              # → "387"
lookup_category("Outdoor", industry="Apparel and Footwear")  # → 매칭 카테고리 리스트
```

## 주의사항 & 제약

| 항목 | 내용 |
|------|------|
| **토큰 캐싱** | JWT는 24시간 유효. `backend/.euromonitor_token.json`에 자동 캐싱됨 |
| **Fair Usage** | 명시적 Rate Limit 없으나 과도한 호출 시 429. 분당 1000 req 여유 있음 |
| **데이터 갱신** | 연 1회 (researchYear 필드 확인) |
| **구독 범위** | F&F는 Apparel, Beauty, Personal Accessories 3개 산업만 |
| **제공 불가** | Retail Channel, 소비자 설문조사, 리포트 전문(정성) |
| **통화** | 기본 USD (Fixed ex rates). Local currency 조회 시 `LocalCurrency=true` |

## 응답 출력 규칙

조회 결과를 사용자에게 제시할 때 반드시 지켜라:

1. **실제 시장 규모**는 `value * unitMultiplier`로 계산해서 보여줄 것 (예: "$43.2B USD")
2. **dataType / unitName / researchYear**을 리포트 하단에 명시 (출처/연도)
3. CAGR, YoY, 점유율 % 같은 **파생 지표 계산해서 제공**
4. F&F 관점 **인사이트·시사점** 섹션 추가 (MLB/Discovery 연관성)
5. 출처: "Euromonitor Passport (researchYear: 2025)" 표기

## 일반적인 질문 패턴 매핑

| 사용자 질문 | 호출 |
|------------|------|
| "한국 Sportswear 시장 규모" | `get_market_size([Geo.KOREA], 'CLF', [Apparel.SPORTSWEAR])` |
| "중국 Sportswear Top 10" | `get_company_share([Geo.CHINA], 'CLF', [Apparel.SPORTSWEAR], unit_type=2, limit=15)` |
| "Nike 글로벌 브랜드 점유율" | `get_brand_share([여러 국가], 'CLF', [Apparel.SPORTSWEAR])` 후 "Nike" 필터 |
| "Outdoor Apparel 급성장 국가" | 모든 국가 × Outdoor Apparel 카테고리 CAGR 비교 |
| "바닐라코 중국 뷰티 시장" | `get_market_size([Geo.CHINA], 'CT', [Beauty.BATH_SHOWER 등])` |

## 에러 대응

- **401**: 토큰 만료 → collector가 자동 재발급. 수동 처리 불필요
- **404 "no Route matched"**: 엔드포인트 경로 오류. collector의 URL 구성 로직 점검
- **400 "UnitType not valid"**: `unit_type` 파라미터를 숫자(1 또는 2)로 전달
- **204 No Content**: 경로는 맞지만 조회 범위에 데이터 없음. ShareTypeId 변경 시도
- **403**: 구독 범위 밖. F&F는 CLF/CT/PLG 3개 산업만 가능

## 출력 완료 후 다음 제안 (선택)

분석 결과 하단에 "다음 분석 제안"으로 이어갈 수 있는 질의 2-3개 추천:
- 하위 카테고리 드릴다운
- 다른 국가 비교
- Brand Share 심층 분석
- 엑셀/CSV 내보내기
