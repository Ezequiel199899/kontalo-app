<script>
const AV_KEY="42ac3700f76949f6899936a3eb2ad9d2";
const API_BASE="https://contabilidad-de-datos.onrender.com";
const MONTHS=["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
const COMPANIES=[{name:"El Clavo Ferretería",sector:"Retail",plan:"Growth",seed:[210,195,230,215,240,225],currency:"ARS"},{name:"Soto Import SA",sector:"Importación",plan:"Scale",seed:[450,480,510,490,530,505],currency:"USD"},{name:"Dubois Consulting",sector:"Servicios",plan:"Starter",seed:[80,95,88,102,91,110],currency:"BRL"}];
const INVOICES=[{id:"FAC-001",client:"BuildCo Ltd",amount:12400,date:"10 Jun",status:"paid"},{id:"FAC-002",client:"Suministros Metro",amount:8750,date:"08 Jun",status:"pending"},{id:"FAC-003",client:"Grupo Alfa",amount:31200,date:"05 Jun",status:"paid"},{id:"FAC-004",client:"Obras Municipales",amount:5600,date:"01 Jun",status:"overdue"},{id:"FAC-005",client:"TechParts SA",amount:19800,date:"28 May",status:"paid"},{id:"FAC-006",client:"Harbor Imports",amount:44100,date:"25 May",status:"pending"}];
const ALERTS=[{type:"danger",title:"Riesgo de flujo de caja detectado",desc:"Se proyecta caída por debajo del umbral en mayo.",time:"Hace 2h"},{type:"warn",title:"Factura vencida — Obras Municipales",desc:"FAC-004 por $5.600 tiene 14 días de atraso.",time:"Hace 1d"},{type:"ok",title:"Objetivo mensual alcanzado",desc:"El ingreso de junio superó la proyección un 12,4%.",time:"Hace 2d"},{type:"ok",title:"Oportunidad en tipo de cambio",desc:"Diferencial favorable en USD para pedidos de importación.",time:"Hace 3d"}];
const NAV=[{id:"dashboard",icon:"⬡",label:"Panel"},{id:"markets",icon:"◇",label:"FX & Mercados"},{id:"invoices",icon:"◈",label:"Facturas"},{id:"alerts",icon:"△",label:"Alertas",badge:2},{id:"cashflow",icon:"◎",label:"Flujo de Caja"}];

// Precios actualizados 25 Jul 2026
const FALLBACK={
  EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",
  ARS_BNA_COMPRA:"1470",ARS_BNA_VENTA:"1520",
  ARS_BLUE_COMPRA:"1525",ARS_BLUE_VENTA:"1545",
  ARS_MEP:"1531",
  CLP:"945",COP:"3196",
  gold:"4.068",silver:"58,88",
  oil:"73,40",soy:"281",wheat:"5,64",corn:"281"
};

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
try{const g=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(g["Realtime Currency Exchange Rate"]){mktData.gold=parseFloat(g["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toLocaleString("es-AR",{maximumFractionDigits:2});}else{mktData.gold=FALLBACK.gold;}}catch(e){mktData.gold=FALLBACK.gold;}
try{const s=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAG&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(s["Realtime Currency Exchange Rate"]){mktData.silver=parseFloat(s["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(2);}else{mktData.silver=FALLBACK.silver;}}catch(e){mktData.silver=FALLBACK.silver;}
renderTicker();
if(currentPage==="dashboard"||currentPage==="markets")renderPage(currentPage);
}

function renderTicker(){
const track=document.getElementById("tickerTrack");
const items=[
{label:"USD/ARS BNA",val:"V $"+FALLBACK.ARS_BNA_VENTA+" | C $"+FALLBACK.ARS_BNA_COMPRA,up:false},
{label:"USD/ARS Blue",val:"V $"+FALLBACK.ARS_BLUE_VENTA+" | C $"+FALLBACK.ARS_BLUE_COMPRA,up:false},
{label:"Dólar MEP",val:"$"+FALLBACK.ARS_MEP,up:true},
{label:"EUR/USD",val:(1/parseFloat(getRate("EUR"))).toFixed(4),up:true},
{label:"USD/BRL",val:getRate("BRL"),up:false},
{label:"USD/MXN",val:getRate("MXN"),up:false},
{label:"USD/CLP",val:FALLBACK.CLP,up:false},
{label:"USD/COP",val:FALLBACK.COP,up:false},
{label:"GBP/USD",val:(1/parseFloat(getRate("GBP"))).toFixed(4),up:true},
{label:"ORO oz",val:"USD "+getGold(),up:true},
{label:"PLATA oz",val:"USD "+getSilver(),up:true},
{label:"SOJA",val:"USD "+FALLBACK.soy,up:false},
{label:"TRIGO",val:"USD "+FALLBACK.wheat,up:true},
{label:"MAÍZ",val:"USD "+FALLBACK.corn,up:false},
{label:"PETRÓLEO",val:"USD "+FALLBACK.oil,up:true},
];
const all=items.concat(items);
track.innerHTML=all.map(function(i){return'<span class="ticker-item"><span style="color:var(--faint)">'+i.label+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-weight:600">'+i.val+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-size:.65rem">'+(i.up?'▲':'▼')+'</span></span>';}).join("");
}

function handleLogin(){
const email=document.getElementById("loginEmail").value;
const pass=document.getElementById("loginPass").value;
const err=document.getElementById("loginError");
if(!email||!pass){err.textContent="Por favor completá todos los campos.";err.style.display="block";return;}
err.style.display="none";
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Ingresando...';btn.disabled=true;
setTimeout(function(){
currentUser={name:"Ezequiel Prilusky",email:email};
document.getElementById("userName").textContent=currentUser.name;
document.getElementById("loginScreen").classList.add("hidden");
document.getElementById("appShell").classList.remove("hidden");
document.getElementById("appShell").style.display="flex";
renderTicker();renderNav();renderPage("dashboard");loadMarkets();
},1100);
}

function handleLogout(){currentUser=null;document.getElementById("appShell").classList.add("hidden");document.getElementById("loginScreen").classList.remove("hidden");document.getElementById("loginEmail").value="";document.getElementById("loginPass").value="";document.getElementById("loginBtn").innerHTML="Ingresar";document.getElementById("loginBtn").disabled=false;}
function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}
function renderNav(){const nav=document.getElementById("navMenu");nav.innerHTML=NAV.map(function(n){return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')"><span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+'</div>';}).join("");document.getElementById("userPlan").textContent=COMPANIES[currentCompany].plan+" plan";}
function renderPage(page){currentPage=page;renderNav();const company=COMPANIES[currentCompany];const main=document.getElementById("mainContent");if(page==="dashboard")renderDashboard(main,company);else if(page==="cashflow")renderCashflow(main,company);else if(page==="markets")renderMarkets(main);else if(page==="invoices")renderInvoices(main);else if(page==="alerts")renderAlerts(main);}

async function renderDashboard(main,company){
const latest=company.seed[5]*1000,prev=company.seed[4]*1000;
const delta=(((latest-prev)/prev)*100).toFixed(1);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.25rem;">Panel Principal</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.25rem;">'+company.name+' · '+company.sector+'</p>'+
'<div class="kpi-grid">'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Flujo de caja</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(latest)+'</div><div style="font-size:.75rem;color:'+(+delta>0?'var(--accent)':'var(--danger)')+'">'+( +delta>0?'▲':'▼')+' '+delta+'% vs mes anterior</div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiProjection"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiAvg"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Alertas activas</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">2</div><div style="font-size:.75rem;color:var(--danger)">▼ 1 crítica</div></div>'+
'</div>'+

'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">💱 Tipo de Cambio</div>'+
'<div class="card" style="margin-bottom:1rem;overflow:hidden">'+
[
{label:"Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(2),venta:(1/parseFloat(getRate("EUR"))).toFixed(2),var:"+0,08%",up:true},
{label:"Real BRL",compra:(parseFloat(getRate("BRL"))*0.99).toFixed(4),venta:getRate("BRL"),var:"-0,12%",up:false},
{label:"Peso CLP",compra:FALLBACK.CLP,venta:FALLBACK.CLP,var:"-0,05%",up:false},
{label:"Peso COP",compra:FALLBACK.COP,venta:FALLBACK.COP,var:"+0,10%",up:true},
].map(function(r){return'<div class="fx-table-row"><span style="font-size:.85rem;font-weight:500;min-width:120px">'+r.label+'</span><span style="font-size:.82rem;color:var(--muted)">C: <strong style="color:var(--text)">$'+r.compra+'</strong></span><span style="font-size:.82rem;color:var(--muted)">V: <strong style="color:var(--text)">$'+r.venta+'</strong></span><span style="font-size:.78rem;color:'+(r.up?'var(--accent)':'var(--danger)')+'">'+r.var+'</span></div>';}).join('')+
'</div>'+

'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">🌾 Commodities</div>'+
'<div class="com-mini-grid">'+
[
{label:"🥇 Oro",val:"USD "+getGold()+" /oz"},
{label:"🥈 Plata",val:"USD "+getSilver()+" /oz"},
{label:"🛢️ Petróleo",val:"USD "+FALLBACK.oil+" /barril"},
{label:"🌾 Soja",val:"USD "+FALLBACK.soy+" /ton"},
{label:"🌿 Trigo",val:"USD "+FALLBACK.wheat+" /bushel"},
{label:"🌽 Maíz",val:"USD "+FALLBACK.corn+" /ton"},
].map(function(r){return'<div class="card" style="padding:.75rem 1rem"><div style="font-size:.75rem;color:var(--muted);margin-bottom:.25rem">'+r.label+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:.95rem;color:var(--accent)">'+r.val+'</div></div>';}).join('')+
'</div>'+

'<div id="apiStatus"></div>'+
'<div class="bottom-grid">'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Facturas recientes</div>'+INVOICES.slice(0,4).map(function(inv){return'<div style="display:flex;justify-content:space-between;padding:.5rem 0;border-bottom:1px solid var(--border)"><div><div style="font-size:.82rem">'+inv.client+'</div><div style="font-size:.7rem;color:var(--faint)">'+inv.id+' · '+inv.date+'</div></div><div style="text-align:right"><div style="font-size:.85rem;font-weight:600">$'+inv.amount.toLocaleString()+'</div><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+(inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente')+'</span></div></div>';}).join('')+'</div>'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Alertas activas</div>'+ALERTS.map(function(a){return'<div class="alert-item alert-'+a.type+'"><div style="font-size:.8rem;font-weight:600;margin-bottom:.15rem">'+a.title+'</div><div style="font-size:.72rem;color:var(--muted)">'+a.desc+'</div><div style="font-size:.67rem;color:var(--faint);margin-top:.2rem">'+a.time+'</div></div>';}).join('')+'</div>'+
'</div></div>';

try{const forecast=await fetchForecast(company.seed);
document.getElementById("kpiProjection").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Tendencia '+forecast.tendencia.toFixed(1)+'K/mes</div>';
document.getElementById("kpiAvg").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(Math.round(forecast.promedio*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Desde API en vivo</div>';
}catch(e){document.getElementById("apiStatus").innerHTML='<div class="card" style="padding:.9rem;margin-bottom:1rem;color:var(--danger);font-size:.82rem;border-color:#FF6B6B44">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

async function fetchForecast(values){const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values:values})});if(!res.ok)throw new Error("API "+res.status);return res.json();}

async function renderCashflow(main,company){
main.innerHTML='<div class="fade-in"><h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">Flujo de Fondos</h1><div id="cfContent"><span class="spinner"></span> Consultando tu API...</div></div>';
try{const forecast=await fetchForecast(company.seed);
document.getElementById("cfContent").innerHTML=
'<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:1rem;margin-bottom:1.25rem">'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio 6 meses</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.promedio*1000))+'</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Tendencia mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+(forecast.tendencia>0?'+':'')+forecast.tendencia.toFixed(1)+'K</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Próxima proyección</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div></div>'+
'</div>'+
'<div class="card" style="padding:1.25rem"><div style="font-weight:600;margin-bottom:1rem">Detalle mensual</div><table style="width:100%;border-collapse:collapse;font-size:.85rem"><thead><tr style="color:var(--faint);font-size:.72rem;text-transform:uppercase"><th style="text-align:left;padding:.5rem .75rem">Mes</th><th style="text-align:left;padding:.5rem .75rem">Real</th><th style="text-align:left;padding:.5rem .75rem">Proyectado</th><th style="text-align:left;padding:.5rem .75rem">Estado</th></tr></thead><tbody>'+
MONTHS.slice(0,6).map(function(m,i){return'<tr style="border-top:1px solid var(--border)"><td style="padding:.65rem .75rem;font-weight:500">'+m+'</td><td style="padding:.65rem .75rem">'+fmt(company.seed[i]*1000)+'</td><td style="padding:.65rem .75rem;color:var(--muted)">'+fmt(Math.round((company.seed[5]+forecast.tendencia*(i-5))*1000))+'</td><td style="padding:.65rem .75rem"><span class="badge badge-green">En curso</span></td></tr>';}).join('')+
'</tbody></table></div>';
}catch(e){document.getElementById("cfContent").innerHTML='<div style="color:var(--danger)">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

function renderMarkets(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">FX & Mercados</h1>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Tipo de Cambio <span style="color:var(--accent);font-size:.78rem;font-weight:400">· Actualizado hoy</span></div>'+
'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div style="display:grid;grid-template-columns:1fr 1fr 1fr 1fr;padding:.5rem .85rem;background:var(--bg);font-size:.7rem;color:var(--faint);text-transform:uppercase;font-weight:500">'+
'<span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
[
{label:"🇦🇷 Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"🇦🇷 Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"🇦🇷 Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"🇪🇺 Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(4),venta:(1/parseFloat(getRate("EUR"))).toFixed(4),var:"+0,08%",up:true},
{label:"🇧🇷 Real BRL",compra:getRate("BRL"),venta:(parseFloat(getRate("BRL"))*1.01).toFixed(4),var:"-0,12%",up:false},
{label:"🇲🇽 Peso MXN",compra:getRate("MXN"),venta:(parseFloat(getRate("MXN"))*1.01).toFixed(4),var:"+0,20%",up:true},
{label:"🇨🇴 Peso COP",compra:FALLBACK.COP,venta:FALLBACK.COP,var:"+0,10%",up:true},
{label:"🇨🇱 Peso CLP",compra:FALLBACK.CLP,venta:FALLBACK.CLP,var:"-0,05%",up:false},
{label:"🇬🇧 Libra GBP",compra:(1/parseFloat(getRate("GBP"))*0.99).toFixed(4),venta:(1/parseFloat(getRate("GBP"))).toFixed(4),var:"+0,15%",up:true},
{label:"🇨🇳 Yuan CNY",compra:getRate("CNY"),venta:(parseFloat(getRate("CNY"))*1.01).toFixed(4),var:"-0,08%",up:false},
].map(function(r){return'<div class="fx-table-row"><span style="font-size:.85rem;font-weight:500">'+r.label+'</span><span style="font-size:.85rem;font-weight:600">$'+r.compra+'</span><span style="font-size:.85rem;font-weight:600">$'+r.venta+'</span><span style="font-size:.82rem;color:'+(r.up?'var(--accent)':'var(--danger)')+'">'+r.var+'</span></div>';}).join('')+
'</div>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Commodities</div>'+
'<div class="com-grid">'+
[
{name:"Oro",unit:"troy oz",icon:"🥇",val:"USD "+getGold()},
{name:"Plata",unit:"troy oz",icon:"🥈",val:"USD "+getSilver()},
{name:"Petróleo",unit:"barril",icon:"🛢️",val:"USD "+FALLBACK.oil},
{name:"Soja",unit:"tonelada",icon:"🌾",val:"USD "+FALLBACK.soy},
{name:"Trigo",unit:"bushel",icon:"🌿",val:"USD "+FALLBACK.wheat},
{name:"Maíz",unit:"tonelada",icon:"🌽",val:"USD "+FALLBACK.corn},
].map(function(c){return'<div class="fx-card"><div style="font-size:1.4rem;margin-bottom:.4rem">'+c.icon+'</div><div style="font-size:.72rem;color:var(--faint);margin-bottom:.2rem">'+c.name+' / '+c.unit+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.2rem;color:var(--accent)">'+c.val+'</div></div>';}).join('')+
'</div></div>';
}

function renderInvoices(main){
main.innerHTML='<div class="fade-in">'<script>
const AV_KEY="42ac3700f76949f6899936a3eb2ad9d2";
const API_BASE="https://contabilidad-de-datos.onrender.com";
const MONTHS=["Ene","Feb","Mar","Abr","May","Jun","Jul","Ago","Sep","Oct","Nov","Dic"];
const COMPANIES=[{name:"El Clavo Ferretería",sector:"Retail",plan:"Growth",seed:[210,195,230,215,240,225],currency:"ARS"},{name:"Soto Import SA",sector:"Importación",plan:"Scale",seed:[450,480,510,490,530,505],currency:"USD"},{name:"Dubois Consulting",sector:"Servicios",plan:"Starter",seed:[80,95,88,102,91,110],currency:"BRL"}];
const INVOICES=[{id:"FAC-001",client:"BuildCo Ltd",amount:12400,date:"10 Jun",status:"paid"},{id:"FAC-002",client:"Suministros Metro",amount:8750,date:"08 Jun",status:"pending"},{id:"FAC-003",client:"Grupo Alfa",amount:31200,date:"05 Jun",status:"paid"},{id:"FAC-004",client:"Obras Municipales",amount:5600,date:"01 Jun",status:"overdue"},{id:"FAC-005",client:"TechParts SA",amount:19800,date:"28 May",status:"paid"},{id:"FAC-006",client:"Harbor Imports",amount:44100,date:"25 May",status:"pending"}];
const ALERTS=[{type:"danger",title:"Riesgo de flujo de caja detectado",desc:"Se proyecta caída por debajo del umbral en mayo.",time:"Hace 2h"},{type:"warn",title:"Factura vencida — Obras Municipales",desc:"FAC-004 por $5.600 tiene 14 días de atraso.",time:"Hace 1d"},{type:"ok",title:"Objetivo mensual alcanzado",desc:"El ingreso de junio superó la proyección un 12,4%.",time:"Hace 2d"},{type:"ok",title:"Oportunidad en tipo de cambio",desc:"Diferencial favorable en USD para pedidos de importación.",time:"Hace 3d"}];
const NAV=[{id:"dashboard",icon:"⬡",label:"Panel"},{id:"markets",icon:"◇",label:"FX & Mercados"},{id:"invoices",icon:"◈",label:"Facturas"},{id:"alerts",icon:"△",label:"Alertas",badge:2},{id:"cashflow",icon:"◎",label:"Flujo de Caja"}];

// Precios actualizados 25 Jul 2026
const FALLBACK={
  EUR:"0.8790",BRL:"5.0827",MXN:"17.48",GBP:"0.7505",CNY:"6.7722",
  ARS_BNA_COMPRA:"1470",ARS_BNA_VENTA:"1520",
  ARS_BLUE_COMPRA:"1525",ARS_BLUE_VENTA:"1545",
  ARS_MEP:"1531",
  CLP:"945",COP:"3196",
  gold:"4.068",silver:"58,88",
  oil:"73,40",soy:"281",wheat:"5,64",corn:"281"
};

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
try{const g=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(g["Realtime Currency Exchange Rate"]){mktData.gold=parseFloat(g["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toLocaleString("es-AR",{maximumFractionDigits:2});}else{mktData.gold=FALLBACK.gold;}}catch(e){mktData.gold=FALLBACK.gold;}
try{const s=await fetch("https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAG&to_currency=USD&apikey="+AV_KEY).then(function(r){return r.json();});if(s["Realtime Currency Exchange Rate"]){mktData.silver=parseFloat(s["Realtime Currency Exchange Rate"]["5. Exchange Rate"]).toFixed(2);}else{mktData.silver=FALLBACK.silver;}}catch(e){mktData.silver=FALLBACK.silver;}
renderTicker();
if(currentPage==="dashboard"||currentPage==="markets")renderPage(currentPage);
}

function renderTicker(){
const track=document.getElementById("tickerTrack");
const items=[
{label:"USD/ARS BNA",val:"V $"+FALLBACK.ARS_BNA_VENTA+" | C $"+FALLBACK.ARS_BNA_COMPRA,up:false},
{label:"USD/ARS Blue",val:"V $"+FALLBACK.ARS_BLUE_VENTA+" | C $"+FALLBACK.ARS_BLUE_COMPRA,up:false},
{label:"Dólar MEP",val:"$"+FALLBACK.ARS_MEP,up:true},
{label:"EUR/USD",val:(1/parseFloat(getRate("EUR"))).toFixed(4),up:true},
{label:"USD/BRL",val:getRate("BRL"),up:false},
{label:"USD/MXN",val:getRate("MXN"),up:false},
{label:"USD/CLP",val:FALLBACK.CLP,up:false},
{label:"USD/COP",val:FALLBACK.COP,up:false},
{label:"GBP/USD",val:(1/parseFloat(getRate("GBP"))).toFixed(4),up:true},
{label:"ORO oz",val:"USD "+getGold(),up:true},
{label:"PLATA oz",val:"USD "+getSilver(),up:true},
{label:"SOJA",val:"USD "+FALLBACK.soy,up:false},
{label:"TRIGO",val:"USD "+FALLBACK.wheat,up:true},
{label:"MAÍZ",val:"USD "+FALLBACK.corn,up:false},
{label:"PETRÓLEO",val:"USD "+FALLBACK.oil,up:true},
];
const all=items.concat(items);
track.innerHTML=all.map(function(i){return'<span class="ticker-item"><span style="color:var(--faint)">'+i.label+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-weight:600">'+i.val+'</span><span style="color:'+(i.up?'var(--accent)':'var(--danger)')+';font-size:.65rem">'+(i.up?'▲':'▼')+'</span></span>';}).join("");
}

function handleLogin(){
const email=document.getElementById("loginEmail").value;
const pass=document.getElementById("loginPass").value;
const err=document.getElementById("loginError");
if(!email||!pass){err.textContent="Por favor completá todos los campos.";err.style.display="block";return;}
err.style.display="none";
const btn=document.getElementById("loginBtn");
btn.innerHTML='<span class="spinner"></span> Ingresando...';btn.disabled=true;
setTimeout(function(){
currentUser={name:"Ezequiel Prilusky",email:email};
document.getElementById("userName").textContent=currentUser.name;
document.getElementById("loginScreen").classList.add("hidden");
document.getElementById("appShell").classList.remove("hidden");
document.getElementById("appShell").style.display="flex";
renderTicker();renderNav();renderPage("dashboard");loadMarkets();
},1100);
}

function handleLogout(){currentUser=null;document.getElementById("appShell").classList.add("hidden");document.getElementById("loginScreen").classList.remove("hidden");document.getElementById("loginEmail").value="";document.getElementById("loginPass").value="";document.getElementById("loginBtn").innerHTML="Ingresar";document.getElementById("loginBtn").disabled=false;}
function changeCompany(){currentCompany=parseInt(document.getElementById("companySelect").value);renderPage(currentPage);}
function renderNav(){const nav=document.getElementById("navMenu");nav.innerHTML=NAV.map(function(n){return'<div class="nav-item '+(currentPage===n.id?'active':'')+'" onclick="renderPage(\''+n.id+'\')"><span>'+n.icon+'</span><span style="flex:1">'+n.label+'</span>'+(n.badge?'<span class="badge badge-red">'+n.badge+'</span>':'')+'</div>';}).join("");document.getElementById("userPlan").textContent=COMPANIES[currentCompany].plan+" plan";}
function renderPage(page){currentPage=page;renderNav();const company=COMPANIES[currentCompany];const main=document.getElementById("mainContent");if(page==="dashboard")renderDashboard(main,company);else if(page==="cashflow")renderCashflow(main,company);else if(page==="markets")renderMarkets(main);else if(page==="invoices")renderInvoices(main);else if(page==="alerts")renderAlerts(main);}

async function renderDashboard(main,company){
const latest=company.seed[5]*1000,prev=company.seed[4]*1000;
const delta=(((latest-prev)/prev)*100).toFixed(1);
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:.25rem;">Panel Principal</h1>'+
'<p style="color:var(--muted);font-size:.82rem;margin-bottom:1.25rem;">'+company.name+' · '+company.sector+'</p>'+
'<div class="kpi-grid">'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Flujo de caja</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(latest)+'</div><div style="font-size:.75rem;color:'+(+delta>0?'var(--accent)':'var(--danger)')+'">'+( +delta>0?'▲':'▼')+' '+delta+'% vs mes anterior</div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiProjection"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem" id="kpiAvg"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div><span class="spinner"></span></div></div>'+
'<div class="card" style="padding:1rem 1.2rem"><div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Alertas activas</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">2</div><div style="font-size:.75rem;color:var(--danger)">▼ 1 crítica</div></div>'+
'</div>'+

'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">💱 Tipo de Cambio</div>'+
'<div class="card" style="margin-bottom:1rem;overflow:hidden">'+
[
{label:"Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(2),venta:(1/parseFloat(getRate("EUR"))).toFixed(2),var:"+0,08%",up:true},
{label:"Real BRL",compra:(parseFloat(getRate("BRL"))*0.99).toFixed(4),venta:getRate("BRL"),var:"-0,12%",up:false},
{label:"Peso CLP",compra:FALLBACK.CLP,venta:FALLBACK.CLP,var:"-0,05%",up:false},
{label:"Peso COP",compra:FALLBACK.COP,venta:FALLBACK.COP,var:"+0,10%",up:true},
].map(function(r){return'<div class="fx-table-row"><span style="font-size:.85rem;font-weight:500;min-width:120px">'+r.label+'</span><span style="font-size:.82rem;color:var(--muted)">C: <strong style="color:var(--text)">$'+r.compra+'</strong></span><span style="font-size:.82rem;color:var(--muted)">V: <strong style="color:var(--text)">$'+r.venta+'</strong></span><span style="font-size:.78rem;color:'+(r.up?'var(--accent)':'var(--danger)')+'">'+r.var+'</span></div>';}).join('')+
'</div>'+

'<div style="font-size:.78rem;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.06em;margin-bottom:.6rem">🌾 Commodities</div>'+
'<div class="com-mini-grid">'+
[
{label:"🥇 Oro",val:"USD "+getGold()+" /oz"},
{label:"🥈 Plata",val:"USD "+getSilver()+" /oz"},
{label:"🛢️ Petróleo",val:"USD "+FALLBACK.oil+" /barril"},
{label:"🌾 Soja",val:"USD "+FALLBACK.soy+" /ton"},
{label:"🌿 Trigo",val:"USD "+FALLBACK.wheat+" /bushel"},
{label:"🌽 Maíz",val:"USD "+FALLBACK.corn+" /ton"},
].map(function(r){return'<div class="card" style="padding:.75rem 1rem"><div style="font-size:.75rem;color:var(--muted);margin-bottom:.25rem">'+r.label+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:.95rem;color:var(--accent)">'+r.val+'</div></div>';}).join('')+
'</div>'+

'<div id="apiStatus"></div>'+
'<div class="bottom-grid">'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Facturas recientes</div>'+INVOICES.slice(0,4).map(function(inv){return'<div style="display:flex;justify-content:space-between;padding:.5rem 0;border-bottom:1px solid var(--border)"><div><div style="font-size:.82rem">'+inv.client+'</div><div style="font-size:.7rem;color:var(--faint)">'+inv.id+' · '+inv.date+'</div></div><div style="text-align:right"><div style="font-size:.85rem;font-weight:600">$'+inv.amount.toLocaleString()+'</div><span class="badge badge-'+(inv.status==='paid'?'green':inv.status==='overdue'?'red':'yellow')+'">'+(inv.status==='paid'?'Pagada':inv.status==='overdue'?'Vencida':'Pendiente')+'</span></div></div>';}).join('')+'</div>'+
'<div class="card" style="padding:1.1rem"><div style="font-weight:600;font-size:.9rem;margin-bottom:.85rem">Alertas activas</div>'+ALERTS.map(function(a){return'<div class="alert-item alert-'+a.type+'"><div style="font-size:.8rem;font-weight:600;margin-bottom:.15rem">'+a.title+'</div><div style="font-size:.72rem;color:var(--muted)">'+a.desc+'</div><div style="font-size:.67rem;color:var(--faint);margin-top:.2rem">'+a.time+'</div></div>';}).join('')+'</div>'+
'</div></div>';

try{const forecast=await fetchForecast(company.seed);
document.getElementById("kpiProjection").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Proyección IA</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Tendencia '+forecast.tendencia.toFixed(1)+'K/mes</div>';
document.getElementById("kpiAvg").innerHTML='<div style="font-size:.68rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.6rem;font-weight:700">'+fmt(Math.round(forecast.promedio*1000))+'</div><div style="font-size:.75rem;color:var(--accent)">▲ Desde API en vivo</div>';
}catch(e){document.getElementById("apiStatus").innerHTML='<div class="card" style="padding:.9rem;margin-bottom:1rem;color:var(--danger);font-size:.82rem;border-color:#FF6B6B44">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

async function fetchForecast(values){const res=await fetch(API_BASE+"/forecast",{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({values:values})});if(!res.ok)throw new Error("API "+res.status);return res.json();}

async function renderCashflow(main,company){
main.innerHTML='<div class="fade-in"><h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">Flujo de Fondos</h1><div id="cfContent"><span class="spinner"></span> Consultando tu API...</div></div>';
try{const forecast=await fetchForecast(company.seed);
document.getElementById("cfContent").innerHTML=
'<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:1rem;margin-bottom:1.25rem">'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Promedio 6 meses</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.promedio*1000))+'</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Tendencia mensual</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+(forecast.tendencia>0?'+':'')+forecast.tendencia.toFixed(1)+'K</div></div>'+
'<div class="card" style="padding:1rem 1.25rem"><div style="font-size:.7rem;color:var(--faint);text-transform:uppercase;margin-bottom:.4rem">Próxima proyección</div><div style="font-family:\'Space Grotesk\',sans-serif;font-size:1.5rem;font-weight:700;color:var(--accent)">'+fmt(Math.round(forecast.proyeccion*1000))+'</div></div>'+
'</div>'+
'<div class="card" style="padding:1.25rem"><div style="font-weight:600;margin-bottom:1rem">Detalle mensual</div><table style="width:100%;border-collapse:collapse;font-size:.85rem"><thead><tr style="color:var(--faint);font-size:.72rem;text-transform:uppercase"><th style="text-align:left;padding:.5rem .75rem">Mes</th><th style="text-align:left;padding:.5rem .75rem">Real</th><th style="text-align:left;padding:.5rem .75rem">Proyectado</th><th style="text-align:left;padding:.5rem .75rem">Estado</th></tr></thead><tbody>'+
MONTHS.slice(0,6).map(function(m,i){return'<tr style="border-top:1px solid var(--border)"><td style="padding:.65rem .75rem;font-weight:500">'+m+'</td><td style="padding:.65rem .75rem">'+fmt(company.seed[i]*1000)+'</td><td style="padding:.65rem .75rem;color:var(--muted)">'+fmt(Math.round((company.seed[5]+forecast.tendencia*(i-5))*1000))+'</td><td style="padding:.65rem .75rem"><span class="badge badge-green">En curso</span></td></tr>';}).join('')+
'</tbody></table></div>';
}catch(e){document.getElementById("cfContent").innerHTML='<div style="color:var(--danger)">⚠ '+e.message+' — Render iniciando. Recargá en 30s.</div>';}
}

function renderMarkets(main){
main.innerHTML='<div class="fade-in">'+
'<h1 style="font-family:\'Space Grotesk\',sans-serif;font-size:1.4rem;font-weight:700;margin-bottom:1.25rem;">FX & Mercados</h1>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Tipo de Cambio <span style="color:var(--accent);font-size:.78rem;font-weight:400">· Actualizado hoy</span></div>'+
'<div class="card" style="margin-bottom:1.5rem;overflow:hidden">'+
'<div style="display:grid;grid-template-columns:1fr 1fr 1fr 1fr;padding:.5rem .85rem;background:var(--bg);font-size:.7rem;color:var(--faint);text-transform:uppercase;font-weight:500">'+
'<span>Moneda</span><span>Compra</span><span>Venta</span><span>Var.</span></div>'+
[
{label:"🇦🇷 Dólar BNA",compra:FALLBACK.ARS_BNA_COMPRA,venta:FALLBACK.ARS_BNA_VENTA,var:"+0,66%",up:true},
{label:"🇦🇷 Dólar Blue",compra:FALLBACK.ARS_BLUE_COMPRA,venta:FALLBACK.ARS_BLUE_VENTA,var:"-0,32%",up:false},
{label:"🇦🇷 Dólar MEP",compra:FALLBACK.ARS_MEP,venta:FALLBACK.ARS_MEP,var:"+0,78%",up:true},
{label:"🇪🇺 Euro",compra:(1/parseFloat(getRate("EUR"))*0.99).toFixed(4),venta:(1/parseFloat(getRate("EUR"))).toFixed(4),var:"+0,08%",up:true},
{label:"🇧🇷 Real BRL",compra:getRate("BRL"),venta:(parseFloat(getRate("BRL"))*1.01).toFixed(4),var:"-0,12%",up:false},
{label:"🇲🇽 Peso MXN",compra:getRate("MXN"),venta:(parseFloat(getRate("MXN"))*1.01).toFixed(4),var:"+0,20%",up:true},
{label:"🇨🇴 Peso COP",compra:FALLBACK.COP,venta:FALLBACK.COP,var:"+0,10%",up:true},
{label:"🇨🇱 Peso CLP",compra:FALLBACK.CLP,venta:FALLBACK.CLP,var:"-0,05%",up:false},
{label:"🇬🇧 Libra GBP",compra:(1/parseFloat(getRate("GBP"))*0.99).toFixed(4),venta:(1/parseFloat(getRate("GBP"))).toFixed(4),var:"+0,15%",up:true},
{label:"🇨🇳 Yuan CNY",compra:getRate("CNY"),venta:(parseFloat(getRate("CNY"))*1.01).toFixed(4),var:"-0,08%",up:false},
].map(function(r){return'<div class="fx-table-row"><span style="font-size:.85rem;font-weight:500">'+r.label+'</span><span style="font-size:.85rem;font-weight:600">$'+r.compra+'</span><span style="font-size:.85rem;font-weight:600">$'+r.venta+'</span><span style="font-size:.82rem;color:'+(r.up?'var(--accent)':'var(--danger)')+'">'+r.var+'</span></div>';}).join('')+
'</div>'+
'<div style="font-weight:600;font-size:.85rem;margin-bottom:.75rem">Commodities</div>'+
'<div class="com-grid">'+
[
{name:"Oro",unit:"troy oz",icon:"🥇",val:"USD "+getGold()},
{name:"Plata",unit:"troy oz",icon:"🥈",val:"USD "+getSilver()},
{name:"Petróleo",unit:"barril",icon:"🛢️",val:"USD "+FALLBACK.oil},
{name:"Soja",unit:"tonelada",icon:"🌾",val:"USD "+FALLBACK.soy},
{name:"Trigo",unit:"bushel",icon:"🌿",val:"USD "+FALLBACK.wheat},
{name:"Maíz",unit:"tonelada",icon:"🌽",val:"USD "+FALLBACK.corn},
].map(function(c){return'<div class="fx-card"><div style="font-size:1.4rem;margin-bottom:.4rem">'+c.icon+'</div><div style="font-size:.72rem;color:var(--faint);margin-bottom:.2rem">'+c.name+' / '+c.unit+'</div><div style="font-family:\'Space Grotesk\',sans-serif;font-weight:700;font-size:1.2rem;color:var(--accent)">'+c.val+'</div></div>';}).join('')+
'</div></div>';
}

function renderInvoices(main){
main.innerHTML='<div class="fade-in">'