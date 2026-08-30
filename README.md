 import os, sqlite3, requests
from flask import Flask, request, jsonify, render_template_string

app = Flask(__name__)
DB = os.getenv("DB_PATH", "kontalo.db")

HTML = r"""
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Kontalo</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#071019;color:#edf3f8;font-family:Arial,sans-serif}
.top{height:64px;background:#09131e;border-bottom:1px solid #1d3040;display:flex;align-items:center;justify-content:space-between;padding:0 22px}
.logo{font-size:23px;font-weight:800}.logo b{color:#00e5a0}
.pill{border:1px solid #1d3040;padding:7px 10px;border-radius:8px;color:#9db0c1;font-size:12px}
.layout{display:grid;grid-template-columns:220px 1fr}
.side{padding:15px;background:#09131e;min-height:calc(100vh - 64px);border-right:1px solid #1d3040}
.nav{display:block;width:100%;border:0;background:transparent;color:#9db0c1;text-align:left;padding:11px;border-radius:8px;margin:3px 0;cursor:pointer}
.nav:hover,.nav.active{background:#0d2d25;color:#00e5a0}
.main{padding:24px;max-width:1500px;width:100%;margin:auto}
.title{font-size:27px;font-weight:800}
.sub{color:#8fa2b4;font-size:13px;margin:4px 0 20px}
.grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
.grid3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
.card{background:#0d1824;border:1px solid #1d3040;border-radius:12px;padding:16px}
.label,.small{color:#8fa2b4;font-size:12px}
.value{font-size:24px;font-weight:800;margin:7px 0}
.ok{color:#00e5a0}.bad{color:#ff6b6b}.warn{color:#f6b73c}
.table{width:100%;border-collapse:collapse}
.table th,.table td{padding:10px 7px;border-bottom:1px solid #1d3040;font-size:12px;text-align:left}
.table th{color:#8fa2b4}
.btn{border:1px solid #1d3040;background:#111f2d;color:#edf3f8;padding:9px 12px;border-radius:8px;cursor:pointer}
.primary{background:#00e5a0;color:#06120e;font-weight:800}
.toolbar{display:flex;gap:8px;margin-bottom:12px;flex-wrap:wrap}
.alert{padding:11px;margin:7px 0;border-left:3px solid #f6b73c;background:#101c27;border-radius:0 8px 8px 0}
.alert.bad{border-left-color:#ff6b6b}.alert.ok{border-left-color:#00e5a0}
.market{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}
.badge{font-size:10px;padding:4px 8px;border-radius:999px;background:#332914;color:#f6b73c}
.green{background:#0d2d25;color:#00e5a0}.red{background:#321b22;color:#ff6b6b}
@media(max-width:900px){
.layout{grid-template-columns:1fr}
.side{display:flex;overflow:auto;min-height:auto}
.nav{min-width:max-content}
.grid,.grid3,.market{grid-template-columns:1fr 1fr}
}
@media(max-width:550px){
.grid,.grid3,.market{grid-template-columns:1fr}
.main{padding:14px}
}
</style>
</head>

<body>

<header class="top">
<div class="logo">kontalo<b>.</b></div>
<div class="pill">BCRA · IA · PostgreSQL-ready</div>
</header>

<div class="layout">

<aside class="side" id="nav"></aside>

<main class="main" id="main"></main>

</div>

<script>

const menus=[
["dashboard","⬡ Dashboard"],
["cashflow","◎ Flujo de caja"],
["invoices","◈ Facturación"],
["stock","▦ Stock"],
["budget","▤ Presupuesto"],
["investments","◇ Inversiones"],
["markets","◌ Cotizaciones BCRA"],
["commodities","◆ Commodities"],
["alerts","△ Alertas"]
];

let page="dashboard";

function nav(){
document.getElementById("nav").innerHTML=
menus.map(x=>
`<button class="nav ${page===x[0]?"active":""}" onclick="go('${x[0]}')">${x[1]}</button>`
).join("");
}

function money(n){
return new Intl.NumberFormat("es-AR",{
style:"currency",
currency:"ARS",
maximumFractionDigits:0
}).format(n);
}

async function api(url,options){
const response=await fetch(url,options);
return response.json();
}

async function go(p){
page=p;
nav();
await render();
}

function card(title,value,description=""){
return `
<div class="card">
<div class="label">${title}</div>
<div class="value">${value}</div>
<div class="small">${description}</div>
</div>`;
}

async function render(){

let html="";

if(page==="dashboard"){

html=`

<div class="title">Dashboard</div>

<div class="sub">
Kontalo centraliza caja, facturación, stock, presupuesto,
inversiones y mercados.
</div>

<div class="grid">

${card("Caja actual",money(12845000),"Saldo registrado")}

${card("Por cobrar",money(5115000),"Facturas pendientes")}

${card("Inventario",money(5450000),"Valor estimado a costo")}

${card("Alertas","3","Vencimientos y stock")}

</div>

<div class="grid3" style="margin-top:18px">

<div class="card">

<b>🤖 IA financiera</b>

<div class="alert bad">
Hay facturas vencidas que pueden afectar la caja.
</div>

<div class="alert ok">
La IA podrá analizar datos y proyectar escenarios.
</div>

</div>

<div class="card">

<b>📊 Proyección</b>

<div class="value ok">
${money(13872600)}
</div>

<div class="small">
Escenario base a 30 días.
</div>

</div>

<div class="card">

<b>🏗 Arquitectura</b>

<p class="small">
Frontend → Flask/API
</p>

<p class="small">
API → PostgreSQL
</p>

<p class="small">
IA → Forecasting
</p>

<p class="small">
BCRA → Cotizaciones
</p>

</div>

</div>
`;

}

else if(page==="cashflow"){

html=`

<div class="title">Flujo de caja</div>

<div class="sub">
Control y proyección de liquidez.
</div>

<div class="grid">

${card("Caja inicial",money(12845000))}

${card("Ingresos",money(4670000))}

${card("Egresos",money(3210000))}

${card("Caja proyectada",money(14260000))}

</div>

<div class="card" style="margin-top:18px">

<b>Escenarios</b>

<table class="table">

<tr>
<th>Escenario</th>
<th>Proyección</th>
<th>Lectura</th>
</tr>

<tr>
<td>Pesimista</td>
<td class="bad">${money(12331200)}</td>
<td>Menores cobranzas</td>
</tr>

<tr>
<td>Base</td>
<td class="ok">${money(13872600)}</td>
<td>Comportamiento esperado</td>
</tr>

<tr>
<td>Optimista</td>
<td class="ok">${money(14771750)}</td>
<td>Cobranzas aceleradas</td>
</tr>

</table>

</div>
`;

}

else if(page==="invoices"){

const data=await api("/api/invoices");

html=`

<div class="title">Facturación</div>

<div class="sub">
Cuentas por cobrar y vencimientos.
</div>

<div class="toolbar">

<button class="btn primary" onclick="addInvoice()">
+ Nueva factura
</button>

</div>

<div class="card">

<table class="table">

<tr>
<th>ID</th>
<th>Cliente</th>
<th>Importe</th>
<th>Vencimiento</th>
<th>Estado</th>
</tr>

${data.map(x=>`

<tr>

<td>${x.id}</td>

<td>${x.client}</td>

<td>${money(x.amount)}</td>

<td>${x.due}</td>

<td>

<span class="badge ${
x.status==="Pagada"
?"green"
:x.status==="Vencida"
?"red"
:""
}">

${x.status}

</span>

</td>

</tr>

`).join("")}

</table>

</div>
`;

}

else if(page==="stock"){

const data=await api("/api/stock");

html=`

<div class="title">Stock</div>

<div class="sub">
Inventario y reposición.
</div>

<div class="toolbar">

<button class="btn primary" onclick="addStock()">
+ Producto
</button>

</div>

<div class="card">

<table class="table">

<tr>
<th>SKU</th>
<th>Producto</th>
<th>Unidades</th>
<th>Mínimo</th>
<th>Costo</th>
<th>Estado</th>
</tr>

${data.map(x=>`

<tr>

<td>${x.sku}</td>

<td>${x.name}</td>

<td>${x.qty}</td>

<td>${x.min}</td>

<td>${money(x.cost)}</td>

<td>

${
x.qty<x.min
?'<span class="badge">Reponer</span>'
:'<span class="badge green">OK</span>'
}

</td>

</tr>

`).join("")}

</table>

</div>
`;

}

else if(page==="budget"){

const data=await api("/api/budgets");

html=`

<div class="title">Presupuesto</div>

<div class="sub">
Planificado contra ejecutado.
</div>

<div class="grid3">

${data.map(x=>{

const percentage=Math.round(
x.spent/x.budget*100
);

return `

<div class="card">

<b>${x.name}</b>

<div class="value">
${percentage}%
</div>

<div class="small">
${money(x.spent)} / ${money(x.budget)}
</div>