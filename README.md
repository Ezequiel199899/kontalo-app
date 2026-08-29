
  > import os, sqlite3, requests
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
.logo{font-size:22px;font-weight:800}.logo b{color:#00e5a0}
.pill{border:1px solid #1d3040;padding:7px 10px;border-radius:8px;color:#9db0c1;font-size:12px}
.layout{display:grid;grid-template-columns:220px 1fr}
.side{padding:15px;background:#09131e;min-height:calc(100vh - 64px);border-right:1px solid #1d3040}
.nav{display:block;width:100%;border:0;background:transparent;color:#9db0c1;text-align:left;padding:11px;border-radius:8px;margin:3px 0;cursor:pointer}
.nav:hover,.nav.active{background:#0d2d25;color:#00e5a0}
.main{padding:24px;max-width:1500px;width:100%;margin:auto}
.title{font-size:27px;font-weight:800}.sub,.small{color:#8fa2b4;font-size:12px}
.sub{margin:4px 0 20px}
.grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
.grid3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
.card{background:#0d1824;border:1px solid #1d3040;border-radius:12px;padding:16px}
.value{font-size:24px;font-weight:800;margin:7px 0}
.ok{color:#00e5a0}.bad{color:#ff6b6b}
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
.layout{grid-template-columns:1fr}.side{display:flex;overflow:auto;min-height:auto}
.nav{min-width:max-content}.grid,.grid3,.market{grid-template-columns:1fr 1fr}
}
@media(max-width:550px){
.grid,.grid3,.market{grid-template-columns:1fr}.main{padding:14px}
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

const nav=()=>{
document.getElementById("nav").innerHTML=
menus.map(x=>`
<button class="nav ${page==x[0]?"active":""}"
onclick="go('${x[0]}')">${x[1]}</button>
`).join("");
};

const money=n=>
new Intl.NumberFormat("es-AR",{
style:"currency",
currency:"ARS",
maximumFractionDigits:0
}).format(n);

async function api(u,o){
let r=await fetch(u,o);
return r.json();
}

async function go(p){
page=p;
nav();
render();
}

function card(a,b,c=""){
return `
<div class="card">
<div class="small">${a}</div>
<div class="value">${b}</div>
<div class="small">${c}</div>
</div>`;
}

async function render(){

let h="";

if(page=="dashboard"){

h=`
<div class="title">Dashboard</div>
<div class="sub">
Caja, facturación, stock, presupuesto, inversiones y mercados.
</div>

<div class="grid">

${card("Caja actual",money(12845000),"Demo")}

${card("Por cobrar",money(5115000),"Facturas pendientes")}

${card("Inventario",money(5450000),"Valor a costo")}

${card("Alertas","3","Vencimientos y stock")}

</div>

<div class="grid3" style="margin-top:18px">

<div class="card">
<b>🤖 IA financiera</b>

<div class="alert bad">
Hay facturas vencidas que pueden afectar la caja.
</div>

<div class="alert ok">
La IA podrá explicar variaciones y proyectar escenarios.
</div>

</div>

<div class="card">
<b>📊 Proyección</b>

<div class="value ok">
${money(13872600)}
</div>

<div class="small">
Escenario base a 30 días
</div>

</div>

<div class="card">
<b>🏗 Arquitectura</b>

<p class="small">
Frontend → API → Base de datos
</p>

<p class="small">
BCRA → cotizaciones
</p>

<p class="small">
IA / Forecasting → análisis
</p>

</div>

</div>
`;

}

else if(page=="cashflow"){

h=`
<div class="title">Flujo de caja</div>
<div class="sub">Control y proyección de liquidez.</div>

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
<td>Esperado</td>
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

else if(page=="invoices"){

let d=await api("/api/invoices");

h=`
<div class="title">Facturación</div>
<div class="sub">Cuentas por cobrar y vencimientos.</div>

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

${d.map(x=>`

<tr>
<td>${x.id}</td>
<td>${x.client}</td>
<td>${money(x.amount)}</td>
<td>${x.due}</td>

<td>
<span class="badge ${
x.status=="Pagada"
?"green"
:x.status=="Vencida"
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

else if(page=="stock"){

let d=await api("/api/stock");

h=`
<div class="title">Stock</div>
<div class="sub">Inventario y reposición.</div>

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

${d.map(x=>`

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

else if(page=="budget"){

let d=await api("/api/budgets");

h=`
<div class="title">Presupuesto</div>
<div class="sub">Planificado contra ejecutado.</div>

<div class="grid3">

${d.map(x=>{

let p=Math.round(x.spent/x.budget*100);

return`

<div class="card">

<b>${x.name}</b>

<div class="value">
${p}%
</div>

<div class="small">
${money(x.spent)} / ${money(x.budget)}
</div>

</div>

`;

}).join("")}

</div>
`;

}

else if(page=="investments"){

let d=await api("/api/investments");

h=`
<div class="title">Inversiones</div>
<div class="sub">Registro y rendimiento estimado.</div>

<div class="card">

<table class="table">

<tr>
<th>Instrumento</th>
<th>Capital</th>
<th>Tasa</th>
<th>Rendimiento anual estimado</th>
</tr>

${d.map(x=>`

<tr>

<td>${x.name}</td>

<td>${money(x.amount)}</td>

<td>${x.rate}%</td>

<td class="ok">
${money(x.amount*x.rate/100)}
</td>

</tr>

`).join("")}

</table>

</div>
`;

}

else if(page=="markets"){

let d=await api("/api/bcra");

h=`
<div class="title">Cotizaciones BCRA</div>

<div class="sub">
Cotizaciones obtenidas desde el backend mediante la API pública del BCRA.
</div>

<div class="market">

${Object.entries(d).map(([k,v])=>`

<div class="card">

<div class="small">${k}</div>

<div class="value">
${v??"—"}
</div>

<div class="small">
ARS · BCRA
</div>

</div>

`).join("")}

</div>
`;

}

else if(page=="commodities"){

h=`
<div class="title">Commodities</div>

<div class="sub">
Módulo preparado para proveedor externo.
Las claves quedan en el backend.
</div>

<div class="market">

${["Oro","Plata","Petróleo","Soja","Trigo","Maíz"].map(x=>`

<div class="card">

<b>${x}</b>

<div class="value">—</div>

<div class="small">
Sin proveedor configurado
</div>

</div>

`).join("")}

</div>
`;

}

else{

let d=await api("/api/alerts");

h=`
<div class="title">Alertas</div>

<div class="sub">
Eventos financieros e inventario.
</div>

${d.map(x=>`

<div class="alert ${x.type}">

<b>${x.title}</b>

<br>

${x.message}

</div>

`).join("")
||
'<div class="alert ok">No hay alertas.</div>'}

`;

}

document.getElementById("main").innerHTML=h;
nav();

}

async function addInvoice(){

let c=prompt("Cliente");
let a=Number(prompt("Importe ARS"));

if(c&&a){

await api("/api/invoices",{
method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({
client:c,
amount:a
})
});

render();

}

}

async function addStock(){

let n=prompt("Producto");
let q=Number(prompt("Cantidad"));
let c=Number(prompt("Costo unitario ARS"));

if(n&&q&&c){

await api("/api/stock",{
method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({
name:n,
qty:q,
cost:c,
min:10
})
});

render();

}

}

render();

</script>

</body>
</html>
"""

def conn():
    c=sqlite3.connect(DB)
    c.row_factory=sqlite3.Row
    return c


def init():

    c=conn()

    c.executescript("""

    CREATE TABLE IF NOT EXISTS invoices(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        client TEXT,
        amount REAL,
        due TEXT,
        status TEXT
    );

    CREATE TABLE IF NOT EXISTS stock(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        sku TEXT,
        name TEXT,
        qty INTEGER,
        min INTEGER,
        cost REAL
    );

    CREATE TABLE IF NOT EXISTS budgets(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT,
        budget REAL,
        spent REAL
    );

    CREATE TABLE IF NOT EXISTS investments(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT,
        amount REAL,
        rate REAL
    );

    """)

    if c.execute(
        "SELECT COUNT(*) n FROM invoices"
    ).fetchone()["n"]==0:

        c.executemany(
            "INSERT INTO invoices(client,amount,due,status) VALUES(?,?,?,?)",
            [
                ("BuildCo Ltd",1240000,"2026-09-05","Pendiente"),
                ("Metro Supplies",875000,"2026-08-18","Vencida"),
                ("Alpha Group",3120000,"2026-09-12","Pagada")
            ]
        )

    if c.execute(
        "SELECT COUNT(*) n FROM stock"
    ).fetchone()["n"]==0:

        c.executemany(
            "INSERT INTO stock(sku,name,qty,min,cost) VALUES(?,?,?,?,?)",
            [
                ("SKU-001","Taladro profesional",24,10,185000),
                ("SKU-002","Amoladora 900W",7,12,132000),
                ("SKU-003","Disco de corte",80,30,4200)
            ]
        )

    if c.execute(
        "SELECT COUNT(*) n FROM budgets"
    ).fetchone()["n"]==0:

        c.executemany(
            "INSERT INTO budgets(name,budget,spent) VALUES(?,?,?)",
            [
                ("Marketing",500000,420000),
                ("Compras",3500000,2980000),
                ("Operación",1800000,1710000)
            ]
        )

    if c.execute(
        "SELECT COUNT(*) n FROM investments"
    ).fetchone()["n"]==0:

        c.executemany(
            "INSERT INTO investments(name,amount,rate) VALUES(?,?,?)",
            [
                ("Plazo fijo",2000000,32),
                ("FCI money market",850000,28)
            ]
        )

    c.commit()
    c.close()


init()


@app.get("/")
def home():
    return render_template_string(HTML)


@app.get("/api/invoices")
def invoices():

    return jsonify([
        dict(x)
        for x in conn().execute(
            "SELECT * FROM invoices ORDER BY id DESC"
        )
    ])


@app.post("/api/invoices")
def add_invoice():

    x=request.json or {}

    c=conn()

    c.execute(
        """
        INSERT INTO invoices(client,amount,due,status)
        VALUES(?,?,date('now'),'Pendiente')
        """,
        (
            x.get("client","Cliente"),
            float(x.get("amount",0))
        )
    )

    c.commit()
    c.close()

    return jsonify({"ok":True})


@app.get("/api/stock")
def stock():

    return jsonify([
        dict(x)
        for x in conn().execute(
            "SELECT * FROM stock ORDER BY id DESC"
        )
    ])


@app.post("/api/stock")
def add_stock():

    x=request.json or {}

    c=conn()

    c.execute(
        """
        INSERT INTO stock(sku,name,qty,min,cost)
        VALUES(?,?,?,?,?)
        """,
        (
            x.get("sku","SKU-"+str(os.getpid())),
            x.get("name","Producto"),
            int(x.get("qty",0)),
            int(x.get("min",10)),
            float(x.get("cost",0))
        )
    )

    c.commit()
    c.close()

    return jsonify({"ok":True})


@app.get("/api/budgets")
def budgets():

    return jsonify([
        dict(x)
        for x in conn().execute(
            "SELECT * FROM budgets"
        )
    ])


@app.get("/api/investments")
def investments():

    return jsonify([
        dict(x)
        for x in conn().execute(
            "SELECT * FROM investments"
        )
    ])


@app.get("/api/alerts")
def alerts():

    c=conn()
    a=[]

    for x in c.execute(
        "SELECT * FROM invoices WHERE status='Vencida'"
    ):

        a.append({
            "type":"bad",
            "title":"Factura vencida",
            "message":
            f"{x['client']} · ARS {x['amount']:,.0f}"
        })

    for x in c.execute(
        "SELECT * FROM stock WHERE qty<min"
    ):

        a.append({
            "type":"",
            "title":"Stock bajo",
            "message":
            f"{x['name']} · {x['qty']} unidades "
            f"(mínimo {x['min']})"
        })

    c.close()

    return jsonify(a)


@app.get("/api/bcra")
def bcra():

    try:

        url=(
            "https://api.bcra.gob.ar/"
            "estadisticascambiarias/v1.0/"
            "Cotizaciones"
        )

        r=requests.get(
            url,
            timeout=8
        )

        r.raise_for_status()

        data=r.json()

        rows=data.get(
            "results",
            {}
        ).get(
            "detalle",
            []
        )

        wanted={
            "USD",
            "EUR",
            "BRL",
            "CLP",
            "COP",
            "GBP",
            "CNY",
            "UYU"
        }

        return jsonify({

            x["codigoMoneda"]:
            x.get("tipoCotizacion")

            for x in rows

            if x.get("codigoMoneda") in wanted

        })

    except Exception:

        return jsonify({

            "USD":"Sin conexión",
            "EUR":"Sin conexión",
            "BRL":"Sin conexión"

        })


@app.get("/health")
def health():

    return jsonify({
        "status":"ok",
        "app":"Kontalo"
    })


if __name__=="__main__":

    app.run(
        host="0.0.0.0",
        port=int(
            os.getenv("PORT","5000")
        ),
        debug=False
    )