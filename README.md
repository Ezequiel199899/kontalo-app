<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Kontalo</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap');
:root{--bg:#0A0F1A;--surface:#111827;--border:#1E2D3D;--accent:#00E5A0;--accentDim:#0D2A1F;--accentHover:#00FFB3;--text:#E8EDF5;--muted:#8899AA;--faint:#4A5A6A;--danger:#FF6B6B;--warn:#F59E0B;}
*{box-sizing:border-box;margin:0;padding:0;}
body{background:var(--bg);color:var(--text);font-family:'Inter',system-ui,sans-serif;}
input,button,select{font-family:'Inter',sans-serif;}
@keyframes fadeIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
@keyframes spin{to{transform:rotate(360deg)}}
@keyframes ticker{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}
.fade-in{animation:fadeIn .3s ease forwards;}
.btn{border:none;border-radius:7px;cursor:pointer;font-weight:600;font-size:.9rem;display:inline-flex;align-items:center;gap:.5rem;}
.btn-primary{background:var(--accent);color:var(--bg);padding:.75rem 1.5rem;}
.btn-primary:hover{background:var(--accentHover);}
.btn-danger{background:transparent;color:var(--danger);border:1px solid #FF6B6B22;padding:.5rem 1rem;font-size:.8rem;}
.input{background:#0A0F1A;border:1px solid var(--border);border-radius:7px;color:var(--text);font-size:.9rem;padding:.75rem 1rem;width:100%;outline:none;}
.input:focus{border-color:var(--accent);}
.card{background:var(--surface);border:1px solid var(--border);border-radius:12px;}
.nav-item{color:var(--muted);font-size:.875rem;padding:.6rem .9rem;border-radius:7px;cursor:pointer;display:flex;align-items:center;gap:.6rem;}
.nav-item:hover{background:var(--accentDim);color:var(--accent);}
.nav-item.active{background:var(--accentDim);color:var(--accent);font-weight:600;}
.badge{font-size:.7rem;font-weight:700;padding:.2rem .55rem;border-radius:999px;}
.badge-green{background:var(--accentDim);color:var(--accent);}
.badge-red{background:#FF6B6B18;color:var(--danger);}
.badge-yellow{background:#F59E0B18;color:var(--warn);}
.spinner{width:18px;height:18px;border:2px solid var(--border);border-top-color:var(--accent);border-radius:50%;animation:spin .7s linear infinite;display:inline-block;}
.alert-item{border-left:3px solid;padding:.75rem 1rem;border-radius:0 8px 8px 0;margin-bottom:.5rem;}
.alert-danger{border-color:var(--danger);background:#FF6B6B08;}
.alert-warn{border-color:var(--warn);background:#F59E0B08;}
.alert-ok{border-color:var(--accent);background:#0D2A1F22;}
.ticker-wrap{overflow:hidden;background:var(--surface);border-bottom:1px solid var(--border);padding:.45rem 0;}
.ticker-track{display:flex;white-space:nowrap;animation:ticker 50s linear infinite;}
.ticker-item{display:inline-flex;align-items:center;gap:.5rem;padding:0 2rem;font-size:.78rem;border-right:1px solid var(--border);}
.fx-card{background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.85rem 1rem;}
.logo{font-family:'Space Grotesk',sans-serif;font-weight:700;}
.logo span{color:var(--accent);}
.hidden{display:none !important;}
.kpi-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.85rem;margin-bottom:1rem;}
.fx-mini-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.75rem;margin-bottom:1rem;}
.com-mini-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.75rem;margin-bottom:1rem;}
.bottom-grid{display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
.fx-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.85rem;margin-bottom:2rem;}
.com-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.85rem;margin-bottom:2rem;}
@media(max-width:768px){
.sidebar{display:none !important;}
.kpi-grid{grid-template-columns:1fr 1fr !important;}
.fx-mini-grid{grid-template-columns:1fr 1fr !important;}
.com-mini-grid{grid-template-columns:1fr 1fr !important;}
.bottom-grid{grid-template-columns:1fr !important;}
.fx-grid,.com-grid{grid-template-columns:1fr 1fr !important;}
}
</style>
</head>
<body>
<div id="loginScreen" style="min-height:100vh;display:flex;align-items:center;justify-content:center;padding:1.5rem;">
<div style="width:100%;max-width:400px;" class="fade-in">
<div style="text-align:center;margin-bottom:2.5rem;">
<span class="logo" style="font-size:1.9rem;"><span>kon</span>talo</span>
<p style="color:var(--muted);margin-top:.6rem;font-size:.9rem;">Sign in to your account</p>
</div>
<div class="card" style="padding:2rem;">
<div style="margin-bottom:1rem;"><label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Email</label><input class="input" id="loginEmail" type="email" placeholder="you@company.com"/></div>
<div style="margin-bottom:1.4rem;"><label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Password</label><input class="input" id="loginPass" type="password" placeholder="••••••••"/></div>
<div id="loginError" style="color:var(--danger);font-size:.8rem;margin-bottom:.9rem;display:none;"></div>
<button class="btn btn-primary" style="width:100%;justify-content:center;" onclick="handleLogin()" id="loginBtn">Sign in</button>
<p style="text-align:center;margin-top:1.1rem;font-size:.75rem;color:var(--faint);">Demo: any email + any password</p>
</div>
</div>
</div>
<div id="appShell" class="hidden" style="display:flex;min-height:100vh;">
<div class="sidebar" style="width:220px;background:var(--surface);border-right:1px solid var(--border);display:flex;flex-direction:column;height:100vh;position:sticky;top:0;flex-shrink:0;">
<div style="padding:1.4rem 1.2rem;border-bottom:1px solid var(--border);"><span class="logo" style="font-size:1.3rem;"><span>kon</span>talo</span></div>
<div style="padding:1rem 1.2rem;border-bottom:1px solid var(--border);">
<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.5rem;">Active company</div>
<select id="companySelect" class="input" style="padding:.5rem .7rem;font-size:.82rem;cursor:pointer;" onchange="changeCompany()">
<option value="0">El Clavo Hardware</option>
<option value="1">Soto Import SA</option>
<option value="2">Dubois Consulting</option>
</select>
</div>
<nav style="padding:.75rem;flex:1;" id="navMenu"></nav>
<div style="padding:1rem 1.2rem;border-top:1px solid var(--border);">
<div style="display:flex;align-items:center;gap:.6rem;margin-bottom:.75rem;">
<div style="width:32px;height:32px;border-radius:50%;background:var(--accentDim);display:flex;align-items:center;justify-content:center;font-size:.7rem;font-weight:700;color:var(--accent);" id="userAvatar">EP</div>
<div><div style="font-size:.8rem;font-weight:600;" id="userName">—</div><div style="font-size:.68rem;color:var(--faint);" id="userPlan">—</div></div>
</div>
<button class="btn btn-danger" style="width:100%;" onclick="handleLogout()">Sign out</button>
</div>
</div>
<div style="flex:1;display:flex;flex-direction:column;">
<div class="ticker-wrap"><div class="ticker-track" id="tickerTrack"><span class="ticker-item"><span class="spinner"></span> Loading live market rates…</span></div></div>
<main style="flex:1;padding:1.5rem 2rem;overflow-y:auto;" id="mainContent"></main>
</div>
</div><script>
const AV_KEY="42ac3700f76949f6899936a3eb2ad9d2";
const API_BASE="https://contabilidad-de-datos.onrender.com";
const MONTHS=["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const COMPANIES=[{name:"El Clavo Hardware",sector:"Retail",plan:"Growth",seed:[210,195,230,215,240,225],currency:"ARS"},{name:"Soto Import SA",sector:"Import/Export",plan:"Scale",seed:[450,480,510,490,530,505],currency:"USD"},{name:"Dubois Consulting",sector:"Services",plan:"Starter",seed:[80,95,88,102,91,110],currency:"BRL"}];
const INVOICES=[{id:"INV-001",client:"BuildCo Ltd",amount:12400,date:"Jun 10",status:"paid"},{id:"INV-002",client:"Metro Supplies",amount:8750,date:"Jun 08",status:"pending"},{id:"INV-003",client:"Alpha Group",amount:31200,date:"Jun 05",status:"paid"},{id:"INV-004",client:"City Works",amount:5600,date:"Jun 01",status:"overdue"},{id:"INV-005",client:"TechParts SA",amount:19800,date:"May 28",status:"paid"},{id:"INV-006",client:"Harbor Imports",amount:44100,date:"May 25",status:"pending"}];
const ALERTS=[{type:"danger",title:"Cash flow risk detected",desc:"Projected cash drops below threshold in May.",time:"2h ago"},{type:"warn",title:"Invoice overdue — City Works",desc:"INV-004 for $5,600 is 14 days overdue.",time:"1d ago"},{type:"ok",title:"Monthly target reached",desc:"June inflow exceeded forecast by 12.4%.",time:"2d ago"},{type:"ok",title:"FX rate opportunity",desc:"Favorable USD spread — optimal for import orders.",time:"3d ago"}];
const NAV=[{id:"dashboard",icon:"⬡",label:"Dashboard"},{id:"markets",icon:"◇",label:"FX & Markets"},{id:"invoices",icon:"◈",label:"Invoices"},{id:"alerts",icon:"△",label:"Alerts",badge:2},{id:"cashflow",icon:"◎",label:"Cash Flow"}];

const FALLBACK={EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",ARS_OFICIAL:"1497",ARS_BLUE:"1525",CLP:"945",COP:"3196",gold:"4,068",silver:"58.88",oil:"73.40",soy:"10.82",wheat:"5.64",corn:"4.38"};

let currentUser=null,currentCompany=0,currentPage="dashboard";
let mktData={rates:{},gold:null,silver:null};

function fmt(n){if(n>=1e6)return"$"+(n/1e6).toFixed(1)+"M";if(n>=1000)return"$"+(n/1000).toFixed(0)+"K";return"$"+n;}
function getRate(c){return mktData.rates[c]||FALLBACK[c]||"—";}
function getGold(){return mktData.gold||FALLBACK.gold;}
function getSilver(){return mktData.silver||FALLBACK.silver;}

async function loadMarkets(){
try{
const pairs=["EUR","BRL","MXN","GBP","CNY"];
const results=await Promise.allSettled(pairs.map(function(p){return fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency="+p+"&apikey="+AV_KEY).then(function(r){return r.json();});}));
results.forEach(function(r,i){if(r.status==="fulfilled"&&r.value["Realtime Currency Exchange Rate"]){mktData.rates[pairs[i]]=parseFloat(r.value["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(4);}else{mktData.rates[pairs[i]]=FALLBACK[pairs[i]];}});
}catch(e){}
try{const g=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(g["Realtime Currency Exchange Rate"]){mktData.gold=parseFloat(g["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toLocaleString("en-US",{maximumFractionDigits:2});}else{mktData.gold=FALLBACK.gold;}}catch(e){mktData.gold=FALLBACK.gold;}
try{const s=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAG&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(s["Realtime Currency Exchange Rate"]){mktData.silver=parseFloat(s["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(2);}else{mktData.silver=FALLBACK.silver;}}catch(e){mktData.silver=FALLBACK.silver;}
renderTicker();
if(currentPage==="dashboard"||currentPage==="markets")renderPage(currentPage);
}

function renderTicker(){
const track=document.getElementById("tickerTrack");
const items=[
{label:"USD/ARS oficial",val:FALLBACK.ARS_OFICIAL,up:false},
{label:"USD/ARS blue",val:FALLBACK.ARS_BLUE,up:false},
{label:"EUR/USD",val:(1/parseFloat(getRate("EUR"))).toFixed(4),up:true},
{label:"USD/BRL",val:getRate("BRL"),up:false},
{label:"USD/MXN",val:getRate("MXN"),up:false},
{label:"USD/CLP",val:FALLBACK.CLP,up:false},
{label:"USD/COP",val:FALLBACK.COP,up:false},
{label:"GBP/USD",val:(1/parseFloat(getRate("GBP"))).toFixed(4),up:true},
{label:"USD/CNY",val:getRate("CNY"),up:false},
{label:"GOLD oz",val:"$"+getGold(),up:true},
{label:"SILVER oz",val:"$"+getSilver(),up:true},
{label:"CRUDE OIL",val:"$"+FALLBACK.oil,up:true},
{label:"SOYBEANS",val:"$"+FALLBACK.soy,up:false},
{label:"WHEAT",val:"$"+FALLBACK.wheat,up:true},
{label:"CORN",val:"$"+FALLBACK.corn,up:false},
];
const all=items.concat(items);
track.innerHTML=all.map(function(i){return'<span class="ticker-item"><span style="color:var(--faint)">'+i.label+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-weight:600">'+i.val+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-size:.65rem">'+(i.up?'▲':'▼')+'</span></span>';}).join("");
}

function handleLogin(){
const email=document.getElementById("loginEmail").value;
const pass=document.getElementById("loginPass").value;
const err=document.getElementById("loginError");
if(!email||!pass){err.textContent="Please fill in all fields.";err.style.display="block";return;}
err.style.display="none";
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Signing in...';btn.disabled=true;
setTimeout(function(){
currentUser={name:"Ezequiel Prilusky",email:email};
document.getElementById("userName").textContent=currentUser.name;
document.getElementById("loginScreen").classList.add("hidden");
document.getElementById("appShell").classList.remove("hidden");
document.getElementById("appShell").style.display="flex";
renderTicker();renderNav();renderPage("dashboard");loadMarkets();
},1100);
}
function handleLogout(){currentUser=null;document.getElementById("appShell").classList.add("hidden");document.getElementById("loginScreen").classList.remove("hidden");document.getElementById("loginEmail").value="";document.getElementById("loginPass").value="";document.getElementById("loginBtn").innerHTML="Sign in";document.getElementById("loginBtn").disabled=false;}
function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}
function renderNav(){const nav=document.getElementById("navMenu");nav.innerHTML=NAV.map(function(n){return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')"><span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+'</div>';}).join("");document.getElementById("userPlan").textContent=COMPANIES[currentCompany].plan+" plan";}
function renderPage(page){currentPage=page;renderNav();const company=COMPANIES[currentCompany];const main=document.getElementById("mainContent");if(page==="dashboard")renderDashboard(main,company);else if(page==="cashflow")renderCashflow(main,company);else if(page==="markets")renderMarkets(main);else if(page==="invoices")renderInvoices(main);else if(page==="alerts")renderAlerts(main);}

async function renderDashboard(main,company){
const latest=company.seed[5]*1000,prev=company.seed[4]*1000;
const delta=(((latest-prev)/prev)*100).toFixed(1);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.25rem;">Dashboard</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.25rem;">'+company.name+' · '+company.sector+'</p>'+

// KPIs
'<div class="kpi-grid">'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Cash flow</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(latest)+'</div><div style="font-size:.75rem;color:'+(+delta>0?'var(--accent)':'var(--danger)')+'">'+( +delta>0?'▲':'▼')+' '+delta+'% vs last month</div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiProjection"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">AI projection</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiAvg"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Avg monthly</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Active alerts</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">2</div><div style="font-size:.75rem;color:var(--danger)">▼ 1 critical</div></div>'+
'</div>'+

// FX mini
'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">💱 Live FX Rates</div>'+
'<div class="fx-mini-grid">'+
[{label:"USD/ARS blue",val:FALLBACK.ARS_BLUE,up:false},{label:"USD/BRL",val:getRate("BRL"),up:false},{label:"EUR/USD",val:(1/parseFloat(getRate("EUR"))).toFixed(4),up:true},{label:"USD/CLP",val:FALLBACK.CLP,up:false}].map(function(r){return'<div class="card" style="padding:.75rem 1rem;display:flex;justify-content:space-between;align-items:center"><span style="font-size:.75rem;color:var(--muted)">'+r.label+'</span><span style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:.95rem;color:'+(r.up?'var(--accent)':'var(--text)')+'">'+r.val+'</span></div>';}).join('')+
'</div>'+

// Commodities mini
'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">🌾 Commodities</div>'+
'<div class="com-mini-grid">'+
[{label:"🥇 Gold",val:"$"+getGold()},{label:"🥈 Silver",val:"$"+getSilver()},{label:"🌾 Soybeans",val:"$"+FALLBACK.soy}].map(function(r){return'<div class="card" style="padding:.75rem 1rem;display:flex;justify-content:space-between;align-items:center"><span style="font-size:.75rem;color:var(--muted)">'+r.label+'</span><span style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:.95rem;color:var(--accent)">'+r.val+'</span></div>';}).join('')+
'</div>'+

// Bottom
'<div id="apiStatus"></div>'+
'<div class="bottom-grid">'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Recent invoices</div>'+INVOICES.slice(0,4).map(function(inv){return'<div style="display:flex;justify-content:space-between;padding:.5rem 0;border-bottom:1px solid var(--border)"><div><div style="font-size:.82rem">'+inv.client+'</div><div style="font-size:.7rem;color:var(--faint)">'+inv.id+' · '+inv.date+'</div></div><div style="text-align:right"><div style="font-size:.85rem;font-weight:600">$'+inv.amount.toLocaleString()+'</div><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+inv.status+'</span></div></div>';}).join('')+'</div>'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Active alerts</div>'+ALERTS.map(function(a){return'<div class="alert-item alert-'+a.type+'"><div style="font-size:.8rem;font-weight:600;margin-bottom:.15rem">'+a.title+'</div><div style="font-size:.72rem;color:var(--muted)">'+a.desc+'</div><div style="font-size:.67rem;color:var(--faint);margin-top:.2rem">'+a.time+'</div></div>';}).join('')+'</div>'+
'</div></div>';

try{const forecast=await fetchForecast(company.seed);
document.getElementById("kpiProjection").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">AI projection</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Trend '+forecast.tendencia.toFixed(1)+'K/mo</div>';
document.getElementById("kpiAvg").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Avg monthly</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(Math.round(forecast.promedio*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ From live API</div>';
}catch(e){document.getElementById("apiStatus").innerHTML='<div class="card" style="padding:.9rem;margin-bottom:1rem;color:var(--danger);font-size:.82rem;border-color:#FF6B6B44">⚠ '+e.message+' — Render cold-starting. Reload in 30s.</div>';}
}

async function fetchForecast(values){const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values:values})});if(!res.ok)throw new Error("API "+res.status);return res.json();}

async function renderCashflow(main,company){
main.innerHTML='<div class="fade-in"><h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">Cash Flow</h1><div id="cfContent"><span class="spinner"></span> Fetching from your API...</div></div>';
try{const forecast=await fetchForecast(company.seed);
document.getElementById("cfContent").innerHTML=
'<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:1rem;margin-bottom:1.25rem">'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">6-month average</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.promedio*1000))+'</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Monthly trend</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+(forecast.tendencia>0?'+':'')+forecast.tendencia.toFixed(1)+'K</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Next projection</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div></div>'+
'</div>'+
'<div class="card" style="padding:1.25rem"><div style="font-weight:600;margin-bottom:1rem">Monthly breakdown</div><table style="width:100%;border-collapse:collapse;font-size:.85rem"><thead><tr style="color:var(--faint);font-size:.72rem;text-transform:uppercase"><th style="text-align:left;padding:.5rem .75rem">Month</th><th style="text-align:left;padding:.5rem .75rem">Actual</th><th style="text-align:left;padding:.5rem .75rem">Projected</th><th style="text-align:left;padding:.5rem .75rem">Status</th></tr></thead><tbody>'+
MONTHS.slice(0,6).map(function(m,i){return'<tr style="border-top:1px solid var(--border)"><td style="padding:.65rem .75rem;font-weight:500">'+m+'</td><td style="padding:.65rem .75rem">'+fmt(company.seed[i]*1000)+'</td><td style="padding:.65rem .75rem;color:var(--muted)">'+fmt(Math.round((company.seed[5]+forecast.tendencia*(i-5))*1000))+'</td><td style="padding:.65rem .75rem"><span class="badge badge-green">On track</span></td></tr>';}).join('')+
'</tbody></table></div>';
}catch(e){document.getElementById("cfContent").innerHTML='<div style="color:var(--danger)">⚠ '+e.message+' — Render cold-starting. Reload in 30s.</div>';}
}

function renderMarkets(main){
const currencies=[
{code:"ARS",flag:"🇦🇷",name:"Argentine Peso (oficial)",rate:FALLBACK.ARS_OFICIAL+" ARS"},
{code:"ARS",flag:"🇦🇷",name:"Argentine Peso (blue)",rate:FALLBACK.ARS_BLUE+" ARS"},
{code:"EUR",flag:"🇪🇺",name:"Euro",rate:getRate("EUR")+" EUR"},
{code:"BRL",flag:"🇧🇷",name:"Brazilian Real",rate:getRate("BRL")+" BRL"},
{code:"MXN",flag:"🇲🇽",name:"Mexican Peso",rate:getRate("MXN")+" MXN"},
{code:"COP",flag:"🇨🇴",name:"Colombian Peso",rate:FALLBACK.COP+" COP"},
{code:"CLP",flag:"🇨🇱",name:"Chilean Peso",rate:FALLBACK.CLP+" CLP"},
{code:"GBP",flag:"🇬🇧",name:"British Pound",rate:getRate("GBP")+" GBP"},
];
const comCards=[
{name:"Gold",unit:"troy oz",icon:"🥇",val:"$"+getGold()},
{name:"Silver",unit:"troy oz",icon:"🥈",val:"$"+getSilver()},
{name:"Crude Oil",unit:"barrel",icon:"🛢️",val:"$"+FALLBACK.oil},
{name:"Soybeans",unit:"bushel",icon:"🌾",val:"$"+FALLBACK.soy},
{name:"Wheat",unit:"bushel",icon:"🌿",val:"$"+FALLBACK.wheat},
{name:"Corn",unit:"bushel",icon:"🌽",val:"$"+FALLBACK.corn},
];
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">FX & Markets</h1>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Currency Pairs <span style="color:var(--accent);font-size:.78rem;font-weight:400">vs USD · Alpha Vantage + Reference</span></div>'+
'<div class="fx-grid">'+currencies.map(function(c){return'<div class="fx-card"><div style="display:flex;justify-content:space-between;margin-bottom:.5rem"><span style="font-size:1.3rem">'+c.flag+'</span><span class="badge badge-green">'+c.code+'</span></div><div style="font-size:.72rem;color:var(--faint);margin-bottom:.2rem">'+c.name+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.1rem">'+c.rate+'</div></div>';}).join('')+'</div>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Commodities</div>'+
'<div class="com-grid">'+comCards.map(function(c){return'<div class="fx-card"><div style="font-size:1.4rem;margin-bottom:.4rem">'+c.icon+'</div><div style="font-size:.72rem;color:var(--faint);margin-bottom:.2rem">'+c.name+' / '+c.unit+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.25rem;color:var(--accent)">'+c.val+'</div></div>';}).join('')+'</div>'+
'</div>';
}

function renderInvoices(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">Invoices</h1>'+
'<div class="card" style="overflow:hidden">'+
'<table style="width:100%;border-collapse:collapse;font-size:.875rem">'+
'<thead style="background:var(--bg)"><tr style="color:var(--faint);font-size:.72rem;text-transform:uppercase">'+
'<th style="text-align:left;padding:.75rem 1.25rem">Invoice</th>'+
'<th style="text-align:left;padding:.75rem 1.25rem">Client</th>'+
'<th style="text-align:left;padding:.75rem 1.25rem">Amount</th>'+
'<th style="text-align:left;padding:.75rem 1.25rem">Date</th>'+
'<th style="text-align:left;padding:.75rem 1.25rem">Status</th>'+
'</tr></thead>'+
'<tbody>'+INVOICES.map(function(inv){return'<tr style="border-top:1px solid var(--border)"><td style="padding:.9rem 1.25rem;font-weight:600;color:var(--accent)">'+inv.id+'</td><td style="padding:.9rem 1.25rem">'+inv.client+'</td><td style="padding:.9rem 1.25rem;font-weight:600">$'+inv.amount.toLocaleString()+'</td><td style="padding:.9rem 1.25rem;color:var(--muted)">'+inv.date+'</td><td style="padding:.9rem 1.25rem"><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+inv.status+'</span></td></tr>';}).join('')+
'</tbody></table></div></div>';
}

function renderAlerts(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">Alerts</h1>'+
'<div style="max-width:680px">'+ALERTS.map(function(a){return'<div class="alert-item alert-'+a.type+' card" style="margin-bottom:.85rem;padding:1.1rem 1.25rem"><div style="display:flex;justify-content:space-between"><div style="font-size:.9rem;font-weight:600;margin-bottom:.3rem">'+a.title+'</div><span style="font-size:.7rem;color:var(--faint)">'+a.time+'</span></div><di
 