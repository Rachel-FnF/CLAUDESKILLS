---
name: dart-sync
description: DART 전자공시에서 상장 브랜드의 재무데이터를 자동으로 동기화합니다. 재무 정보 업데이트가 필요할 때 사용하세요.
disable-model-invocation: true
allowed-tools: Bash(py *), Read
---

# DART 재무데이터 동기화

상장 브랜드의 재무제표를 DART API에서 자동으로 수집합니다.

## 실행 단계

1. **상장 브랜드 목록 확인**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from database import get_db_session, Brand

   db = get_db_session()
   listed = db.query(Brand).filter(Brand.is_listed == True).all()

   print('=== Listed Brands ===')
   for b in listed:
       print(f'{b.id}. {b.name} ({b.company_name or \"-\"})')
   print(f'\\nTotal: {len(listed)}')
   db.close()
   "
   ```

2. **DART 재무데이터 수집**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from collectors.dart_collector import DartCollector

   dart = DartCollector()
   results = dart.collect_all_listed()

   print('=== DART Sync Complete ===')
   total = sum(results.values())
   print(f'Total: {total} years of data')
   for brand, count in results.items():
       if count > 0:
           print(f'  - {brand}: {count} years')
   "
   ```

3. **수집된 재무데이터 확인**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from database import get_db_session, Brand, BrandFinancial

   db = get_db_session()
   financials = db.query(BrandFinancial, Brand.name).join(Brand).order_by(
       BrandFinancial.fiscal_year.desc()
   ).limit(10).all()

   print('=== Recent Financial Data ===')
   for f, brand_name in financials:
       revenue = f'{f.revenue/100000000:,.0f}억' if f.revenue else '-'
       op_margin = f'{f.operating_margin:.1f}%' if f.operating_margin else '-'
       print(f'{brand_name} ({f.fiscal_year}): 매출 {revenue}, 영업이익률 {op_margin}')
   db.close()
   "
   ```

## 출력 형식

1. 상장 브랜드 목록
2. DART 수집 결과 (브랜드별 연도 수)
3. 최신 재무데이터 요약
4. 수집 실패한 브랜드 (있는 경우) 및 원인

## 주의사항

- DART API 키가 .env에 설정되어 있어야 합니다
- 회사명이 DART 정식 명칭과 일치해야 수집 가능
- 매칭 안 되는 경우 '기업명 매칭' 탭에서 수동 매칭 필요
