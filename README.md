<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="Капитал">
<title>Капитал — Трекер</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Bebas+Neue&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #0d0f14; }
  input::-webkit-inner-spin-button,
  input::-webkit-outer-spin-button { -webkit-appearance: none; }
  input { -moz-appearance: textfield; }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const { useState, useEffect } = React;
const fmt = (n) => "$ " + Number(n).toLocaleString("en-US", { minimumFractionDigits: 0, maximumFractionDigits: 2 });
const card = { background: "#161922", border: "1px solid #252a38", borderRadius: 16, padding: 32, width: "100%", maxWidth: 520, marginBottom: 20 };
const labelStyle = { fontSize: "0.65rem", letterSpacing: "0.25em", textTransform: "uppercase", color: "#5a6080", marginBottom: 10 };
const inputStyle = { flex: 1, background: "#0d0f14", border: "1px solid #252a38", borderRadius: 10, color: "#e8eaf0", fontFamily: "'Space Mono', monospace", fontSize: "1rem", padding: "12px 16px", width: "100%" };
const btnStyle = { background: "#c8ff00", color: "#000", border: "none", borderRadius: 10, fontFamily: "'Bebas Neue', sans-serif", fontSize: "1.3rem", letterSpacing: "0.08em", padding: "12px 22px", cursor: "pointer", whiteSpace: "nowrap", height: 50 };

function App() {
  const [goal, setGoal] = useState(() => parseFloat(localStorage.getItem('cap_goal') || '0'));
  const [current, setCurrent] = useState(() => parseFloat(localStorage.getItem('cap_current') || '0'));
  const [log, setLog] = useState(() => JSON.parse(localStorage.getItem('cap_log') || '[]'));
  const [goalInput, setGoalInput] = useState("");
  const [addInput, setAddInput] = useState("");
  const [error, setError] = useState("");

  useEffect(() => { localStorage.setItem('cap_goal', goal); }, [goal]);
  useEffect(() => { localStorage.setItem('cap_current', current); }, [current]);
  useEffect(() => { localStorage.setItem('cap_log', JSON.stringify(log)); }, [log]);

  const pct = goal > 0 ? Math.min((current / goal) * 100, 100) : 0;
  const pctDisplay = goal > 0 ? Math.round((current / goal) * 100) : 0;
  const over = current > goal && goal > 0;

  function handleSetGoal() {
    const v = parseFloat(goalInput);
    if (!v || v <= 0) { setError("Введите корректную сумму цели"); return; }
    setGoal(v); setGoalInput(""); setError("");
  }
  function handleAdd() {
    if (goal <= 0) { setError("Сначала установите цель"); return; }
    const v = parseFloat(addInput);
    if (!v || v <= 0) { setError("Введите сумму пополнения"); return; }
    const now = new Date();
    const time = now.toLocaleString("ru-RU", { day:"2-digit", month:"2-digit", hour:"2-digit", minute:"2-digit" });
    setCurrent(c => c + v);
    setLog(l => [...l, { amount: v, time }]);
    setAddInput(""); setError("");
  }
  function handleReset() {
    setGoal(0); setCurrent(0); setLog([]);
    setGoalInput(""); setAddInput(""); setError("");
    localStorage.removeItem('cap_goal');
    localStorage.removeItem('cap_current');
    localStorage.removeItem('cap_log');
  }

  return (
    <div style={{ minHeight: "100vh", background: "#0d0f14", backgroundImage: "radial-gradient(ellipse 80% 50% at 50% -10%, rgba(200,255,0,0.06) 0%, transparent 70%)", color: "#e8eaf0", fontFamily: "'Space Mono', monospace", display: "flex", flexDirection: "column", alignItems: "center", padding: "40px 20px 60px" }}>
      <div style={{ textAlign: "center", marginBottom: 48 }}>
        <div style={{ fontFamily: "'Bebas Neue', sans-serif", fontSize: "clamp(2.5rem,8vw,5rem)", color: "#c8ff00", letterSpacing: "0.08em", lineHeight: 1 }}>КАПИТАЛ</div>
        <div style={{ color: "#5a6080", fontSize: "0.75rem", letterSpacing: "0.2em", textTransform: "uppercase", marginTop: 6 }}>Трекер прогресса</div>
      </div>
      <div style={card}>
        <div style={labelStyle}>Прогресс к цели</div>
        <div style={{ fontFamily: "'Bebas Neue', sans-serif", fontSize: "clamp(5rem,20vw,9rem)", lineHeight: 1, color: over ? "#ff4d6d" : "#c8ff00", textAlign: "center" }}>{pctDisplay}%</div>
        <div style={{ height: 28, background: "#252a38", borderRadius: 99, overflow: "hidden", margin: "16px 0 8px" }}>
          <div style={{ height: "100%", width: pct + "%", background: over ? "linear-gradient(90deg,#c0002a,#ff4d6d)" : "linear-gradient(90deg,#8aaf00,#c8ff00)", borderRadius: 99, transition: "width 0.6s" }} />
        </div>
        <div style={{ display: "flex", justifyContent: "space-between", fontSize: "0.65rem", color: "#5a6080" }}>
          <span>{fmt(current)}</span>
          <span>Цель: {goal > 0 ? fmt(goal) : "— $"}</span>
        </div>
      </div>
      <div style={card}>
        <div style={labelStyle}>Установить цель</div>
        <div style={{ display: "flex", gap: 12 }}>
          <input type="number" value={goalInput} min="1" placeholder="Сумма цели, $" onChange={e => setGoalInput(e.target.value)} onKeyDown={e => e.key === "Enter" && handleSetGoal()} style={inputStyle} />
          <button onClick={handleSetGoal} style={btnStyle}>ЗАДАТЬ</button>
        </div>
      </div>
      <div style={card}>
        <div style={labelStyle}>Добавить в капитал</div>
        <div style={{ display: "flex", gap: 12 }}>
          <input type="number" value={addInput} min="0.01" step="0.01" placeholder="Сумма пополнения, $" onChange={e => setAddInput(e.target.value)} onKeyDown={e => e.key === "Enter" && handleAdd()} style={inputStyle} />
          <button onClick={handleAdd} style={btnStyle}>+ ДОБАВИТЬ</button>
        </div>
        {error && <div style={{ marginTop: 10, color: "#ff4d6d", fontSize: "0.75rem" }}>{error}</div>}
      </div>
      <div style={card}>
        <div style={{ ...labelStyle, marginBottom: 14 }}>Статистика</div>
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
          {[{ l: "Накоплено", v: fmt(current), c: "#c8ff00" }, { l: "Цель", v: goal > 0 ? fmt(goal) : "— $", c: "#e8eaf0" }, { l: "Осталось", v: goal > 0 ? fmt(Math.max(goal - current, 0)) : "— $", c: "#ff4d6d" }, { l: "Пополнений", v: log.length, c: "#e8eaf0" }].map(({ l, v, c }) => (
            <div key={l} style={{ background: "#0d0f14", border: "1px solid #252a38", borderRadius: 12, padding: 16 }}>
              <div style={labelStyle}>{l}</div>
              <div style={{ fontFamily: "'Bebas Neue', sans-serif", fontSize: "1.8rem", color: c, marginTop: 4 }}>{v}</div>
            </div>
          ))}
        </div>
        <button onClick={handleReset} style={{ ...btnStyle, background: "transparent", border: "1px solid #252a38", color: "#5a6080", fontSize: "1rem", width: "100%", marginTop: 16, height: 46 }}>↺ СБРОСИТЬ ВСЁ</button>
      </div>
      <div style={card}>
        <div style={{ ...labelStyle, marginBottom: 14 }}>История пополнений</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 8, maxHeight: 220, overflowY: "auto" }}>
          {log.length === 0
            ? <div style={{ textAlign: "center", color: "#5a6080", fontSize: "0.75rem", padding: "24px 0" }}>Пополнений пока нет</div>
            : [...log].reverse().map((e, i) => (
              <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", background: "#0d0f14", border: "1px solid #252a38", borderRadius: 10, padding: "10px 14px", fontSize: "0.8rem" }}>
                <span style={{ color: "#c8ff00", fontWeight: 700 }}>+ {fmt(e.amount)}</span>
                <span style={{ color: "#5a6080", fontSize: "0.65rem" }}>{e.time}</span>
              </div>
            ))}
        </div>
      </div>
    </div>
  );
}
ReactDOM.createRoot(document.getElementById("root")).render(<App />);
</script>
</body>
</html>
