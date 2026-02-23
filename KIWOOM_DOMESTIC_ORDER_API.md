# 키움증권 REST API - 국내주식 주문 (kt10001)

**최종 업데이트:** 2026-02-23

---

## 1. API 개요

### 기본 정보

| 항목 | 값 |
|------|-----|
| **API명** | 국내주식 주문 |
| **TR명** | kt10001 |
| **API ID** | kt10001 |
| **Method** | POST |
| **운영 도메인** | https://api.kiwoom.com |
| **모의투자 도메인** | https://mockapi.kiwoom.com (KRX만 지원) |
| **URL** | /api/dostk/ordr |
| **Format** | JSON |
| **Content-Type** | application/json;charset=UTF-8 |

---

## 2. 인증 (Authentication)

### 접근 토큰 (Authorization Header)

```
Authorization: Bearer {ACCESS_TOKEN}
```

**예시:**
```
Authorization: Bearer Egicyx1a2b3c4d5e6f...
```

- **Type:** String
- **Required:** Yes
- **Length:** 최대 1000자
- **설명:** OAuth 2.0 Bearer 토큰 형식
- **유효기간:** 24시간 (별도 설정 가능)

### 토큰 발급 흐름

1. 인증 서버에서 Access Token 획득
2. Authorization Header에 "Bearer " + 토큰 추가
3. 모든 API 요청시 포함

---

## 3. 요청 (Request)

### Header

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **authorization** | 접근토큰 | String | **Y** | 1000 | Bearer {TOKEN} 형식 |
| **cont-yn** | 연속조회여부 | String | N | 1 | 응답 Header의 cont-yn값 Y일 경우 세팅 |
| **next-key** | 연속조회키 | String | N | 50 | 응답 Header의 next-key값 세팅 |
| **api-id** | TR명 | String | **Y** | 10 | kt10001 (고정값) |

### Body Parameters

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **dmst_stex_tp** | 국내거래소구분 | String | **Y** | 3 | **KRX** (유가증권), **NXT** (코스닥), **SOR** (상장폐지예정) |
| **stk_cd** | 종목코드 | String | **Y** | 12 | 예) 005930 (삼성전자) |
| **ord_qty** | 주문수량 | String | **Y** | 12 | 정수값 (예: 10) |
| **ord_uv** | 주문단가 | String | N | 12 | 지정가 주문시만 필수 |
| **trde_tp** | 매매구분 | String | **Y** | 2 | 아래 "매매구분 코드" 참조 |
| **cond_uv** | 조건단가 | String | N | 12 | 조건부지정가(5)일 경우만 필수 |

### 매매구분 (trde_tp) 코드

| Code | 설명 | 필수 Parameters |
|------|------|-----------------|
| **0** | 보통주문 | ord_qty, stk_cd |
| **3** | 시장가 | ord_qty, stk_cd |
| **5** | 조건부지정가 | ord_qty, ord_uv, cond_uv, stk_cd |
| **6** | 최유리지정가 | ord_qty, ord_uv, stk_cd |
| **7** | 최우선지정가 | ord_qty, ord_uv, stk_cd |
| **10** | 보통(IOC) | ord_qty, ord_uv, stk_cd |
| **13** | 시장가(IOC) | ord_qty, stk_cd |
| **16** | 최유리(IOC) | ord_qty, ord_uv, stk_cd |
| **20** | 보통(FOK) | ord_qty, ord_uv, stk_cd |
| **23** | 시장가(FOK) | ord_qty, stk_cd |
| **26** | 최유리(FOK) | ord_qty, ord_uv, stk_cd |
| **28** | 스톱지정가 | ord_qty, ord_uv, stk_cd |
| **29** | 중간가 | ord_qty, stk_cd |
| **30** | 중간가(IOC) | ord_qty, stk_cd |
| **31** | 중간가(FOK) | ord_qty, stk_cd |
| **61** | 장시작전시간외 | ord_qty, ord_uv, stk_cd |
| **62** | 시간외단일가 | ord_qty, ord_uv, stk_cd |
| **81** | 장마감후시간외 | ord_qty, ord_uv, stk_cd |

### 거래소 구분 (dmst_stex_tp)

| Code | 설명 | 시간 |
|------|------|------|
| **KRX** | 유가증권(코스피) | 09:00 ~ 15:30 |
| **NXT** | 코스닥 | 09:00 ~ 15:30 |
| **SOR** | 상장폐지예정 | - |

---

## 4. 요청 예제

### 4.1 지정가 매수 (삼성전자, 10주, ₩70,000)

```json
POST /api/dostk/ordr HTTP/1.1
Host: api.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10001

{
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "10",
  "ord_uv": "70000",
  "trde_tp": "0"
}
```

### 4.2 시장가 매도 (SK하이닉스, 5주)

```json
POST /api/dostk/ordr HTTP/1.1
Host: api.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10001

{
  "dmst_stex_tp": "KRX",
  "stk_cd": "000660",
  "ord_qty": "5",
  "trde_tp": "3"
}
```

### 4.3 조건부지정가 (NAVER, 3주, ₩250,000, 조건: ₩252,000 이상)

```json
POST /api/dostk/ordr HTTP/1.1
Host: api.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10001

{
  "dmst_stex_tp": "KRX",
  "stk_cd": "035420",
  "ord_qty": "3",
  "ord_uv": "250000",
  "cond_uv": "252000",
  "trde_tp": "5"
}
```

### 4.4 최우선지정가 (코스닥, 20주, ₩80,000)

```json
POST /api/dostk/ordr HTTP/1.1
Host: mockapi.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10001

{
  "dmst_stex_tp": "NXT",
  "stk_cd": "348210",
  "ord_qty": "20",
  "ord_uv": "80000",
  "trde_tp": "7"
}
```

---

## 5. 응답 (Response)

### Response Header

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **cont-yn** | 연속조회여부 | String | N | 1 | 다음 데이터 있을시 Y값 전달 |
| **next-key** | 연속조회키 | String | N | 50 | 다음 데이터 있을시 다음 키값 전달 |
| **api-id** | TR명 | String | **Y** | 10 | kt10001 |

### Response Body

#### 성공 응답

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **ord_no** | 주문번호 | String | **Y** | 7 | 주문 고유 번호 (예: 1234567) |
| **dmst_stex_tp** | 국내거래소구분 | String | N | 6 | KRX, NXT, SOR |
| **ord_dt** | 주문일자 | String | N | 8 | YYYYMMDD 형식 |
| **ord_tm** | 주문시간 | String | N | 6 | HHMMSS 형식 |
| **stk_cd** | 종목코드 | String | N | 12 | 주문한 종목코드 |
| **ord_qty** | 주문수량 | String | N | 12 | 실행된 주문수량 |
| **trad_tp_cd** | 매매구분코드 | String | N | 2 | 주문시 입력한 매매구분 |

### HTTP Status Codes

| Status | 설명 | 처리 |
|--------|------|------|
| **200** | 주문 성공 | ord_no 확인 후 저장 |
| **400** | 잘못된 요청 | Body 파라미터 재검증 |
| **401** | 인증 실패 | 토큰 갱신 후 재시도 |
| **403** | 권한 없음 | 계좌 권한 확인 |
| **429** | Rate Limit 초과 | 대기 후 재시도 |
| **500** | 서버 오류 | 서버 상태 확인 후 재시도 |

### 응답 예제

#### 성공

```json
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "ord_dt": "20260223",
  "ord_tm": "143050",
  "stk_cd": "005930",
  "ord_qty": "10",
  "trad_tp_cd": "0"
}
```

#### 오류

```json
{
  "rt_cd": "1",
  "msg_cd": "EGW0001",
  "msg1": "잘못된 요청입니다.",
  "msg2": "필수 파라미터 dmst_stex_tp 누락"
}
```

---

## 6. 에러 코드 및 처리

### 주요 에러 코드

| msg_cd | 설명 | 원인 | 대응 |
|--------|------|------|------|
| **EGW0001** | 잘못된 요청 | Body 파라미터 오류 | 파라미터 재검증 |
| **EGW0002** | 인증 실패 | 토큰 만료/invalid | 토큰 갱신 |
| **EGW0003** | 권한 없음 | 계좌 권한 부족 | 관리자 문의 |
| **EGW0004** | 중복 주문 | 동일 주문 재시도 | 몇 초 대기 후 재시도 |
| **EGW0005** | 주문가격 오류 | 상한가/하한가 초과 | 가격 범위 재확인 |
| **EGW0006** | 매매가능시간 아님 | 장 종료 후 주문 | 장 시간 내 주문 |
| **EGW0007** | 주문수량 오류 | 최소/최대 수량 초과 | 수량 범위 확인 |
| **EGW9999** | 시스템 오류 | 서버 점검/오류 | 잠시 후 재시도 |

---

## 7. Python 구현 예제

### 7.1 기본 주문 함수

```python
import requests
import json
from datetime import datetime

class KiwoomTradingAPI:
    def __init__(self, access_token, is_mock=False):
        self.access_token = access_token
        self.base_url = "https://mockapi.kiwoom.com" if is_mock else "https://api.kiwoom.com"
        self.headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json;charset=UTF-8",
            "api-id": "kt10001"
        }
    
    def order_stock(self, dmst_stex_tp, stk_cd, ord_qty, trde_tp, 
                   ord_uv=None, cond_uv=None):
        """
        국내주식 주문 API
        
        Args:
            dmst_stex_tp: 거래소 구분 (KRX, NXT, SOR)
            stk_cd: 종목코드 (6자리)
            ord_qty: 주문수량
            trde_tp: 매매구분 코드
            ord_uv: 주문단가 (지정가 주문시)
            cond_uv: 조건단가 (조건부지정가)
        
        Returns:
            dict: {ord_no, dmst_stex_tp, ord_dt, ord_tm, stk_cd, ord_qty, ...}
        """
        payload = {
            "dmst_stex_tp": dmst_stex_tp,
            "stk_cd": stk_cd,
            "ord_qty": str(ord_qty),
            "trde_tp": str(trde_tp)
        }
        
        # 지정가 주문시 주문단가 필수
        if ord_uv is not None:
            payload["ord_uv"] = str(ord_uv)
        
        # 조건부지정가시 조건단가 필수
        if cond_uv is not None:
            payload["cond_uv"] = str(cond_uv)
        
        try:
            response = requests.post(
                f"{self.base_url}/api/dostk/ordr",
                headers=self.headers,
                json=payload,
                timeout=10
            )
            
            # 응답 처리
            if response.status_code == 200:
                result = response.json()
                print(f"✅ 주문 성공: {result.get('ord_no')}")
                return result
            else:
                error = response.json()
                print(f"❌ 주문 실패: {error.get('msg1')} ({error.get('msg_cd')})")
                return None
                
        except requests.exceptions.RequestException as e:
            print(f"❌ 네트워크 오류: {e}")
            return None
    
    def buy_limit(self, stk_cd, ord_qty, ord_uv, exchange="KRX"):
        """지정가 매수"""
        return self.order_stock(exchange, stk_cd, ord_qty, 0, ord_uv)
    
    def sell_limit(self, stk_cd, ord_qty, ord_uv, exchange="KRX"):
        """지정가 매도"""
        return self.order_stock(exchange, stk_cd, ord_qty, 1, ord_uv)
    
    def buy_market(self, stk_cd, ord_qty, exchange="KRX"):
        """시장가 매수"""
        return self.order_stock(exchange, stk_cd, ord_qty, 3)
    
    def sell_market(self, stk_cd, ord_qty, exchange="KRX"):
        """시장가 매도"""
        return self.order_stock(exchange, stk_cd, ord_qty, 13)
```

### 7.2 사용 예제

```python
# API 초기화 (모의투자)
api = KiwoomTradingAPI(access_token="YOUR_TOKEN", is_mock=True)

# 삼성전자(005930) 10주 70,000원에 매수
result = api.buy_limit("005930", 10, 70000, "KRX")
if result:
    order_no = result['ord_no']
    print(f"주문번호: {order_no}")

# SK하이닉스(000660) 5주 시장가 매도
result = api.sell_market("000660", 5, "KRX")

# 코스닥 종목 (NXT) 20주 매수
result = api.buy_limit("348210", 20, 80000, "NXT")
```

---

## 8. 매매 전략별 활용

### 8.1 데이 트레이딩 (당일 청산)

```python
def day_trading_order(api, stk_cd, qty, price, side="BUY"):
    """
    당일 청산 매매
    - 09:00 ~ 15:20 사이에만 가능
    - 시장가 사용 권장 (체결 확실성)
    """
    if side.upper() == "BUY":
        return api.buy_market(stk_cd, qty)
    else:
        return api.sell_market(stk_cd, qty)
```

### 8.2 스윙 트레이딩 (5일 보유)

```python
def swing_trading_order(api, stk_cd, qty, price):
    """
    스윙 매매 (5일 보유)
    - 지정가 사용으로 정확한 진입 가격 관리
    - 손절/익절 설정
    """
    # 진입
    entry = api.buy_limit(stk_cd, qty, price)
    
    # 진입 후 손절/익절은 별도 API 호출 필요
    # (주문 취소 후 재주문)
    
    return entry
```

### 8.3 시간외 거래

```python
def after_hours_order(api, stk_cd, qty, price, timing="CLOSE"):
    """
    시간외 거래
    - timing: "PRE" (장시작전), "CLOSE" (장마감후), "SINGLE" (시간외단일가)
    """
    timing_map = {
        "PRE": "61",      # 장시작전시간외
        "CLOSE": "81",    # 장마감후시간외
        "SINGLE": "62"    # 시간외단일가
    }
    
    return api.order_stock("KRX", stk_cd, qty, timing_map[timing], price)
```

---

## 9. 실전 거래 체크리스트

### 주문 전

- [ ] 접근토큰(Authorization) 유효성 확인
- [ ] 거래소 구분(dmst_stex_tp) 올바른지 확인
- [ ] 종목코드 정확성 검증 (6자리)
- [ ] 주문수량 범위 확인 (최소 1주, 최대 999,999,999)
- [ ] 주문가격 상한가/하한가 범위 확인
- [ ] 장 시간 확인 (09:00 ~ 15:30)
- [ ] 매수/매도 방향 재확인 ⚠️

### 주문 후

- [ ] ord_no 수신 확인
- [ ] 주문번호 DB 저장
- [ ] 체결 상태 모니터링
- [ ] 에러 발생시 에러코드 기록
- [ ] 주문 취소/정정 필요시 즉시 처리

---

## 10. Rate Limit & 제한사항

### API Rate Limit

- **초당 요청 수:** 최대 10 req/sec
- **분당 요청 수:** 최대 600 req/min
- **초과시:** HTTP 429 (Too Many Requests)

### 거래량 제한

| 항목 | 제한 |
|------|------|
| **최소 주문수량** | 1주 |
| **최대 주문수량** | 999,999,999주 |
| **최소 주문가격** | 1원 |
| **소수점 자릿수** | 정수만 가능 |

---

## 11. 참고 자료

- **공식 API 가이드:** https://openapi.kiwoom.com/m/guide/apiguide
- **OAuth 토큰 발급:** https://openapi.kiwoom.com/auth/authorize
- **상태 확인 API:** https://openapi.kiwoom.com/api/dostk/inquire
- **모의투자:** https://mockapi.kiwoom.com (테스트용)

---

## 12. 변경 이력

| 날짜 | 내용 | 비고 |
|------|------|------|
| 2026-02-23 | 초판 작성 | kt10001 기준 |
| - | - | - |
