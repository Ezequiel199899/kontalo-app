<<<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Kontalo — Plataforma Financiera</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap');
:root{--bg:#0A0F1A;--surface:#111827;--border:#1E2D3D;--accent:#00E5A0;--accentDim:#0D2A1F;--text:#E8EDF5;--muted:#8899AA;--faint:#4A5A6A;--danger:#FF6B6B;--warn:#F59E0B;}
*{box-sizing:border-box;margin:0;padding:0;}
body{background:var(--bg);color:var(--text);font-family:'Inter',system-ui,sans-serif;}
input,button,select{font-family:'Inter',sans-serif;}
@keyframes fadeIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:translateY(0)}}
@keyframes spin{to{transform:rotate(360deg)}}
@keyframes ticker{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}
.fade-in{animation:fadeIn .3s ease forwards;}
.btn{border:none;border-radius:7px;cursor:pointer;font-weight:600;font-size:.9rem;display:inline-flex;align-items:center;gap:.5rem;}
.btn-primary{background:var(--accent);color:var(--bg);padding:.75rem 1.5rem;}
.btn-primary:hover{background:#00FFB3;}
.btn-ghost{background:transparent;color:var(--text);border:1px solid var(--border);padding:.6rem 1.2rem;}
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
.logo{font-family:'Space Grotesk',sans-serif;font-weight:700;}
.logo span{color:var(--accent);}
.hidden{display:none !important;}
.kpi-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:.85rem;margin-bottom:1.5rem;}
.com-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:.85rem;margin-bottom:1.5rem;}
.bottom-grid{display:grid;grid-template-columns:1fr 1fr;gap:1rem;margin-bottom:1.5rem;}
.sec{font-family:'Space Grotesk',sans-serif;font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:.75rem;padding-bottom:.5rem;border-bottom:1px solid var(--border);}
.fx-card{background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:.85rem 1rem;}
.tbl-h{display:grid;padding:.45rem 1rem;background:#0D1520;font-size:.68rem;color:var(--faint);text-transform:uppercase;font-weight:500;}
.tbl-r{display:grid;align-items:center;padding:.72rem 1rem;border-top:1px solid var(--border);}
.fx-cols{grid-template-columns:1.6fr 1fr 1fr .7fr;}
.inv-cols{grid-template-columns:.7fr 1.8fr 1fr .75fr .8fr;}
.stk-cols{grid-template-columns:.75fr 2fr .65fr .65fr 1fr .7fr;}
.caj-cols{grid-template-columns:1fr 1fr 1fr 1fr 1fr .7fr;}
@media(max-width:768px){
.sidebar{display:none !important;}
.kpi-grid{grid-template-columns:1fr 1fr !important;}
.com-grid{grid-template-columns:1fr 1fr !important;}
.bottom-grid{grid-template-columns:1fr !important;}
}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" style="min-height:100vh;display:flex;align-items:center;justify-content:center;padding:1.5rem;">
<div style="width:100%;max-width:420px;" class="fade-in">
<div style="text-align:center;margin-bottom:2.5rem;">
<span class="logo" style="font-size:2rem;"><span>kon</span>talo</span>
<p style="color:var(--muted);margin-top:.6rem;">Tu plataforma financiera inteligente</p>
</div>
<div class="card" style="padding:2rem;">
<div style="margin-bottom:1rem;">
<label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Email</label>
<input class="input" id="loginEmail" type="email" placeholder="tu@empresa.com"/>
</div>
<div style="margin-bottom:1.5rem;">
<label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Contraseña</label>
<input class="input" id="loginPass" type="password" placeholder="••••••••" onkeydown="if(event.key==='Enter')handleLogin()"/>
</div>
<div id="loginError" style="color:var(--danger);font-size:.8rem;margin-bottom:.9rem;display:none;"></div>
<button class="btn btn-primary" style="width:100%;justify-content:center;" onclick="handleLogin()" id="loginBtn">Ingresar</button>
<p style="text-align:center;margin-top:1rem;font-size:.75rem;color:var(--faint);">Demo: cualquier email y contraseña</p>
</div>
<div style="margin-top:2rem;text-align:center;">
<p style="color:var(--muted);font-size:.85rem;margin-bottom:1rem;">¿Querés usar Kontalo en tu empresa?</p>
<div style="display:flex;gap:.75rem;justify-content:center;flex-wrap:wrap;">
<div class="card" style="padding:1.25rem 1.5rem;text-align:center;min-width:160px;">
<div style="font-family:'Space Grotesk',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent);">$25<span style="font-size:.85rem;color:var(--muted)">/mes</span></div>
<div style="font-size:.82rem;font-weight:600;color:var(--text);margin:.3rem 0;">Plan PYME</div>
<div style="font-size:.72rem;color:var(--muted);">1 empresa · Panel completo</div>
</div>
<div class="card" style="padding:1.25rem 1.5rem;text-align:center;min-width:160px;border-color:var(--accent);">
<div style="font-family:'Space Grotesk',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent);">$99<span style="font-size:.85rem;color:var(--muted)">/mes</span></div>
<div style="font-size:.82rem;font-weight:600;color:var(--text);margin:.3rem 0;">Plan Estudio Contable</div>
<div style="font-size:.72rem;color:var(--muted);">Empresas ilimitadas</div>
</div>
</div>
<p style="margin-top:1rem;font-size:.72rem;color:var(--faint);">14 días gratis · Sin tarjeta de crédito · Cancelás cuando querés</p>
</div>
</div>
</div>

<!-- APP -->
<div id="appShell" class="hidden" style="display:flex;min-height:100vh;">
<div class="sidebar" style="width:225px;background:var(--surface);border-right:1px solid var(--border);display:flex;flex-direction:column;height:100vh;position:sticky;top:0;flex-shrink:0;">
<div style="padding:1.4rem 1.2rem;border-bottom:1px solid var(--border);">
<span class="logo" style="font-size:1.3rem;"><span>kon</span>talo</span>
</div>
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
<div style="width:32px;height:32px;border-radius:50%;background:var(--accentDim);display:flex;align-items:center;justify-content:center;font-size:.7rem;font-weight:700;color:var(--accent);" id="userAvatar">--</div>
<div>
<div style="font-size:.8rem;font-weight:600;color:var(--text);" id="userName">—</div>
<div style="font-size:.68rem;color:var(--faint);" id="userPlan">Plan PYME</div>
</div>
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
const MONTHS=["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
const COMPANIES=[
{name:"El Clavo Ferretería",sector:"Retail",plan:"PYME",seed:[210,195,230,215,240,225]},
{name:"Soto Import SA",sector:"Importación",plan:"Estudio",seed:[450,480,510,490,530,505]},
{name:"Dubois Consulting",sector:"Servicios",plan:"PYME",seed:[80,95,88,102,91,110]}
];
const INVOICES=[
{id:"FAC-001",client:"BuildCo Ltd",amount:12400,date:"10 Jun",status:"paid"},
{id:"FAC-002",client:"Suministros Metro",amount:8750,date:"08 Jun",status:"pending"},
{id:"FAC-003",client:"Grupo Alfa",amount:31200,date:"05 Jun",status:"paid"},
{id:"FAC-004",client:"Obras Municipales",amount:5600,date:"01 Jun",status:"overdue"},
{id:"FAC-005",client:"TechParts SA",amount:19800,date:"28 May",status:"paid"},
{id:"FAC-006",client:"Harbor Imports",amount:44100,date:"25 May",status:"pending"}
];
const STOCK=[
{cod:"FER-001",prod:"Tornillos 1/2\" x100",stock:450,min:100,precio:2800,est:"ok"},
{cod:"FER-002",prod:"Pintura látex blanco 4L",stock:28,min:30,precio:15400,est:"low"},
{cod:"FER-003",prod:"Cable eléctrico 2.5mm",stock:12,min:20,precio:48000,est:"low"},
{cod:"FER-004",prod:"Cemento Portland 50kg",stock:95,min:50,precio:8900,est:"ok"},
{cod:"FER-005",prod:"Caño PVC 3\" x6m",stock:5,min:15,precio:12500,est:"critical"},
{cod:"FER-006",prod:"Llave inglesa 12\"",stock:33,min:20,precio:9800,est:"ok"},
{cod:"FER-007",prod:"Discos de corte x10",stock:8,min:25,precio:6500,est:"critical"},
{cod:"FER-008",prod:"Silicona transp. 280ml",stock:62,min:30,precio:3200,est:"ok"}
];
const CIERRES=[
{fecha:"25 Jul",apertura:150000,ventas:384500,gastos:89200,cierre:445300,dif:300},
{fecha:"24 Jul",apertura:148000,ventas:412000,gastos:95000,cierre:465000,dif:-200},
{fecha:"23 Jul",apertura:145000,ventas:298000,gastos:78000,cierre:365000,dif:0},
{fecha:"22 Jul",apertura:140000,ventas:520000,gastos:110000,cierre:550000,dif:500},
{fecha:"21 Jul",apertura:138000,ventas:189000,gastos:65000,cierre:262000,dif:-800}
];
const ALERTS=[
{type:"danger",title:"Riesgo de flujo de caja",desc:"Se proyecta caída por debajo del umbral en mayo.",time:"Hace 2h"},
{type:"warn",title:"Factura vencida — Obras Municipales",desc:"FAC-004 por $5.600 tiene 14 días de atraso.",time:"Hace 1d"},
{type:"warn",title:"Stock crítico — Caño PVC",desc:"Solo 5 unidades. Mínimo recomendado: 15.",time:"Hace 3h"},
{type:"ok",title:"Objetivo mensual alcanzado",desc:"El ingreso de junio superó la proyección un 12,4%.",time:"Hace 2d"},
{type:"ok",title:"Oportunidad tipo de cambio",desc:"Diferencial favorable en USD para importación.",time:"Hace 3d"}
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

// Fallback precios referencia
const FB={
EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",
ARS_BNA_C:"1470",ARS_BNA_V:"1520",
ARS_BLUE_C:"1525",ARS_BLUE_V:"1545",
ARS_MEP:"1531",CLP:"945",COP:"3196",
gold:"4.068",silver:"58.88",oil:"73.40",soy:"281",wheat:"5.64",corn:"281"
};

let currentCompany=0,currentPage="dashboard";
let mkt={rates:{},gold:null,silver:null,bcra:{oficial_c:null,oficial_v:null,blue_c:null,blue_v:null,mep:null}};

const AR=function(n){return Number(n).toLocaleString("es-AR");};
function fmt(n){if(n>=1e6)return"$"+(n/1e6).toFixed(1)+"M";if(n>=1000)return"$"+(n/1000).toFixed(0)+"K";return"$"+n;}
function getR(c){return mkt.rates[c]||FB[c]||"—";}
function getGold(){return mkt.gold||FB.gold;}
function getSilver(){return mkt.silver||FB.silver;}
function getBnac(){return mkt.bcra.oficial_c||FB.ARS_BNA_C;}
function getBnav(){return mkt.bcra.oficial_v||FB.ARS_BNA_V;}
function getBluec(){return mkt.bcra.blue_c||FB.ARS_BLUE_C;}
function getBluev(){return mkt.bcra.blue_v||FB.ARS_BLUE_V;}
function getMep(){return mkt.bcra.mep||FB.ARS_MEP;}

// BCRA API — estadisticasbcra.com (sin token, datos del día)
async function loadBCRA(){
try{
const today=new Date().toISOString().split("T")[0];
const [ofRes,blueRes]=await Promise.allSettled([
fetch("https://api.estadisticasbcra.com/usd_of_minorista").then(function(r){return r.json();}),
fetch("https://api.estadisticasbcra.com/usd").then(function(r){return r.json();})
]);
if(ofRes.status==="fulfilled"&&ofRes.value.length){
const last=ofRes.value[ofRes.value.length-1];
mkt.bcra.oficial_v=Math.round(last.v).toString();
mkt.bcra.oficial_c=Math.round(last.v*0.967).toString();
}
if(blueRes.status==="fulfilled"&&blueRes.value.length){
const last=blueRes.value[blueRes.value.length-1];
mkt.bcra.blue_v=Math.round(last.v).toString();
mkt.bcra.blue_c=Math.round(last.v*0.987).toString();
}
}catch(e){}
}

// Alpha Vantage — pares internacionales + metales
async function loadMarkets(){
await loadBCRA();
try{
const pp=["EUR","BRL","MXN","GBP","CNY"];
const rr=await Promise.allSettled(pp.map(function(p){
return fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency="+p+"&apikey="+AV_KEY).then(function(r){return r.json();});
}));
rr.forEach(function(r,i){
if(r.status==="fulfilled"&&r.value["Realtime Currency Exchange Rate"]){
mkt.rates[pp[i]]=parseFloat(r.value["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(4);
}else{mkt.rates[pp[i]]=FB[pp[i]];}
});
}catch(e){}
try{
const g=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});
if(g["Realtime Currency Exchange Rate"]){mkt.gold=parseFloat(g["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toLocaleString("es-AR",{maximumFractionDigits:2});}
else{mkt.gold=FB.gold;}
}catch(e){mkt.gold=FB.gold;}
try{
const s=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAG&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});
if(s["Realtime Currency Exchange Rate"]){mkt.silver=parseFloat(s["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(2);}
else{mkt.silver=FB.silver;}
}catch(e){mkt.silver=FB.silver;}
renderTicker();
if(currentPage==="dashboard"||currentPage==="markets")renderPage(currentPage);
}

function renderTicker(){
const track=document.getElementById("tickerTrack");
const items=[
{label:"USD/ARS BNA",val:"C $"+getBnac()+" V $"+getBnav(),up:false},
{label:"USD/ARS Blue",val:"C $"+getBluec()+" V $"+getBluev(),up:false},
{label:"Dólar MEP",val:"$"+getMep(),up:true},
{label:"EUR/USD",val:(1/parseFloat(getR("EUR"))).toFixed(4),up:true},
{label:"USD/BRL",val:getR("BRL"),up:false},
{label:"USD/MXN",val:getR("MXN"),up:false},
{label:"USD/CLP",val:FB.CLP,up:false},
{label:"USD/COP",val:FB.COP,up:false},
{label:"ORO oz",val:"USD "+getGold(),up:true},
{label:"PLATA oz",val:"USD "+getSilver(),up:true},
{label:"SOJA",val:"USD "+FB.soy,up:false},
{label:"MAÍZ",val:"USD "+FB.corn,up:false},
{label:"PETRÓLEO",val:"USD "+FB.oil,up:true},
];
const all=items.concat(items);
track.innerHTML=all.map(function(i){
return'<span class="ticker-item"><span style="color:var(--faint)">'+i.label+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-weight:600">'+i.val+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-size:.65rem">'+(i.up?'▲':'▼')+'</span></span>';
}).join("");
}

function handleLogin(){
const email=document.getElementById("loginEmail").value;
const pass=document.getElementById("loginPass").value;
const err=document.getElementById("loginError");
if(!email||!pass){err.textContent="Completá email y contraseña.";err.style.display="block";return;}
err.style.display="none";
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Ingresando...';btn.disabled=true;
setTimeout(function(){
const initials=email.split("@")[0].slice(0,2).toUpperCase();
document.getElementById("userAvatar").textContent=initials;
document.getElementById("userName").textContent=email.split("@")[0];
document.getElementById("loginScreen").classList.add("hidden");
document.getElementById("appShell").classList.remove("hidden");
document.getElementById("appShell").style.display="flex";
renderNav();renderPage("dashboard");loadMarkets();
},900);
}

function handleLogout(){
document.getElementById("appShell").classList.add("hidden");
document.getElementById("loginScreen").classList.remove("hidden");
document.getElementById("loginEmail").value="";
document.getElementById("loginPass").value="";
document.getElementById("loginBtn").innerHTML="Ingresar";
document.getElementById("loginBtn").disabled=false;
}

function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}

function renderNav(){
document.getElementById("navMenu").innerHTML=NAV.map(function(n){
return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')"><span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+'</div>';
}).join("");
}

function renderPage(page){
currentPage=page;renderNav();
const co=COMPANIES[currentCompany];
const main=document.getElementById("mainContent");
if(page==="dashboard")renderDashboard(main,co);
else if(page==="markets")renderMarkets(main);
else if(page==="invoices")renderInvoices(main);
else if(page==="stock")renderStock(main);
else if(page==="cierre")renderCierre(main);
else if(page==="alerts")renderAlerts(main);
else if(page==="cashflow")renderCashflow(main,co);
}

function kpiCard(label,val,delta,deltaUp){
return'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">'+label+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.55rem;font-weight:700;color:var(--text)">'+val+'</div>'+(delta?'<div style="font-size:.75rem;color:'+(deltaUp?'var(--accent)':'var(--danger)')+'">'+delta+'</div>':'')+'</div>';
}

function fxRow(l,c,v,va,up){
return'<div class="tbl-r fx-cols"><span style="font-size:.88rem;font-weight:500;color:var(--text)">'+l+'</span><span style="font-size:.92rem;font-weight:700;color:#1a1a2e;background:#E8EDF5;padding:.15rem .5rem;border-radius:5px">$'+c+'</span><span style="font-size:.92rem;font-weight:700;color:#1a1a2e;background:#E8EDF5;padding:.15rem .5rem;border-radius:5px">$'+v+'</span><span style="font-size:.82rem;color:'+(up?'var(--accent)':'var(--danger)')+'">'+va+'</span></div>';
}

function comCard(n,u,v){
return'<div class="fx-card"><div style="font-size:.75rem;color:var(--muted);margin-bottom:.2rem">'+n+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.05rem;color:var(--accent)">'+v+'</div><div style="font-size:.68rem;color:var(--faint)">por '+u+'</div></div>';
}

function buildFxTable(){
return'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div class="tbl-h fx-cols"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
fxRow("🇦🇷 Dólar BNA",getBnac(),getBnav(),"+0,66%",true)+
fxRow("🇦🇷 Dólar Blue",getBluec(),getBluev(),"-0,32%",false)+
fxRow("🇦🇷 Dólar MEP",getMep(),getMep(),"+0,78%",true)+
fxRow("🇪🇺 Euro",(1/parseFloat(getR("EUR"))*0.99).toFixed(2),(1/parseFloat(getR("EUR"))).toFixed(2),"+0,08%",true)+
fxRow("🇧🇷 Real BRL",getR("BRL"),(parseFloat(getR("BRL"))*1.01).toFixed(4),"-0,12%",false)+
fxRow("🇨🇱 Peso CLP",FB.CLP,FB.CLP,"-0,05%",false)+
fxRow("🇨🇴 Peso COP",FB.COP,FB.COP,"+0,10%",true)+
'</div>';
}

function buildComGrid(){
return'<div class="com-grid">'+
comCard("🥇 Oro","troy oz","USD "+getGold())+
comCard("🥈 Plata","troy oz","USD "+getSilver())+
comCard("🛢️ Petróleo","barril","USD "+FB.oil)+
comCard("🌾 Soja","tonelada","USD "+FB.soy)+
comCard("🌿 Trigo","bushel","USD "+FB.wheat)+
comCard("🌽 Maíz","tonelada","USD "+FB.corn)+
'</div>';
}

function renderDashboard(main,co){
const latest=co.seed[5]*1000,prev=co.seed[4]*1000;
const delta=(((latest-prev)/prev)*100).toFixed(1);
const avg=Math.round(co.seed.reduce(function(a,b){return a+b;},0)/co.seed.length*1000);
const proj=Math.round(co.seed[5]*1000*1.08);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.2rem;">Panel Principal</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.5rem;">'+co.name+' · '+co.sector+'</p>'+
'<div class="sec">📊 Resumen financiero</div>'+
'<div class="kpi-grid">'+
kpiCard("Flujo de caja",fmt(latest),(+delta>0?'▲':'▼')+' '+delta+'% vs mes ant.',+delta>0)+
kpiCard("Proyección estimada",fmt(proj),"▲ +8% estimado",true)+
kpiCard("Promedio mensual",fmt(avg),"▲ Últimos 6 meses",true)+
kpiCard("Alertas activas","3","▼ 1 crítica",false)+
'</div>'+
'<div class="sec">💱 Cotizaciones del día — BCRA + Mercado</div>'+
buildFxTable()+
'<div class="sec">🌾 Commodities</div>'+
buildComGrid()+
'<div class="bottom-grid">'+
'<div><div class="sec">◈ Facturas recientes</div>'+
'<div class="card" style="overflow:hidden">'+
'<div class="tbl-h inv-cols"><span>Nº</span><span>Cliente</span><span>Monto</span><span>Fecha</span><span>Estado</span></div>'+
INVOICES.slice(0,5).map(function(inv){
return'<div class="tbl-r inv-cols"><span style="font-size:.82rem;font-weight:700;color:var(--accent)">'+inv.id+'</span><span style="font-size:.88rem;font-weight:500;color:var(--text)">'+inv.client+'</span><span style="font-size:.95rem;font-weight:700;color:var(--text)">$'+AR(inv.amount)+'</span><span style="font-size:.82rem;color:var(--muted)">'+inv.date+'</span><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+(inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente')+'</span></div>';
}).join('')+
'</div></div>'+
'<div><div class="sec">△ Alertas activas</div>'+
ALERTS.map(function(a){
return'<div class="alert-item alert-'+a.type+' card" style="margin-bottom:.6rem;padding:1rem 1.1rem"><div style="display:flex;justify-content:space-between;margin-bottom:.2rem"><span style="font-size:.85rem;font-weight:600;color:var(--text)">'+a.title+'</span><span style="font-size:.7rem;color:var(--faint)">'+a.time+'</span></div><div style="font-size:.78rem;color:var(--muted)">'+a.desc+'</div></div>';
}).join('')+
'</div></div></div>';
}

function renderMarkets(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.5rem;">FX & Mercados</h1>'+
'<div class="sec">💱 Divisas — Compra y Venta · BCRA + Mercado</div>'+
'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div class="tbl-h fx-cols"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
fxRow("🇦🇷 Dólar BNA",getBnac(),getBnav(),"+0,66%",true)+
fxRow("🇦🇷 Dólar Blue",getBluec(),getBluev(),"-0,32%",false)+
fxRow("🇦🇷 Dólar MEP",getMep(),getMep(),"+0,78%",true)+
fxRow("🇪🇺 Euro",(1/parseFloat(getR("EUR"))*0.99).toFixed(2),(1/parseFloat(getR("EUR"))).toFixed(2),"+0,08%",true)+
fxRow("🇧🇷 Real BRL",getR("BRL"),(parseFloat(getR("BRL"))*1.01).toFixed(4),"-0,12%",false)+
fxRow("🇲🇽 Peso MXN",getR("MXN"),(parseFloat(getR("MXN"))*1.01).toFixed(4),"+0,20%",true)+
fxRow("🇨🇴 Peso COP",FB.COP,FB.COP,"+0,10%",true)+
fxRow("🇨🇱 Peso CLP",FB.CLP,FB.CLP,"-0,05%",false)+
fxRow("🇬🇧 Libra GBP",(1/parseFloat(getR("GBP"))*0.99).toFixed(4),(1/parseFloat(getR("GBP"))).toFixed(4),"+0,15%",true)+
fxRow("🇨🇳 Yuan CNY",getR("CNY"),(parseFloat(getR("CNY"))*1.01).toFixed(4),"-0,08%",false)+
'</div>'+
'<div class="sec">🌾 Commodities</div>'+
'<div class="com-grid">'+
comCard("🥇 Oro","troy oz","USD "+getGold())+
comCard("🥈 Plata","troy oz","USD "+getSilver())+
comCard("🛢️ Petróleo","barril","USD "+FB.oil)+
comCard("🌾 Soja","tonelada","USD "+FB.soy)+
comCard("🌿 Trigo","bushel","USD "+FB.wheat)+
comCard("🌽 Maíz","tonelada","USD "+FB.corn)+
'</div>'+
'<p style="font-size:.72rem;color:var(--faint);margin-top:.5rem;">🇦🇷 Dólar oficial y blue: API estadisticasbcra.com · Divisas internacionales: Alpha Vantage · Metales: Alpha Vantage · Commodities agrícolas: referencia de mercado</p>'+
'</div>';
}

function renderInvoices(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.5rem;">Facturas</h1>'+
'<div class="card" style="overflow:hidden">'+
'<div class="tbl-h inv-cols" style="background:#0D1520"><span>Nº</span><span>Cliente</span><span>Monto</span><span>Fecha</span><span>Estado</span></div>'+
INVOICES.map(function(inv){
return'<div class="tbl-r inv-cols" style="background:var(--surface)"><span style="font-size:.85rem;font-weight:700;color:var(--accent)">'+inv.id+'</span><span style="font-size:.9rem;font-weight:600;color:var(--text)">'+inv.client+'</span><span style="font-size:.98rem;font-weight:700;color:var(--text)">$'+AR(inv.amount)+'</span><span style="font-size:.85rem;color:var(--muted)">'+inv.date+'</span><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+(inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente')+'</span></div>';
}).join('')+
'</div></div>';
}

function renderStock(main){
const crit=STOCK.filter(function(s){return s.est==='critical';}).length;
const low=STOCK.filter(function(s){return s.est==='low';}).length;
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.5rem;">Stock</h1>'+
'<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:.85rem;margin-bottom:1.5rem">'+
kpiCard("Total productos",STOCK.length.toString(),"",true)+
kpiCard("Stock crítico",crit.toString(),"▼ Reponer urgente",false)+
kpiCard("Stock bajo",low.toString(),"⚠ Bajo mínimo",false)+
'</div>'+
'<div class="card" style="overflow:hidden">'+
'<div class="tbl-h stk-cols"><span>Código</span><span>Producto</span><span>Stock</span><span>Mín.</span><span>Precio</span><span>Estado</span></div>'+
STOCK.map(function(s){
const ec=s.est==='critical'?'var(--danger)':s.est==='low'?'var(--warn)':'var(--accent)';
const el=s.est==='critical'?'Crítico':s.est==='low'?'Bajo':'OK';
const eb=s.est==='critical'?'red':s.est==='low'?'yellow':'green';
return'<div class="tbl-r stk-cols"><span style="font-size:.82rem;font-weight:600;color:var(--accent)">'+s.cod+'</span><span style="font-size:.88rem;font-weight:500;color:var(--text)">'+s.prod+'</span><span style="font-size:.95rem;font-weight:700;color:'+ec+'">'+s.stock+'</span><span style="font-size:.88rem;color:var(--muted)">'+s.min+'</span><span style="font-size:.9rem;font-weight:700;color:var(--text)">$'+AR(s.precio)+'</span><span class="badge badge-'+eb+'">'+el+'</span></div>';
}).join('')+
'</div></div>';
}

function renderCierre(main){
const ul=CIERRES[0];
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.5rem;">Cierre de Caja</h1>'+
'<div class="kpi-grid">'+
kpiCard("Apertura hoy","$"+AR(ul.apertura),"",true)+
kpiCard("Ventas del día","$"+AR(ul.ventas),"▲ Del día",true)+
kpiCard("Gastos del día","$"+AR(ul.gastos),"▼ Del día",false)+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Cierre</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.55rem;font-weight:700;color:var(--text)">$'+AR(ul.cierre)+'</div><div style="font-size:.75rem;color:'+(ul.dif>=0?'var(--accent)':'var(--danger)')+'">Diferencia: '+(ul.dif>=0?'+':'')+ul.dif+'</div></div>'+
'</div>'+
'<div class="sec">📋 Historial de cierres</div>'+
'<div class="card" style="overflow:hidden">'+
'<div class="tbl-h caj-cols"><span>Fecha