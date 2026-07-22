<!DOCTYPE html><html>
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
.alert-item{border-left:3px solid;padding:.9rem 1rem;border-radius:0 8px 8px 0;margin-bottom:.6rem;}
.alert-danger{border-color:var(--danger);background:#FF6B6B08;}
.alert-warn{border-color:var(--warn);background:#F59E0B08;}
.alert-ok{border-color:var(--accent);background:#0D2A1F22;}
.ticker-wrap{overflow:hidden;background:var(--surface);border-bottom:1px solid var(--border);padding:.45rem 0;}
.ticker-track{display:flex;white-space:nowrap;animation:ticker 50s linear infinite;}
.ticker-item{display:inline-flex;align-items:center;gap:.5rem;padding:0 2rem;font-size:.78rem;border-right:1px solid var(--border);}
.fx-card{background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:1rem 1.2rem;}
.logo{font-family:'Space Grotesk',sans-serif;font-weight:700;}
.logo span{color:var(--accent);}
.hidden{display:none !important;}
.kpi-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:1rem;margin-bottom:1.5rem;}
.fx-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.85rem;margin-bottom:2rem;}
.com-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.85rem;margin-bottom:2rem;}
@media(max-width:768px){.sidebar{display:none !important;}.kpi-grid{grid-template-columns:1fr 1fr !important;}.fx-grid,.com-grid{grid-template-columns:1fr 1fr !important;}}
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
<main style="flex:1;padding:2rem 2.25rem;overflow-y:auto;" id="mainContent"></main>
</div>
</div><script>
const API_BASE="https://contabilidad-de-datos.onrender.com";
const MONTHS=["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const COMPANIES=[{name:"El Clavo Hardware",sector:"Retail",plan:"Growth",seed:[210,195,230,215,240,225],currency:"ARS"},{name:"Soto Import SA",sector:"Import/Export",plan:"Scale",seed:[450,480,510,490,530,505],currency:"USD"},{name:"Dubois Consulting",sector:"Services",plan:"Starter",seed:[80,95,88,102,91,110],currency:"BRL"}];
const INVOICES=[{id:"INV-001",client:"BuildCo Ltd",amount:12400,date:"Jun 10",status:"paid"},{id:"INV-002",client:"Metro Supplies",amount:8750,date:"Jun 08",status:"pending"},{id:"INV-003",client:"Alpha Group",amount:31200,date:"Jun 05",status:"paid"},{id:"INV-004",client:"City Works",amount:5600,date:"Jun 01",status:"overdue"},{id:"INV-005",client:"TechParts SA",amount:19800,date:"May 28",status:"paid"},{id:"INV-006",client:"Harbor Imports",amount:44100,date:"May 25",status:"pending"}];
const ALERTS=[{type:"danger",title:"Cash flow risk detected",desc:"Projected cash drops below threshold in May.",time:"2h ago"},{type:"warn",title:"Invoice overdue — City Works",desc:"INV-004 for $5,600 is 14 days overdue.",time:"1d ago"},{type:"ok",title:"Monthly target reached",desc:"June inflow exceeded forecast by 12.4%.",time:"2d ago"},{type:"ok",title:"FX rate opportunity",desc:"Favorable USD spread — optimal for import orders.",time:"3d ago"}];
const NAV=[{id:"dashboard",icon:"⬡",label:"Dashboard"},{id:"cashflow",icon:"◎",label:"Cash Flow"},{id:"markets",icon:"◇",label:"FX & Markets"},{id:"invoices",icon:"◈",label:"Invoices"},{id:"alerts",icon:"△",label:"Alerts",badge:2}];
let currentUser=null,currentCompany=0,fxRates=null,commodities=null,currentPage="dashboard";
function fmt(n){if(n>=1e6)return"$"+(n/1e6).toFixed(1)+"M";if(n>=1000)return"$"+(n/1000).toFixed(0)+"K";return"$"+n;}
function handleLogin(){
const email=document.getElementById("loginEmail").value;
const pass=document.getElementById("loginPass").value;
const err=document.getElementById("loginError");
if(!email||!pass){err.textContent="Please fill in all fields.";err.style.display="block";return;}
err.style.display="none";
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Signing in…';btn.disabled=true;
setTimeout(()=>{currentUser={name:"Ezequiel Prilusky",email};document.getElementById("userName").textContent=currentUser.name;document.getElementById("loginScreen").classList.add("hidden");document.getElementById("appShell").classList.remove("hidden");document.getElementById("appShell").style.display="flex";renderNav();loadMarkets();renderPage("dashboard");},1100);
}
function handleLogout(){currentUser=null;document.getElementById("appShell").classList.add("hidden");document.getElementById("loginScreen").classList.remove("hidden");document.getElementById("loginEmail").value="";document.getElementById("loginPass").value="";document.getElementById("loginBtn").innerHTML="Sign in";document.getElementById("loginBtn").disabled=false;}
function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}
function renderNav(){const nav=document.getElementById("navMenu");nav.innerHTML=NAV.map(n=>`<div class="nav-item ${currentPage===n.id?'active':''}" onclick="renderPage('${n.id}')"><span>${n.icon}</span><span style="flex:1">${n.label}</span>${n.badge?`<span class="badge badge-red">${n.badge}</span>`:''}</div>`).join("");document.getElementById("userPlan").textContent=COMPANIES[currentCompany].plan+" plan";}
async function fetchForecast(values){const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values})});if(!res.ok)throw new Error("API "+res.status);return res.json();}
async function fetchFXRates(){const res=await fetch("https://api.frankfurter.dev/v1/latest?from=USD&to=EUR,ARS,BRL,MXN,COP,CLP,GBP,CNY");if(!res.ok)throw new Error("FX error");return(await res.json()).rates;}
async function fetchCommodities(){try{const[g,s]=await Promise.allSettled([fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xau.json").then(r=>r.json()),fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xag.json").then(r=>r.json())]);return{goldUSD:g.status==="fulfilled"?+(1/g.value.xau.usd).toFixed(2):null,silverUSD:s.status==="fulfilled"?+(1/s.value.xag.usd).toFixed(2):null};}catch(e){return{goldUSD:null,silverUSD:null};}}
async function loadMarkets(){try{fxRates=await fetchFXRates();}catch(e){fxRates=null;}try{commodities=await fetchCommodities();}catch(e){commodities=null;}renderTicker();if(currentPage==="markets"||currentPage==="dashboard")renderPage(currentPage);}
function renderTicker(){const track=document.getElementById("tickerTrack");if(!fxRates){track.innerHTML='<span class="ticker-item" style="color:var(--danger)">⚠ Live rates unavailable</span>';r