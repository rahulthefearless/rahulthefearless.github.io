<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sleep Lab AI</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Bebas+Neue&family=Inter:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
:root {
  --bg: #06060e;
  --s1: #0c0c18;
  --s2: #111120;
  --s3: #16162a;
  --border: #1e1e35;
  --border2: #262640;
  --text: #ddddf5;
  --muted: #5a5a80;
  --muted2: #3a3a58;
  --accent: #6c63ff;
  --accent-glow: rgba(108,99,255,0.3);
  --green: #00e5a0;
  --green-dim: rgba(0,229,160,0.12);
  --red: #ff4d6d;
  --red-dim: rgba(255,77,109,0.12);
  --amber: #ffaa00;
  --amber-dim: rgba(255,170,0,0.12);
  --blue: #4da6ff;
  --purple: #c084fc;
  --mono: 'Space Mono', monospace;
  --sans: 'Inter', sans-serif;
  --display: 'Bebas Neue', sans-serif;
}

* { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; }

body {
  font-family: var(--sans);
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  overflow-x: hidden;
}

/* Scanline overlay */
body::after {
  content: '';
  position: fixed; inset: 0; pointer-events: none; z-index: 9999;
  background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,0,0,0.03) 2px, rgba(0,0,0,0.03) 4px);
}

/* Ambient glow */
.glow-orb {
  position: fixed; border-radius: 50%; filter: blur(80px);
  pointer-events: none; z-index: 0; opacity: 0.4;
}
.orb1 { width: 400px; height: 400px; background: var(--accent); top: -100px; right: -100px; }
.orb2 { width: 300px; height: 300px; background: var(--green); bottom: -80px; left: -80px; }

.app { position: relative; z-index: 1; max-width: 800px; margin: 0 auto; padding: 2rem 1.25rem 5rem; }

/* ── HEADER ── */
.header { margin-bottom: 2.5rem; display: flex; align-items: flex-start; justify-content: space-between; }
.header-left {}
.eyebrow { font-family: var(--mono); font-size: 9px; letter-spacing: 4px; color: var(--accent); text-transform: uppercase; margin-bottom: 8px; }
.logo { font-family: var(--display); font-size: 52px; letter-spacing: 2px; line-height: 1; color: #fff; }
.logo span { color: var(--green); }
.tagline { font-size: 11px; color: var(--muted); margin-top: 6px; line-height: 1.6; max-width: 320px; font-family: var(--mono); }
.nights-badge {
  background: var(--s2); border: 1px solid var(--border2);
  border-radius: 12px; padding: 12px 16px; text-align: center;
}
.nb-num { font-family: var(--display); font-size: 36px; color: var(--accent); line-height: 1; }
.nb-label { font-family: var(--mono); font-size: 9px; color: var(--muted); letter-spacing: 2px; margin-top: 2px; }

/* ── TABS ── */
.tabs { display: flex; border-bottom: 1px solid var(--border); margin-bottom: 2rem; gap: 0; }
.tab {
  font-family: var(--mono); font-size: 10px; letter-spacing: 2px; text-transform: uppercase;
  padding: 12px 18px; background: transparent; border: none; color: var(--muted);
  cursor: pointer; border-bottom: 2px solid transparent; margin-bottom: -1px;
  transition: all 0.2s; white-space: nowrap;
}
.tab.on { color: var(--green); border-bottom-color: var(--green); }
.tab:hover:not(.on) { color: var(--text); }

.pane { display: none; }
.pane.on { display: block; }

/* ── CARDS ── */
.card {
  background: var(--s1); border: 1px solid var(--border);
  border-radius: 16px; padding: 1.5rem; margin-bottom: 1.25rem;
  position: relative; overflow: hidden;
}
.card::before {
  content: ''; position: absolute; top: 0; left: 0; right: 0; height: 1px;
  background: linear-gradient(90deg, transparent, var(--border2), transparent);
}
.card-label {
  font-family: var(--mono); font-size: 9px; letter-spacing: 3px;
  color: var(--muted); text-transform: uppercase; margin-bottom: 1.25rem;
  display: flex; align-items: center; gap: 8px;
}
.card-label::before { content: '//'; color: var(--accent); }

/* ── UPLOAD ZONE ── */
.upload-zone {
  border: 2px dashed var(--border2); border-radius: 12px;
  padding: 2.5rem 1rem; text-align: center; cursor: pointer;
  transition: all 0.2s; position: relative; background: var(--s2);
}
.upload-zone:hover, .upload-zone.drag { border-color: var(--green); background: var(--green-dim); }
.upload-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; width: 100%; }
.uz-icon { font-size: 36px; margin-bottom: 10px; }
.uz-title { font-size: 14px; font-weight: 600; margin-bottom: 4px; }
.uz-sub { font-size: 11px; color: var(--muted); font-family: var(--mono); }
.uz-preview { max-width: 200px; border-radius: 8px; margin: 12px auto 0; display: none; }

/* ── AI STATUS ── */
.ai-status {
  display: none; align-items: center; gap: 12px;
  background: var(--s3); border: 1px solid var(--border2); border-radius: 10px;
  padding: 12px 16px; margin-top: 12px; font-family: var(--mono); font-size: 11px;
}
.ai-status.show { display: flex; }
.ai-dot { width: 8px; height: 8px; border-radius: 50%; background: var(--green); flex-shrink: 0; }
.ai-dot.pulse { animation: pulse 1s infinite; }
@keyframes pulse { 0%,100% { opacity: 1; transform: scale(1); } 50% { opacity: 0.4; transform: scale(0.8); } }
.ai-text { color: var(--muted); flex: 1; }
.ai-text strong { color: var(--text); }

/* ── EXTRACTED DATA ── */
.extracted-grid {
  display: grid; grid-template-columns: repeat(3,1fr); gap: 10px; margin: 1rem 0;
}
.ex-box {
  background: var(--s3); border: 1px solid var(--border2); border-radius: 10px;
  padding: 12px; text-align: center;
}
.ex-val { font-family: var(--display); font-size: 28px; letter-spacing: 1px; line-height: 1; }
.ex-lbl { font-family: var(--mono); font-size: 9px; color: var(--muted); letter-spacing: 1px; margin-top: 4px; }

/* FIELDS */
.field { display: flex; flex-direction: column; gap: 5px; margin-bottom: 12px; }
.field label { font-family: var(--mono); font-size: 9px; color: var(--muted); letter-spacing: 2px; text-transform: uppercase; }
.field input, .field select, .field textarea {
  background: var(--s2); border: 1px solid var(--border2); color: var(--text);
  border-radius: 8px; padding: 9px 12px; font-size: 13px; font-family: var(--sans);
  outline: none; transition: border-color 0.15s; width: 100%;
}
.field input:focus, .field select:focus, .field textarea:focus { border-color: var(--accent); }
.field textarea { resize: vertical; min-height: 65px; line-height: 1.6; font-size: 12px; }
.frow2 { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.frow3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; }

/* HABITS */
.habit-section { margin: 14px 0 8px; font-family: var(--mono); font-size: 9px; letter-spacing: 2px; color: var(--muted); text-transform: uppercase; }
.habit-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 7px; }
.hc {
  display: flex; align-items: center; gap: 8px; padding: 8px 11px;
  background: var(--s2); border: 1px solid var(--border); border-radius: 8px;
  cursor: pointer; transition: all 0.15s; user-select: none;
}
.hc:hover { border-color: var(--border2); }
.hc.yes { border-color: var(--green); background: var(--green-dim); }
.hc.no { border-color: var(--red); background: var(--red-dim); }
.hc-box {
  width: 15px; height: 15px; border-radius: 4px; border: 1px solid var(--border2);
  flex-shrink: 0; display: flex; align-items: center; justify-content: center;
  font-size: 9px; font-weight: 700; transition: all 0.15s;
}
.hc.yes .hc-box { background: var(--green); border-color: var(--green); color: #000; }
.hc.no .hc-box { background: var(--red); border-color: var(--red); color: #fff; }
.hc-lbl { font-size: 11px; color: var(--muted); line-height: 1.3; }
.hc.yes .hc-lbl, .hc.no .hc-lbl { color: var(--text); }

/* BTN */
.btn {
  font-family: var(--mono); font-size: 11px; letter-spacing: 2px; text-transform: uppercase;
  padding: 12px 24px; border-radius: 10px; border: none; cursor: pointer; transition: all 0.2s;
  display: flex; align-items: center; justify-content: center; gap: 8px;
}
.btn-primary {
  background: linear-gradient(135deg, var(--accent), #9c55ff);
  color: #fff; width: 100%; margin-top: 8px;
  box-shadow: 0 4px 24px rgba(108,99,255,0.3);
}
.btn-primary:hover { transform: translateY(-1px); box-shadow: 0 6px 32px rgba(108,99,255,0.4); }
.btn-primary:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }
.btn-green {
  background: linear-gradient(135deg, #00b37a, var(--green));
  color: #000; font-weight: 700; width: 100%; margin-top: 8px;
  box-shadow: 0 4px 24px rgba(0,229,160,0.2);
}
.btn-green:hover { transform: translateY(-1px); }
.btn-ghost { background: transparent; color: var(--muted); border: 1px solid var(--border2); }

/* CONFIRM */
.confirm-bar {
  display: none; text-align: center; padding: 12px;
  font-family: var(--mono); font-size: 11px; color: var(--green);
  background: var(--green-dim); border: 1px solid rgba(0,229,160,0.2);
  border-radius: 8px; margin-top: 10px; letter-spacing: 1px;
}

/* STATS */
.stat-row { display: grid; grid-template-columns: repeat(4,1fr); gap: 8px; margin-bottom: 1.25rem; }
.stat-box { background: var(--s2); border: 1px solid var(--border); border-radius: 12px; padding: 14px; text-align: center; }
.stat-val { font-family: var(--display); font-size: 26px; letter-spacing: 1px; line-height: 1; }
.stat-lbl { font-family: var(--mono); font-size: 8px; color: var(--muted); letter-spacing: 1px; text-transform: uppercase; margin-top: 4px; }

/* TREND CHART */
.chart-wrap { height: 90px; display: flex; align-items: flex-end; gap: 5px; }
.chart-col { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; }
.chart-bar {
  width: 100%; border-radius: 4px 4px 0 0; min-height: 4px;
  transition: height 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
  cursor: pointer; position: relative;
}
.chart-bar:hover::after {
  content: attr(data-tip); position: absolute; bottom: 105%; left: 50%;
  transform: translateX(-50%); background: var(--s3); border: 1px solid var(--border2);
  border-radius: 6px; padding: 4px 8px; font-size: 10px; white-space: nowrap;
  font-family: var(--mono); color: var(--text); z-index: 10;
}
.chart-lbl { font-family: var(--mono); font-size: 8px; color: var(--muted); text-align: center; }

/* LOG ITEMS */
.log-item {
  background: var(--s2); border: 1px solid var(--border); border-radius: 12px;
  padding: 14px; margin-bottom: 8px; transition: border-color 0.15s;
}
.log-item:hover { border-color: var(--border2); }
.li-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.li-date { font-family: var(--display); font-size: 18px; letter-spacing: 1px; }
.li-score { font-family: var(--display); font-size: 22px; letter-spacing: 1px; padding: 2px 14px; border-radius: 99px; }
.li-stages { display: flex; gap: 12px; flex-wrap: wrap; font-family: var(--mono); font-size: 10px; color: var(--muted); }
.li-stages span { color: var(--text); }
.li-tags { display: flex; gap: 5px; flex-wrap: wrap; margin-top: 8px; }
.tag { font-size: 10px; padding: 2px 8px; border-radius: 99px; font-family: var(--mono); }
.tag-g { background: var(--green-dim); color: var(--green); }
.tag-b { background: var(--red-dim); color: var(--red); }
.li-note { font-size: 11px; color: var(--muted); margin-top: 6px; font-style: italic; line-height: 1.5; }
.li-delta { font-family: var(--mono); font-size: 10px; margin-left: 8px; }

/* HABIT IMPACT */
.impact-row {
  display: flex; align-items: center; gap: 10px; padding: 10px 0;
  border-bottom: 1px solid var(--border); font-size: 12px;
}
.impact-row:last-child { border-bottom: none; }
.ir-name { width: 200px; flex-shrink: 0; font-size: 12px; }
.ir-track { flex: 1; height: 5px; background: var(--s3); border-radius: 3px; overflow: hidden; }
.ir-fill { height: 100%; border-radius: 3px; transition: width 0.6s; }
.ir-val { font-family: var(--mono); font-size: 11px; font-weight: 700; min-width: 55px; text-align: right; }
.ir-n { font-family: var(--mono); font-size: 9px; color: var(--muted); }

/* INSIGHT */
.insight {
  border-radius: 10px; padding: 12px 14px; margin-bottom: 10px;
  font-size: 12px; line-height: 1.7; display: flex; gap: 10px;
}
.ins-g { background: var(--green-dim); border: 1px solid rgba(0,229,160,0.2); }
.ins-r { background: var(--red-dim); border: 1px solid rgba(255,77,109,0.2); }
.ins-a { background: var(--amber-dim); border: 1px solid rgba(255,170,0,0.2); }
.ins-p { background: rgba(192,132,252,0.08); border: 1px solid rgba(192,132,252,0.2); }
.ins-icon { font-size: 16px; flex-shrink: 0; }
.ins-text { color: var(--muted); }
.ins-text strong { color: var(--text); }

/* PROTOCOL */
.proto-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.proto-col { background: var(--s2); border-radius: 12px; padding: 1rem; border: 1px solid var(--border); }
.proto-col h4 { font-family: var(--mono); font-size: 9px; letter-spacing: 3px; text-transform: uppercase; margin-bottom: 14px; }
.proto-col.do h4 { color: var(--green); }
.proto-col.avoid h4 { color: var(--red); }
.proto-item { display: flex; gap: 8px; margin-bottom: 10px; font-size: 11px; color: var(--muted); line-height: 1.55; align-items: flex-start; }
.proto-item:last-child { margin-bottom: 0; }
.pi-ico { flex-shrink: 0; }
.pi-status { font-family: var(--mono); font-size: 9px; margin-top: 2px; display: block; }
.pi-confirmed { color: var(--green); }
.pi-pending { color: var(--amber); }
.pi-harmful { color: var(--red); }

/* EMPTY */
.empty { text-align: center; padding: 3rem 1rem; color: var(--muted); font-size: 12px; font-family: var(--mono); }
.empty-ico { font-size: 32px; margin-bottom: 10px; }

/* SCROLLBAR */
::-webkit-scrollbar { width: 3px; }
::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 2px; }

/* SLEEP ARCH BAR */
.arch-bar { height: 28px; border-radius: 8px; overflow: hidden; display: flex; margin: 8px 0 4px; }
.arch-seg { display: flex; align-items: center; justify-content: center; font-size: 9px; font-weight: 700; color: rgba(255,255,255,0.7); }

.score-hero-num { font-family: var(--display); font-size: 72px; letter-spacing: 2px; line-height: 1; }

@media (max-width: 540px) {
  .proto-grid { grid-template-columns: 1fr; }
  .stat-row { grid-template-columns: repeat(2,1fr); }
  .extracted-grid { grid-template-columns: repeat(2,1fr); }
  .habit-grid { grid-template-columns: 1fr; }
  .frow2, .frow3 { grid-template-columns: 1fr; }
  .header { flex-direction: column; gap: 14px; }
  .logo { font-size: 40px; }
  .tabs { overflow-x: auto; }
}
</style>
</head>
<body>
<div class="glow-orb orb1"></div>
<div class="glow-orb orb2"></div>

<div class="app">

  <!-- API KEY SETUP SCREEN -->
  <div id="setup-screen" style="display:none; min-height:100vh; display:flex; align-items:center; justify-content:center; padding:2rem;">
    <div style="max-width:480px; width:100%;">
      <div class="eyebrow" style="text-align:center; margin-bottom:8px;">// First-time setup</div>
      <div class="logo" style="text-align:center; margin-bottom:8px;">Sleep <span>Lab</span> AI</div>
      <p style="font-family:var(--mono); font-size:11px; color:var(--muted); text-align:center; line-height:1.8; margin-bottom:2rem;">To auto-read your screenshots, this app needs an Anthropic API key.<br>Your key is stored only in <strong style="color:var(--text)">your browser</strong> — never sent anywhere else.</p>

      <div style="background:var(--s1); border:1px solid var(--border); border-radius:12px; padding:1.5rem;">
        <div style="font-family:var(--mono); font-size:10px; color:var(--muted); letter-spacing:2px; margin-bottom:16px;">HOW TO GET YOUR FREE API KEY</div>

        <div style="display:flex; flex-direction:column; gap:10px; margin-bottom:20px;">
          <div style="display:flex; gap:10px; font-size:12px; color:var(--muted); line-height:1.6; align-items:flex-start;">
            <div style="width:22px; height:22px; border-radius:50%; background:var(--accent); color:#fff; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:700; flex-shrink:0;">1</div>
            <div>Go to <a href="https://console.anthropic.com" target="_blank" style="color:var(--accent)">console.anthropic.com</a> and sign up free</div>
          </div>
          <div style="display:flex; gap:10px; font-size:12px; color:var(--muted); line-height:1.6; align-items:flex-start;">
            <div style="width:22px; height:22px; border-radius:50%; background:var(--accent); color:#fff; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:700; flex-shrink:0;">2</div>
            <div>Click <strong style="color:var(--text)">API Keys</strong> in the left sidebar → <strong style="color:var(--text)">Create Key</strong></div>
          </div>
          <div style="display:flex; gap:10px; font-size:12px; color:var(--muted); line-height:1.6; align-items:flex-start;">
            <div style="width:22px; height:22px; border-radius:50%; background:var(--accent); color:#fff; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:700; flex-shrink:0;">3</div>
            <div>Copy the key (starts with <code style="color:var(--green); font-size:11px;">sk-ant-...</code>) and paste below</div>
          </div>
          <div style="display:flex; gap:10px; font-size:12px; color:var(--muted); line-height:1.6; align-items:flex-start;">
            <div style="width:22px; height:22px; border-radius:50%; background:var(--accent); color:#fff; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:700; flex-shrink:0;">4</div>
            <div>Free tier gives enough credits to scan hundreds of screenshots</div>
          </div>
        </div>

        <div style="margin-bottom:12px;">
          <label style="font-family:var(--mono); font-size:10px; color:var(--muted); letter-spacing:1px; display:block; margin-bottom:6px;">PASTE YOUR API KEY</label>
          <input type="password" id="api-key-input" placeholder="sk-ant-api03-..." style="width:100%; background:var(--s2); border:1px solid var(--border2); color:var(--text); border-radius:8px; padding:10px 12px; font-size:13px; font-family:var(--mono); outline:none;">
        </div>

        <button onclick="saveApiKey()" style="width:100%; background:linear-gradient(135deg,var(--accent),#9c55ff); color:#fff; border:none; padding:12px; border-radius:8px; font-family:var(--mono); font-size:11px; letter-spacing:2px; text-transform:uppercase; cursor:pointer; font-weight:700;">
          Save Key & Open App →
        </button>
        <div id="key-error" style="display:none; text-align:center; margin-top:8px; font-size:11px; color:var(--red); font-family:var(--mono);">Key must start with sk-ant-</div>
      </div>
    </div>
  </div>

  <!-- HEADER -->
  <div id="main-app" style="display:none">
  <div class="header">
    <div class="header-left">
      <div class="eyebrow">// Boat Wave 2 · Personal Sleep Science</div>
      <div class="logo">Sleep <span>Lab</span> AI</div>
      <div class="tagline">Upload screenshot → AI reads it → auto-saved → patterns emerge</div>
    </div>
    <div style="display:flex; flex-direction:column; align-items:flex-end; gap:8px;">
      <div class="nights-badge">
        <div class="nb-num" id="nights-count">0</div>
        <div class="nb-label">Nights<br>Tracked</div>
      </div>
      <button onclick="changeKey()" style="font-family:var(--mono); font-size:9px; color:var(--muted); background:transparent; border:1px solid var(--border2); border-radius:6px; padding:4px 10px; cursor:pointer; letter-spacing:1px;">⚙ API KEY</button>
    </div>
  </div>

  <!-- TABS -->
  <div class="tabs">
    <button class="tab on" onclick="switchTab('scan',this)">📸 Scan</button>
    <button class="tab" onclick="switchTab('dashboard',this)">📊 Dashboard</button>
    <button class="tab" onclick="switchTab('science',this)">🧪 Habit Science</button>
    <button class="tab" onclick="switchTab('protocol',this)">📋 Protocol</button>
  </div>


  <!-- ══════════════════ SCAN PANE ══════════════════ -->
  <div id="pane-scan" class="pane on">

    <div class="card">
      <div class="card-label">Upload Tonight's Screenshot</div>

      <div class="upload-zone" id="upload-zone">
        <input type="file" accept="image/*" id="img-input" onchange="handleImage(this)">
        <div class="uz-icon">📱</div>
        <div class="uz-title">Drop your Boat Wave 2 / Zepp screenshot</div>
        <div class="uz-sub">Tap to select · Any screenshot with sleep stages visible</div>
        <img class="uz-preview" id="img-preview">
      </div>

      <div class="ai-status" id="ai-status">
        <div class="ai-dot pulse" id="ai-dot"></div>
        <div class="ai-text" id="ai-text">Initialising...</div>
      </div>
    </div>

    <!-- EXTRACTED + CONFIRM -->
    <div class="card" id="extracted-card" style="display:none">
      <div class="card-label">Extracted Sleep Data</div>

      <div class="extracted-grid">
        <div class="ex-box"><div class="ex-val" id="ex-score" style="color:var(--amber)">—</div><div class="ex-lbl">Sleep Score</div></div>
        <div class="ex-box"><div class="ex-val" id="ex-deep" style="color:var(--red)">—</div><div class="ex-lbl">Deep (min)</div></div>
        <div class="ex-box"><div class="ex-val" id="ex-rem" style="color:var(--purple)">—</div><div class="ex-lbl">REM (min)</div></div>
        <div class="ex-box"><div class="ex-val" id="ex-light" style="color:var(--green)">—</div><div class="ex-lbl">Light (min)</div></div>
        <div class="ex-box"><div class="ex-val" id="ex-awake" style="color:var(--amber)">—</div><div class="ex-lbl">Awake (min)</div></div>
        <div class="ex-box"><div class="ex-val" id="ex-total" style="color:var(--blue)">—</div><div class="ex-lbl">Total (min)</div></div>
      </div>

      <div class="frow2">
        <div class="field">
          <label>Slept at (actual)</label>
          <input type="time" id="ex-slept">
        </div>
        <div class="field">
          <label>Woke at</label>
          <input type="time" id="ex-woke">
        </div>
      </div>

      <div class="field">
        <label>Date</label>
        <input type="date" id="ex-date">
      </div>

      <!-- GOOD HABITS -->
      <div class="habit-section">✓ What I did GOOD tonight</div>
      <div class="habit-grid" id="good-grid"></div>

      <!-- BAD HABITS -->
      <div class="habit-section">✗ What I did BAD / missed</div>
      <div class="habit-grid" id="bad-grid"></div>

      <div class="field" style="margin-top:14px;">
        <label>Notes / confounding factors</label>
        <textarea id="ex-notes" placeholder="Stress, travel, noise, illness, unusual day? Flag it here — noisy nights are excluded from habit correlations."></textarea>
      </div>

      <button class="btn btn-green" onclick="confirmSave()">✓ Confirm & Save to Lab</button>
      <div class="confirm-bar" id="confirm-bar">✓ NIGHT LOGGED — dashboard updated</div>
    </div>

  </div>

  <!-- ══════════════════ DASHBOARD PANE ══════════════════ -->
  <div id="pane-dashboard" class="pane">

    <div id="dash-empty" class="card">
      <div class="empty"><div class="empty-ico">📊</div>No nights logged yet.<br>Upload your first screenshot in the Scan tab.</div>
    </div>

    <div id="dash-main" style="display:none">

      <div class="card" id="dash-overview"></div>

      <div class="card" id="dash-trend-card">
        <div class="card-label">Score Trend</div>
        <div class="chart-wrap" id="trend-chart"></div>
        <div style="display:flex;font-family:var(--mono);font-size:8px;color:var(--muted);margin-top:6px;" id="trend-labels"></div>
      </div>

      <div class="card" id="dash-insights-card">
        <div class="card-label">Auto Insights</div>
        <div id="dash-insights"></div>
      </div>

      <div class="card">
        <div class="card-label">All Nights</div>
        <div id="all-logs"></div>
      </div>

    </div>
  </div>

  <!-- ══════════════════ HABIT SCIENCE PANE ══════════════════ -->
  <div id="pane-science" class="pane">
    <div class="card">
      <div class="card-label">Habit Impact on Sleep Score</div>
      <div style="font-family:var(--mono);font-size:10px;color:var(--muted);margin-bottom:16px;line-height:1.7;">
        Average score delta: nights WITH habit vs nights WITHOUT.<br>
        Needs 3+ data points. Single anomalies excluded via notes flag.
      </div>
      <div id="habit-impact"></div>
    </div>
    <div class="card">
      <div class="card-label">Experiment Progress</div>
      <div id="exp-progress"></div>
    </div>
  </div>

  <!-- ══════════════════ PROTOCOL PANE ══════════════════ -->
  <div id="pane-protocol" class="pane">
    <div class="card">
      <div class="card-label">Error & Noise Policy</div>
      <div class="insight ins-a">
        <div class="ins-icon">⚗️</div>
        <div class="ins-text"><strong>3-night minimum, 7-night ideal.</strong> One change at a time. Flag confounders in notes — they're excluded from habit scoring. A +5 pt difference over 3+ clean nights = meaningful signal.</div>
      </div>
    </div>
    <div class="card">
      <div class="card-label">Do's & Avoid's</div>
      <div class="proto-grid">
        <div class="proto-col do">
          <h4>✓ Do These</h4>
          <div class="proto-item"><span class="pi-ico">🕙</span><div>Sleep before <strong>11 PM</strong> — catches HGH + melatonin peak<span class="pi-status pi-confirmed">🟢 Confirmed by science</span></div></div>
          <div class="proto-item"><span class="pi-ico">🌡️</span><div>Room <strong>18–20°C</strong> — core temp must drop for deep sleep<span class="pi-status pi-confirmed">🟢 Confirmed</span></div></div>
          <div class="proto-item"><span class="pi-ico">🌑</span><div><strong>Complete darkness</strong> — even dim light delays melatonin<span class="pi-status pi-confirmed">🟢 Confirmed</span></div></div>
          <div class="proto-item"><span class="pi-ico">🏃</span><div><strong>Exercise</strong> before 6 PM — boosts slow-wave sleep<span class="pi-status pi-confirmed">🟢 Confirmed</span></div></div>
          <div class="proto-item"><span class="pi-ico">🥗</span><div>Last meal <strong>2h+ before</strong> bed<span class="pi-status pi-confirmed">🟢 Confirmed</span></div></div>
          <div class="proto-item"><span class="pi-ico">📖</span><div><strong>30m wind-down</strong> — reading, no screens, dim light<span class="pi-status pi-pending">🟡 Track it — your data will confirm</span></div></div>
          <div class="proto-item"><span class="pi-ico">⏰</span><div><strong>Same wake time</strong> daily — anchors circadian rhythm<span class="pi-status pi-confirmed">🟢 Confirmed</span></div></div>
          <div class="proto-item"><span class="pi-ico">💧</span><div>No liquids <strong>after 8 PM</strong> — prevents wake-ups<span class="pi-status pi-pending">🟡 Track it</span></div></div>
        </div>
        <div class="proto-col avoid">
          <h4>✗ Avoid These</h4>
          <div class="proto-item"><span class="pi-ico">☕</span><div><strong>Caffeine after 1–2 PM</strong> — 6hr half-life kills deep sleep<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
          <div class="proto-item"><span class="pi-ico">📱</span><div><strong>Screens within 45m</strong> of bed — delays melatonin 3h<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
          <div class="proto-item"><span class="pi-ico">🍺</span><div><strong>Alcohol</strong> — destroys REM, fragments second half<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
          <div class="proto-item"><span class="pi-ico">🍕</span><div><strong>Heavy meal within 2h</strong> — raises core temp<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
          <div class="proto-item"><span class="pi-ico">😤</span><div><strong>Stressful content</strong> before bed — raises cortisol<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
          <div class="proto-item"><span class="pi-ico">💡</span><div><strong>Bright lights after 9 PM</strong><span class="pi-status pi-pending">🟡 Track it</span></div></div>
          <div class="proto-item"><span class="pi-ico">🕐</span><div><strong>Sleeping past midnight</strong> — misses golden window<span class="pi-status pi-harmful">🔴 Your data confirms this</span></div></div>
          <div class="proto-item"><span class="pi-ico">⏰</span><div><strong>Snooze button</strong> — incomplete cycle = grogginess<span class="pi-status pi-harmful">🔴 Proven harmful</span></div></div>
        </div>
      </div>
    </div>
    <div class="card">
      <div class="card-label">Experiment Queue</div>
      <div id="exp-queue">
        <div class="impact-row"><div class="ir-name">🕙 Sleep before 11 PM</div><div style="font-family:var(--mono);font-size:10px;color:var(--amber)">→ START HERE — Experiment 1</div></div>
        <div class="impact-row"><div class="ir-name">📵 No screens 45m before bed</div><div style="font-family:var(--mono);font-size:10px;color:var(--muted)">Experiment 2</div></div>
        <div class="impact-row"><div class="ir-name">☕ No caffeine after 1 PM</div><div style="font-family:var(--mono);font-size:10px;color:var(--muted)">Experiment 3</div></div>
        <div class="impact-row"><div class="ir-name">🏃 Morning / early exercise</div><div style="font-family:var(--mono);font-size:10px;color:var(--muted)">Experiment 4</div></div>
        <div class="impact-row"><div class="ir-name">🌑 Complete room darkness</div><div style="font-family:var(--mono);font-size:10px;color:var(--muted)">Experiment 5</div></div>
        <div class="impact-row"><div class="ir-name">📖 30m wind-down routine</div><div style="font-family:var(--mono);font-size:10px;color:var(--muted)">Experiment 6</div></div>
      </div>
    </div>
  </div>

</div><!-- /main-app -->
</div><!-- /app -->

<script>
// ───────────────────────────────────────────── HABITS DATA
const GOOD = [
  {id:'before11', label:'🕙 Slept before 11 PM'},
  {id:'noscreens', label:'📵 No screens 45m before'},
  {id:'nocaff', label:'☕ No caffeine after 1 PM'},
  {id:'exercise', label:'🏃 Exercised today'},
  {id:'dark', label:'🌑 Room completely dark'},
  {id:'cool', label:'🌡️ Room cool (18–20°C)'},
  {id:'winddown', label:'📖 30m wind-down done'},
  {id:'earlymeal', label:'🥗 Last meal 2h+ before'},
  {id:'nowater', label:'💧 No liquids after 8 PM'},
  {id:'samewake', label:'⏰ Same wake time kept'},
];
const BAD = [
  {id:'latebed', label:'🕐 Slept after midnight'},
  {id:'screens', label:'📱 Screens in bed'},
  {id:'alcohol', label:'🍺 Had alcohol'},
  {id:'latemeal', label:'🍕 Heavy meal within 2h'},
  {id:'latecaff', label:'☕ Caffeine after 2 PM'},
  {id:'stress', label:'😤 Stressful content before bed'},
  {id:'snooze', label:'⏰ Used snooze'},
  {id:'brightlight', label:'💡 Bright lights past 9 PM'},
];

// ───────────────────────────────────────────── STORAGE
function load() { try { return JSON.parse(localStorage.getItem('sleeplab_ai_v1')||'[]'); } catch { return []; } }
function save(d) { try { localStorage.setItem('sleeplab_ai_v1', JSON.stringify(d)); } catch {} }

// ───────────────────────────────────────────── INIT
let currentExtracted = {};

function init() {
  document.getElementById('ex-date').value = new Date().toISOString().split('T')[0];
  renderHabitGrids();
  updateNightsCount();

  const uz = document.getElementById('upload-zone');
  uz.addEventListener('dragover', e => { e.preventDefault(); uz.classList.add('drag'); });
  uz.addEventListener('dragleave', () => uz.classList.remove('drag'));
  uz.addEventListener('drop', e => {
    e.preventDefault(); uz.classList.remove('drag');
    const f = e.dataTransfer.files[0];
    if (f && f.type.startsWith('image/')) processFile(f);
  });
}

function updateNightsCount() {
  document.getElementById('nights-count').textContent = load().length;
}

// ───────────────────────────────────────────── HABIT GRIDS
function renderHabitGrids() {
  document.getElementById('good-grid').innerHTML = GOOD.map(h => habitHTML(h,'yes')).join('');
  document.getElementById('bad-grid').innerHTML = BAD.map(h => habitHTML(h,'no')).join('');
  document.querySelectorAll('.hc').forEach(el => el.addEventListener('click', () => toggleHC(el)));
}

function habitHTML(h, type) {
  return `<div class="hc" data-id="${h.id}" data-type="${type}"><div class="hc-box"></div><div class="hc-lbl">${h.label}</div></div>`;
}

function toggleHC(el) {
  const cls = el.dataset.type;
  const active = el.classList.contains(cls);
  el.classList.toggle(cls, !active);
  el.querySelector('.hc-box').textContent = !active ? (cls==='yes'?'✓':'✗') : '';
}

// ───────────────────────────────────────────── IMAGE HANDLING
function handleImage(input) {
  const f = input.files[0];
  if (f) processFile(f);
}

function processFile(file) {
  const reader = new FileReader();
  reader.onload = e => {
    const dataUrl = e.target.result;
    const preview = document.getElementById('img-preview');
    preview.src = dataUrl;
    preview.style.display = 'block';
    const base64 = dataUrl.split(',')[1];
    const mime = file.type || 'image/jpeg';
    analyseWithClaude(base64, mime);
  };
  reader.readAsDataURL(file);
}

// ───────────────────────────────────────────── CLAUDE API CALL
async function analyseWithClaude(base64, mime) {
  const status = document.getElementById('ai-status');
  const dot = document.getElementById('ai-dot');
  const txt = document.getElementById('ai-text');
  const extracted = document.getElementById('extracted-card');

  status.classList.add('show');
  dot.classList.add('pulse');
  txt.innerHTML = '<strong>Reading screenshot...</strong> Extracting sleep stages, duration, and times';
  extracted.style.display = 'none';

  const prompt = `You are a sleep data extractor. Analyse this Boat Wave 2 / Zepp sleep tracking screenshot and extract ALL available sleep data.

Return ONLY a JSON object with these exact keys (use null if not visible):
{
  "score": <sleep score 0-100 or null>,
  "sleptAt": "<HH:MM in 24hr format or null>",
  "wokeAt": "<HH:MM in 24hr format or null>",
  "totalMinutes": <total sleep in minutes or null>,
  "deepMinutes": <deep sleep in minutes or null>,
  "remMinutes": <REM sleep in minutes or null>,
  "lightMinutes": <light sleep in minutes or null>,
  "awakeMinutes": <awake time in minutes or null>,
  "notes": "<any notable observations from the screenshot e.g. unusual patterns, or empty string>"
}

Convert all hours:minutes to total minutes (e.g. 1hr 20min = 80). 
For times shown as 12:26 AM convert to 00:26. For PM times after noon convert properly to 24hr.
Return ONLY the JSON object, no other text.`;

  try {
    const resp = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': getKey(),
        'anthropic-version': '2023-06-01',
        'anthropic-dangerous-direct-browser-access': 'true'
      },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 1000,
        messages: [{
          role: 'user',
          content: [{
            type: 'image',
            source: { type: 'base64', media_type: mime, data: base64 }
          }, {
            type: 'text',
            text: prompt
          }]
        }]
      })
    });

    if (!resp.ok) throw new Error('API error ' + resp.status);
    const data = await resp.json();
    const raw = data.content.find(b => b.type === 'text')?.text || '';
    const clean = raw.replace(/```json|```/g, '').trim();
    const parsed = JSON.parse(clean);

    currentExtracted = parsed;
    populateExtracted(parsed);

    dot.classList.remove('pulse');
    dot.style.background = 'var(--green)';
    txt.innerHTML = '<strong>✓ Data extracted</strong> — review below, add habits, then save';
    extracted.style.display = 'block';
    extracted.scrollIntoView({ behavior: 'smooth', block: 'start' });

  } catch (err) {
    dot.classList.remove('pulse');
    dot.style.background = 'var(--red)';
    txt.innerHTML = '<strong>Could not auto-extract</strong> — fill in manually below';
    extracted.style.display = 'block';
    console.error(err);
  }
}

// ───────────────────────────────────────────── POPULATE
function populateExtracted(d) {
  setEx('ex-score', d.score);
  setEx('ex-deep', d.deepMinutes);
  setEx('ex-rem', d.remMinutes);
  setEx('ex-light', d.lightMinutes);
  setEx('ex-awake', d.awakeMinutes);
  setEx('ex-total', d.totalMinutes);

  if (d.sleptAt) document.getElementById('ex-slept').value = d.sleptAt;
  if (d.wokeAt) document.getElementById('ex-woke').value = d.wokeAt;
  if (d.notes) document.getElementById('ex-notes').value = d.notes;

  // Auto-tick late bedtime if slept after midnight
  if (d.sleptAt) {
    const [h] = d.sleptAt.split(':').map(Number);
    if (h >= 0 && h < 6) {
      const lateEl = document.querySelector('[data-id="latebed"]');
      if (lateEl && !lateEl.classList.contains('no')) {
        lateEl.classList.add('no');
        lateEl.querySelector('.hc-box').textContent = '✗';
      }
    }
  }
}

function setEx(id, val) {
  const el = document.getElementById(id);
  el.textContent = val !== null && val !== undefined ? val : '—';
}

// ───────────────────────────────────────────── SAVE
function confirmSave() {
  const date = document.getElementById('ex-date').value;
  if (!date) { alert('Please set the date.'); return; }

  const entry = {
    date,
    score: currentExtracted.score || 0,
    sleptAt: document.getElementById('ex-slept').value,
    wokeAt: document.getElementById('ex-woke').value,
    totalMinutes: currentExtracted.totalMinutes || 0,
    deep: currentExtracted.deepMinutes || 0,
    rem: currentExtracted.remMinutes || 0,
    light: currentExtracted.lightMinutes || 0,
    awake: currentExtracted.awakeMinutes || 0,
    notes: document.getElementById('ex-notes').value,
    goodDone: [...document.querySelectorAll('#good-grid .hc.yes')].map(e => e.dataset.id),
    badDone: [...document.querySelectorAll('#bad-grid .hc.no')].map(e => e.dataset.id),
    ts: Date.now()
  };

  const data = load();
  const idx = data.findIndex(d => d.date === date);
  if (idx >= 0) data[idx] = entry; else data.unshift(entry);
  data.sort((a,b) => b.date.localeCompare(a.date));
  save(data);
  updateNightsCount();

  const bar = document.getElementById('confirm-bar');
  bar.style.display = 'block';
  setTimeout(() => bar.style.display = 'none', 3000);

  // Reset habits
  document.querySelectorAll('.hc').forEach(el => {
    el.classList.remove('yes','no');
    el.querySelector('.hc-box').textContent = '';
  });
  document.getElementById('ex-notes').value = '';
  currentExtracted = {};
}

// ───────────────────────────────────────────── DASHBOARD
function renderDashboard() {
  const data = load();
  const empty = document.getElementById('dash-empty');
  const main = document.getElementById('dash-main');

  if (!data.length) { empty.style.display='block'; main.style.display='none'; return; }
  empty.style.display = 'none'; main.style.display = 'block';

  const r7 = data.slice(0,7);
  const avg = k => Math.round(r7.reduce((s,d)=>s+(d[k]||0),0)/r7.length);
  const avgScore = avg('score');
  const sc = avgScore>=80?'var(--green)':avgScore>=65?'var(--amber)':'var(--red)';
  const delta = data.length>=2 ? data[0].score - data[1].score : null;
  const dHtml = delta!==null ? `<span style="font-family:var(--mono);font-size:11px;color:${delta>0?'var(--green)':delta<0?'var(--red)':'var(--muted)'}">${delta>0?'▲':delta<0?'▼':'='} ${Math.abs(delta)} vs prev</span>` : '';
  const best = data.reduce((b,d)=>d.score>b.score?d:b,data[0]);

  document.getElementById('dash-overview').innerHTML = `
    <div class="card-label">Overview — ${data.length} night${data.length!==1?'s':''} logged</div>
    <div style="display:flex;align-items:center;gap:20px;margin-bottom:16px;">
      <div class="score-hero-num" style="color:${sc}">${avgScore}</div>
      <div>
        <div style="font-family:var(--mono);font-size:10px;color:var(--muted);">7-day avg score</div>
        <div style="margin-top:6px;">${dHtml}</div>
        <div style="font-family:var(--mono);font-size:10px;color:var(--muted);margin-top:4px;">Best: ${best.score} on ${best.date}</div>
      </div>
    </div>
    <div class="stat-row">
      <div class="stat-box"><div class="stat-val" style="color:var(--red)">${avg('deep')}m</div><div class="stat-lbl">Avg Deep</div></div>
      <div class="stat-box"><div class="stat-val" style="color:var(--purple)">${avg('rem')}m</div><div class="stat-lbl">Avg REM</div></div>
      <div class="stat-box"><div class="stat-val" style="color:var(--amber)">${avg('awake')}m</div><div class="stat-lbl">Avg Awake</div></div>
      <div class="stat-box"><div class="stat-val" style="color:var(--accent)">${data.length}</div><div class="stat-lbl">Nights</div></div>
    </div>`;

  // Trend
  const recent = data.slice(0,14).reverse();
  document.getElementById('trend-chart').innerHTML = recent.map(d => {
    const h = Math.max(5, Math.round((d.score/100)*85));
    const c = d.score>=80?'var(--green)':d.score>=65?'var(--amber)':'var(--red)';
    return `<div class="chart-col"><div class="chart-bar" style="height:${h}px;background:${c}" data-tip="${d.date}: ${d.score}"></div></div>`;
  }).join('');
  document.getElementById('trend-labels').innerHTML = recent.map(d =>
    `<div style="flex:1;text-align:center;">${d.date.slice(5)}</div>`
  ).join('');

  // Insights
  document.getElementById('dash-insights').innerHTML = generateInsights(data).join('');

  // Logs
  document.getElementById('all-logs').innerHTML = data.map((e,i) => {
    const sc2 = e.score>=80?'var(--green)':e.score>=65?'var(--amber)':'var(--red)';
    const prev = data[i+1];
    const d2 = prev ? e.score - prev.score : null;
    const dStr = d2!==null ? `<span class="li-delta" style="color:${d2>0?'var(--green)':d2<0?'var(--red)':'var(--muted)'}">${d2>0?'▲':d2<0?'▼':'='} ${Math.abs(d2)}</span>` : '';
    const gtags = e.goodDone.map(id=>{ const h=GOOD.find(g=>g.id===id); return h?`<span class="tag tag-g">${h.label}</span>`:''; }).join('');
    const btags = e.badDone.map(id=>{ const h=BAD.find(b=>b.id===id); return h?`<span class="tag tag-b">${h.label}</span>`:''; }).join('');
    return `<div class="log-item">
      <div class="li-head">
        <div class="li-date">${e.date} ${dStr}</div>
        <div class="li-score" style="background:${sc2}22;color:${sc2}">${e.score}</div>
      </div>
      <div class="li-stages">${e.sleptAt||'—'} → ${e.wokeAt||'—'} &nbsp;|&nbsp; Deep: <span>${e.deep}m</span> &nbsp;REM: <span>${e.rem}m</span> &nbsp;Awake: <span>${e.awake}m</span></div>
      ${gtags||btags ? `<div class="li-tags">${gtags}${btags}</div>` : ''}
      ${e.notes ? `<div class="li-note">"${e.notes}"</div>` : ''}
    </div>`;
  }).join('');
}

function generateInsights(data) {
  const out = [];
  if (data.length < 2) return [`<div class="insight ins-a"><div class="ins-icon">📊</div><div class="ins-text">Log more nights for auto insights.</div></div>`];

  const r3 = data.slice(0,3), r36 = data.slice(3,6);
  if (r3.length>=2 && r36.length>=2) {
    const a3 = r3.reduce((s,d)=>s+d.score,0)/r3.length;
    const a36 = r36.reduce((s,d)=>s+d.score,0)/r36.length;
    if (a3-a36>5) out.push(`<div class="insight ins-g"><div class="ins-icon">📈</div><div class="ins-text"><strong>Improving!</strong> Last 3 nights avg ${Math.round(a3)} vs ${Math.round(a36)} earlier.</div></div>`);
    else if (a36-a3>5) out.push(`<div class="insight ins-r"><div class="ins-icon">📉</div><div class="ins-text"><strong>Declining trend.</strong> Last 3 nights avg ${Math.round(a3)} vs ${Math.round(a36)} earlier. Check what changed.</div></div>`);
  }

  const avgAwake = data.slice(0,5).reduce((s,d)=>s+d.awake,0)/Math.min(data.length,5);
  if (avgAwake>25) out.push(`<div class="insight ins-r"><div class="ins-icon">⚠️</div><div class="ins-text"><strong>High awake time: ${Math.round(avgAwake)} min/night.</strong> Limit is 25 min. Fragmenting your cycles. Check room environment and phone use at night.</div></div>`);

  const avgRem = data.slice(0,5).reduce((s,d)=>s+d.rem,0)/Math.min(data.length,5);
  if (avgRem<60) out.push(`<div class="insight ins-p"><div class="ins-icon">🧠</div><div class="ins-text"><strong>Chronic REM deficit: ${Math.round(avgRem)} min avg.</strong> Target 90+ min. Affects mood, memory, learning. Sleep earlier and longer.</div></div>`);

  const avgDeep = data.slice(0,5).reduce((s,d)=>s+d.deep,0)/Math.min(data.length,5);
  if (avgDeep<70) out.push(`<div class="insight ins-r"><div class="ins-icon">💤</div><div class="ins-text"><strong>Deep sleep deficiency: ${Math.round(avgDeep)} min avg.</strong> Target 90+ min. Body repair, immunity, memory consolidation all suffer.</div></div>`);

  const lateN = data.filter(d=>d.badDone.includes('latebed'));
  const earlyN = data.filter(d=>d.goodDone.includes('before11'));
  if (lateN.length>=2 && earlyN.length>=1) {
    const lA = lateN.reduce((s,d)=>s+d.score,0)/lateN.length;
    const eA = earlyN.reduce((s,d)=>s+d.score,0)/earlyN.length;
    const imp = Math.round(eA-lA);
    if (imp>0) out.push(`<div class="insight ins-g"><div class="ins-icon">🕙</div><div class="ins-text"><strong>Early bedtime gives you +${imp} pts</strong> (${earlyN.length} early vs ${lateN.length} late nights). This is YOUR personal data confirming it.</div></div>`);
  }

  if (!out.length) out.push(`<div class="insight ins-g"><div class="ins-icon">✓</div><div class="ins-text">No major flags. Keep logging for deeper patterns.</div></div>`);
  return out;
}

// ───────────────────────────────────────────── HABIT SCIENCE
function renderHabitScience() {
  const data = load();
  if (data.length < 3) {
    document.getElementById('habit-impact').innerHTML = '<div class="empty"><div class="empty-ico">🧪</div>Need 3+ nights to show impact data.</div>';
    document.getElementById('exp-progress').innerHTML = '<div class="empty"><div class="empty-ico">⚗️</div>Experiments start after 7+ nights.</div>';
    return;
  }

  const allH = [...GOOD.map(h=>({...h,type:'good'})), ...BAD.map(h=>({...h,type:'bad'}))];
  const rows = allH.map(h => {
    const w = data.filter(d => h.type==='good' ? d.goodDone.includes(h.id) : d.badDone.includes(h.id));
    const wo = data.filter(d => h.type==='good' ? !d.goodDone.includes(h.id) : !d.badDone.includes(h.id));
    if (w.length<2 || wo.length<1) return null;
    const aw = w.reduce((s,d)=>s+d.score,0)/w.length;
    const awo = wo.reduce((s,d)=>s+d.score,0)/wo.length;
    const imp = Math.round(aw-awo);
    const display = h.type==='bad' ? -imp : imp;
    return {...h, imp, display, samples: w.length};
  }).filter(Boolean).sort((a,b)=>Math.abs(b.display)-Math.abs(a.display));

  if (!rows.length) {
    document.getElementById('habit-impact').innerHTML = '<div class="empty">Not enough varied data yet.</div>';
    return;
  }

  document.getElementById('habit-impact').innerHTML = rows.map(r => {
    const pos = r.display > 0;
    const c = pos ? 'var(--green)' : 'var(--red)';
    const pct = Math.min(100, Math.abs(r.display)*4);
    return `<div class="impact-row">
      <div class="ir-name">${r.label}</div>
      <div class="ir-track"><div class="ir-fill" style="width:${pct}%;background:${c}"></div></div>
      <div class="ir-val" style="color:${c}">${r.display>0?'+':''}${r.display} pts</div>
      <div class="ir-n">${r.samples}n</div>
    </div>`;
  }).join('');

  document.getElementById('exp-progress').innerHTML = rows.slice(0,6).map(r => {
    const status = r.samples>=5 ? '✓ Confirmed' : r.samples>=3 ? '~ Trending' : '? Early';
    const sc = r.samples>=5 ? 'var(--green)' : r.samples>=3 ? 'var(--amber)' : 'var(--muted)';
    return `<div class="impact-row">
      <div class="ir-name">${r.label}</div>
      <div style="font-family:var(--mono);font-size:10px;color:${sc}">${status} (${r.samples} nights)</div>
      <div class="ir-val" style="color:${r.display>0?'var(--green)':'var(--red)'}">${r.display>0?'+':''}${r.display} pts</div>
    </div>`;
  }).join('');
}

// ───────────────────────────────────────────── TABS
function switchTab(name, btn) {
  document.querySelectorAll('.pane').forEach(p=>p.classList.remove('on'));
  document.querySelectorAll('.tab').forEach(b=>b.classList.remove('on'));
  document.getElementById('pane-'+name).classList.add('on');
  btn.classList.add('on');
  if (name==='dashboard') renderDashboard();
  if (name==='science') renderHabitScience();
}

// ───────────────────────────────────────────── API KEY
function getKey() { return localStorage.getItem('slabai_apikey') || ''; }

function saveApiKey() {
  const k = document.getElementById('api-key-input').value.trim();
  if (!k.startsWith('sk-ant-')) {
    document.getElementById('key-error').style.display = 'block';
    return;
  }
  localStorage.setItem('slabai_apikey', k);
  document.getElementById('setup-screen').style.display = 'none';
  document.getElementById('main-app').style.display = 'block';
  init();
}

function changeKey() {
  const k = prompt('Paste your Anthropic API key (sk-ant-...):', getKey());
  if (k && k.startsWith('sk-ant-')) {
    localStorage.setItem('slabai_apikey', k);
    alert('Key saved!');
  } else if (k) {
    alert('Invalid key — must start with sk-ant-');
  }
}

// ───────────────────────────────────────────── STARTUP
(function startup() {
  const key = getKey();
  if (key) {
    document.getElementById('setup-screen').style.display = 'none';
    document.getElementById('main-app').style.display = 'block';
    init();
  } else {
    document.getElementById('setup-screen').style.display = 'flex';
    document.getElementById('main-app').style.display = 'none';
  }
})();
</script>
</body>
</html>



