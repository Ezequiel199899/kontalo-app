<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Kontalo — Financial Intelligence</title>
<style>
:root{--bg:#071019;--panel:#0d1824;--panel2:#111f2d;--line:#1d3040;--text:#e9f0f7;--muted:#8fa2b4;--green:#00e5a0;--green2:#0d2d25;--red:#ff6b6b;--yellow:#f6b73c;--blue:#64b5ff}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:Inter,system-ui,-apple-system,sans-serif}
button,input,select{font:inherit}.app{min-height:100vh}.top{height:64px;border-bottom:1px solid var(--line);display:flex;align-items:center;justify-content:space-between;padding:0 22px;background:#09131e;position:sticky;top:0;z-index:10}.logo{font-weight:800;font-size:22px;letter-spacing:-.5px}.logo span{color:var(--green)}.top-right{display:flex;gap:10px;align-items:center}.pill{border:1px solid var(--line);background:var(--panel);padding:7px 10px;border-radius:8px;color:var(--muted);font-size:12px}.layout{display:grid;grid-template-columns:225px 1fr}.side{border-right:1px solid var(--line);min-height:calc(100vh - 64px);padding:18px 12px;background:#09131e}.nav{width:100%;text-align:left;border:0;background:transparent;color:var(--muted);padding:11px 12px;border-radius:8px;cursor:pointer;margin-bottom:4px}.nav:hover,.nav.active{background:var(--green2);color:var(--green)}.main{padding:24px;max-width:1500px;width:100%;margin:auto}.page-title{font-size:26px;font-weight:750;margin-bottom:4px}.sub{color:var(--muted);font-size:13px;margin-bottom:22px}.grid4{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}.grid3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}.card{background:var(--panel);border:1px solid var(--line);border-radius:12px;padding:16px}.label{font-size:12px;color:var(--muted)}.value{font-size:25px;font-weight:750;margin-top:7px}.positive{color:var(--green)}.negative{color:var(--red)}.warning{color:var(--yellow)}.section{margin-top:18px}.section h2{font-size:15px;margin:0 0 10px}.table{width:100%;border-collapse:collapse}.table th,.table td{padding:11px 8px;border-bottom:1px solid var(--line);font-size:12px;text-align:left}.table th{color:var(--muted);font-weight:600}.badge{padding:4px 8px;border-radius:999px;font-size:10px;font-weight:700}.bgreen{background:var(--green2);color:var(--green)}.bred{background:#321b22;color:var(--red)}.byellow{background:#332914;color:var(--yellow)}.toolbar{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:12px}.btn{border:1px solid var(--line);background:var(--panel2);color:var(--text);padding:9px 12px;border-radius:8px;cursor:pointer}.btn.primary{background:var(--green);color:#06120e;border-color:var(--green);font-weight:700}.btn:hover{filter:brightness(1.1)}.form{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:14px}.input{background:#08121c;color:var(--text);border:1px solid var(--line);padding:9px;border-radius:8px;width:100%}.market{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}.market .card{padding:13px}.small{font-size:11px;color:var(--muted)}.alert{border-left:3px solid var(--yellow);background:#0e1a24;padding:12px;border-radius:0 8px 8px 0;margin-bottom:8px}.alert.ok{border-color:var(--green)}.alert.bad{border-color:var(--red)}.empty{color:var(--muted);padding:25px;text-align:center}.footer{margin-top:24px;color:#617589;font-size:11px}.error{color:var(--red);font-size:11px;margin-top:7px}
@media(max-width:900px){.layout{grid-template-columns:1fr}.side{display:flex;overflow:auto;min-height:auto;border-right:0;border-bottom:1px solid var(--line);gap:5px}.nav{min-width:max-content}.grid4,.grid3,.market{grid-template-columns:1fr 1fr}.form{grid-template-columns:1fr 1fr}}
@media(max-width:560px){.main{padding:15px}.grid4,.grid3,.market,.form{grid-template-columns:1fr}.top{padding:0 13px}}
</style>
</head>
<body>
<div class="app">
<header class="top">
  <div class="logo">kontalo<span>.</span></div>
  <div class="top-right"><div class="pill" id="bcraStatus">BCRA: cargando…</div><div class="pill">El Clavo Hardware · Growth</div></div>
</header>
<div class="layout">
<aside class="side" id="nav"></aside>
<main class="main" id="main"></main>
</div>
</div>

<script>
const BCRA="https://api.bcra.gob.ar/estadisticascambiarias/v1.0/Cotizaciones";
const state={
  page:"dashboard",
  cash:12845000,
  invoices:[
    {id:"INV-001",client:"BuildCo Ltd",amount:1240000,due:"2026-09-05",status:"Pendiente"},
    {id:"INV-002",client:"Metro Supplies",amount:875000,due:"2026-08-18",status:"Vencida"},
    {id:"INV-003",client:"Alpha Group",amount:3120000,due:"2026-09-12",status:"Pagada"}
  ],
  stock:[
    {sku:"SKU-001",name:"Taladro profesional",qty:24,min:10,cost:185000},
    {sku:"SKU-002",name:"Amoladora 900W",qty:7,min:12,cost:132000},
    {sku:"SKU-003",name:"Disco de corte",qty:80,min:30,cost:4200}
  ],
  budgets:[
    {name:"Marketing",budget:500000,spent:420000},
    {name:"Compras",budget:3500000,spent:2980000},
    {name:"Operación",budget:1800000,spent:1710000}
  ],
  investments:[
    {name:"Plazo fijo",amount:2000000,rate:32},
    {name:"FCI money market",amount:850000,rate:28}
  ],
  fx:{},
  fxDate:null,
  commodities:[
    {name:"Oro",symbol:"XAU",value:"—",source:"Proveedor de commodities"},
    {name:"Plata",symbol:"XAG",value:"—",source:"Proveedor de commodities"},
    {name:"Petróleo",symbol:"WTI",value:"—",source:"Proveedor de commodities"},
    {name:"Soja",symbol:"SOY",value:"—",source:"Proveedor de commodities"},
    {name:"Trigo",symbol:"WHEAT",value:"—",source:"Proveedor de commodities"},
    {name:"Maíz",symbol:"CORN",value:"—",source:"Proveedor de commodities"}
  ]
};

const navItems=[
 ["dashboard","⬡","Dashboard"],["cashflow","◎","Flujo de caja"],["invoices","◈","Facturación"],
 ["stock","▦","Stock"],["budget","▤","Presupuesto"],["investments","◇","Inversiones"],
 ["markets","◌","Cotizaciones"],["commodities","◆","Commodities"],["alerts","△","Alertas"]
];

function money(n){return new Intl.NumberFormat("es-AR",{style:"currency",currency:"ARS",maximumFractionDigits:0}).format(n)}
function esc(s){return String(s).replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
function save(){localStorage.setItem("kontalo_v2",JSON.stringify({invoices:state.invoices,stock:state.stock,budgets:state.budgets,investments:state.investments}))}
function load(){try{const x=JSON.parse(localStorage.getItem("kontalo_v2")||"null");if(x){Object.assign(state,x)}}catch(e){}}
function nav(){document.getElementById("nav").innerHTML=navItems.map(x=>`<button class="nav ${state.page===x[0]?"active":""}" onclick="go('${x[0]}')">${x[1]} &nbsp; ${x[2]}</button>`).join("")}
function go(p){state.page=p;nav();render()}

function kpi(label,value,extra=""){return `<div class="card"><div class="label">${label}</div><div class="value">${value}</div><div class="small">${extra}</div></div>`}

function dashboard(){
 const receivable=state.invoices.filter(x=>x.status!=="Pagada").reduce((a,x)=>a+x.amount,0);
 const overdue=state.invoices.filter(x=>x.status==="Vencida").reduce((a,x)=>a+x.amount,0);
 const stockValue=state.stock.reduce((a,x)=>a+x.qty*x.cost,0);
 return `<div class="page-title">Dashboard</div><div class="sub">Inteligencia financiera para pymes · datos, caja y decisiones en un solo lugar</div>
 <div class="grid4">${kpi("Caja actual",money(state.cash),"Saldo registrado")}
 ${kpi("Cuentas por cobrar",money(receivable),overdue?`<span class="negative">${money(overdue)} vencido</span>`:"Sin vencimientos críticos")}
 ${kpi("Inventario",money(stockValue),"Valor estimado a costo")}
 ${kpi("Alertas activas",String(state.stock.filter(x=>x.qty<x.min).length+state.invoices.filter(x=>x.status==="Vencida").length),"Monitoreo automático")}</div>
 <div class="grid3 section">
 <div class="card"><h2>🤖 IA financiera</h2><div class="alert bad">Riesgo: hay facturas vencidas y stock por debajo del mínimo.</div><div class="alert ok">Oportunidad: cobrar ${money(receivable)} mejoraría la posición de caja.</div><div class="small">La IA puede analizar estos datos y generar explicaciones, escenarios y recomendaciones. No toma decisiones financieras por sí sola.</div></div>
 <div class="card"><h2>📊 Proyección</h2><div class="value positive">${money(state.cash*1.08)}</div><div class="small">Escenario base a 30 días · demo</div><div style="margin-top:15px" class="small">Escenario pesimista: ${money(state.cash*.96)}<br>Escenario optimista: ${money(state.cash*1.15)}</div></div>
 <div class="card"><h2>🧩 Arquitectura</h2><div class="small">Frontend de una página</div><div class="small">Spring Boot · API</div><div class="small">Python/Flask · Forecasting/IA</div><div class="small">PostgreSQL · persistencia</div><div class="small">BCRA · cotizaciones públicas</div><div class="small">Docker · despliegue</div></div>
 </div>
 <div class="section card"><h2>Actividad reciente</h2><table class="table"><tr><th>Evento</th><th>Estado</th></tr>
 <tr><td>Actualización BCRA</td><td><span class="badge bgreen">Automática</span></td></tr>
 <tr><td>Factura INV-002</td><td><span class="badge bred">Vencida</span></td></tr>
 <tr><td>Stock Amoladora 900W</td><td><span class="badge byellow">Reponer</span></td></tr></table></div>`;
}

function cashflow(){return `<div class="page-title">Flujo de caja</div><div class="sub">Ingresos, egresos y escenarios para anticipar necesidades de liquidez</div>
<div class="grid4">${kpi("Caja inicial",money(state.cash),"Demo")}${kpi("Ingresos próximos",money(4670000),"Facturas pendientes")}${kpi("Egresos estimados",money(3210000),"Presupuesto operativo")}${kpi("Caja proyectada",money(state.cash+1460000),"Escenario base")}</div>
<div class="section card"><h2>Escenarios</h2><table class="table"><tr><th>Escenario</th><th>Proyección 30 días</th><th>Lectura</th></tr>
<tr><td>Pesimista</td><td class="negative">${money(state.cash*.96)}</td><td>Mayor demora en cobranzas</td></tr>
<tr><td>Base</td><td class="positive">${money(state.cash*1.08)}</td><td>Comportamiento esperado</td></tr>
<tr><td>Optimista</td><td class="positive">${money(state.cash*1.15)}</td><td>Cobranzas aceleradas</td></tr></table></div>`}

function invoices(){return `<div class="page-title">Facturación</div><div class="sub">Cuentas por cobrar y vencimientos</div>
<div class="toolbar"><button class="btn primary" onclick="addInvoice()">+ Nueva factura</button></div>
<div class="card"><table class="table"><tr><th>ID</th><th>Cliente</th><th>Importe</th><th>Vencimiento</th><th>Estado</th><th></th></tr>
${state.invoices.map((x,i)=>`<tr><td>${esc(x.id)}</td><td>${esc(x.client)}</td><td>${money(x.amount)}</td><td>${esc(x.due)}</td><td><span class="badge ${x.status==="Pagada"?"bgreen":x.status==="Vencida"?"bred":"byellow"}">${x.status}</span></td><td><button class="btn" onclick="toggleInvoice(${i})">Cambiar</button></td></tr>`).join("")}</table></div>`}
function addInvoice(){const client=prompt("Cliente:");if(!client)return;const amount=Number(prompt("Importe ARS:")||0);if(!amount)return;state.invoices.push({id:"INV-"+String(Date.now()).slice(-5),client,amount,due:new Date().toISOString().slice(0,10),status:"Pendiente"});save();render()}
function toggleInvoice(i){const s=["Pendiente","Pagada","Vencida"];state.invoices[i].status=s[(s.indexOf(state.invoices[i].status)+1)%3];save();render()}

function stock(){return `<div class="page-title">Stock</div><div class="sub">Inventario, costos y alertas de reposición</div>
<div class="toolbar"><button class="btn primary" onclick="addStock()">+ Producto</button></div><div class="card"><table class="table"><tr><th>SKU</th><th>Producto</th><th>Unidades</th><th>Mínimo</th><th>Costo</th><th>Valor</th><th>Estado</th></tr>
${state.stock.map(x=>`<tr><td>${esc(x.sku)}</td><td>${esc(x.name)}</td><td>${x.qty}</td><td>${x.min}</td><td>${money(x.cost)}</td><td>${money(x.qty*x.cost)}</td><td>${x.qty<x.min?'<span class="badge byellow">Reponer</span>':'<span class="badge bgreen">OK</span>'}</td></tr>`).join("")}</table></div>`}
function addStock(){const name=prompt("Producto:");if(!name)return;const qty=Number(prompt("Cantidad:")||0);const cost=Number(prompt("Costo unitario ARS:")||0);state.stock.push({sku:"SKU-"+String(Date.now()).slice(-5),name,qty,min:10,cost});save();render()}

function budget(){return `<div class="page-title">Presupuesto</div><div class="sub">Compará lo planificado con lo ejecutado</div><div class="grid3">${state.budgets.map(x=>{const pct=Math.round(x.spent/x.budget*100);return `<div class="card"><div class="label">${esc(x.name)}</div><div class="value">${pct}%</div><div class="small">${money(x.spent)} / ${money(x.budget)}</div><div style="height:7px;background:#162532;border-radius:9px;margin-top:12px"><div style="width:${Math.min(pct,100)}%;height:100%;background:${pct>100?"var(--red)":"var(--green)"};border-radius:9px"></div></div></div>`}).join("")}</div>`}

function investments(){return `<div class="page-title">Inversiones</div><div class="sub">Registro de inversiones de la empresa y seguimiento de rendimiento</div>
<div class="toolbar"><button class="btn primary" onclick="addInvestment()">+ Inversión</button></div><div class="card"><table class="table"><tr><th>Instrumento</th><th>Capital</th><th>Tasa/estimación</th><th>Rendimiento anual estimado</th></tr>
${state.investments.map(x=>`<tr><td>${esc(x.name)}</td><td>${money(x.amount)}</td><td>${x.rate}%</td><td class="positive">${money(x.amount*x.rate/100)}</td></tr>`).join("")}</table></div>
<div class="section card"><div class="small">Importante: esta sección registra inversiones y puede consumir datos de mercado, pero no ejecuta operaciones ni constituye asesoramiento financiero.</div></div>`}
function addInvestment(){const name=prompt("Instrumento:");if(!name)return;const amount=Number(prompt("Capital ARS:")||0);const rate=Number(prompt("Tasa/estimación %:")||0);state.investments.push({name,amount,rate});save();render()}

function markets(){const names=[["USD","Dólar"],["EUR","Euro"],["BRL","Real brasileño"],["CLP","Peso chileno"],["COP","Peso colombiano"],["GBP","Libra"],["CNY","Yuan"],["UYU","Peso uruguayo"]];return `<div class="page-title">Cotizaciones</div><div class="sub">Datos de divisas publicados por el BCRA · última cotización disponible</div>
<div class="toolbar"><button class="btn primary" onclick="loadBcra()">↻ Actualizar BCRA</button><span class="pill">${state.fxDate||"sin fecha"}</span></div>
<div class="market">${names.map(([c,n])=>`<div class="card"><div class="label">${c}</div><div style="font-weight:700;margin-top:4px">${n}</div><div class="value">${state.fx[c]??"—"}</div><div class="small">ARS por unidad · BCRA</div></div>`).join("")}</div>
<div class="section card"><h2>Fuente</h2><div class="small">BCRA API de Estadísticas Cambiarias. No requiere autenticación. La app no guarda claves secretas en el frontend.</div></div>`}

function commodities(){return `<div class="page-title">Commodities</div><div class="sub">Módulo preparado para conectar proveedores de precios sin exponer claves en el navegador</div><div class="market">${state.commodities.map(x=>`<div class="card"><div class="label">${x.symbol}</div><div style="font-weight:700;margin-top:4px">${x.name}</div><div class="value">${x.value}</div><div class="small">${x.source}</div></div>`).join("")}</div>
<div class="section card"><div class="small">Para precios de commodities en producción, el backend Spring/Flask debe consumir el proveedor elegido y almacenar/cachear los datos en PostgreSQL. No conviene poner una API key de commodities en index.html.</div></div>`}

function alerts(){const a=[];state.invoices.filter(x=>x.status==="Vencida").forEach(x=>a.push(`<div class="alert bad"><b>Factura vencida</b><br>${esc(x.client)} · ${money(x.amount)}</div>`));state.stock.filter(x=>x.qty<x.min).forEach(x=>a.push(`<div class="alert"><b>Stock bajo</b><br>${esc(x.name)} · ${x.qty} unidades (mínimo ${x.min})</div>`));if(!a.length)a.push(`<div class="alert ok">No hay alertas críticas.</div>`);return `<div class="page-title">Alertas</div><div class="sub">Kontalo detecta eventos que pueden afectar caja, inventario y presupuesto</div>${a.join("")}`}

function render(){const p={dashboard,cashflow,invoices,stock,budget,investments,markets,commodities,alerts}[state.page];document.getElementById("main").innerHTML=p?p():dashboard();nav()}
async function loadBcra(){
 document.getElementById("bcraStatus").textContent="BCRA: actualizando…";
 try{
  const r=await fetch(BCRA); if(!r.ok)throw new Error("HTTP "+r.status);
  const j=await r.json(); const d=j?.results?.detalle||[];
  const map={}; d.forEach(x=>map[x.codigoMoneda]=Number(x.tipoCotizacion));
  const codes={USD:"USD",EUR:"EUR",BRL:"BRL",CLP:"CLP",COP:"COP",GBP:"GBP",CNY:"CNY",UYU:"UYU"};
  Object.keys(codes).forEach(k=>{if(map[codes[k]]!=null)state.fx[k]=map[codes[k]].toLocaleString("es-AR",{minimumFractionDigits:2,maximumFractionDigits:4})});
  state.fxDate=j?.results?.fecha||null;document.getElementById("bcraStatus").textContent="BCRA: conectado";render();
 }catch(e){document.getElementById("bcraStatus").textContent="BCRA: sin conexión";render();console.warn(e)}
}
load();render();loadBcra();
</script>
</body>
</html>