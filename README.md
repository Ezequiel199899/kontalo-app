cat > /mnt/user-data/outputs/index.html << 'ENDOFFILE'
<!DOCTYPE html>
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
.card{background:var(--surface);border:1px solid var(--border);border-radius:12px;}
.input{background:#0A0F1A;border:1px solid var(--border);border-radius:7px;color:var(--text);font-size:.9rem;padding:.75rem 1rem;width:100%;outline:none;}
.input:focus{border-color:var(--accent);}
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
.btn-login{width:100%;background:var(--accent);color:var(--bg);border:none;border-radius:7px;padding:1rem;font-size:1rem;font-weight:700;cursor:pointer;margin-bottom:.75rem;}
.btn-login:hover{background:#00FFB3;}
.btn-danger{background:transparent;color:var(--danger);border:1px solid #FF6B6B22;padding:.5rem 1rem;font-size:.8rem;border-radius:7px;cursor:pointer;width:100%;}
@media(max-width:768px){
.sidebar{display:none !important;}
.kpi-grid{grid-template-columns:1fr 1fr !important;}
.com-grid{grid-template-columns:1fr 1fr !important;}
.bottom-grid{grid-template-columns:1fr !important;}
}
</style>
</head>
<body>

<div id="loginScreen" style="min-height:100vh;display:flex;align-items:center;justify-content:center;padding:1.5rem;">
<div style="width:100%;max-width:420px;" class="fade-in">
<div style="text-align:center;margin-bottom:2.5rem;">
<span class="logo" style="font-size:2rem;"><span>kon</span>talo</span>
<p style="color:var(--muted);margin-top:.6rem;">Tu plataforma financiera inteligente</p>
</div>
<div class="card" style="padding:2rem;margin-bottom:1.5rem;">
<div style="margin-bottom:1rem;">
<label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Email</label>
<input class="input" id="loginEmail" type="email" placeholder="tu@empresa.com"/>
</div>
<div style="margin-bottom:1.5rem;">
<label style="display:block;font-size:.78rem;color:var(--muted);margin-bottom:.35rem;">Contraseña</label>
<input class="input" id="loginPass" type="password" placeholder="••••••••"/>
</div>
<div id="loginError" style="color:var(--danger);font-size:.8rem;margin-bottom:.9rem;display:none;">Completá email y contraseña.</div>
<button class="btn-login" id="loginBtn" type="button">Ingresar</button>
<p style="text-align:center;font-size:.75rem;color:var(--faint);">Demo: cualquier email y contraseña</p>
</div>
<p style="color:var(--muted);font-size:.85rem;margin-bottom:1rem;text-align:center;">¿Querés usar Kontalo en tu empresa?</p>
<div style="display:flex;gap:.75rem;justify-content:center;">
<div class="card" style="padding:1.25rem 1.5rem;text-align:center;flex:1;">
<div style="font-family:'Space Grotesk',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent);">$25<span style="font-size:.85rem;color:var(--muted)">/mes</span></div>
<div style="font-size:.82rem;font-weight:600;color:var(--text);margin:.3rem 0;">Plan PYME</div>
<div style="font-size:.72rem;color:var(--muted);">1 empresa · Panel completo</div>
</div>
<div class="card" style="padding:1.25rem 1.5rem;text-align:center;flex:1;border-color:var(--accent);">
<div style="font-family:'Space Grotesk',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent);">$99<span style="font-size:.85rem;color:var(--muted)">/mes</span></div>
<div style="font-size:.82rem;font-weight:600;color:var(--text);margin:.3rem 0;">Plan Estudio Contable</div>
<div style="font-size:.72rem;color:var(--muted);">Empresas ilimitadas</div>
</div>
</div>
<p style="text-align:center;margin-top:1rem;font-size:.72rem;color:var(--faint);">14 días gratis · Sin tarjeta · Cancelás cuando querés</p>
</div>
</div>

<div id="appShell" class="hidden" style="display:flex;min-height:100vh;">
<div class="sidebar" style="width:225px;background:var(--surface);border-right:1px solid var(--border);display:flex;flex-direction:column;height:100vh;position:sticky;top:0;flex-shrink:0;">
<div style="padding:1.4rem 1.2rem;border-bottom:1px solid var(--border);"><span class="logo" style="font-size:1.3rem;"><span>kon</span>talo</span></div>
<div style="padding:1rem 1.2rem;border-bottom:1px solid var(--border);">
<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.5rem;">Empresa activa</div>
<select id="companySelect" class="input" style="padding:.5rem .7rem;font-size:.82rem;cursor:pointer;"></select>
</div>
<nav style="padding:.75rem;flex:1;" id="navMenu"></nav>
<div style="padding:1rem 1.2rem;border-top:1px solid var(--border);">
<div style="display:flex;align-items:center;gap:.6rem;margin-bottom:.75rem;">
<div id="userAvatar" style="width:32px;height:32px;border-radius:50%;background:var(--accentDim);display:flex;align-items:center;justify-content:center;font-size:.7rem;font-weight:700;color:var(--accent);">--</div>
<div><div id="userName" style="font-size:.8rem;font-weight:600;color:var(--text);">—</div><div style="font-size:.68rem;color:var(--faint);">Plan PYME</div></div>
</div>
<button class="btn-danger" id="logoutBtn">Salir</button>
</div>
</div>
<div style="flex:1;display:flex;flex-direction:column;">
<div class="ticker-wrap"><div class="ticker-track" id="tickerTrack"><span class="ticker-item"><span class="spinner"></span> Cargando cotizaciones…</span></div></div>
<main style="flex:1;padding:1.5rem 2rem;overflow-y:auto;" id="mainContent"></main>
</div>
</div>

<script>
var AV_KEY="42ac3700f76949f6899936a3eb2ad9d2";
var MONTHS=["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
var COMPANIES=[
{name:"El Clavo Ferretería",sector:"Retail",seed:[210,195,230,215,240,225]},
{name:"Soto Import SA",sector:"Importación",seed:[450,480,510,490,530,505]},
{name:"Dubois Consulting",sector:"Servicios",seed:[80,95,88,102,91,110]}
];
var INVOICES=[
{id:"FAC-001",client:"BuildCo Ltd",amount:12400,date:"10 Jun",status:"paid"},
{id:"FAC-002",client:"Suministros Metro",amount:8750,date:"08 Jun",status:"pending"},
{id:"FAC-003",client:"Grupo Alfa",amount:31200,date:"05 Jun",status:"paid"},
{id:"FAC-004",client:"Obras Municipales",amount:5600,date:"01 Jun",status:"overdue"},
{id:"FAC-005",client:"TechParts SA",amount:19800,date:"28 May",status:"paid"},
{id:"FAC-006",client:"Harbor Imports",amount:44100,date:"25 May",status:"pending"}
];
var STOCK=[
{cod:"FER-001",prod:"Tornillos 1/2 x100",stock:450,min:100,precio:2800,est:"ok"},
{cod:"FER-002",prod:"Pintura latex 4L",stock:28,min:30,precio:15400,est:"low"},
{cod:"FER-003",prod:"Cable electrico 2.5mm",stock:12,min:20,precio:48000,est:"low"},
{cod:"FER-004",prod:"Cemento Portland 50kg",stock:95,min:50,precio:8900,est:"ok"},
{cod:"FER-005",prod:"Cano PVC 3 x6m",stock:5,min:15,precio:12500,est:"critical"},
{cod:"FER-006",prod:"Llave inglesa 12",stock:33,min:20,precio:9800,est:"ok"},
{cod:"FER-007",prod:"Discos de corte x10",stock:8,min:25,precio:6500,est:"critical"},
{cod:"FER-008",prod:"Silicona transp. 280ml",stock:62,min:30,precio:3200,est:"ok"}
];
var CIERRES=[
{fecha:"25 Jul",apertura:150000,ventas:384500,gastos:89200,cierre:445300,dif:300},
{fecha:"24 Jul",apertura:148000,ventas:412000,gastos:95000,cierre:465000,dif:-200},
{fecha:"23 Jul",apertura:145000,ventas:298000,gastos:78000,cierre:365000,dif:0},
{fecha:"22 Jul",apertura:140000,ventas:520000,gastos:110000,cierre:550000,dif:500},
{fecha:"21 Jul",apertura:138000,ventas:189000,gastos:65000,cierre:262000,dif:-800}
];
var ALERTS=[
{type:"danger",title:"Riesgo de flujo de caja",desc:"Se proyecta caida por debajo del umbral en mayo.",time:"Hace 2h"},
{type:"warn",title:"Factura vencida - Obras Municipales",desc:"FAC-004 por $5.600 tiene 14 dias de atraso.",time:"Hace 1d"},
{type:"warn",title:"Stock critico - Cano PVC",desc:"Solo 5 unidades. Minimo recomendado: 15.",time:"Hace 3h"},
{type:"ok",title:"Objetivo mensual alcanzado",desc:"El ingreso de junio supero la proyeccion un 12,4%.",time:"Hace 2d"},
{type:"ok",title:"Oportunidad tipo de cambio",desc:"Diferencial favorable en USD para importacion.",time:"Hace 3d"}
];
var NAV=[
{id:"dashboard",icon:"⬡",label:"Panel"},
{id:"markets",icon:"◇",label:"FX y Mercados"},
{id:"invoices",icon:"◈",label:"Facturas"},
{id:"stock",icon:"📦",label:"Stock"},
{id:"cierre",icon:"🔒",label:"Cierre de Caja"},
{id:"alerts",icon:"△",label:"Alertas",badge:3},
{id:"cashflow",icon:"◎",label:"Flujo de Caja"}
];
var FB={EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",ARS_BNA_C:"1470",ARS_BNA_V:"1520",ARS_BLUE_C:"1525",ARS_BLUE_V:"1545",ARS_MEP:"1531",CLP:"945",COP:"3196",gold:"4.068",silver:"58.88",oil:"73.40",soy:"281",wheat:"5.64",corn:"281"};
var currentCompany=0;
var currentPage="dashboard";
var mkt={rates:{},gold:null,silver:null,bna_c:null,bna_v:null,blue_c:null,blue_v:null};

function AR(n){return Number(n).toLocaleString("es-AR");}
function fmt(n){if(n>=1000000)return"$"+(n/1000000).toFixed(1)+"M";if(n>=1000)return"$"+(n/1000).toFixed(0)+"K";return"$"+n;}
function getR(c){return mkt.rates[c]||FB[c]||"—";}
function getGold(){return mkt.gold||FB.gold;}
function getSilver(){return mkt.silver||FB.silver;}
function getBnac(){return mkt.bna_c||FB.ARS_BNA_C;}
function getBnav(){return mkt.bna_v||FB.ARS_BNA_V;}
function getBluec(){return mkt.blue_c||FB.ARS_BLUE_C;}
function getBluev(){return mkt.blue_v||FB.ARS_BLUE_V;}

function doLogin(){
var email=document.getElementById("loginEmail").value;
var pass=document.getElementById("loginPass").value;
var err=document.getElementById("loginError");
if(!email||!pass){err.style.display="block";return;}
err.style.display="none";
var btn=document.getElementById("loginBtn");
btn.textContent="Ingresando...";
btn.disabled=true;
setTimeout(function(){
document.getElementById("userAvatar").textContent=email.slice(0,2).toUpperCase();
document.getElementById("userName").textContent=email.split("@")[0];
document.getElementById("loginScreen").classList.add("hidden");
var shell=document.getElementById("appShell");
shell.classList.remove("hidden");
shell.style.display="flex";
buildCompanySelect();
renderNav();
renderPage("dashboard");
loadMarkets();
},800);
}

function doLogout(){
document.getElementById("appShell").classList.add("hidden");
document.getElementById("loginScreen").classList.remove("hidden");
document.getElementById("loginEmail").value="";
document.getElementById("loginPass").value="";
document.getElementById("loginBtn").textContent="Ingresar";
document.getElementById("loginBtn").disabled=false;
}

function buildCompanySelect(){
var sel=document.getElementById("companySelect");
sel.innerHTML=COMPANIES.map(function(c,i){return'<option value="'+i+'">'+c.name+'</option>';}).join("");
sel.onchange=function(){currentCompany=parseInt(sel.value);renderPage(currentPage);};
}

function renderNav(){
document.getElementById("navMenu").innerHTML=NAV.map(function(n){
return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')">'+
'<span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+
(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+
'</div>';
}).join("");
}

function renderPage(p){
currentPage=p;
renderNav();
var co=COMPANIES[currentCompany];
var main=document.getElementById("mainContent");
if(p==="dashboard")renderDashboard(main,co);
else if(p==="markets")renderMarkets(main);
else if(p==="invoices")renderInvoices(main);
else if(p==="stock")renderStock(main);
else if(p==="cierre")renderCierre(main);
else if(p==="alerts")renderAlerts(main);
else if(p==="cashflow")renderCashflow(main,co);
}

function kpi(label,val,delta,up){
return'<div class="card" style="padding:1rem 1.2rem">'+
'<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">'+label+'</div>'+
'<div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.55rem;font-weight:700;color:var(--text)">'+val+'</div>'+
(delta?'<div style="font-size:.75rem;color:'+(up?'var(--accent)':'var(--danger)')+'">'+delta+'</div>':'')+
'</div>';
}

function fxRow(l,c,v,va,up){
return'<div class="tbl-r fx-cols">'+
'<span style="font-size:.88rem;font-weight:500;color:var(--text)">'+l+'</span>'+
'<span style="font-size:.92rem;font-weight:700;color:var(--text)">$'+c+'</span>'+
'<span style="font-size:.92rem;font-weight:700;color:var(--text)">$'+v+'</span>'+
'<span style="font-size:.82rem;color:'+(up?'var(--accent)':'var(--danger)')+'">'+va+'</span>'+
'</div>';
}

function comCard(n,u,v){
return'<div class="fx-card">'+
'<div style="font-size:.75rem;color:var(--muted);margin-bottom:.2rem">'+n+'</div>'+
'<div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.05rem;color:var(--accent)">'+v+'</div>'+
'<div style="font-size:.68rem;color:var(--faint)">por '+u+'</div>'+
'</div>';
}

function buildFxTable(){
return'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div class="tbl-h fx-cols"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
fxRow("🇦🇷 Dolar BNA",getBnac(),getBnav(),"+0,66%",true)+
fxRow("🇦🇷 Dolar Blue",getBluec(),getBluev(),"-0,32%",false)+
fxRow("🇦🇷 Dolar MEP",FB.ARS_MEP,FB.ARS_MEP,"+0,78%",true)+
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
comCard("🛢️ Petroleo","barril","USD "+FB.oil)+
comCard("🌾 Soja","tonelada","USD "+FB.soy)+
comCard("🌿 Trigo","bushel","USD "+FB.wheat)+
comCard("🌽 Maiz","tonelada","USD "+FB.corn)+
'</div>';
}

function renderDashboard(main,co){
var latest=co.seed[5]*1000;
var prev=co.seed[4]*1000;
var delta=(((latest-prev)/prev)*100).toFixed(1);
var avg=Math.round(co.seed.reduce(function(a,b){return a+b;},0)/co.seed.length*1000);
var proj=Math.round(co.seed[5]*1000*1.08);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.2rem;">Panel Principal</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.5rem;">'+co.name+' · '+co.sector+'</p>'+
'<div class="sec">📊 Resumen financiero</div>'+
'<div class="kpi-grid">'+
kpi("Flujo de caja",fmt(latest),(+delta>0?'▲':'▼')+' '+delta+'% vs mes ant.',+delta>0)+
kpi("Proyeccion estimada",fmt(proj),"▲ +8% estimado",true)+
kpi("Promedio mensual",fmt(avg),"▲ Ultimos 6 meses",true)+
kpi("Alertas activas","3","▼ 1 critica",false)+
'</div>'+
'<div class="sec">💱 Cotizaciones del dia</div>'+
buildFxTable()+
'<div class="sec">🌾 Commodities</div>'+
buildComGrid()+
'<div class="bottom-grid">'+
'<div><div class="sec">◈ Facturas recientes</div>'+
'<div class="card" style="overflow:hidden">'+
'<div class="tbl-h inv-cols"><span>N</span><span>Cliente</span><span>Monto</span><span>Fecha</span><span>Estado</span></div>'+
INVOICES.slice(0,5).map(function(inv){
var st=inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente';
var bc=inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow';
return'<div class="tbl-r inv-cols">'+
'<span style="font-size:.82rem;font-weight:700;color:var(--accent)">'+inv.id+'</span>'+
'<span style="font-size:.88rem;font-weight:500;color:var(--text)">'+inv.client+'</span>'+
'<span style="font-size:.95rem;font-weight:700;color:var(--text)">$'+AR(inv.amount)+'</span>'+
'<span style="font-size:.82rem;color:var(--muted)">'+inv.date+'</span>'+
'<span class="badge badge-'+bc+'">'+st+'</span>'+
'</div>';
}).join('')+
'</div></div>'+
'<div><div class="sec">△ Alertas activas</div>'+
ALERTS.map(function(a){
return'<div class="alert-item alert-'+a.type+' card" style="margin-bottom:.6rem;padding:1rem 1.1rem">'+
'<div style="display:flex;justify-content:space-between;margin-bottom:.2rem">'+
'<span style="font-size:.85rem;font-weight:600;color:var(--text)">'+a.title+'</span>'+
'<span style="font-size:.7rem;color:var(--faint)">'+a.time+'</span>'+
'</div>'+
'<div style="font-size:.78rem;color:var(--muted)">'+a.desc+'</div>'+
'</div>';
}).join('')+
'</div>'+
'</div></div>';
}

function renderMarkets(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.5rem;">FX y Mercados</h1>'+
'<div class="sec">💱 Divisas — Compra y Venta</div>'+
'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div class="tbl-h fx-cols"><span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
fxRow("🇦🇷 Dolar BNA",getBnac(),getBnav(),"+0,66%",true)+
fxRow("🇦🇷 Dolar Blue",getBluec(),get