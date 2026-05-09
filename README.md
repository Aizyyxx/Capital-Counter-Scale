<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="Капитал">
<title>Капитал — Трекер</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Bebas+Neue&display=swap" rel="stylesheet">
<style>
* { box-sizing: border-box; margin: 0; padding: 0; }
body { background: #0d0f14; font-family: 'Space Mono', monospace; color: #e8eaf0; }
input::-webkit-inner-spin-button, input::-webkit-outer-spin-button { -webkit-appearance: none; }
input { -moz-appearance: textfield; }
</style>
</head>
<body>
<div id="app"></div>
<script>
const DB = {
  save(goal, current, log) {
    try {
      localStorage.setItem('ccs_v2', JSON.stringify({goal, current, log}));
      sessionStorage.setItem('ccs_v2', JSON.stringify({goal, current, log}));
    } catch(e) {}
    try {
      document.cookie = "ccs=" + encodeURIComponent(JSON.stringify({goal, current, log})) + ";max-age=31536000;path=/";
    } catch(e) {}
  },
  load() {
    try {
      const ls = localStorage.getItem('ccs_v2');
      if (ls) return JSON.parse(ls);
    } catch(e) {}
    try {
      const ss = sessionStorage.getItem('ccs_v2');
      if (ss) return JSON.parse(ss);
    } catch(e) {}
    try {
      const c = document.cookie.split(';').find(x => x.trim().startsWith('ccs='));
      if (c) return JSON.parse(decodeURIComponent(c.split('=')[1]));
    } catch(e) {}
    return {goal: 0, current: 0, log: []};
  },
  clear() {
    try { localStorage.removeItem('ccs_v2'); } catch(e) {}
    try { sessionStorage.removeItem('ccs_v2'); } catch(e) {}
    try { document.cookie = "ccs=;max-age=0;path=/"; } catch(e) {}
  }
};

let state = DB.load();

const fmt = n => "$ " + Number(n).toLocaleString("en-US", {minimumFractionDigits:0, maximumFractionDigits:2});

function render() {
  const {goal, current, log} = state;
  const pct = goal > 0 ? Math.min((current/goal)*100, 100) : 0;
  const pctD = goal > 0 ? Math.round((current/goal)*100) : 0;
  const over = current > goal && goal > 0;
  const color = over ? "#ff4d6d" : "#c8ff00";
  const barBg = over ? "linear-gradient(90deg,#c0002a,#ff4d6d)" : "linear-gradient(90deg,#8aaf00,#c8ff00)";

  document.getElementById('app').innerHTML = `
    <div style="min-height:100vh;background:#0d0f14;background-image:radial-gradient(ellipse 80% 50% at 50% -10%,rgba(200,255,0,0.06) 0%,transparent 70%);display:flex;flex-direction:column;align-items:center;padding:40px 20px 60px;">
      <div style="text-align:center;margin-bottom:48px;">
        <div style="font-family:'Bebas Neue',sans-serif;font-size:clamp(2.5rem,8vw,5rem);color:#c8ff00;letter-spacing:0.08em;line-height:1;">КАПИТАЛ</div>
        <div style="color:#5a6080;font-size:0.75rem;letter-spacing:0.2em;text-transform:uppercase;margin-top:6px;">Трекер прогресса</div>
      </div>

      <div style="background:#161922;border:1px solid #252a38;border-radius:16px;padding:32px;width:100%;max-width:520px;margin-bottom:20px;">
        <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:10px;">Прогресс к цели</div>
        <div style="font-family:'Bebas Neue',sans-serif;font-size:clamp(5rem,20vw,9rem);line-height:1;color:${color};text-align:center;">${pctD}%</div>
        <div style="height:28px;background:#252a38;border-radius:99px;overflow:hidden;margin:16px 0 8px;">
          <div style="height:100%;width:${pct}%;background:${barBg};border-radius:99px;transition:width 0.6s;"></div>
        </div>
        <div style="display:flex;justify-content:space-between;font-size:0.65rem;color:#5a6080;">
          <span>${fmt(current)}</span>
          <span>Цель: ${goal > 0 ? fmt(goal) : '— $'}</span>
        </div>
      </div>

      <div style="background:#161922;border:1px solid #252a38;border-radius:16px;padding:32px;width:100%;max-width:520px;margin-bottom:20px;">
        <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:10px;">Установить цель</div>
        <div style="display:flex;gap:12px;">
          <input id="goalInp" type="number" min="1" placeholder="Сумма цели, $" style="flex:1;background:#0d0f14;border:1px solid #252a38;border-radius:10px;color:#e8eaf0;font-family:'Space Mono',monospace;font-size:1rem;padding:12px 16px;width:100%;" />
          <button onclick="setGoal()" style="background:#c8ff00;color:#000;border:none;border-radius:10px;font-family:'Bebas Neue',sans-serif;font-size:1.3rem;padding:12px 22px;cursor:pointer;height:50px;">ЗАДАТЬ</button>
        </div>
      </div>

      <div style="background:#161922;border:1px solid #252a38;border-radius:16px;padding:32px;width:100%;max-width:520px;margin-bottom:20px;">
        <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:10px;">Добавить в капитал</div>
        <div style="display:flex;gap:12px;">
          <input id="addInp" type="number" min="0.01" step="0.01" placeholder="Сумма пополнения, $" style="flex:1;background:#0d0f14;border:1px solid #252a38;border-radius:10px;color:#e8eaf0;font-family:'Space Mono',monospace;font-size:1rem;padding:12px 16px;width:100%;" />
          <button onclick="addAmt()" style="background:#c8ff00;color:#000;border:none;border-radius:10px;font-family:'Bebas Neue',sans-serif;font-size:1.3rem;padding:12px 22px;cursor:pointer;height:50px;">+ ДОБАВИТЬ</button>
        </div>
        <div id="err" style="margin-top:10px;color:#ff4d6d;font-size:0.75rem;"></div>
      </div>

      <div style="background:#161922;border:1px solid #252a38;border-radius:16px;padding:32px;width:100%;max-width:520px;margin-bottom:20px;">
        <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:14px;">Статистика</div>
        <div style="display:grid;grid-template-columns:1fr 1fr;gap:12px;">
          ${[['Накоплено', fmt(current), '#c8ff00'], ['Цель', goal>0?fmt(goal):'— $', '#e8eaf0'], ['Осталось', goal>0?fmt(Math.max(goal-current,0)):'— $', '#ff4d6d'], ['Пополнений', log.length, '#e8eaf0']].map(([l,v,c])=>`
            <div style="background:#0d0f14;border:1px solid #252a38;border-radius:12px;padding:16px;">
              <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:4px;">${l}</div>
              <div style="font-family:'Bebas Neue',sans-serif;font-size:1.8rem;color:${c};">${v}</div>
            </div>`).join('')}
        </div>
        <button onclick="resetAll()" style="background:transparent;border:1px solid #252a38;color:#5a6080;border-radius:10px;font-family:'Bebas Neue',sans-serif;font-size:1rem;width:100%;margin-top:16px;height:46px;cursor:pointer;">↺ СБРОСИТЬ ВСЁ</button>
      </div>

      <div style="background:#161922;border:1px solid #252a38;border-radius:16px;padding:32px;width:100%;max-width:520px;margin-bottom:20px;">
        <div style="font-size:0.65rem;letter-spacing:0.25em;text-transform:uppercase;color:#5a6080;margin-bottom:14px;">История пополнений</div>
        <div style="display:flex;flex-direction:column;gap:8px;max-height:220px;overflow-y:auto;">
          ${log.length === 0
            ? '<div style="text-align:center;color:#5a6080;font-size:0.75rem;padding:24px 0;">Пополнений пока нет</div>'
            : [...log].reverse().map(e=>`
              <div style="display:flex;justify-content:space-between;align-items:center;background:#0d0f14;border:1px solid #252a38;border-radius:10px;padding:10px 14px;font-size:0.8rem;">
                <span style="color:#c8ff00;font-weight:700;">+ ${fmt(e.amount)}</span>
                <span style="color:#5a6080;font-size:0.65rem;">${e.time}</span>
              </div>`).join('')}
        </div>
      </div>
    </div>`;
}

function setGoal() {
  const v = parseFloat(document.getElementById('goalInp').value);
  if (!v || v <= 0) { document.getElementById('err').textContent = 'Введите корректную сумму цели'; return; }
  state.goal = v;
  document.getElementById('goalInp').value = '';
  document.getElementById('err').textContent = '';
  DB.save(state.goal, state.current, state.log);
  render();
}

function addAmt() {
  if (state.goal <= 0) { document.getElementById('err').textContent = 'Сначала установите цель'; return; }
  const v = parseFloat(document.getElementById('addInp').value);
  if (!v || v <= 0) { document.getElementById('err').textContent = 'Введите сумму пополнения'; return; }
  const now = new Date();
  const time = now.toLocaleString('ru-RU', {day:'2-digit',month:'2-digit',hour:'2-digit',minute:'2-digit'});
  state.current += v;
  state.log.push({amount: v, time});
  document.getElementById('addInp').value = '';
  document.getElementById('err').textContent = '';
  DB.save(state.goal, state.current, state.log);
  render();
}

function resetAll() {
  state = {goal:0, current:0, log:[]};
  DB.clear();
  render();
}

render();
</script>
</body>
</html> 
