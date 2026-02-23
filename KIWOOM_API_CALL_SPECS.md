# 키움증권 REST API - API별 호출 스펙 완전 가이드

**최종 업데이트:** 2026-02-23

---

## 📌 API 호출 기본 정보

### 공통 설정

| 항목 | 값 |
|------|-----|
| **운영 도메인** | https://api.kiwoom.com |
| **모의투자 도메인** | https://mockapi.kiwoom.com (KRX만 지원) |
| **Format** | JSON |
| **Content-Type** | application/json;charset=UTF-8 |
| **인증** | Authorization: Bearer {ACCESS_TOKEN} |
| **Rate Limit** | 10 req/sec, 600 req/min |

---

## 1️⃣ 계좌 (Account) API 호출 스펙

### 1-1. 예수금상세현황요청 (kt00001)

```
API ID: kt00001
Method: POST
URL: /api/dostk/acnt
분류: 계좌
설명: 계좌별 예수금 및 투자현황 조회

[요청]
필수 파라미터:
- dmst_stex_tp: 국내거래소구분 (KRX, NXT, SOR)
- acnt_no: 계좌번호

선택 파라미터:
- (없음)

[응답]
- acnt_no: 계좌번호
- psbl_psbl_amt: 사용가능금액
- nAsset: 순자산
- etc_psbl_amt: 기타사용가능금액

[예제]
POST https://api.kiwoom.com/api/dostk/acnt
Authorization: Bearer {TOKEN}
Content-Type: application/json;charset=UTF-8

{
  "dmst_stex_tp": "KRX",
  "acnt_no": "50010000001"
}

응답:
{
  "acnt_no": "50010000001",
  "psbl_psbl_amt": "5000000",
  "nAsset": "15000000"
}
```

---

### 1-2. 계좌번호조회 (ka00001)

```
API ID: ka00001
Method: GET
URL: /api/account/accounts
분류: 계좌
설명: 보유 계좌 목록 조회

[요청]
필수 파라미터:
- (없음 - 토큰으로 인증)

선택 파라미터:
- (없음)

[응답]
- account: 계좌번호 배열
- accountNick: 계좌별칭
- accTpCode: 계좌유형

[예제]
GET https://api.kiwoom.com/api/account/accounts
Authorization: Bearer {TOKEN}

응답:
{
  "account": [
    {
      "acnt_no": "50010000001",
      "acnt_alias": "메인계좌",
      "acnt_tp_cd": "1"
    }
  ]
}
```

---

### 1-3. 미체결요청 (ka10075)

```
API ID: ka10075
Method: GET
URL: /api/dostk/inquire/order
분류: 계좌
설명: 미체결 주문 조회

[요청]
필수 파라미터:
- acnt_no: 계좌번호
- dmst_stex_tp: 거래소구분 (KRX, NXT)

선택 파라미터:
- ord_dt_from: 시작일 (YYYYMMDD)
- ord_dt_to: 종료일 (YYYYMMDD)
- cont_yn: 연속조회여부 (Y/N)
- next_key: 다음조회키

[응답]
- ord_no: 주문번호
- ord_dt: 주문일자
- stk_cd: 종목코드
- ord_qty: 주문수량
- ord_uv: 주문단가
- trde_tp_cd: 매매구분

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/order?acnt_no=50010000001&dmst_stex_tp=KRX
Authorization: Bearer {TOKEN}

응답:
{
  "output1": [
    {
      "ord_no": "1234567",
      "ord_dt": "20260223",
      "stk_cd": "005930",
      "ord_qty": "10",
      "ord_uv": "70000"
    }
  ]
}
```

---

### 1-4. 체결요청 (ka10076)

```
API ID: ka10076
Method: GET
URL: /api/dostk/inquire/execution
분류: 계좌
설명: 체결 내역 조회

[요청]
필수 파라미터:
- acnt_no: 계좌번호
- dmst_stex_tp: 거래소구분

선택 파라미터:
- ord_dt_from: 시작일
- ord_dt_to: 종료일
- cont_yn: 연속조회여부

[응답]
- org_ord_no: 원주문번호
- exec_no: 체결번호
- exec_qty: 체결수량
- exec_uv: 체결가
- exec_dt: 체결일시

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/execution?acnt_no=50010000001&dmst_stex_tp=KRX
Authorization: Bearer {TOKEN}

응답:
{
  "output1": [
    {
      "org_ord_no": "1234567",
      "exec_no": "100",
      "exec_qty": "10",
      "exec_uv": "70100",
      "exec_dt": "20260223143050"
    }
  ]
}
```

---

## 2️⃣ 주문 (Order) API 호출 스펙

### 2-1. 주식 매도주문 (kt10001)

```
API ID: kt10001
Method: POST
URL: /api/dostk/ordr
분류: 주문
설명: 국내주식 매도 주문

[요청]
필수 파라미터:
- dmst_stex_tp: 거래소구분 (KRX, NXT, SOR)
- stk_cd: 종목코드 (6자리)
- ord_qty: 주문수량
- trde_tp: 매매구분 (0:보통, 3:시장가, 5:조건부, 6:최유리, 7:최우선, 등)

선택 파라미터:
- ord_uv: 주문단가 (지정가 필수)
- cond_uv: 조건단가 (조건부지정가 필수)

[응답]
- ord_no: 주문번호
- dmst_stex_tp: 거래소구분
- stk_cd: 종목코드
- ord_qty: 주문수량
- ord_dt: 주문일자
- ord_tm: 주문시간

[예제]
POST https://api.kiwoom.com/api/dostk/ordr
Authorization: Bearer {TOKEN}
Content-Type: application/json;charset=UTF-8

{
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "10",
  "ord_uv": "70000",
  "trde_tp": "0"
}

응답:
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "10",
  "ord_dt": "20260223",
  "ord_tm": "143050"
}
```

---

### 2-2. 주식 정정주문 (kt10002)

```
API ID: kt10002
Method: POST
URL: /api/dostk/ordr
분류: 주문
설명: 주식 주문 정정 (수량/가격 변경)

[요청]
필수 파라미터:
- ord_no: 기존주문번호
- dmst_stex_tp: 거래소구분
- stk_cd: 종목코드
- ord_qty: 새로운 주문수량
- trde_tp: 매매구분

선택 파라미터:
- ord_uv: 새로운 주문단가

[응답]
- ord_no: 정정된 주문번호
- ord_dt: 정정일자
- ord_uv: 정정가격

[예제]
POST https://api.kiwoom.com/api/dostk/ordr
Authorization: Bearer {TOKEN}
Content-Type: application/json;charset=UTF-8

{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "5",
  "ord_uv": "71000",
  "trde_tp": "0"
}

응답:
{
  "ord_no": "1234567",
  "ord_dt": "20260223",
  "ord_uv": "71000"
}
```

---

### 2-3. 주식 취소주문 (kt10003)

```
API ID: kt10003
Method: POST
URL: /api/dostk/ordr
분류: 주문
설명: 미체결 주문 취소

[요청]
필수 파라미터:
- ord_no: 취소할 주문번호
- dmst_stex_tp: 거래소구분
- stk_cd: 종목코드

선택 파라미터:
- (없음 - 최소한의 정보만 필요)

[응답]
- ord_no: 취소된 주문번호
- ord_dt: 취소일자

[예제]
POST https://api.kiwoom.com/api/dostk/ordr
Authorization: Bearer {TOKEN}
Content-Type: application/json;charset=UTF-8

{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930"
}

응답:
{
  "ord_no": "1234567",
  "ord_dt": "20260223"
}
```

---

## 3️⃣ 시세 (Market Price) API 호출 스펙

### 3-1. 주식현재가 (ka10100)

```
API ID: ka10100
Method: GET
URL: /api/dostk/inquire/price
분류: 시세
설명: 주식 현재가 조회

[요청]
필수 파라미터:
- stk_cd: 종목코드 (6자리)
- dmst_stex_tp: 거래소구분 (기본값: KRX)

선택 파라미터:
- (없음)

[응답]
- stk_prce: 현재가
- stk_oprc: 시가
- stk_hgpr: 고가
- stk_lwpr: 저가
- acml_vol: 거래량
- acml_tr_pbmn: 거래대금

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/price?stk_cd=005930&dmst_stex_tp=KRX
Authorization: Bearer {TOKEN}

응답:
{
  "output": {
    "stk_prce": "70500",
    "stk_oprc": "70000",
    "stk_hgpr": "70900",
    "stk_lwpr": "69800",
    "acml_vol": "5000000",
    "acml_tr_pbmn": "354000000000"
  }
}
```

---

### 3-2. 호가조회 (ka10104)

```
API ID: ka10104
Method: GET
URL: /api/dostk/inquire/quotation
분류: 시세
설명: 호가(매도/매수 호가) 조회

[요청]
필수 파라미터:
- stk_cd: 종목코드

선택 파라미터:
- (없음)

[응답]
- bidp1: 매수호가 1순위
- askp1: 매도호가 1순위
- bidp2 ~ bidp10: 매수호가 2~10순위
- askp2 ~ askp10: 매도호가 2~10순위

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/quotation?stk_cd=005930
Authorization: Bearer {TOKEN}

응답:
{
  "output": {
    "bidp1": "70400",
    "askp1": "70500",
    "bidp2": "70300",
    "askp2": "70600"
  }
}
```

---

## 4️⃣ 차트 (Chart) API 호출 스펙

### 4-1. 주식일차트조회요청 (ka10070)

```
API ID: ka10070
Method: GET
URL: /api/dostk/inquire/chart
분류: 차트
설명: 일봉 차트 데이터 조회 (OHLCV)

[요청]
필수 파라미터:
- stk_cd: 종목코드
- inq_strt_dt: 조회시작일 (YYYYMMDD)
- inq_end_dt: 조회종료일 (YYYYMMDD)

선택 파라미터:
- cont_yn: 연속조회여부 (Y/N)
- next_key: 다음조회키

[응답]
- stk_cntg_date: 거래일자
- open_price: 시가
- high_price: 고가
- low_price: 저가
- close_price: 종가
- volume: 거래량

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/chart?stk_cd=005930&inq_strt_dt=20260101&inq_end_dt=20260223
Authorization: Bearer {TOKEN}

응답:
{
  "output": [
    {
      "stk_cntg_date": "20260223",
      "open_price": "70000",
      "high_price": "70900",
      "low_price": "69800",
      "close_price": "70500",
      "volume": "5000000"
    }
  ]
}
```

---

### 4-2. 주식분차트조회요청 (ka10064)

```
API ID: ka10064
Method: GET
URL: /api/dostk/inquire/chart/minute
분류: 차트
설명: 분봉 차트 데이터 조회

[요청]
필수 파라미터:
- stk_cd: 종목코드
- inq_hour: 조회시간 (HH:MM)

선택 파라미터:
- inq_min: 분봉 단위 (1, 5, 10, 15, 30, 60 - 기본값: 1)
- cont_yn: 연속조회여부

[응답]
- stk_cntg_time: 거래시간 (HHMMSS)
- open_price: 시가
- high_price: 고가
- low_price: 저가
- close_price: 종가
- volume: 거래량

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/chart/minute?stk_cd=005930&inq_hour=14:30&inq_min=5
Authorization: Bearer {TOKEN}

응답:
{
  "output": [
    {
      "stk_cntg_time": "143000",
      "open_price": "70400",
      "high_price": "70600",
      "low_price": "70300",
      "close_price": "70500",
      "volume": "50000"
    }
  ]
}
```

---

## 5️⃣ 순위정보 (Rankings) API 호출 스펙

### 5-1. 당일거래량상위요청 (ka10030)

```
API ID: ka10030
Method: GET
URL: /api/dostk/inquire/ranking/volume
분류: 순위정보
설명: 금일 거래량 상위 종목 조회

[요청]
필수 파라미터:
- dmst_stex_tp: 거래소구분 (KRX, NXT)

선택 파라미터:
- search_cnt: 조회개수 (기본값: 50)

[응답]
- rank: 순위
- stk_cd: 종목코드
- stk_nm: 종목명
- acml_vol: 누적거래량
- vol_rnk: 거래량순위

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/ranking/volume?dmst_stex_tp=KRX&search_cnt=10
Authorization: Bearer {TOKEN}

응답:
{
  "output": [
    {
      "rank": "1",
      "stk_cd": "005930",
      "stk_nm": "삼성전자",
      "acml_vol": "50000000"
    }
  ]
}
```

---

### 5-2. 전일대비등락률상위요청 (ka10027)

```
API ID: ka10027
Method: GET
URL: /api/dostk/inquire/ranking/price
분류: 순위정보
설명: 전일 대비 등락률 상위 종목 조회

[요청]
필수 파라미터:
- dmst_stex_tp: 거래소구분

선택 파라미터:
- search_cnt: 조회개수

[응답]
- rank: 순위
- stk_cd: 종목코드
- stk_nm: 종목명
- prc_chg_rt: 등락률 (%)
- prc_chg_amt: 등락액

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/ranking/price?dmst_stex_tp=KRX&search_cnt=10
Authorization: Bearer {TOKEN}

응답:
{
  "output": [
    {
      "rank": "1",
      "stk_cd": "000660",
      "stk_nm": "SK하이닉스",
      "prc_chg_rt": "5.25",
      "prc_chg_amt": "3500"
    }
  ]
}
```

---

## 6️⃣ 종목정보 (Stock Info) API 호출 스펙

### 6-1. 주식기본정보요청 (ka10001)

```
API ID: ka10001
Method: GET
URL: /api/dostk/inquire/stock/basics
분류: 종목정보
설명: 종목 기본 정보 조회

[요청]
필수 파라미터:
- stk_cd: 종목코드

선택 파라미터:
- (없음)

[응답]
- stk_nm: 종목명
- per: PER
- pbr: PBR
- roe: ROE
- market_cap: 시가총액
- outstanding_shares: 상장주식수

[예제]
GET https://api.kiwoom.com/api/dostk/inquire/stock/basics?stk_cd=005930
Authorization: Bearer {TOKEN}

응답:
{
  "output": {
    "stk_nm": "삼성전자",
    "per": "8.5",
    "pbr": "1.2",
    "roe": "14.2",
    "market_cap": "350000000000000"
  }
}
```

---

## 📊 API 호출 패턴 요약

### GET API (조회성)
```
GET https://api.kiwoom.com/api/dostk/inquire/{endpoint}?param1=value1&param2=value2
Authorization: Bearer {TOKEN}
```

### POST API (주문성)
```
POST https://api.kiwoom.com/api/dostk/ordr
Authorization: Bearer {TOKEN}
Content-Type: application/json;charset=UTF-8

{
  "param1": "value1",
  "param2": "value2"
}
```

---

## 🔑 필수 파라미터 정리

| API 분류 | 필수 파라미터 |
|---------|-------------|
| 계좌 조회 | acnt_no, dmst_stex_tp |
| 미/체결 조회 | acnt_no, dmst_stex_tp |
| 주식 주문 | dmst_stex_tp, stk_cd, ord_qty, trde_tp |
| 주식 정정 | ord_no, dmst_stex_tp, stk_cd, ord_qty |
| 주식 취소 | ord_no, dmst_stex_tp, stk_cd |
| 현재가 조회 | stk_cd |
| 호가 조회 | stk_cd |
| 일차트 조회 | stk_cd, inq_strt_dt, inq_end_dt |
| 분차트 조회 | stk_cd, inq_hour |
| 순위 조회 | dmst_stex_tp |
| 기본정보 조회 | stk_cd |

---

## 🚀 Python 구현 예제

```python
import requests
import json

class KiwoomAPI:
    def __init__(self, access_token):
        self.base_url = "https://api.kiwoom.com"
        self.headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json;charset=UTF-8"
        }
    
    # 현재가 조회
    def get_price(self, stk_cd):
        url = f"{self.base_url}/api/dostk/inquire/price"
        params = {"stk_cd": stk_cd, "dmst_stex_tp": "KRX"}
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    # 주문
    def place_order(self, dmst_stex_tp, stk_cd, ord_qty, ord_uv, trde_tp):
        url = f"{self.base_url}/api/dostk/ordr"
        payload = {
            "dmst_stex_tp": dmst_stex_tp,
            "stk_cd": stk_cd,
            "ord_qty": str(ord_qty),
            "ord_uv": str(ord_uv),
            "trde_tp": str(trde_tp)
        }
        response = requests.post(url, headers=self.headers, json=payload)
        return response.json()
    
    # 미체결 조회
    def get_pending_orders(self, acnt_no, dmst_stex_tp):
        url = f"{self.base_url}/api/dostk/inquire/order"
        params = {"acnt_no": acnt_no, "dmst_stex_tp": dmst_stex_tp}
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()

# 사용 예
api = KiwoomAPI("your_access_token")

# 현재가
price = api.get_price("005930")
print(f"삼성전자 현재가: {price['output']['stk_prce']}")

# 주문
order = api.place_order("KRX", "005930", 10, 70000, 0)
print(f"주문번호: {order['ord_no']}")

# 미체결
pending = api.get_pending_orders("50010000001", "KRX")
print(f"미체결 주문: {pending['output1']}")
```

---

## 📝 변경 이력

| 날짜 | 내용 | 비고 |
|------|------|------|
| 2026-02-23 | API별 호출 스펙 완전 정리 | URL, Method, 파라미터, 응답 포함 |
