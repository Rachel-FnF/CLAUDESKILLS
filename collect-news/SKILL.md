---
name: collect-news
description: 워치리스트 브랜드의 뉴스를 자동 수집하고 M&A 시그널을 분석합니다. 뉴스 수집이나 M&A 동향 파악이 필요할 때 사용하세요.
disable-model-invocation: false
allowed-tools: Bash(py *), Read, Grep
---

# 뉴스 수집 자동화

워치리스트에 등록된 모든 브랜드의 최신 뉴스를 수집하고 M&A 시그널을 분석합니다.

## 실행 단계

1. **뉴스 수집 실행**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from collectors.news_collector import NewsCollector

   collector = NewsCollector()
   results = collector.collect_all_watchlist()

   print('=== News Collection Complete ===')
   total = sum(results.values())
   print(f'Total: {total} articles')
   for brand, count in results.items():
       if count > 0:
           print(f'  - {brand}: {count}')
   "
   ```

2. **M&A 시그널 확인**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from database import get_db_session, NewsArticle, Brand

   db = get_db_session()
   ma_news = db.query(NewsArticle, Brand.name).join(Brand).filter(
       NewsArticle.is_ma_signal == True
   ).order_by(NewsArticle.created_at.desc()).limit(10).all()

   print('=== M&A Signals ===')
   if ma_news:
       for news, brand_name in ma_news:
           print(f'[{brand_name}] {news.title[:50]}...')
           print(f'  Type: {news.ma_signal_type}')
   else:
       print('No M&A signals found')
   db.close()
   "
   ```

3. **결과 요약 제공**
   - 수집된 총 뉴스 수
   - 브랜드별 수집 현황
   - M&A 시그널 목록 (있는 경우)

## 출력 형식

수집 완료 후 다음을 보고하세요:
- 총 수집 뉴스 수
- M&A 시그널 발견 여부 및 상세 내용
- 주목할 만한 뉴스 하이라이트
