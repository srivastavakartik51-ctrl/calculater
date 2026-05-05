<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Calculator</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      background: #f0f0f0;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    }

    .calc {
      background: #ffffff;
      border: 1px solid #ddd;
      border-radius: 16px;
      padding: 1.5rem;
      width: 320px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.10);
    }

    .display {
      background: #f5f5f5;
      border-radius: 12px;
      padding: 1rem 1.25rem;
      margin-bottom: 1rem;
      text-align: right;
      min-height: 80px;
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
      gap: 4px;
    }

    .expr {
      font-size: 13px;
      color: #999;
      min-height: 18px;
      word-break: break-all;
    }

    .main-val {
      font-size: 36px;
      font-weight: 500;
      color: #111;
      word-break: break-all;
      line-height: 1.1;
    }

    .btn-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 8px;
    }

    button.btn {
      border: none;
      border-radius: 8px;
      font-size: 17px;
      font-weight: 400;
      height: 60px;
      cursor: pointer;
      transition: transform 0.08s, opacity 0.1s;
      color: #111;
      background: #f0f0f0;
    }

    button.btn:active { transform: scale(0.94); opacity: 0.8; }
    button.btn.op     { background: #ddeeff; color: #0a4080; font-weight: 500; }
    button.btn.eq     { background: #185FA5; color: #fff; font-size: 22px; font-weight: 500; }
    button.btn.clear  { background: #fde8e8; color: #a32d2d; font-weight: 500; }
    button.btn.zero   { grid-column: span 2; }
    button.btn.func   { background: #f0f0f0; color: #555; font-size: 14px; }
    .error            { color: #a32d2d; font-size: 20px; }
  </style>
</head>
<body>

  <div class="calc">
    <div class="display">
      <div class="expr" id="expr"></div>
      <div class="main-val" id="display">0</div>
    </div>
    <div class="btn-grid">
      <button class="btn clear" onclick="clearAll()">AC</button>
      <button class="btn func"  onclick="toggleSign()">+/-</button>
      <button class="btn func"  onclick="percent()">%</button>
      <button class="btn op"    onclick="setOp('/')">÷</button>

      <button class="btn" onclick="input('7')">7</button>
      <button class="btn" onclick="input('8')">8</button>
      <button class="btn" onclick="input('9')">9</button>
      <button class="btn op" onclick="setOp('*')">×</button>

      <button class="btn" onclick="input('4')">4</button>
      <button class="btn" onclick="input('5')">5</button>
      <button class="btn" onclick="input('6')">6</button>
      <button class="btn op" onclick="setOp('-')">−</button>

      <button class="btn" onclick="input('1')">1</button>
      <button class="btn" onclick="input('2')">2</button>
      <button class="btn" onclick="input('3')">3</button>
      <button class="btn op" onclick="setOp('+')">+</button>

      <button class="btn zero" onclick="input('0')">0</button>
      <button class="btn"      onclick="dot()">.</button>
      <button class="btn eq"   onclick="calculate()">=</button>
    </div>
  </div>

  <script>
    let cur = '0', prev = null, op = null, fresh = false, justCalc = false;

    function show() {
      const d = document.getElementById('display');
      const e = document.getElementById('expr');
      let val = cur;
      if (val.length > 12) val = parseFloat(val).toExponential(5);
      d.textContent = val;
      d.className = 'main-val';
      const sym = { '/':'÷', '*':'×', '-':'−', '+':'+' };
      if (op && prev !== null) e.textContent = formatNum(prev) + ' ' + (sym[op] || op);
      else e.textContent = '';
    }

    function formatNum(n) {
      let s = String(n);
      if (s.length > 12) s = parseFloat(s).toExponential(4);
      return s;
    }

    function input(d) {
      if (fresh || justCalc) { cur = d; fresh = false; justCalc = false; }
      else cur = cur === '0' ? d : (cur.length < 15 ? cur + d : cur);
      show();
    }

    function dot() {
      if (fresh || justCalc) { cur = '0.'; fresh = false; justCalc = false; show(); return; }
      if (!cur.includes('.')) cur += '.';
      show();
    }

    function setOp(o) {
      if (op && !fresh && !justCalc) { calculate(true); }
      prev = parseFloat(cur);
      op = o; fresh = true; justCalc = false;
      show();
    }

    function calculate(chain) {
      if (op === null || prev === null) return;
      const a = prev, b = parseFloat(cur);
      let res;
      if (op === '/') { res = b === 0 ? null : a / b; }
      else if (op === '*') res = a * b;
      else if (op === '-') res = a - b;
      else if (op === '+') res = a + b;

      const d = document.getElementById('display');
      const e = document.getElementById('expr');
      const sym = { '/':'÷', '*':'×', '-':'−', '+':'+' };
      if (!chain) e.textContent = formatNum(a) + ' ' + (sym[op]||op) + ' ' + formatNum(b) + ' =';

      if (res === null) {
        d.textContent = 'Error'; d.className = 'main-val error';
        cur = '0'; op = null; prev = null; fresh = false; justCalc = true; return;
      }
      const r = parseFloat(res.toPrecision(12));
      cur = String(r);
      if (!chain) { op = null; prev = null; }
      justCalc = true; fresh = false;
      show();
    }

    function clearAll() { cur = '0'; prev = null; op = null; fresh = false; justCalc = false; show(); }

    function toggleSign() {
      if (cur !== '0') cur = cur.startsWith('-') ? cur.slice(1) : '-' + cur;
      show();
    }

    function percent() {
      let v = parseFloat(cur);
      if (op && prev !== null) v = prev * v / 100;
      else v = v / 100;
      cur = String(parseFloat(v.toPrecision(12)));
      show();
    }

    document.addEventListener('keydown', e => {
      if ('0123456789'.includes(e.key))             input(e.key);
      else if (e.key === '.')                        dot();
      else if (['+','-','*','/'].includes(e.key))   setOp(e.key);
      else if (e.key === 'Enter' || e.key === '=')  calculate();
      else if (e.key === 'Escape')                  clearAll();
      else if (e.key === 'Backspace') {
        if (!justCalc) { cur = cur.length > 1 ? cur.slice(0, -1) : '0'; show(); }
      }
      else if (e.key === '%') percent();
    });

    show();
  </script>

</body>
</html>
