# Claude Skills (F&F)

F&F 브랜드 M&A 리서치 프로젝트에서 사용하는 Claude Code 스킬 모음입니다.

## Skills

### 📊 데이터 수집

| 스킬 | 설명 |
|------|------|
| **collect-stock** | 기업 주가, 시가총액, PER/PBR 등 주식 데이터 수집 + AI 밸류에이션 분석 |
| **collect-financials** | 재무제표(매출·영업이익·순이익·자산·부채) 수집 + AI 재무 건전성 분석 |
| **collect-analyst** | 애널리스트 투자의견, 목표가, 리서치 리포트 수집 |
| **collect-news** | 워치리스트 브랜드 뉴스 자동 수집 + M&A 시그널 분석 |
| **collect-consensus** | 주가·재무·뉴스·애널리스트 데이터를 종합한 마켓 컨센서스 리포트 |
| **dart-sync** | DART 기업 공시 자동 동기화 |

### 🌍 글로벌 시장

| 스킬 | 설명 |
|------|------|
| **euromonitor** | 유로모니터 Passport Stats API 연동 — 글로벌 시장 규모, 기업 점유율, 브랜드 점유율 |

### 📈 분석·리포트

| 스킬 | 설명 |
|------|------|
| **analyze-brand** | 브랜드 종합 분석 워크플로우 |
| **trend-report** | 업종 트렌드 리포트 생성 |

## 설치 방법

```bash
# 사용자 스킬 디렉토리에 클론
cd ~/.claude/
git clone https://github.com/Rachel-FnF/CLAUDESKILLS.git skills
```

## 업데이트 방법

```bash
cd ~/.claude/skills
git pull origin main
```

## 주요 트리거 예시

- "F&F 주가 분석" → `collect-stock`
- "한섬 재무제표" → `collect-financials`
- "나이키 컨센서스" → `collect-consensus`
- "패션업계 뉴스" → `collect-news`
- "중국 Sportswear Top 10" → `euromonitor`
- "글로벌 시장 규모" → `euromonitor`
- "애널리스트 목표가" → `collect-analyst`
