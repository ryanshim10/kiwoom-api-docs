# Autostock2026 봇 - 보완 계획

**작성일**: 2026-02-24 00:04 KST  
**목표**: TOP 10 선택 → 자동 매매까지 완전 자동화

---

## 📋 현재 상태

### ✅ 완성된 부분
- TOP 10 선택 (거래량 +30%, MA20 상향)
- Telegram 봇 연동
- 실시간 데이터 수집

### ❌ 보완 필요 부분
1. **신호 정확도** (떨사오팔 MA5 기준 추가)
2. **자동 매수** (선택된 종목 자동 주문)
3. **포지션 관리** (익절/손절 자동화)
4. **실시간 모니터링** (지속적 체크)
5. **성과 추적** (거래 기록 + 리포트)

---

## 🔧 보완 1: 신호 정확도 개선

### 기존 신호 (단순)
```python
# 거래량 + MA20만 사용
if volume_increase > 30 and ma20_direction == "UP":
    signal = "STRONG_BUY"
```

### 개선된 신호 (떨사오팔 기준)
```python
def get_tteolsa_signal(symbol, current_price, ma5, close_prices):
    """
    떨사오팔 신호 판정
    
    조건:
    1. 현재가 < MA(5) × 0.95 → 진입 신호
    2. 거래량 > 5일 평균 × 1.2 → 유동성 확인
    3. 5일 모멘텀 음수 → 하락장 진입
    """
    
    ma5_entry = ma5 * 0.95
    momentum_5d = (close_prices[-1] - close_prices[-5]) / close_prices[-5]
    
    # 신호 강도 점수 (0~100)
    score = 0
    
    # 진입 가격 조건 (50점)
    if current_price < ma5_entry:
        score += 50
        entry_triggered = True
    else:
        entry_triggered = False
    
    # 거래량 조건 (30점)
    if volume_increase > 20:
        score += 30
    
    # 모멘텀 조건 (20점)
    if momentum_5d < 0:
        score += 20
    
    # 신호 판정
    if score >= 50 and entry_triggered:
        return "STRONG_BUY"
    elif score >= 40:
        return "BUY"
    else:
        return "WAIT"
    
    return {
        "signal": signal,
        "score": score,
        "ma5": ma5,
        "entry_price": ma5_entry,
        "current_price": current_price,
        "gap": current_price - ma5_entry
    }
```

### Telegram 봇 업데이트
```python
def update_telegram_message(symbol_data):
    """개선된 신호 표시"""
    
    message = "🤖 Autostock2026 - TOP 10 (떨사오팔)\n\n"
    message += "필터: 거래량+30% | MA20↑ | 떨사오팔 신호\n\n"
    
    message += "| # | 종목 | 코드 | 현재가 | MA5 진입 | 신호 | 점수 |\n"
    message += "|---|------|------|--------|---------|------|------|\n"
    
    for i, stock in enumerate(symbol_data[:10], 1):
        signal = stock['signal']
        score = stock['signal_score']
        
        # 신호 표시
        if signal == "STRONG_BUY":
            signal_icon = "🔴"
        elif signal == "BUY":
            signal_icon = "🟠"
        else:
            signal_icon = "⚪"
        
        message += f"| {i} | {stock['name']} | {stock['code']} | "
        message += f"{stock['price']:,.0f} | {stock['ma5']*0.95:,.0f} | "
        message += f"{signal_icon} {signal} | {score}/100 |\n"
    
    return message
```

---

## 🔧 보완 2: 자동 매수 로직

### 매수 엔진
```python
class AutoBuyer:
    def __init__(self, kiwoom_api, capital=10000000):
        self.api = kiwoom_api
        self.capital = capital
        self.capital_per_symbol = capital * 0.02  # 2% per trade
        self.active_positions = {}
    
    def execute_buys(self, top_symbols):
        """
        상위 5개 종목 자동 매수
        
        Args:
            top_symbols: [{name, code, price, ma5, signal, ...}, ...]
        """
        
        # STRONG_BUY 신호만 매수
        buy_candidates = [
            s for s in top_symbols[:10] 
            if s['signal'] == "STRONG_BUY"
        ]
        
        print(f"🛒 매수 후보: {len(buy_candidates)}개")
        
        for stock in buy_candidates[:5]:  # 상위 5개만
            try:
                self.buy_stock(stock)
            except Exception as e:
                logger.error(f"매수 실패 {stock['code']}: {e}")
    
    def buy_stock(self, stock):
        """개별 종목 매수"""
        
        symbol = stock['code']
        ma5_entry = stock['ma5'] * 0.95
        current_price = stock['price']
        
        # 1. 포지션 확인
        if symbol in self.active_positions:
            print(f"⚠️  {symbol}: 이미 보유 중")
            return
        
        # 2. 매수 수량 계산
        quantity = int(self.capital_per_symbol / current_price)
        
        if quantity < 1:
            print(f"❌ {symbol}: 자본금 부족 (필요: {current_price}원)")
            return
        
        # 3. 매수 주문
        order = self.api.buy_limit(
            symbol_code=symbol,
            quantity=quantity,
            price=int(ma5_entry)  # MA5 진입가로 지정가 주문
        )
        
        if not order or 'ord_no' not in order:
            raise Exception("주문 실패")
        
        # 4. 포지션 기록
        position = {
            'symbol': symbol,
            'name': stock['name'],
            'entry_price': ma5_entry,
            'quantity': quantity,
            'buy_amount': quantity * ma5_entry,
            'order_no': order['ord_no'],
            'entry_date': datetime.now(),
            'target_price': ma5_entry * 1.03,  # +3% 목표
            'stop_loss': ma5_entry * 0.99,    # -1% 손절
            'status': 'pending'
        }
        
        self.active_positions[symbol] = position
        
        # 5. Telegram 알림
        self.notify_buy(position)
        
        print(f"✅ {stock['name']} 매수 주문 완료")
        print(f"   수량: {quantity}주 @ {ma5_entry:,.0f}원 = {quantity * ma5_entry:,.0f}원")
    
    def notify_buy(self, position):
        """매수 알림"""
        message = f"""
🔴 매수 신호 발생!

종목: {position['name']} ({position['symbol']})
진입가: {position['entry_price']:,.0f}원
수량: {position['quantity']:,}주
총액: {position['buy_amount']:,.0f}원

목표가: {position['target_price']:,.0f}원 (+3%)
손절가: {position['stop_loss']:,.0f}원 (-1%)
        """.strip()
        
        telegram.send_message(message)
```

---

## 🔧 보완 3: 포지션 관리 (익절/손절)

### 실시간 모니터링
```python
class PositionManager:
    def __init__(self, kiwoom_api, telegram):
        self.api = kiwoom_api
        self.telegram = telegram
    
    def monitor_positions(self, positions, interval=60):
        """
        활성 포지션 모니터링 (1분 간격)
        """
        
        while positions:
            current_time = datetime.now()
            
            for symbol, position in list(positions.items()):
                try:
                    # 현재 가격 조회
                    current_price = self.api.get_current_price(symbol)
                    
                    if current_price is None:
                        continue
                    
                    # 손익 계산
                    profit_loss = (current_price - position['entry_price']) * position['quantity']
                    profit_pct = (current_price - position['entry_price']) / position['entry_price'] * 100
                    
                    # 1. 익절 체크 (+3%)
                    if current_price >= position['target_price']:
                        self.sell_partial(symbol, position, current_price, "익절")
                    
                    # 2. 손절 체크 (-1%)
                    elif current_price <= position['stop_loss']:
                        self.sell_all(symbol, position, current_price, "손절")
                    
                    # 3. 5일 경과 강제청산
                    elif (current_time - position['entry_date']).days >= 5:
                        self.sell_all(symbol, position, current_price, "강제청산(5일)")
                    
                    # 4. 실시간 상태 표시
                    self.update_status(symbol, position, current_price, profit_loss, profit_pct)
                
                except Exception as e:
                    logger.error(f"모니터링 오류 {symbol}: {e}")
            
            time.sleep(interval)
    
    def sell_partial(self, symbol, position, current_price, reason):
        """부분 매도 (50%)"""
        
        sell_qty = position['quantity'] // 2
        
        # 50% 매도
        order = self.api.sell_limit(
            symbol_code=symbol,
            quantity=sell_qty,
            price=int(current_price)
        )
        
        profit = sell_qty * (current_price - position['entry_price'])
        
        message = f"""
✅ 익절 {reason}

종목: {position['name']} ({symbol})
매도가: {current_price:,.0f}원
수량: {sell_qty:,}주 (50%)
수익: {profit:,.0f}원

남은 포지션: {position['quantity'] - sell_qty}주 (트레일링)
        """.strip()
        
        self.telegram.send_message(message)
        
        # 포지션 업데이트
        position['quantity'] = position['quantity'] - sell_qty
        position['partial_sell_price'] = current_price
    
    def sell_all(self, symbol, position, current_price, reason):
        """전량 매도"""
        
        order = self.api.sell_limit(
            symbol_code=symbol,
            quantity=position['quantity'],
            price=int(current_price)
        )
        
        profit = position['quantity'] * (current_price - position['entry_price'])
        profit_pct = (current_price - position['entry_price']) / position['entry_price'] * 100
        
        message = f"""
🔵 매도 완료 ({reason})

종목: {position['name']} ({symbol})
매입가: {position['entry_price']:,.0f}원
매도가: {current_price:,.0f}원
수익률: {profit_pct:+.2f}%
수익: {profit:+,.0f}원

보유기간: {(datetime.now() - position['entry_date']).days}일
        """.strip()
        
        self.telegram.send_message(message)
        
        # 포지션 삭제
        del positions[symbol]
```

---

## 🔧 보완 4: 실시간 모니터링

### 통합 모니터링 시스템
```python
class RealTimeMonitor:
    def __init__(self):
        self.buyer = AutoBuyer()
        self.manager = PositionManager()
    
    def start(self):
        """메인 루프"""
        
        print("🤖 Autostock2026 자동 매매 시작")
        
        # 1. 매수 스레드
        buy_thread = threading.Thread(
            target=self.buy_loop,
            daemon=True
        )
        buy_thread.start()
        
        # 2. 모니터링 스레드
        monitor_thread = threading.Thread(
            target=self.monitor_loop,
            daemon=True
        )
        monitor_thread.start()
        
        # 3. 리포트 스레드
        report_thread = threading.Thread(
            target=self.report_loop,
            daemon=True
        )
        report_thread.start()
        
        print("✅ 모든 스레드 시작됨")
    
    def buy_loop(self):
        """매 10분마다 TOP 10 재평가"""
        
        while True:
            try:
                # TOP 10 조회
                top_symbols = self.get_top_symbols()
                
                # 매수 신호 있는 종목 매수
                self.buyer.execute_buys(top_symbols)
                
            except Exception as e:
                logger.error(f"매수 루프 오류: {e}")
            
            time.sleep(600)  # 10분
    
    def monitor_loop(self):
        """1분마다 포지션 체크"""
        
        while True:
            try:
                self.manager.monitor_positions(
                    self.buyer.active_positions,
                    interval=60
                )
            except Exception as e:
                logger.error(f"모니터링 오류: {e}")
    
    def report_loop(self):
        """매일 09:00, 18:00 리포트"""
        
        while True:
            now = datetime.now()
            
            if now.hour == 9 and now.minute == 0:
                self.generate_report("morning")
            elif now.hour == 18 and now.minute == 0:
                self.generate_report("evening")
            
            time.sleep(60)
```

---

## 🔧 보완 5: 성과 추적

### 거래 기록 저장
```python
class TradingLogger:
    def __init__(self, db_conn):
        self.conn = db_conn
        self.cursor = db_conn.cursor()
    
    def log_buy(self, position):
        """매수 기록"""
        
        self.cursor.execute("""
            INSERT INTO trading_log 
            (symbol, symbol_name, buy_price, quantity, buy_amount, 
             target_price, stop_loss, entry_date, status)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, 'open')
        """, (
            position['symbol'],
            position['name'],
            position['entry_price'],
            position['quantity'],
            position['buy_amount'],
            position['target_price'],
            position['stop_loss'],
            position['entry_date']
        ))
        self.conn.commit()
    
    def log_sell(self, symbol, sell_price, quantity, reason):
        """매도 기록"""
        
        self.cursor.execute("""
            UPDATE trading_log
            SET sell_price = %s, sell_date = %s, 
                profit_loss = (sell_price - buy_price) * quantity,
                profit_pct = ((sell_price - buy_price) / buy_price) * 100,
                close_reason = %s, status = 'closed'
            WHERE symbol = %s AND status = 'open'
        """, (
            sell_price,
            datetime.now(),
            reason,
            symbol
        ))
        self.conn.commit()
    
    def generate_daily_report(self):
        """일일 성과 리포트"""
        
        self.cursor.execute("""
            SELECT 
                COUNT(*) as total_trades,
                SUM(CASE WHEN profit_loss > 0 THEN 1 ELSE 0 END) as wins,
                SUM(CASE WHEN profit_loss < 0 THEN 1 ELSE 0 END) as losses,
                SUM(profit_loss) as total_profit,
                AVG(profit_pct) as avg_profit_pct
            FROM trading_log
            WHERE DATE(sell_date) = CURRENT_DATE
        """)
        
        stats = self.cursor.fetchone()
        
        if stats[0] > 0:
            win_rate = (stats[1] / stats[0]) * 100
            
            report = f"""
📊 일일 성과 리포트 ({datetime.now().strftime('%Y-%m-%d')})

거래: {stats[0]}건
승리: {stats[1]}건 (승률 {win_rate:.1f}%)
패배: {stats[2]}건
수익: {stats[3]:+,.0f}원
평균: {stats[4]:+.2f}%
            """.strip()
            
            telegram.send_message(report)
```

---

## 📋 구현 순서

### Phase 1: 신호 개선 (1일)
```bash
1. 떨사오팔 신호 함수 추가
2. Telegram 메시지 업데이트
3. 테스트 실행
```

### Phase 2: 자동 매수 (1일)
```bash
1. AutoBuyer 클래스 구현
2. Kiwoom API 연동
3. 모의투자(Mock) 테스트
```

### Phase 3: 포지션 관리 (1일)
```bash
1. PositionManager 구현
2. 익절/손절 로직 추가
3. 실시간 모니터링
```

### Phase 4: 통합 및 배포 (1일)
```bash
1. 모든 모듈 통합
2. 완전 자동화 테스트
3. 실전 배포
```

---

## 🎯 최종 목표

```
구조:
  Kiwoom API
     ↓
  TOP 10 선택 (신호 강화)
     ↓
  자동 매수 (5개 종목)
     ↓
  실시간 모니터링 (익절/손절)
     ↓
  성과 추적 (DB 저장)
     ↓
  Telegram 알림 (실시간)

결과:
  ✅ 완전 자동 매매
  ✅ 신호 기반 거래
  ✅ 리스크 관리
  ✅ 성과 추적
```

---

**이 계획으로 진행할까요?** 🚀
