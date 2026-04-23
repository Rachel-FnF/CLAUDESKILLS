---
name: trend-report
description: 패션 업계 트렌드 뉴스를 수집하고 AI로 종합 분석 리포트를 생성합니다. 시장 동향 파악이 필요할 때 사용하세요.
disable-model-invocation: true
allowed-tools: Bash(py *), Read, Write
---

# 패션 트렌드 리포트 생성

최신 패션 업계 트렌드를 수집하고 AI가 종합 분석합니다.

## 실행 단계

1. **트렌드 뉴스 수집**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from collectors.trend_collector import TrendCollector
   import json

   collector = TrendCollector()
   articles = collector.collect_trend_news(days=7)

   print(f'=== Collected {len(articles)} Trend Articles ===')
   for a in articles[:10]:
       print(f'- {a[\"title\"][:60]}...')
       print(f'  Source: {a[\"source\"]}')

   # 임시 저장
   with open('data/trend_articles.json', 'w', encoding='utf-8') as f:
       json.dump(articles, f, ensure_ascii=False, default=str)
   print(f'\\nSaved to data/trend_articles.json')
   "
   ```

2. **AI 트렌드 분석**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from collectors.trend_collector import TrendAnalyzer
   import json

   # 뉴스 로드
   with open('data/trend_articles.json', 'r', encoding='utf-8') as f:
       articles = json.load(f)

   analyzer = TrendAnalyzer()
   if analyzer.is_available():
       analysis = analyzer.analyze_trends(articles)
       if analysis:
           print('=== AI Trend Analysis ===')
           print(json.dumps(analysis, ensure_ascii=False, indent=2))

           # 분석 결과 저장
           with open('data/trend_analysis.json', 'w', encoding='utf-8') as f:
               json.dump(analysis, f, ensure_ascii=False, indent=2)
       else:
           print('Analysis failed')
   else:
       print('AI API not available')
   "
   ```

3. **리포트 생성**

   분석 결과를 바탕으로 다음 형식의 리포트를 생성하세요:

   ```markdown
   # 패션 업계 트렌드 리포트

   생성일: [오늘 날짜]

   ## 1. 트렌드 요약
   [trend_summary 내용]

   ## 2. HOT 카테고리
   - [hot_categories 항목들]

   ## 3. 주목 브랜드
   [rising_brands 항목들]

   ## 4. 소비자 인사이트
   [consumer_insights 내용]

   ## 5. 비즈니스 기회
   [business_opportunities 내용]

   ## 6. 핵심 키워드
   [keywords 항목들]

   ---

   ## 원본 뉴스 (상위 10개)
   [뉴스 제목 및 출처 목록]
   ```

## 출력 형식

사용자에게 다음을 제공하세요:
1. 수집된 트렌드 뉴스 수
2. AI 분석 결과 요약 (핵심 포인트만)
3. 전체 리포트 (마크다운 형식)

## 활용 팁

- 주 1회 정기적으로 실행하여 시장 동향 파악
- M&A 타겟 발굴에 활용 (rising_brands 참고)
- 투자/인수 보고서 작성 시 참고자료로 활용
