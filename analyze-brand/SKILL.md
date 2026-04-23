---
name: analyze-brand
description: 특정 브랜드의 정보를 AI로 분석하고 워치리스트에 등록합니다. 새로운 브랜드를 조사하거나 등록할 때 사용하세요.
argument-hint: <브랜드명>
disable-model-invocation: true
allowed-tools: Bash(py *), Read, Write
---

# 브랜드 AI 분석 및 등록

브랜드명: $ARGUMENTS

## 실행 단계

1. **AI로 브랜드 정보 조회**
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from analyzers.brand_info_generator import BrandInfoGenerator
   import json

   generator = BrandInfoGenerator()
   if generator.is_available():
       result = generator.generate('$ARGUMENTS')
       if result:
           print('=== Brand Info ===')
           print(json.dumps(result, ensure_ascii=False, indent=2))
       else:
           print('Failed to generate brand info')
   else:
       print('AI API not available')
   "
   ```

2. **DART에서 유사 기업 검색** (상장사인 경우)
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from collectors.dart_collector import DartCollector

   dart = DartCollector()
   candidates = dart.search_corp_code_candidates('$ARGUMENTS', top_n=5, min_score=0.3)

   print('=== DART Matches ===')
   if candidates:
       for name, code, score in candidates:
           print(f'{name} (similarity: {score:.0%})')
   else:
       print('No DART matches found')
   "
   ```

3. **워치리스트 등록**
   조회된 정보를 사용자에게 보여주고 확인 후 등록할지 물어보세요.

   등록 코드:
   ```bash
   cd "C:/Users/AC1145/TEST.py/brand_ma_dashboard" && py -c "
   import sys
   sys.path.insert(0, '.')
   from database import get_db_session, Brand

   db = get_db_session()

   # 중복 체크
   existing = db.query(Brand).filter(Brand.name == '브랜드명').first()
   if existing:
       print('Already registered')
   else:
       brand = Brand(
           name='브랜드명',
           name_en='English Name',
           company_name='Company',
           category='Category',
           is_watchlist=True,
           watchlist_priority=7,
       )
       db.add(brand)
       db.commit()
       print(f'Registered: {brand.name}')
   db.close()
   "
   ```

## 출력 형식

1. 브랜드 기본 정보 (AI 분석 결과)
2. DART 매칭 후보 (상장사 확인)
3. 등록 여부 확인 질문
4. 등록 완료 메시지
