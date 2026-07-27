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
.com-mini-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.75rem;margin-bottom:1rem;}
.bottom-grid{display:grid;grid-template-columns:1fr 1fr;gap:1rem;}
.fx-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.85rem;margin-bottom:2rem;}
.com-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.85rem;margin-bottom:2rem;}
.fx-table-row{display:grid;grid-template-columns:1.5fr 1fr 1fr 0.7fr;align-items:center;padding:.65rem .85rem;border-bottom:1px solid var(--border);}
.fx-table-row:last-child{border-bottom:none;}
.tab-btn{padding:.5rem 1.1rem;border-radius:6px;font-size:.85rem;cursor:pointer;color:var(--muted);background:transparent;border:none;font-weight:500;}
.tab-btn.active{background:var(--accentDim);color:var(--accent);font-weight:600;}
@media(max-width:768px){
.sidebar{display:none !important;}
.kpi-grid{grid-template-columns:1fr 1fr !important;}
.com-mini-grid{grid-template-columns:1fr 1fr !important;}
.bottom-grid{grid-template-columns:1fr !important;}
.fx-grid,.com-grid{grid-template-columns:1fr 1fr !important;}
}
</style>
</head>
<body>
<div id="loginScreen" style="min-height:100vh;display:flex;align-items:center;justify-content:center;padding:1.5rem;">
<div style="width:100%;max-width:420px;" class="fade-in">
<div style="text-align:center;margin-bottom:2.5rem;">
<span class="logo" style="font-size:2rem;"><span>kon</span>talo</span>
<p style="color:var(--muted);margin-top:.6rem;font-size:.95rem;">Tu plataforma financiera inteligente</p>
</div>
<div class="card" style="padding:2.5rem;text-align:center;">
<p style="color:var(--muted);font-size:.9rem;margin-bottom:1.75rem;">Panel de demostración — acceso libre</p>
<button class="btn btn-primary" style="width:100%;justify-content:center;font-size:1.05rem;padding:1rem;" onclick="handleLogin()" id="loginBtn">
Ingresar a Kontalo →
</button>
<p style="margin-top:1rem;font-size:.75rem;color:var(--faint);">Sin registro ni contraseña</p>
</div>
</div>
</div>
<div id="appShell" class="hidden" style="display:flex;min-height:100vh;">
<div class="sidebar" style="width:225px;background:var(--surface);border-right:1px solid var(--border);display:flex;flex-direction:column;height:100vh;position:sticky;top:0;flex-shrink:0;">
<div style="padding:1.4rem 1.2rem;border-bottom:1px solid var(--border);"><span class="logo" style="font-size:1.3rem;"><span>kon</span>talo</span></div>
<div style="padding:1rem 1.2rem;border-bottom:1px solid var(--border);">
<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.5rem;">Empresa activa</div>
<select id="companySelect" class="input" style="padding:.5rem .7rem;font-size:.82rem;cursor:pointer;" onchange="changeCompany()">
<option value="0">El Clavo Ferretería</option>
<option value="1">Soto Import SA</option>
<option value="2">Dubois Consulting</option>
</select>
</div>
<nav style="padding:.75rem;flex:1;" id="navMenu"></nav>
<div style="padding:1rem 1.2rem;border-top:1px solid var(--border);">
<div style="display:flex;align-items:center;gap:.6rem;margin-bottom:.75rem;">
<div style="width:32px;height:32px;border-radius:50%;background:var(--accentDim);display:flex;align-items:center;justify-content:center;font-size:.75rem;font-weight:700;color:var(--accent);">K</div>
<div><div style="font-size:.8rem;font-weight:600;color:var(--text);">Kontalo Demo</div><div style="font-size:.68rem;color:var(--faint);">Plan Growth</div></div>
</div>
<button class="btn btn-danger" style="width:100%;" onclick="handleLogout()">Salir</button>
</div>
</div>
<div style="flex:1;display:flex;flex-direction:column;">
<div class="ticker-wrap"><div class="ticker-track" id="tickerTrack"><span class="ticker-item"><span class="spinner"></span> Cargando cotizaciones…</span></div></div>
<main style="flex:1;padding:1.5rem 2rem;overflow-y:auto;" id="mainContent"></main>
</div>
</div><script>
const AV_KEY="42ac3700f76949f6899936a3eb2ad9d2";
const API_BASE="https://contabilidad-de-datos.onrender.com";
const MONTHS=["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
const COMPANIES=[
{name:"El Clavo Ferretería",sector:"Retail",plan:"Growth",seed:[210,195,230,215,240,225],currency:"ARS"},
{name:"Soto Import SA",sector:"Importación",plan:"Scale",seed:[450,480,510,490,530,505],currency:"USD"},
{name:"Dubois Consulting",sector:"Servicios",plan:"Starter",seed:[80,95,88,102,91,110],currency:"BRL"}
];
const INVOICES=[
{id:"FAC-001",client:"BuildCo Ltd",amount:12400,date:"10 Jun",status:"paid",items:3},
{id:"FAC-002",client:"Suministros Metro",amount:8750,date:"08 Jun",status:"pending",items:5},
{id:"FAC-003",client:"Grupo Alfa",amount:31200,date:"05 Jun",status:"paid",items:8},
{id:"FAC-004",client:"Obras Municipales",amount:5600,date:"01 Jun",status:"overdue",items:2},
{id:"FAC-005",client:"TechParts SA",amount:19800,date:"28 May",status:"paid",items:6},
{id:"FAC-006",client:"Harbor Imports",amount:44100,date:"25 May",status:"pending",items:4}
];
const STOCK=[
{codigo:"FER-001",producto:"Tornillos 1/2 pulgada x100",stock:450,minimo:100,precio:2800,estado:"ok"},
{codigo:"FER-002",producto:"Pintura látex blanco 4L",stock:28,minimo:30,precio:15400,estado:"low"},
{codigo:"FER-003",producto:"Cable eléctrico 2.5mm x100m",stock:12,minimo:20,precio:48000,estado:"low"},
{codigo:"FER-004",producto:"Cemento Portland 50kg",stock:95,minimo:50,precio:8900,estado:"ok"},
{codigo:"FER-005",producto:"Caño PVC 3 pulgada x6m",stock:5,minimo:15,precio:12500,estado:"critical"},
{codigo:"FER-006",producto:"Llave inglesa 12 pulgada",stock:33,minimo:20,precio:9800,estado:"ok"},
{codigo:"FER-007",producto:"Discos de corte 115mm x10",stock:8,minimo:25,precio:6500,estado:"critical"},
{codigo:"FER-008",producto:"Silicona transparente 280ml",stock:62,minimo:30,precio:3200,estado:"ok"}
];
const CIERRES=[
{fecha:"25 Jul 2026",apertura:150000,ventas:384500,gastos:89200,cierre:445300,diferencia:+300,estado:"ok"},
{fecha:"24 Jul 2026",apertura:148000,ventas:412000,gastos:95000,cierre:465000,diferencia:-200,estado:"warn"},
{fecha:"23 Jul 2026",apertura:145000,ventas:298000,gastos:78000,cierre:365000,diferencia:0,estado:"ok"},
{fecha:"22 Jul 2026",apertura:140000,ventas:520000,gastos:110000,cierre:550000,diferencia:+500,estado:"ok"},
{fecha:"21 Jul 2026",apertura:138000,ventas:189000,gastos:65000,cierre:262000,diferencia:-800,estado:"danger"}
];
const ALERTS=[
{type:"danger",title:"Riesgo de flujo de caja",desc:"Se proyecta caída por debajo del umbral en mayo.",time:"Hace 2h"},
{type:"warn",title:"Factura vencida — Obras Municipales",desc:"FAC-004 por $5.600 tiene 14 días de atraso.",time:"Hace 1d"},
{type:"warn",title:"Stock crítico — Caño PVC",desc:"Solo 5 unidades. Mínimo recomendado: 15.",time:"Hace 3h"},
{type:"ok",title:"Objetivo mensual alcanzado",desc:"El ingreso de junio superó la proyección un 12,4%.",time:"Hace 2d"},
{type:"ok",title:"Oportunidad en tipo de cambio",desc:"Diferencial favorable en USD para pedidos de importación.",time:"Hace 3d"}
];
const NAV=[
{id:"dashboard",icon:"⬡",label:"Panel"},
{id:"markets",icon:"◇",label:"FX & Mercados"},
{id:"invoices",icon:"◈",label:"Facturas"},
{id:"stock",icon:"📦",label:"Stock"},
{id:"cierre",icon:"🔒",label:"Cierre de Caja"},
{id:"alerts",icon:"△",label:"Alertas",badge:3},
{id:"cashflow",icon:"◎",label:"Flujo de Caja"}
];
const FALLBACK={
EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",
ARS_BNA_COMPRA:"1470",ARS_BNA_VENTA:"1520",
ARS_BLUE_COMPRA:"1525",ARS_BLUE_VENTA:"1545",
ARS_MEP:"1531",CLP:"945",COP:"3196",
gold:"4.068",silver:"58.88",oil:"73.40",soy:"281",wheat:"5.64",corn:"281"
};
let currentCompany=0,currentPage="dashboard";
let mktData={rates:{},gold:null,silver:null};
function fmt(n){if(n>=1e6)return"$"+(n/1e6).toFixed(1)+"M";if(n>=1000)return"$"+(n/1000).toFixed(0)+"K";return"$"+n;}
function fmtNum(n){return n.toLocaleString("es-AR");}
function getRate(c){return mktData.rates[c]||FALLBACK[c]||"—";}
function getGold(){return mktData.gold||FALLBACK.gold;}
function getSilver(){return mktData.silver||FALLBACK.silver;}

async function loadMarkets(){
try{
const pairs=["EUR","BRL","MXN","GBP","CNY"];
const results=await Promise.allSettled(pairs.map(function(p){return fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency="+p+"&apikey="+AV_KEY).then(function(r){return r.json();});}));
results.forEach(function(r,i){if(r.status==="fulfilled"&&r.value["Realtime Currency Exchange Rate"]){mktData.rates[pairs[i]]=parseFloat(r.value["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(4);}else{mktData.rates[pairs[i]]=FALLBACK[pairs[i]];}});
}catch(e){}
try{const g=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(g["Realtime Currency Exchange Rate"]){mktData.gold=parseFloat(g["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toLocaleString("es-AR",{maximumFractionDigits:2});}else{mktData.gold=FALLBACK.gold;}}catch(e){mktData.gold=FALLBACK.gold;}
try{const s=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAG&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(s["Realtime Currency Exchange Rate"]){mktData.silver=parseFloat(s["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(2);}else{mktData.silver=FALLBACK.silver;}}catch(e){mktData.silver=FALLBACK.silver;}
renderTicker();
if(currentPage==="dashboard"||currentPage==="markets")renderPage(currentPage);
}

function renderTicker(){
const track=document.getElementById("tickerTrack");
const items=[
{label:"USD/ARS BNA",val:"V $"+FALLBACK.ARS_BNA_VENTA+" C $"+FALLBACK.ARS_BNA_COMPRA,up:false},
{label:"USD/ARS Blue",val:"V $"+FALLBACK.ARS_BLUE_VENTA+" C $"+FALLBACK.ARS_BLUE_COMPRA,up:false},
{label:"Dólar MEP",val:"$"+FALLBACK.ARS_MEP,up:true},
{label:"EUR/USD",val:(1/parseFloat(getRate("EUR"))).toFixed(4),up:true},
{label:"USD/BRL",val:getRate("BRL"),up:false},
{label:"USD/MXN",val:getRate("MXN"),up:false},
{label:"USD/CLP",val:FALLBACK.CLP,up:false},
{label:"USD/COP",val:FALLBACK.COP,up:false},
{label:"ORO oz",val:"USD "+getGold(),up:true},
{label:"PLATA oz",val:"USD "+getSilver(),up:true},
{label:"SOJA",val:"USD "+FALLBACK.soy,up:false},
{label:"MAÍZ",val:"USD "+FALLBACK.corn,up:false},
{label:"PETRÓLEO",val:"USD "+FALLBACK.oil,up:true},
];
const all=items.concat(items);
track.innerHTML=all.map(function(i){return'<span class="ticker-item"><span style="color:var(--faint)">'+i.label+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-weight:600">'+i.val+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-size:.65rem">'+(i.up?'▲':'▼')+'</span></span>';}).join("");
}

function handleLogin(){
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Ingresando...';btn.disabled=true;
setTimeout(function(){
document.getElementById("loginScreen").classList.add("hidden");
document.getElementById("appShell").classList.remove("hidden");
document.getElementById("appShell").style.display="flex";
renderTicker();renderNav();renderPage("dashboard");loadMarkets();
},900);
}
function handleLogout(){
document.getElementById("appShell").classList.add("hidden");
document.getElementById("loginScreen").classList.remove("hidden");
document.getElementById("loginBtn").innerHTML="Ingresar a Kontalo →";
document.getElementById("loginBtn").disabled=false;
}
function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}
function renderNav(){
const nav=document.getElementById("navMenu");
nav.innerHTML=NAV.map(function(n){return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')"><span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+'</div>';}).join("");
}
function renderPage(page){
currentPage=page;renderNav();
const company=COMPANIES[currentCompany];
const main=document.getElementById("mainContent");
if(page==="dashboard")renderDashboard(main,company);
else if(page==="cashflow")renderCashflow(main,company);
else if(page==="markets")renderMarkets(main);
else if(page==="invoices")renderInvoices(main);
else if(page==="stock")renderStock(main);
else if(page==="cierre")renderCierre(main);
else if(page==="alerts")renderAlerts(main);
}

async function renderDashboard(main,company){
const latest=company.seed[5]*1000,prev=company.seed[4]*1000;
const delta=(((latest-prev)/prev)*100).toFixed(1);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.25rem;">Panel Principal</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.25rem;">'+company.name+' · '+company.sector+'</p>'+
'<div class="kpi-grid">'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Flujo de caja</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--text)">'+fmt(latest)+'</div><div style="font-size:.75rem;color:'+(+delta>0?'var(--accent)':'var(--danger)')+'">'+( +delta>0?'▲':'▼')+' '+delta+'% vs mes anterior</div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiProjection"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiAvg"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Alertas activas</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--text)">3</div><div style="font-size:.75rem;color:var(--danger)">▼ 1 crítica</div></div>'+
'</div>'+
'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">💱 Tipo de Cambio</div>'+
'<div class="card" style="margin-bottom:1rem;overflow:hidden">'+
'<div style="display:grid;grid-template-columns:1.5fr 1fr 1fr 0.7fr;padding:.45rem .85rem;background:#0D1520;font-size:.7rem;color:var(--faint);text-transform:uppercase;font-weight:500"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
[
{label:"🇦🇷 Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"🇦🇷 Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"🇦🇷 Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"🇪🇺 Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(2),venta:(1/parseFloat(getRate("EUR"))).toFixed(2),var:"+0,08%",up:true},
{label:"🇧🇷 Real BRL",compra:getRate("BRL"),venta:(parseFloat(getRate("BRL"))*1.01).toFixed(4),var:"-0,12%",up:false},
].map(function(r){return'<div style="display:grid;grid-template-columns:1.5fr 1fr 1fr 0.7fr;align-items:center;padding:.6rem .85rem;border-top:1px solid var(--border)"><span style="font-size:.85rem;font-weight:500;color:var(--text)">'+r.label+'</span><span style="font-size:.85rem;font-weight:600;color:var(--text)">$'+r.compra+'</span><span style="font-size:.85rem;font-weight:600;color:var(--text)">$'+r.venta+'</span><span style="font-size:.82rem;color:'+(r.up?'var(--accent)':'var(--danger)')+'">'+r.var+'</span></div>';}).join('')+
'</div>'+
'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">🌾 Commodities</div>'+
'<div class="com-mini-grid">'+
[
{label:"🥇 Oro",val:"USD "+getGold()+" /oz"},
{label:"🥈 Plata",val:"USD "+getSilver()+" /oz"},
{label:"🌾 Soja",val:"USD "+FALLBACK.soy+" /ton"},
{label:"🌽 Maíz",val:"USD "+FALLBACK.corn+" /ton"},
{label:"🌿 Trigo",val:"USD "+FALLBACK.wheat+" /bu"},
{label:"🛢️ Petróleo",val:"USD "+FALLBACK.oil+" /barril"},
].map(function(r){return'<div class="card" style="padding:.75rem 1rem"><div style="font-size:.75rem;color:var(--muted);margin-bottom:.25rem">'+r.label+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:.95rem;color:var(--accent)">'+r.val+'</div></div>';}).join('')+
'</div>'+
'<div id="apiStatus"></div>'+
'<div class="bottom-grid">'+
'<div class="card" style="padding:1.1rem">'+
'<div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem;color:var(--text)">Facturas recientes</div>'+
INVOICES.slice(0,4).map(function(inv){return'<div style="display:flex;justify-content:space-between;align-items:center;padding:.55rem 0;border-bottom:1px solid var(--border)"><div><div style="font-size:.85rem;font-weight:500;color:var(--text)">'+inv.client+'</div><div style="font-size:.72rem;color:var(--muted)">'+inv.id+' · '+inv.date+'</div></div><div style="text-align:right"><div style="font-size:.9rem;font-weight:700;color:var(--text)">$'+inv.amount.toLocaleString("es-AR")+'</div><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+(inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente')+'</span></div></div>';}).join('')+
'</div>'+
'<div class="card" style="padding:1.1rem">'+
'<div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem;color:var(--text)">Alertas activas</div>'+
ALERTS.slice(0,4).map(function(a){return'<div class="alert-item alert-'+a.type+'"><div style="font-size:.8rem;font-weight:600;margin-bottom:.15rem;color:var(--text)">'+a.title+'</div><div style="font-size:.72rem;color:var(--muted)">'+a.desc+'</div><div style="font-size:.67rem;color:var(--faint);margin-top:.2rem">'+a.time+'</div></div>';}).join('')+
'</div>'+
'</div></div>';
try{const forecast=await fetchForecast(company.seed);
document.getElementById("kpiProjection").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Tendencia '+forecast.tendencia.toFixed(1)+'K/mes</div>';
document.getElementById("kpiAvg").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--text)">'+fmt(Math.round(forecast.promedio*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Desde API en vivo</div>';
}catch(e){document.getElementById("apiStatus").innerHTML='<div class="card" style="padding:.9rem;margin-bottom:1rem;color:var(--danger);font-size:.82rem;border-color:#FF6B6B44">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

async function fetchForecast(values){const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values:values})});if(!res.ok)throw new Error("API "+res.status);return res.json();}

async function renderCashflow(main,company){
main.innerHTML='<div class="fade-in"><h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;color:var(--text)">Flujo de Fondos</h1><div id="cfContent"><span class="spinner"></span> Consultando tu API...</div></div>';
try{const forecast=await fetchForecast(company.seed);
document.getElementById("cfContent").innerHTML=
'<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:1rem;margin-bottom:1.25rem">'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio 6 meses</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.promedio*1000))+'</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Tendencia mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+(forecast.tendencia>0?'+':'')+forecast.tendencia.toFixed(1)+'K</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Próxima proyección</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div></div>'+
'</div>'+
'<div class="card" style="padding:1.25rem"><div style="font-weight:600;margin-bottom:1rem;color:var(--text)">Detalle mensual</div>'+
'<table style="width:100%;border-collapse:collapse;font-size:.85rem">'+
'<thead><tr style="color:var(--faint);font-size:.72rem;text-transform:uppercase">'+
'<th style="text-align:left;padding:.5rem .75rem">Mes</th>'+
'<th style="text-align:left;padding:.5rem .75rem">Real</th>'+
'<th style="text-align:left;padding:.5rem .75rem">Proyectado</th>'+
'<th style="text-align:left;padding:.5rem .75rem">Estado</th>'+
'</tr></thead><tbody>'+
MONTHS.slice(0,6).map(function(m,i){return'<tr style="border-top:1px solid var(--border)"><td style="padding:.65rem .75rem;font-weight:500;color:var(--text)">'+m+'</td><td style="padding:.65rem .75rem;font-weight:600;color:var(--text)">'+fmt(company.seed[i]*1000)+'</td><td style="padding:.65rem .75rem;color:var(--muted)">'+fmt(Math.round((company.seed[5]+forecast.tendencia*(i-5))*1000))+'</td><td style="padding:.65rem .75rem"><span class="badge badge-green">En curso</span></td></tr>';}).join('')+
'</tbody></table></div>';
}catch(e){document.getElementById("cfContent").innerHTML='<div style="color:var(--danger)">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

function renderMarkets(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;color:var(--text)">FX & Mercados</h1>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem;color:var(--text)">Tipo de Cambio</div>'+
'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div style="display:grid;grid-template-columns:1.5fr 1fr 1fr 0.7fr;padding:.5rem .85rem;background:#0D1520;font-size:.7rem;color:var(--faint);text-transform:uppercase;font-weight:500"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
[
{label:"🇦🇷 Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"🇦🇷 Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"🇦🇷 Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"🇪🇺 Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(2),venta:(1/parseFloat(getRate("EUR"))).toFixed(2),var:"+0,08%",up:true},
{label:"🇧🇷 Real BRL",compra:getRate("BRL"),venta:(parseFloat(getRate("BRL"))*1.01).toFixed(4),var:"-0,12%",up:false},
{label:"🇲🇽 Peso MXN",compra:getRate("MXN"),venta:(parseFloat(getRate("MXN"))*1.01).toFixed(4),var:"+0,20%",up:true},
{label:"🇨🇴 Peso COP",compra:FALLBACK.COP,venta:FALLBACK.COP,var:"+0,10%",up:true},
{label:"🇨🇱 Peso CLP",compra:FALLBACK.CLP,venta:FALLBACK.CLP,var:"-0,05%",up:false},
{label:"🇬🇧 Libra GBP",compra:(1/parseFloat(getRate("GBP"))*0.99).toFixed(4),venta:(1/parseFloat(getRate("GBP"))).toFixed(4),var:"+0,15%",u%",u>