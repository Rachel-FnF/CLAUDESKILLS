---
name: collect-financials
description: 기업의 재무제표(매출, 영업이익, 순이익, 자산, 부채)를 수집하고 AI로 재무 건전성과 성장성을 심층 분석합니다. 재무제표 조회나 재무 분석이 필요할 때 사용하세요.
disable-model-invocation: false
allowed-tools: Bash(py *), Read, Grep, WebSearch, WebFetch
---

# 재무제표 수집 + AI 심층 분석 스킬

기업의 재무제표 데이터를 수집하고, AI로 주석/비용/투자 분석까지 포함한 심층 인사이트를 도출합니다.

## 수집 대상 데이터

- 매출액 (Revenue)
- 영업이익 (Operating Income)
- 당기순이익 (Net Income)
- 총자산 (Total Assets)
- 총부채 (Total Liabilities)
- 자본총계 (Total Equity)
- 영업이익률, 순이익률, ROE, 부채비율 (계산)

## API 키 조회 순서

API 키는 다음 순서로 확인합니다:
1. `backend/.env` 파일 (프로젝트 환경변수) — Read 도구로 파일을 읽어 해당 키 값을 확인
2. `.claude/market-research-agent.local.md` (플러그인 설정) — 1번에 없을 경우 확인

키 매핑:
| backend/.env | local.md | 용도 |
|---|---|---|
| `DART_API_KEY` | `dart_api_key` | DART 재무제표 |
| `FSC_API_KEY` | — | 금융위원회 기업재무정보 |

## 수집 분기 로직

### 1. 국내 기업 (상장/비상장 모두)

DART API를 우선 사용합니다. **DART API 키 필요.**

**수집 절차:**
1. 위 API 키 조회 순서에 따라 `DART_API_KEY` 확인
2. 키가 있으면:
   - DART API `/fnlttSinglAcnt.json`으로 재무제표 조회
   - 최근 3개년 연간 + 최근 분기 데이터 수집
   - `backend/collectors/dart_collector.py`의 `DartCollector` 활용
3. 키가 없으면:
   - WebSearch로 "{기업명} 재무제표 매출액 영업이익 {현재연도}" 검색
   - WebFetch로 DART 공시페이지, 네이버 금융, FnGuide 등에서 수집

**비상장 외감법인:**
- DART에 공시 의무가 있으므로 DART API로 동일하게 수집 가능
- 종목코드 없이 기업고유번호(corp_code)로 조회

**비상장 비외감법인:**
- DART에 데이터 없음
- WebSearch로 대체: "{기업명} 매출 영업이익", 기업 IR페이지, 언론보도 등에서 수집

### 2. 미국 기업

SEC EDGAR를 우선 사용합니다. **API 키 불필요.**

**수집 절차:**
1. WebSearch로 "{company name} SEC EDGAR 10-K filing" 검색
2. SEC EDGAR에서 10-K (연간) 또는 10-Q (분기) 보고서 확인
3. WebFetch로 EDGAR XBRL 재무데이터 페이지 접근
4. 매출, 영업이익, 순이익, 자산, 부채 추출

**대체 경로:**
- WebSearch로 "{company name} annual revenue net income {year}" 검색
- Bloomberg, Reuters, Macrotrends 등에서 재무 요약 수집

### 3. 영국 기업

Companies House를 우선 시도합니다.

### 4. 기타 해외 기업

WebSearch를 통한 유연한 탐색:
1. "{company name} financial statements {year}"
2. "{company name} revenue operating income annual report"
3. 기업 공식 IR 페이지, Bloomberg, Reuters 등에서 수집

## AI 심층 분석 (수집 후 반드시 수행)

### 분석 항목

**1. 성장성 분석**
- 매출/영업이익/순이익 성장률 YoY (3개년 추세)
- 성장 드라이버 추정

**2. 수익성/비용 구조 분석**
- 영업이익률, 순이익률 추이
- 비용 증가가 매출 증가를 초과하는지 여부

**3. 재무 건전성 분석**
- 부채비율 추이, 유동비율, 이자보상배율 추정
- 자본잠식 위험 여부

**4. 투자 활동 분석**
- CAPEX 규모 및 추이, R&D 투자 비중
- WebSearch로 "{기업명} 설비투자 CAPEX" 검색하여 보강

**5. M&A 관점 재무 매력도**
- 순자산가치(BPS) 대비 시장 평가
- 안정적 현금 창출력, 부채 인수 시 부담 규모

## 출력 형식

```
## 재무제표: {기업명}

### 최근 3개년 요약 (단위: 억원 또는 $M)

| 항목 | 2023 | 2024 | 2025 | YoY 추이 |
|------|------|------|------|----------|
| 매출액 | 0,000 | 0,000 | 0,000 | +0.0% |
| 영업이익 | 000 | 000 | 000 | +0.0% |
| 당기순이익 | 000 | 000 | 000 | +0.0% |
| 총자산 | 0,000 | 0,000 | 0,000 | |
| 총부채 | 0,000 | 0,000 | 0,000 | |
| 자본총계 | 0,000 | 0,000 | 0,000 | |

### 주요 지표
| 항목 | 2023 | 2024 | 2025 | 판단 |
|------|------|------|------|------|
| 영업이익률 | 0.0% | 0.0% | 0.0% | 개선/악화 |
| 순이익률 | 0.0% | 0.0% | 0.0% | 개선/악화 |
| ROE | 0.0% | 0.0% | 0.0% | |
| 부채비율 | 000% | 000% | 000% | 안정/주의 |

### AI 심층 인사이트

#### 성장성 진단
- (매출/이익 성장 추세, 성장 드라이버 분석)

#### 비용 구조 분석
- (마진 변화 원인, 비용 효율성 판단)

#### 재무 건전성
- (부채 수준, 현금흐름 안정성, 리스크 요인)

#### M&A 관점 재무 매력도
- (순자산가치, 현금 창출력, 인수 시 부담/시너지)

출처: DART API / SEC EDGAR / WebSearch ({구체적 출처})
기준일: YYYY-MM-DD
```
