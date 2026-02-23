# 키움증권 REST API 개발 문서

**주인님의 매매 시스템용 통합 가이드**

---

## 📚 문서 구성

### 📖 전체 API 목록
1. **KIWOOM_ALL_DOMESTIC_APIs.md** ⭐ (NEW)
   - 국내주식 전체 102개 API
   - 14개 섹션별 정리 (계좌, 공매도, 주문, 차트 등)
   - TR명, API ID, 한글명 명시
   - 주요 기능 설명

### 상세 가이드
2. **KIWOOM_DOMESTIC_ORDER_API.md** (주문 API)
   - 국내주식 주문 (kt10001)
   - 지정가, 시장가, 조건부 주문
   - Python 구현 예제
   - 에러 처리 및 재시도 로직

3. **KIWOOM_MODIFY_CANCEL_API.md** (정정/취소)
   - 주문 정정 (kt10002)
   - 주문 취소 (kt10003)
   - 부분 취소 처리
   - 실전 패턴 (손절, 추가 매수)

### 투자 전략 가이드
4. **TTEOLSA_OPAL_INVESTMENT_GUIDE.md** ⭐ (NEW)
   - 떨사오팔 전략 완전 가이드
   - 종목 선정 (거래량 기준)
   - 매수/매도 신호 (MA기반)
   - 포지션 관리 (동시 보유)
   - 실전 실행 명령어 (Mock/Live)
   - Q&A (7가지 주요 질문)

### 예정 문서
5. **KIWOOM_INQUIRY_API.md** (조회 API)
   - 체결 조회 (ka10076)
   - 미체결 조회 (ka10075)
   - 보유 종목 조회

6. **KIWOOM_AUTHENTICATION.md** (인증)
   - OAuth 2.0 토큰 발급
   - 토큰 갱신 (Refresh)
   - 토큰 관리 전략

---

## 🚀 빠른 시작

### 1단계: 인증

```python
from kiwoom_api import KiwoomAuth

auth = KiwoomAuth()
token = auth.get_access_token(
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET"
)
```

### 2단계: 매매

```python
from kiwoom_api import KiwoomTradingAPI

api = KiwoomTradingAPI(access_token=token, is_mock=True)

# 삼성전자 10주 70,000원 매수
order = api.buy_limit("005930", 10, 70000)
print(f"주문번호: {order['ord_no']}")
```

### 3단계: 체결 확인

```python
inquire = KiwoomInquireAPI(access_token=token)
filled = inquire.get_execution_list()
print(f"체결 건수: {len(filled)}")
```

---

## 🏗️ 시스템 아키텍처

```
┌─────────────────┐
│   주인님 매매   │
│   전략 (RL)     │
└────────┬────────┘
         │
    ┌────▼─────────────────────┐
    │  KiwoomTradingAPI         │
    │  - 주문 (kt10001)         │
    │  - 취소 (kt10002)         │
    │  - 정정 (kt10003)         │
    └────┬─────────────────────┘
         │
    ┌────▼──────────────────────┐
    │  Kiwoom REST API           │
    │  https://api.kiwoom.com    │
    └────┬──────────────────────┘
         │
    ┌────▼──────────────────────┐
    │  PostgreSQL (Local)        │
    │  - 주문 로그               │
    │  - 체결 기록               │
    │  - 보유 포지션             │
    └───────────────────────────┘
```

---

## 📋 API 목록

| 카테고리 | API 수 | 상태 | 가이드 |
|---------|--------|------|--------|
| 계좌 | 12개 | ✅ 수집완료 | 📖 전체목록 |
| 공매도 | 7개 | ✅ 수집완료 | 📖 전체목록 |
| 기관/외국인 | 4개 | ✅ 수집완료 | 📖 전체목록 |
| 대차거래 | 5개 | ✅ 수집완료 | 📖 전체목록 |
| 순위정보 | 12개 | ✅ 수집완료 | 📖 전체목록 |
| 시세 | 10개 | ✅ 수집완료 | 📖 전체목록 |
| 신용주문 | 4개 | ✅ 수집완료 | 📖 전체목록 |
| 실시간시세 | 8개 | ✅ 수집완료 | 📖 전체목록 |
| 업종 | 6개 | ✅ 수집완료 | 📖 전체목록 |
| 조건검색 | 4개 | ✅ 수집완료 | 📖 전체목록 |
| 종목정보 | 13개 | ✅ 수집완료 | 📖 전체목록 |
| 주문 | 8개 | ✅ 상세가이드 | 📖 완성 |
| 차트 | 6개 | ✅ 수집완료 | 📖 전체목록 |
| 테마 | 3개 | ✅ 수집완료 | 📖 전체목록 |
| **합계** | **102개** | ✅ **완성** | 🚀 **운영중** |

---

## 🔐 보안 사항

### 토큰 관리

```bash
# .env 파일에 저장 (절대 git에 커밋 금지!)
KIWOOM_CLIENT_ID=xxx
KIWOOM_CLIENT_SECRET=yyy
KIWOOM_ACCESS_TOKEN=zzz
```

### 요청/응답 로깅

```python
# 민감한 정보는 마스킹
api_request = {
    "authorization": "Bearer EgicyxXXXXXXXXXXXX...",  # 마스킹
    "stk_cd": "005930",
    "ord_qty": "10"
}
```

---

## 💡 실전 팁

### 1. Rate Limit 관리
- 초당 최대 10 req/sec
- 분당 최대 600 req/min
- 초과시 429 에러

### 2. 에러 처리
```python
try:
    order = api.buy_limit("005930", 10, 70000)
except Exception as e:
    if "429" in str(e):
        time.sleep(5)  # Rate limit
        order = api.buy_limit("005930", 10, 70000)
```

### 3. 장 시간 관리
- **정규 장:** 09:00 ~ 15:30
- **시간외 장시작전:** 08:30 ~ 09:00
- **시간외 장마감후:** 15:30 ~ 18:00

### 4. 체결 확인
```python
# 주문 직후 1초 대기
time.sleep(1)
filled = inquire.get_execution_list(ord_no="1234567")
```

---

## 📞 문제 해결

### Q: 401 Unauthorized
**A:** 토큰 만료. 새 토큰 발급 필요.
```python
token = auth.refresh_token(refresh_token)
```

### Q: 400 Bad Request
**A:** 파라미터 오류. 다음 확인:
- 종목코드 6자리? (앞에 0 패딩)
- 주문수량 정수? (소수점 제거)
- 가격 범위 유효? (상한가/하한가 초과 안함)

### Q: 429 Too Many Requests
**A:** Rate limit 초과. 몇 초 대기 후 재시도.

### Q: 500 Internal Server Error
**A:** 키움 서버 오류. 상태 확인 후 재시도.

---

## 🔄 개선 예정

- [ ] WebSocket 실시간 체결 알림
- [ ] 주문 자동 매칭 (bid/ask)
- [ ] 포트폴리오 최적화
- [ ] 성과 분석 리포트

---

## 📝 문서 업데이트

| 날짜 | 내용 |
|------|------|
| 2026-02-23 | 떨사오팔 투자법 완전 가이드 추가 |
| 2026-02-23 | kt10002(정정), kt10003(취소) 추가 |
| 2026-02-23 | kt10001(주문) 초판 작성 |

---

**마지막 수정:** 2026-02-23
**작성자:** 🤖 Claude (주인님 매매 시스템)
