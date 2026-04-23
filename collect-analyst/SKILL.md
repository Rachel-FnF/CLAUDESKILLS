---
name: collect-analyst
description: 애널리스트 투자의견, 목표가, 리서치 리포트를 수집합니다. 증권사 목표가나 투자의견 조회가 필요할 때 사용하세요.
disable-model-invocation: false
allowed-tools: Bash(py *), Read, Grep, WebSearch, WebFetch
---

# 애널리스트 투자의견 수집 스킬

증권사 애널리스트의 투자의견과 목표가를 수집합니다.

## 수집 대상 데이터

- 증권사명 (broker)
- 투자의견 (매수/중립/매도 또는 Buy/Hold/Sell)
- 목표가 (target price)
- 리포트 제목
- 발행일
- 컨센서스 목표가 (평균)
- 의견 분포 (매수/중립/매도 비율)

## 수집 분기 로직

### 1. 국내 상장사 (종목코드 있음)

네이버 금융에서 스크래핑합니다. API 키 불필요.

**수집 절차:**
1. 프로젝트 DB에서 종목코드(stock_code) 확인
2. `backend/collectors/analyst_collector.py`의 `AnalystCollector` 사용
3. 네이버 금융 리서치 페이지 스크래핑:
   - URL: `https://finance.naver.com/research/company_list.naver`
   - 수집: 리포트 제목, 증권사, 투자의견, 목표가, 날짜

**직접 수집이 안 될 경우:**
- WebSearch로 "{기업명} 증권사 목표가 투자의견 {현재연도}" 검색
- WebFetch로 네이버 금융, 한경 컨센서스, FnGuide 등에서 수집

### 2. 해외 상장사

WebSearch를 통해 수집합니다.

**수집 절차:**
1. WebSearch로 순차 검색:
   - "{company name} analyst ratings target price {current year}"
   - "{company name} consensus estimate buy sell hold"
2. TipRanks, MarketBeat, Yahoo Finance 등에서 수집
3. WebFetch로 증권사별 투자의견, 목표가, 컨센서스 목표가 추출

### 3. 비상장사

공식 애널리스트 커버리지가 없으므로 대체 정보를 수집합니다.

**수집 절차:**
1. WebSearch로 대체 정보 탐색:
   - "{기업명} 기업분석 보고서" 또는 "{company name} company analysis report"
   - "{기업명} 투자 유치 기업가치" (스타트업인 경우)
2. "공식 애널리스트 커버리지 없음"을 명시하고, 찾은 대체 정보를 제공

## API 키 조회 순서

API 키는 다음 순서로 확인합니다:
1. `backend/.env` 파일 — Read 도구로 파일을 읽어 해당 키 값을 확인
2. `.claude/market-research-agent.local.md` — 1번에 없을 경우 확인

## AI 종합분석

수집된 의견을 종합하여:
- 종합의견 (매수/중립/매도/관망)
- 컨센서스 목표가, 상승여력
- 핵심근거 3가지, 리스크 2가지

## 출력 형식

```
## 애널리스트 의견: {기업명}

### 투자의견 현황 (최근 3개월)

| 증권사 | 투자의견 | 목표가 | 날짜 |
|--------|----------|--------|------|
| OO증권 | 매수 | ₩00,000 | MM-DD |
| XX증권 | 매수 | ₩00,000 | MM-DD |

### 컨센서스 요약
| 항목 | 값 |
|------|-----|
| 컨센서스 목표가 | ₩00,000 |
| 현재가 대비 상승여력 | +00.0% |
| 의견 분포 | 매수 0건 / 중립 0건 / 매도 0건 |

### AI 종합분석
(수집된 의견을 종합한 분석 요약)

수집 소스: 네이버 금융 / WebSearch ({구체적 출처})
수집 시각: YYYY-MM-DD HH:MM
```
