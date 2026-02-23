# 키움증권 REST API 개발 문서

**주인님의 매매 시스템용 통합 가이드**

---

## 📚 문서 구성

### 1. **KIWOOM_DOMESTIC_ORDER_API.md** (주문 API)
- 국내주식 주문 (kt10001)
- 지정가, 시장가, 조건부 주문
- Python 구현 예제
- 에러 처리 및 재시도 로직

### 2. **KIWOOM_INQUIRY_API.md** (조회 API)
- 체결 조회 (kt11007)
- 미체결 조회 (kt11008)
- 보유 종목 조회
- 계좌 잔고 확인

### 3. **KIWOOM_CANCEL_MODIFY_API.md** (취소/정정)
- 주문 취소 (kt10002)
- 주문 정정 (kt10003)
- 부분 취소 처리

### 4. **KIWOOM_AUTHENTICATION.md** (인증)
- OAuth 2.0 토큰 발급
- 토큰 갱신 (Refresh)
- 토큰 관리 전략

### 5. **KIWOOM_TRADING_STRATEGY.md** (전략 문서)
- 데이 트레이딩 (당일 청산)
- 스윙 트레이딩 (5일 보유)
- 시간외 거래
- RL 기반 자동 매매

### 6. **KIWOOM_ERROR_CODES.md** (에러 참조)
- 전체 에러 코드 맵
- 원인 및 대응법
- 재시도 전략

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

| API | TR명 | 기능 | 상태 |
|-----|------|------|------|
| 주문 | kt10001 | 국내주식 주문 | ✅ 완성 |
| 취소 | kt10002 | 주문 취소 | 📝 예정 |
| 정정 | kt10003 | 주문 정정 | 📝 예정 |
| 체결 조회 | kt11007 | 체결 내역 조회 | 📝 예정 |
| 미체결 조회 | kt11008 | 미체결 주문 조회 | 📝 예정 |
| 계좌 조회 | kt11009 | 계좌 정보 조회 | 📝 예정 |
| 보유 종목 | kt11010 | 보유 주식 조회 | 📝 예정 |

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
| 2026-02-23 | 초판 작성 (kt10001) |
| - | - |

---

**마지막 수정:** 2026-02-23
**작성자:** 🤖 Claude (주인님 매매 시스템)
