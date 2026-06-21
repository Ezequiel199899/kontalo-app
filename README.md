<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kontalo</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@700&family=Inter:wght@400;600&display=swap');
:root{--bg:#0A0F1A;--surface:#111827;--border:#1E2D3D;--accent:#00E5A0;--accentDim:#0D2A1F;--text:#E8EDF5;--muted:#8899AA;--faint:#4A5A6A;--danger:#FF6B6B;}
*{box-sizing:border-box;margin:0;padding:0;}
body{background:var(--bg);color:var(--text);font-family:'Inter',sans-serif;}
input,button{font-family:'Inter',sans-serif;}
@keyframes spin{to{transform:rotate(360deg)}}
@keyframes ticker{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}
.hidden{display:none!important;}
.spinner{width:16px;height:16px;border:2px solid var(--border);border-top-color:var(--accent);border-radius:50%;animation:spin .7s linear infinite;display:inline-block;}
.card{background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:1.25rem;}
.input{background:#0A0F1A;border:1px solid var(--border);border-radius:7px;color:var(--text);font-size:.9rem;padding:.75rem 1rem;width:100%;outline:none;margin-bottom:1rem;}
.btn{background:var(--accent);color:var(--bg);border:none;border-radius:7px;padding:.75rem 1.5rem;font-weight:600;cursor:pointer;width:100%;}
.logo{font-family:'Space Grotesk',sans-serif;font-weight:700;font-size:1.8rem;}
.logo span{color:var(--accent);}
.ticker-wrap{overflow:hidden;background:var(--surface);border-bottom:1px solid var(--border);padding:.5rem 0;}
.ticker-track{display:flex;white-space:nowrap;animation:ticker 40s linear infinite;}
.ticker-item{display:inline-flex;gap:.5rem;padding:0 2rem;font-size:.8rem;border-right:1px solid var(--border);}
.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:1rem;margin:1.5rem 0;}
.pipeline-step{display:flex;align-items:center;gap:.75rem;padding:.75rem;border-left:2px solid var(--accent);margin-bottom:.5rem;background:var(--accentDim);border-radius:0 8px 8px 0;}
@media(max-width:768px){.grid{grid-template-columns:1fr;}}
</style>
</head>
<body>
<div id="login" style="min-height:100vh;display:flex;align-items:center;justify-content:center;padding:1.5rem;">
<div style="width:100%;max-width:380px;">
<div style="text-align:center;margin-bottom:2rem;"><span class="logo"><span>kon</span>talo</span></div>
<div class="card">
<input class="input" id="email" placeholder="Email">
<input class="input" id="pass" type="password" placeholder="Password">
<button class="btn" onclick="login()">Sign in</button>
</div>
</div>
</div>
<div id="app" class="hidden">
<div class="ticker-wrap"><div class="ticker-track" id="ticker"><span class="ticker-item"><span class="spinner"></span> Loading…</span></div></div>
<div style="padding:2rem;max-width:1000px;margin:0 auto;">
<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:1.5rem;">
<span class="logo" style="font-size:1.3rem"><span>kon</span>talo</span>
<button onclick="logout()" style="background:none;border:1px solid var(--border);color:var(--danger);padding:.5rem 1rem;border-radius:6px;cursor:pointer;">Sign out</button>
</div>
<div id="content"></div>
</div>
</div>. <script>
const API_BASE="https://contabilidad-de-datos.onrender.com";
const seed=[210,195,230,215,240,225];
let fx=null,com=null;

function login(){
if(!document.getElementById("email").value){alert("Enter email");return;}
document.getElementById("login").classList.add("hidden");
document.getElementById("app").classList.remove("hidden");
loadAll();
}
function logout(){
document.getElementById("app").classList.add("hidden");
document.getElementById("login").classList.remove("hidden");
}

function fmt(n){return n>=1e6?"$"+(n/1e6).toFixed(1)+"M":n>=1000?"$"+(n/1000).toFixed(0)+"K":"$"+n;}

async function fetchForecast(){
const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values:seed})});
if(!res.ok)throw new Error("API "+res.status);
return res.json();
}
async function fetchFX(){
const res=await fetch("https://api.frankfurter.dev/v1/latest?from=USD&to=EUR,ARS,BRL,MXN,GBP");
return (await res.json()).rates;
}
async function fetchCom(){
try{
const[g,s]=await Promise.allSettled([
fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xau.json").then(r=>r.json()),
fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xag.json").then(r=>r.json()),
]);
return{gold:g.status==="fulfilled"?+(1/g.value.xau.usd).toFixed(2):null,silver:s.status==="fulfilled"?+(1/s.value.xag.usd).toFixed(2):null};
}catch(e){return{gold:null,silver:null};}
}

function renderTicker(){
const t=document.getElementById("ticker");
if(!fx){t.innerHTML='<span class="ticker-item" style="color:var(--danger)">⚠ Rates unavailable</span>';return;}
const items=[
{l:"EUR/USD",v:(1/fx.EUR).toFixed(4)},
{l:"USD/ARS",v:fx.ARS.toFixed(2)},
{l:"USD/BRL",v:fx.BRL.toFixed(4)},
{l:"USD/MXN",v:fx.MXN.toFixed(4)},
{l:"GBP/USD",v:(1/fx.GBP).toFixed(4)},
com?.gold?{l:"GOLD",v:"$"+com.gold.toLocaleString()}:null,
com?.silver?{l:"SILVER",v:"$"+com.silver.toFixed(2)}:null,
].filter(Boolean);
const all=[...items,...items];
t.innerHTML=all.map(i=>`<span class="ticker-item"><span style="color:var(--faint)">${i.l}</span><span style="color:var(--accent);font-weight:600">${i.v}</span></span>`).join("");
}

async function loadAll(){
const c=document.getElementById("content");
c.innerHTML='<div class="card"><span class="spinner"></span> Loading dashboard…</div>';
try{
fx=await fetchFX();
com=await fetchCom();
renderTicker();
}catch(e){renderTicker();}

let forecastHTML='<span class="spinner"></span>';
try{
const f=await fetchForecast();
forecastHTML=`
<div class="grid">
<div class="card"><div style="color:var(--faint);font-size:.75rem;text-transform:uppercase;margin-bottom:.5rem">6-month avg</div><div style="font-family:'Space Grotesk',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">${fmt(Math.round(f.promedio*1000))}</div></div>
<div class="card"><div style="color:var(--faint);font-size:.75rem;text-transform:uppercase;margin-bottom:.5rem">Monthly trend</div><div style="font-family:'Space Grotesk',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">${f.tendencia>0?"+":""}${f.tendencia.toFixed(1)}K</div></div>
<div class="card"><div style="color:var(--faint);font-size:.75rem;text-transform:uppercase;margin-bottom:.5rem">Next projection</div><div style="font-family:'Space Grotesk',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">${fmt(Math.round(f.proyeccion*1000))}</div></div>
</div>`;
}catch(e){
forecastHTML=`<div class="card" style="color:var(--danger)">⚠ ${e.message} — Render cold-starting, reload in 30s.</div>`;
}

c.innerHTML=`
<h2 style="font-family:'Space Grotesk',sans-serif;margin-bottom:.3rem">Dashboard</h2>
<p style="color:var(--muted);font-size:.85rem;margin-bottom:1rem">El Clavo Hardware · Cash flow projection</p>
${forecastHTML}
<h2 style="font-family:'Space Grotesk',sans-serif;margin:2rem 0 1rem">Data Pipeline</h2>
<p style="color:var(--muted);font-size:.85rem;margin-bottom:1rem">How your data flows through Kontalo's architecture</p>
<div class="pipeline-step"><span>📥</span> <div><b>Ingest</b> — Invoice & transaction data collected</div></div>
<div class="pipeline-step"><span>☁️</span> <div><b>Azure Data Lake</b> — Raw data stored for processing</div></div>
<div class="pipeline-step"><span>⚙️</span> <div><b>Flask ML Service</b> — Statistical forecasting model runs</div></div>
<div class="pipeline-step"><span>📊</span> <div><b>API Layer</b> — Spring Boot exposes clean results</div></div>
<div class="pipeline-step"><span>🟢</span> <div><b>Dashboard</b> — You see live projections, right here</div></div>
`;
}
</script>
</body>
</html>.     