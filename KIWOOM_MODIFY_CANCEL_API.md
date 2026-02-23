# 키움증권 REST API - 주식 정정/취소 주문 (kt10002, kt10003)

**최종 업데이트:** 2026-02-23

---

## 1. API 개요

### 1.1 정정 주문 (kt10002)

| 항목 | 값 |
|------|-----|
| **API명** | 주식 정정주문 |
| **TR명** | kt10002 |
| **API ID** | kt10002 |
| **Method** | POST |
| **운영 도메인** | https://api.kiwoom.com |
| **모의투자 도메인** | https://mockapi.kiwoom.com (KRX만 지원) |
| **URL** | /api/dostk/ordr |
| **Format** | JSON |
| **Content-Type** | application/json;charset=UTF-8 |

### 1.2 취소 주문 (kt10003)

| 항목 | 값 |
|------|-----|
| **API명** | 주식 취소주문 |
| **TR명** | kt10003 |
| **API ID** | kt10003 |
| **Method** | POST |
| **운영 도메인** | https://api.kiwoom.com |
| **모의투자 도메인** | https://mockapi.kiwoom.com (KRX만 지원) |
| **URL** | /api/dostk/ordr |
| **Format** | JSON |
| **Content-Type** | application/json;charset=UTF-8 |

**공통점:** 
- 동일한 엔드포인트 사용
- 동일한 Method (POST)
- API ID로만 구분

---

## 2. 정정 주문 (kt10002)

### 2.1 정정의 의미

**정정 = 기존 주문을 취소하고 새로운 조건으로 재주문**

예시:
```
기존: 삼성전자 10주 70,000원 → 아직 미체결
정정: 삼성전자 10주 71,000원으로 변경
```

### 2.2 요청 헤더

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **authorization** | 접근토큰 | String | **Y** | 1000 | Bearer {TOKEN} 형식 |
| **api-id** | TR명 | String | **Y** | 10 | **kt10002** (고정값) |
| **cont-yn** | 연속조회여부 | String | N | 1 | 응답의 cont-yn값 세팅 |
| **next-key** | 연속조회키 | String | N | 50 | 응답의 next-key값 세팅 |

### 2.3 요청 Body

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **ord_no** | 기존주문번호 | String | **Y** | 7 | 정정할 원래 주문번호 |
| **dmst_stex_tp** | 국내거래소구분 | String | **Y** | 3 | KRX, NXT, SOR |
| **stk_cd** | 종목코드 | String | **Y** | 12 | 종목코드 (변경 불가) |
| **ord_qty** | 주문수량 | String | **Y** | 12 | **새로운 수량** (예: 5) |
| **ord_uv** | 주문단가 | String | N | 12 | **새로운 가격** (지정가) |
| **trde_tp** | 매매구분 | String | **Y** | 2 | 0, 3, 5, 6, 7 등 |
| **cond_uv** | 조건단가 | String | N | 12 | 조건부지정가시 필수 |

### 2.4 정정 요청 예제

#### 예제 1: 가격 인상 (70,000 → 71,000)

```json
POST /api/dostk/ordr HTTP/1.1
Host: api.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10002

{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "10",
  "ord_uv": "71000",
  "trde_tp": "0"
}
```

#### 예제 2: 수량 감소 (10주 → 5주)

```json
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "5",
  "ord_uv": "70000",
  "trde_tp": "0"
}
```

#### 예제 3: 시장가로 전환 (지정가 → 시장가)

```json
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930",
  "ord_qty": "10",
  "trde_tp": "3"
}
```

### 2.5 정정 응답

#### 성공

```json
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "ord_dt": "20260223",
  "ord_tm": "145530",
  "stk_cd": "005930",
  "ord_qty": "10",
  "trad_tp_cd": "0",
  "ord_uv": "71000"
}
```

#### 실패

```json
{
  "rt_cd": "1",
  "msg_cd": "EGW0008",
  "msg1": "이미 체결된 주문입니다.",
  "msg2": "부분 체결 건은 정정 불가"
}
```

---

## 3. 취소 주문 (kt10003)

### 3.1 취소의 의미

**취소 = 미체결 주문을 완전히 없암**

예시:
```
기존: 삼성전자 10주 70,000원 → 아직 미체결
취소: 이 주문을 완전히 취소
```

### 3.2 요청 헤더

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **authorization** | 접근토큰 | String | **Y** | 1000 | Bearer {TOKEN} 형식 |
| **api-id** | TR명 | String | **Y** | 10 | **kt10003** (고정값) |
| **cont-yn** | 연속조회여부 | String | N | 1 | 응답의 cont-yn값 세팅 |
| **next-key** | 연속조회키 | String | N | 50 | 응답의 next-key값 세팅 |

### 3.3 요청 Body

| Element | 한글명 | Type | Required | Length | Description |
|---------|--------|------|----------|--------|-------------|
| **ord_no** | 주문번호 | String | **Y** | 7 | 취소할 주문번호 |
| **dmst_stex_tp** | 국내거래소구분 | String | **Y** | 3 | KRX, NXT, SOR |
| **stk_cd** | 종목코드 | String | **Y** | 12 | 종목코드 |

**주의: 취소는 최소한의 파라미터만 필요**

### 3.4 취소 요청 예제

#### 기본 취소

```json
POST /api/dostk/ordr HTTP/1.1
Host: api.kiwoom.com
Content-Type: application/json;charset=UTF-8
Authorization: Bearer Egicyx1a2b3c4d5e6f...
api-id: kt10003

{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "stk_cd": "005930"
}
```

### 3.5 취소 응답

#### 성공

```json
{
  "ord_no": "1234567",
  "dmst_stex_tp": "KRX",
  "ord_dt": "20260223",
  "ord_tm": "145630",
  "stk_cd": "005930"
}
```

#### 실패 - 이미 체결

```json
{
  "rt_cd": "1",
  "msg_cd": "EGW0008",
  "msg1": "이미 체결된 주문입니다.",
  "msg2": "부분 체결 건은 취소 불가"
}
```

#### 실패 - 주문번호 오류

```json
{
  "rt_cd": "1",
  "msg_cd": "EGW0009",
  "msg1": "존재하지 않는 주문입니다.",
  "msg2": "ord_no 재확인"
}
```

---

## 4. 에러 코드

### 정정/취소 공통 에러

| msg_cd | 설명 | 원인 | 대응 |
|--------|------|------|------|
| **EGW0001** | 잘못된 요청 | Body 파라미터 오류 | 파라미터 재검증 |
| **EGW0002** | 인증 실패 | 토큰 만료 | 토큰 갱신 |
| **EGW0008** | 이미 체결됨 | 주문이 체결됨 | 체결된 주문은 취소 불가 |
| **EGW0009** | 존재하지 않는 주문 | ord_no 오류 | ord_no 확인 |
| **EGW0010** | 장시간 이후 | 장 종료 후 취소 시도 | 다음 장 오픈시 처리 |
| **EGW0011** | 부분 체결 | 일부 체결된 주문 | 부분 취소 후 정정 필요 |
| **EGW9999** | 시스템 오류 | 서버 오류 | 재시도 |

---

## 5. Python 구현

### 5.1 클래스 기본 구조

```python
class KiwoomModifyCancel:
    def __init__(self, access_token, is_mock=False):
        self.access_token = access_token
        self.base_url = "https://mockapi.kiwoom.com" if is_mock else "https://api.kiwoom.com"
        self.headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json;charset=UTF-8"
        }
    
    def modify_order(self, ord_no, dmst_stex_tp, stk_cd, ord_qty, 
                    ord_uv=None, trde_tp="0", cond_uv=None):
        """
        주문 정정 (kt10002)
        
        Args:
            ord_no: 기존 주문번호
            dmst_stex_tp: KRX, NXT, SOR
            stk_cd: 종목코드
            ord_qty: 새로운 수량
            ord_uv: 새로운 가격 (지정가)
            trde_tp: 매매구분 (기본값 0: 보통)
            cond_uv: 조건단가
        """
        payload = {
            "ord_no": str(ord_no),
            "dmst_stex_tp": dmst_stex_tp,
            "stk_cd": stk_cd,
            "ord_qty": str(ord_qty),
            "trde_tp": str(trde_tp)
        }
        
        if ord_uv is not None:
            payload["ord_uv"] = str(ord_uv)
        if cond_uv is not None:
            payload["cond_uv"] = str(cond_uv)
        
        headers = {**self.headers, "api-id": "kt10002"}
        
        try:
            response = requests.post(
                f"{self.base_url}/api/dostk/ordr",
                headers=headers,
                json=payload,
                timeout=10
            )
            
            if response.status_code == 200:
                result = response.json()
                print(f"✅ 정정 성공: {result.get('ord_no')}")
                return result
            else:
                error = response.json()
                print(f"❌ 정정 실패: {error.get('msg1')}")
                return None
        except Exception as e:
            print(f"❌ 네트워크 오류: {e}")
            return None
    
    def cancel_order(self, ord_no, dmst_stex_tp, stk_cd):
        """
        주문 취소 (kt10003)
        
        Args:
            ord_no: 취소할 주문번호
            dmst_stex_tp: KRX, NXT, SOR
            stk_cd: 종목코드
        """
        payload = {
            "ord_no": str(ord_no),
            "dmst_stex_tp": dmst_stex_tp,
            "stk_cd": stk_cd
        }
        
        headers = {**self.headers, "api-id": "kt10003"}
        
        try:
            response = requests.post(
                f"{self.base_url}/api/dostk/ordr",
                headers=headers,
                json=payload,
                timeout=10
            )
            
            if response.status_code == 200:
                result = response.json()
                print(f"✅ 취소 성공: {result.get('ord_no')}")
                return result
            else:
                error = response.json()
                print(f"❌ 취소 실패: {error.get('msg1')}")
                return None
        except Exception as e:
            print(f"❌ 네트워크 오류: {e}")
            return None
```

### 5.2 사용 예제

```python
api = KiwoomModifyCancel(access_token="YOUR_TOKEN", is_mock=True)

# 정정: 70,000 → 71,000으로 가격 올리기
result = api.modify_order(
    ord_no="1234567",
    dmst_stex_tp="KRX",
    stk_cd="005930",
    ord_qty="10",
    ord_uv="71000"
)

# 정정: 10주 → 5주로 수량 줄이기
result = api.modify_order(
    ord_no="1234567",
    dmst_stex_tp="KRX",
    stk_cd="005930",
    ord_qty="5",
    ord_uv="70000"
)

# 취소: 완전 취소
result = api.cancel_order(
    ord_no="1234567",
    dmst_stex_tp="KRX",
    stk_cd="005930"
)
```

---

## 6. 실전 패턴

### 6.1 손절 설정 (지정가 → 시장가)

```python
def emergency_close(api, ord_no, dmst_stex_tp, stk_cd):
    """
    비상 익절/손절 (시장가로 즉시 매도)
    """
    # 기존 지정가 주문을 시장가로 정정
    return api.modify_order(
        ord_no=ord_no,
        dmst_stex_tp=dmst_stex_tp,
        stk_cd=stk_cd,
        ord_qty="10",  # 수량 유지
        trde_tp="3"    # 시장가
    )
```

### 6.2 추가 매수 (기존 주문 취소 후 증량)

```python
def add_to_position(api, ord_no, dmst_stex_tp, stk_cd):
    """
    포지션 증가 (기존 10주 → 새로 20주 주문)
    """
    # 1. 기존 주문 취소
    api.cancel_order(ord_no, dmst_stex_tp, stk_cd)
    
    # 2. 새로운 주문 (20주)
    from kiwoom_api import KiwoomTradingAPI
    order_api = KiwoomTradingAPI(access_token=api.access_token)
    return order_api.buy_limit(stk_cd, 20, 70000, dmst_stex_tp)
```

### 6.3 주문 수정 체인 (가격 조정)

```python
def adjust_price_chain(api, ord_no, dmst_stex_tp, stk_cd, price_adjustments):
    """
    시간대별 가격 조정 (스윙 매매용)
    
    price_adjustments = [
        (70000, "10:30"),  # 10:30에 70,000으로 설정
        (71000, "12:00"),  # 12:00에 71,000으로 올림
        (72000, "14:00")   # 14:00에 72,000으로 올림
    ]
    """
    results = []
    for price, time_str in price_adjustments:
        # 시간 확인 로직 생략
        result = api.modify_order(
            ord_no=ord_no,
            dmst_stex_tp=dmst_stex_tp,
            stk_cd=stk_cd,
            ord_qty="10",
            ord_uv=price
        )
        results.append((time_str, result))
        time.sleep(1)  # Rate limit 대비
    
    return results
```

---

## 7. 체크리스트

### 정정 전

- [ ] ord_no가 정확한가?
- [ ] 주문이 미체결 상태인가?
- [ ] 새 수량/가격이 유효한가?
- [ ] 장 시간인가?

### 취소 전

- [ ] ord_no가 정확한가?
- [ ] 주문이 완전 미체결인가? (부분 체결 주의)
- [ ] 장 시간인가?

### 성공 후

- [ ] ord_no 수신 확인
- [ ] DB에 기록
- [ ] 포지션 업데이트

---

## 8. 주의사항

| 상황 | 가능 여부 | 설명 |
|------|---------|------|
| **완전 미체결 정정** | ✅ 가능 | 전체 취소 후 새 조건으로 재주문 |
| **부분 체결 정정** | ❌ 불가능 | 먼저 취소 후 남은 수량만 새로 주문 |
| **체결 후 취소** | ❌ 불가능 | 매도로만 청산 가능 |
| **장 종료 후 정정** | ❌ 불가능 | 다음 장에만 가능 |
| **시간외 정정** | ✅ 가능 | 시간외 매매 시간대에만 가능 |

---

## 9. 참고 문서

- KIWOOM_DOMESTIC_ORDER_API.md (kt10001)
- KIWOOM_INQUIRY_API.md (체결/미체결 조회)
- KIWOOM_AUTHENTICATION.md (토큰)

---

## 10. 변경 이력

| 날짜 | 내용 |
|------|------|
| 2026-02-23 | kt10002, kt10003 문서 작성 |
