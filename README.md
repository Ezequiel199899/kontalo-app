import os
import sqlite3
from flask import Flask, request, jsonify, render_template_string

try:
    import requests
except ImportError:
    requests = None

app = Flask(__name__)
DB_PATH = os.getenv("DB_PATH", "kontalo.db")

HTML = r"""
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Kontalo</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#071019;color:#edf3f8;font-family:Arial,sans-serif}
header{height:64px;background:#09131e;border-bottom:1px solid #1d3040;
display:flex;align-items:center;justify-content:space-between;padding:0 20px}
.logo{font-size:23px;font-weight:800}.logo span{color:#00e5a0}
.status{font-size:12px;color:#9db0c1;border:1px solid #1d3040;padding:7px 10px;border-radius:8px}
.layout{display:grid;grid-template-columns:210px 1fr;min-height:calc(100vh - 64px)}
aside{background:#09131e;border-right:1px solid #1d3040;padding:12px}
nav button{width:100%;border:0;background:transparent;color:#9db0c1;
text-align:left;padding:11px;border-radius:8px;margin:2px 0;cursor:pointer}
nav button:hover,nav button.active{background:#0d2d25;color:#00e5a0}
main{padding:24px;max-width:1450px;width:100%;margin:auto}
h1{font-size:28px;margin:0 0 5px}.sub{color:#8fa2b4;font-size:13px;margin-bottom:20px}
.grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
.grid3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
.card{background:#0d1824;border:1px solid #1d3040;border-radius:12px;padding:16px}
.label{font-size:12px;color:#8fa2b4}.value{font-size:24px;font-weight:800;margin:8px 0}
.small{font-size:12px;color:#8fa2b4}
.ok{color:#00e5a0}.bad{color:#ff6b6b}.warn{color:#f6b73c}
table{width:100%;border-collapse:collapse}
th,td{padding:10px 7px;border-bottom:1px solid #1d3040;text-align:left;font-size:12px}
th{color:#8fa2b4}
.toolbar{display:flex;gap:8px;margin-bottom:12px;flex-wrap:wrap}
button.action{background:#111f2d;color:white;border:1px solid #1d3040;padding:9px 12px;border-radius:8px;cursor:pointer}
button.primary{background:#00e5a0;color:#06120e;border:0;font-weight:bold}
.badge{font-size:10px;padding:4px 8px;border-radius:999px;background:#332914;color:#f6b73c}
.badge.green{background:#0d2d25;color:#00e5a0}
.badge.red{background:#321b22;color:#ff6b6b}
.alert{padding:12px;margin:8px 0;background:#101c27;border-left:3px solid #f6b73c;border-radius:0 8px 8px 0}
.alert.bad{border-left-color:#ff6b6b}.alert.ok{border-left-color:#00e5a0}
.market{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
@media(max-width:900px){
.layout{grid-template-columns:1fr}
aside{overflow:auto}
nav{display:flex;gap:5px;overflow:auto}
nav button{white-space:nowrap}
.grid,.grid3,.market{grid-template-columns:1fr 1fr}
}
@media(max-width:550px){
main{padding:14px}.grid,.grid3,.market{grid-template-columns:1fr}
}
</style>
</head>

<body>
<header>
<div class="logo">kontalo<span>.</span></div>
<div class="status">KONTALO V2 · Backend Flask</div>
</header>

<div class="layout">
<aside>
<nav id="nav"></nav>
</aside>
<main id="main"></main>
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

function money(n){
 return new Intl.NumberFormat("es-AR",{
   style:"currency",
   currency:"ARS",
   maximumFractionDigits:0
 }).format(Number(n)||0);
}

function nav(){
 document.getElementById("nav").innerHTML=menus.map(x =>
 `<button class="${page===x[0]?'active':''}"
 onclick="go('${x[0]}')">${x[1]}</button>`
 ).join("");
}

async function api(url,options){
 const r=await fetch(url,options);
 if(!r.ok) throw new Error("HTTP "+r.status);
 return await r.json();
}

function card(title,value,description=""){
 return `<div class="card">
 <div class="label">${title}</div>
 <div class="value">${value}</div>
 <div class="small">${description}</div>
 </div>`;
}

async function go(p){
 page=p;
 nav();
 await render();
}

async function render(){

 let h="";

 if(page==="dashboard"){

 h=`
 <h1>Dashboard</h1>
 <div class="sub">Visión general de tu empresa.</div>

 <div class="grid">
 ${card("Caja actual",money(12845000),"Saldo disponible")}
 ${card("Por cobrar",money(5115000),"Facturas pendientes")}
 ${card("Inventario",money(5450000),"Valor estimado")}
 ${card("Alertas","3","Requieren atención")}
 </div>

 <div class="grid3" style="margin-top:18px">

 <div class="card">
 <b>🤖 IA financiera</b>
 <div class="alert bad">
 Hay facturas vencidas que pueden afectar la caja.
 </div>
 <div class="alert ok">
 Kontalo podrá analizar tus datos y generar proyecciones.
 </div>
 </div>

 <div class="card">
 <b>📊 Proyección de caja</b>
 <div class="value ok">${money(13872600)}</div>
 <div class="small">Escenario base a 30 días</div>
 </div>

 <div class="card">
 <b>🏗 Arquitectura</b>
 <p class="small">Frontend → Flask API</p>
 <p class="small">Flask → SQLite/PostgreSQL</p>
 <p class="small">Flask → BCRA</p>
 </div>

 </div>`;

 }

 else if(page==="cashflow"){

 h=`
 <h1>Flujo de caja</h1>
 <div class="sub">Control de ingresos, egresos y proyecciones.</div>

 <div class="grid">
 ${card("Caja inicial",money(12845000))}
 ${card("Ingresos",money(4670000))}
 ${card("Egresos",money(3210000))}
 ${card("Caja proyectada",money(14260000))}
 </div>

 <div class="card" style="margin-top:18px">
 <b>Escenarios</b>
 <table>
 <tr><th>Escenario</th><th>Proyección</th><th>Interpretación</th></tr>
 <tr><td>Pesimista</td><td class="bad">${money(12331200)}</td><td>Menores cobranzas</td></tr>
 <tr><td>Base</td><td class="ok">${money(13872600)}</td><td>Comportamiento esperado</td></tr>
 <tr><td>Optimista</td><td class="ok">${money(14771750)}</td><td>Cobranzas aceleradas</td></tr>
 </table>
 </div>`;

 }

 else if(page==="invoices"){

 let d=await api("/api/invoices");

 h=`
 <h1>Facturación</h1>
 <div class="sub">Cuentas por cobrar y vencimientos.</div>

 <div class="toolbar">
 <button class="action primary" onclick="addInvoice()">+ Nueva factura</button>
 </div>

 <div class="card">
 <table>
 <tr><th>ID</th><th>Cliente</th><th>Importe</th><th>Vencimiento</th><th>Estado</th></tr>
 ${d.map(x=>`
 <tr>
 <td>${x.id}</td>
 <td>${x.client}</td>
 <td>${money(x.amount)}</td>
 <td>${x.due}</td>
 <td>
 <span class="badge ${
 x.status==="Pagada"?"green":
 x.status==="Vencida"?"red":""
 }">${x.status}</span>
 </td>
 </tr>`).join("")}
 </table>
 </div>`;

 }

 else if(page==="stock"){

 let d=await api("/api/stock");

 h=`
 <h1>Stock</h1>
 <div class="sub">Inventario y reposición.</div>

 <div class="toolbar">
 <button class="action primary" onclick="addStock()">+ Producto</button>
 </div>

 <div class="card">
 <table>
 <tr><th>SKU</th><th>Producto</th><th>Unidades</th><th>Mínimo</th><th>Costo</th><th>Estado</th></tr>
 ${d.map(x=>`
 <tr>
 <td>${x.sku}</td>
 <td>${x.name}</td>
 <td>${x.qty}</td>
 <td>${x.min}</td>
 <td>${money(x.cost)}</td>
 <td>${
 x.qty<x.min
 ?'<span class="badge">Reponer</span>'
 :'<span class="badge green">OK</span>'
 }</td>
 </tr>`).join("")}
 </table>
 </div>`;

 }

 else if(page==="budget"){

 let d=await api("/api/budgets");

 h=`
 <h1>Presupuesto</h1>
 <div class="sub">Comparación entre presupuesto y ejecución.</div>

 <div class="grid3">
 ${d.map(x=>{
 let p=Math.round(x.spent/x.budget*100);
 return `
 <div class="card">
 <b>${x.name}</b>
 <div class="value">${p}%</div>
 <div class="small">${money(x.spent)} / ${money(x.budget)}</div>
 </div>`;
 }).join("")}
 </div>`;

 }

 else if(page==="investments"){

 let d=await api("/api/investments");

 h=`
 <h1>Inversiones</h1>
 <div class="sub">Capital y rendimiento estimado.</div>

 <div class="card">
 <table>
 <tr><th>Instrumento</th><th>Capital</th><th>Tasa</th><th>Rendimiento anual</th></tr>
 ${d.map(x=>`
 <tr>
 <td>${x.name}</td>
 <td>${money(x.amount)}</td>
 <td>${x.rate}%</td>
 <td class="ok">${money(x.amount*x.rate/100)}</td>
 </tr>`).join("")}
 </table>
 </div>`;

 }

 else if(page==="markets"){

 let d=await api("/api/bcra");

 h=`
 <h1>Cotizaciones BCRA</h1>
 <div class="sub">Datos consultados por el backend.</div>

 <div class="market">
 ${Object.entries(d).map(([k,v])=>`
 <div class="card">
 <div class="label">${k}</div>
 <div class="value">${v??"—"}</div>
 <div class="small">Cotización BCRA</div>
 </div>`).join("")}
 </div>`;

 }

 else if(page==="commodities"){

 h=`
 <h1>Commodities</h1>
 <div class="sub">Módulo preparado para conectar proveedores externos.</div>

 <div class="market">
 ${["Oro","Plata","Petróleo","Soja","Trigo","Maíz"].map(x=>`
 <div class="card">
 <b>${x}</b>
 <div class="value">—</div>
 <div class="small">Proveedor pendiente</div>
 </div>`).join("")}
 </div>`;

 }

 else if(page==="alerts"){

 let d=await api("/api/alerts");

 h=`
 <h1>Alertas</h1>
 <div class="sub">Eventos financieros e inventario.</div>

 ${d.length
 ? d.map(x=>`
 <div class="alert ${x.type}">
 <b>${x.title}</b><br>${x.message}
 </div>`).join("")
 : '<div class="alert ok">No hay alertas.</div>'
 }`;

 }

 document.getElementById("main").innerHTML=h;
 nav();
}

async function addInvoice(){

 const client=prompt("Cliente:");
 const amount=Number(prompt("Importe ARS:"));

 if(!client || !amount) return;

 await api("/api/invoices",{
   method:"POST",
   headers:{"Content-Type":"application/json"},
   body:JSON.stringify({client,amount})
 });

 await render();
}

async function addStock(){

 const name=prompt("Producto:");
 const qty=Number(prompt("Cantidad:"));
 const cost=Number(prompt("Costo unitario ARS:"));

 if(!name || !qty || !cost) return;

 await api("/api/stock",{
   method:"POST",
   headers:{"Content-Type":"application/json"},
   body:JSON.stringify({
     name,
     qty,
     cost,
     min:10
   })
 });

 await render();
}

render();
</script>
</body>
</html>
"""

def db():
    c=sqlite3.connect(DB_PATH)
    c.row_factory=sqlite3.Row
    return c

def init_db():
    c=db()

    c.execute("""
    CREATE TABLE IF NOT EXISTS invoices(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        client TEXT NOT NULL,
        amount REAL NOT NULL,
        due TEXT NOT NULL,
        status TEXT NOT NULL
    )
    """)

    c.execute("""
    CREATE TABLE IF NOT EXISTS stock(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        sku TEXT NOT NULL,
        name TEXT NOT NULL,
        qty INTEGER NOT NULL,
        min INTEGER NOT NULL,
        cost REAL NOT NULL
    )
    """)

    c.execute("""
    CREATE TABLE IF NOT EXISTS budgets(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        budget REAL NOT NULL,
        spent REAL NOT NULL
    )
    """)

    c.execute("""
    CREATE TABLE IF NOT EXISTS investments(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        amount REAL NOT NULL,
        rate REAL NOT NULL
    )
    """)

    if c.execute("SELECT COUNT(*) FROM invoices").fetchone()[0]==0:
        c.executemany("""
        INSERT INTO invoices(client,amount,due,status)
        VALUES(?,?,?,?)
        """,[
            ("BuildCo Ltd",1240000,"2026-09-05","Pendiente"),
            ("Metro Supplies",875000,"2026-08-18","Vencida"),
            ("Alpha Group",3120000,"2026-09-12","Pagada")
        ])

    if c.execute("SELECT COUNT(*) FROM stock").fetchone()[0]==0:
        c.executemany("""
        INSERT INTO stock(sku,name,qty,min,cost)
        VALUES(?,?,?,?,?)
        """,[
            ("SKU-001","Taladro profesional",24,10,185000),
            ("SKU-002","Amoladora 900W",7,12,132000),
            ("SKU-003","Disco de corte",80,30,4200)
        ])

    if c.execute("SELECT COUNT(*) FROM budgets").fetchone()[0]==0:
        c.executemany("""
        INSERT INTO budgets(name,budget,spent)
        VALUES(?,?,?)
        """,[
            ("Marketing",500000,420000),
            ("Compras",3500000,2980000),
            ("Operación",1800000,1710000)
        ])

    if c.execute("SELECT COUNT(*) FROM investments").fetchone()[0]==0:
        c.executemany("""
        INSERT INTO investments(name,amount,rate)
        VALUES(?,?,?)
        """,[
            ("Plazo fijo",2000000,32),
            ("FCI money market",850000,28)
        ])

    c.commit()
    c.close()

init_db()

@app.get("/")
def home():
    return render_template_string(HTML)

@app.get("/api/invoices")
def get_invoices():
    c=db()
    rows=c.execute("SELECT * FROM invoices ORDER BY id DESC").fetchall()
    c.close()
    return jsonify([dict(x) for x in rows])

@app.post("/api/invoices")
def create_invoice():

    data=request.get_json(silent=True) or {}

    client=str(data.get("client","Cliente"))
    amount=float(data.get("amount",0))

    c=db()

    c.execute("""
    INSERT INTO invoices(client,amount,due,status)
    VALUES(?,?,date('now'),'Pendiente')
    """,(client,amount))

    c.commit()
    c.close()

    return jsonify({"ok":True})

@app.get("/api/stock")
def get_stock():

    c=db()
    rows=c.execute("SELECT * FROM stock ORDER BY id DESC").fetchall()
    c.close()

    return jsonify([dict(x) for x in rows])

@app.post("/api/stock")
def create_stock():

    data=request.get_json(silent=True) or {}

    name=str(data.get("name","Producto"))
    qty=int(data.get("qty",0))
    cost=float(data.get("cost",0))
    minimum=int(data.get("min",10))

    c=db()

    sku="SKU-%04d"%(c.execute(
        "SELECT COUNT(*) FROM stock"
    ).fetchone()[0]+1)

    c.execute("""
    INSERT INTO stock(sku,name,qty,min,cost)
    VALUES(?,?,?,?,?)
    """,(sku,name,qty,minimum,cost))

    c.commit()
    c.close()

    return jsonify({"ok":True,"sku":sku})

@app.get("/api/budgets")
def get_budgets():

    c=db()
    rows=c.execute("SELECT * FROM budgets").fetchall()
    c.close()

    return jsonify([dict(x) for x in rows])

@app.get("/api/investments")
def get_investments():

    c=db()
    rows=c.execute("SELECT * FROM investments").fetchall()
    c.close()

    return jsonify([dict(x) for x in rows])

@app.get("/api/alerts")
def get_alerts():

    c=db()
    alerts=[]

    invoices=c.execute("""
    SELECT * FROM invoices
    WHERE status='Vencida'
    """).fetchall()

    for x in invoices:
        alerts.append({
            "type":"bad",
            "title":"Factura vencida",
            "message":f"{x['client']} · ARS {x['amount']:,.0f}"
        })

    stock=c.execute("""
    SELECT * FROM stock
    WHERE qty < min
    """).fetchall()

    for x in stock:
        alerts.append({
            "type":"",
            "title":"Stock bajo",
            "message":f"{x['name']} · {x['qty']} unidades · mínimo {x['min']}"
        })

    c.close()

    return jsonify(alerts)

@app.get("/api/bcra")
def bcra():

    if requests is None:
        return jsonify({
            "USD":"Instalar requests",
            "EUR":"Instalar requests"
        })

    url="https://api.bcra.gob.ar/estadisticascambiarias/v1.0/Cotizaciones"

    try:

        response=requests.get(
            url,
            timeout=10,
            headers={"User-Agent":"Kontalo/2.0"}
        )

        response.raise_for_status()

        data=response.json()

        result=data.get("results",{})
        details=result.get("detalle",[])

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

        output={}

        for item in details:

            code=item.get("codigoMoneda")

            if code in wanted:

                output[code]=(
                    item.get("tipoCotizacion")
                    or item.get("tipoCotizacionVenta")
                    or item.get("tipoCotizacionCompra")
                )

        if not output:
            return jsonify({
                "USD":"No disponible",
                "EUR":"No disponible"
            })

        return jsonify(output)

    except Exception as e:

        return jsonify({
            "USD":"Sin conexión",
            "EUR":"Sin conexión",
            "error":"No fue posible consultar BCRA"
        })

@app.get("/api/health")
def health():
    return jsonify({
        "status":"ok",
        "app":"Kontalo",
        "database":DB_PATH
    })

if __name__=="__main__":

    port=int(os.getenv("PORT","5000"))

    app.run(
        host="0.0.0.0",
        port=port,
        debug=False
    )