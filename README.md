from flask import Flask, request, jsonify, render_template_string
import sqlite3, os
from datetime import date
try:
    import requests
except ImportError:
    requests = None

APP = Flask(__name__)
DB = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'kontalo.db')


def conn():
    c = sqlite3.connect(DB)
    c.row_factory = sqlite3.Row
    return c


def init_db():
    c = conn()
    c.executescript('''
    CREATE TABLE IF NOT EXISTS invoices (
      id INTEGER PRIMARY KEY AUTOINCREMENT, client TEXT NOT NULL,
      amount REAL NOT NULL, due TEXT NOT NULL, status TEXT NOT NULL
    );
    CREATE TABLE IF NOT EXISTS stock (
      id INTEGER PRIMARY KEY AUTOINCREMENT, sku TEXT UNIQUE NOT NULL,
      name TEXT NOT NULL, qty INTEGER NOT NULL, minimum INTEGER NOT NULL,
      cost REAL NOT NULL
    );
    CREATE TABLE IF NOT EXISTS budgets (
      id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL,
      budget REAL NOT NULL, spent REAL NOT NULL
    );
    CREATE TABLE IF NOT EXISTS investments (
      id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL,
      amount REAL NOT NULL, rate REAL NOT NULL
    );
    CREATE TABLE IF NOT EXISTS cash (
      id INTEGER PRIMARY KEY AUTOINCREMENT, kind TEXT NOT NULL,
      description TEXT NOT NULL, amount REAL NOT NULL, created TEXT NOT NULL
    );
    ''')
    if c.execute('SELECT COUNT(*) FROM invoices').fetchone()[0] == 0:
        c.executemany('INSERT INTO invoices(client,amount,due,status) VALUES(?,?,?,?)', [
            ('BuildCo Ltd',1240000,'2026-09-05','Pendiente'),
            ('Metro Supplies',875000,'2026-08-18','Vencida'),
            ('Alpha Group',3120000,'2026-09-12','Pagada')])
    if c.execute('SELECT COUNT(*) FROM stock').fetchone()[0] == 0:
        c.executemany('INSERT INTO stock(sku,name,qty,minimum,cost) VALUES(?,?,?,?,?)', [
            ('SKU-001','Taladro profesional',24,10,185000),
            ('SKU-002','Amoladora 900W',7,12,132000),
            ('SKU-003','Disco de corte',80,30,4200)])
    if c.execute('SELECT COUNT(*) FROM budgets').fetchone()[0] == 0:
        c.executemany('INSERT INTO budgets(name,budget,spent) VALUES(?,?,?)', [
            ('Marketing',500000,420000),('Compras',3500000,2980000),
            ('Operación',1800000,1710000)])
    if c.execute('SELECT COUNT(*) FROM investments').fetchone()[0] == 0:
        c.executemany('INSERT INTO investments(name,amount,rate) VALUES(?,?,?)', [
            ('Plazo fijo',2000000,32),('FCI money market',850000,28)])
    c.commit(); c.close()

init_db()

HTML = '''<!doctype html>
<html lang="es"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Kontalo</title><style>
:root{--bg:#071019;--panel:#0d1824;--panel2:#09131e;--border:#203445;--text:#eef4f8;--muted:#91a4b5;--green:#00e5a0;--red:#ff6b6b;--yellow:#f6b73c}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:Arial,Helvetica,sans-serif}
header{padding:17px 22px;background:var(--panel2);border-bottom:1px solid var(--border);display:flex;justify-content:space-between;align-items:center}
.logo{font-size:25px;font-weight:800}.logo b{color:var(--green)}nav{padding:10px;background:var(--panel2);border-bottom:1px solid var(--border);display:flex;gap:7px;flex-wrap:wrap}
button{background:#10202d;color:var(--text);border:1px solid #294052;padding:9px 12px;border-radius:8px;cursor:pointer}button.active{background:#0d2d25;color:var(--green)}button.primary{background:var(--green);color:#06120e;font-weight:700}
main{max-width:1450px;margin:auto;padding:22px}h1{margin:0 0 7px}.muted{color:var(--muted);font-size:13px}.grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}.card{background:var(--panel);border:1px solid var(--border);border-radius:12px;padding:16px}.value{font-size:24px;font-weight:800;margin:8px 0}.green{color:var(--green)}.red{color:var(--red)}.yellow{color:var(--yellow)}
.table{width:100%;border-collapse:collapse;margin-top:12px}.table th,.table td{padding:10px 8px;border-bottom:1px solid var(--border);font-size:13px;text-align:left}.table th{color:var(--muted)}
.alert{background:#101c27;border-left:3px solid var(--yellow);padding:11px;margin:8px 0}.alert.red{border-color:var(--red)}.alert.green{border-color:var(--green)}
@media(max-width:850px){.grid{grid-template-columns:1fr 1fr}}@media(max-width:520px){.grid{grid-template-columns:1fr}main{padding:14px}}
</style></head><body>
<header><div class="logo">kontalo<b>.</b></div><div class="muted">Finanzas · operaciones · mercados</div></header>
<nav id="nav"></nav><main id="main"></main>
<script>
let page='dashboard';
const menus=[['dashboard','Dashboard'],['cash','Flujo de caja'],['invoices','Facturación'],['stock','Stock'],['budget','Presupuesto'],['investments','Inversiones'],['markets','Cotizaciones BCRA'],['commodities','Commodities'],['alerts','Alertas']];
const money=n=>new Intl.NumberFormat('es-AR',{style:'currency',currency:'ARS',maximumFractionDigits:0}).format(Number(n)||0);
async function api(u,o){const r=await fetch(u,o);if(!r.ok)throw Error('HTTP '+r.status);return r.json()}
function nav(){document.getElementById('nav').innerHTML=menus.map(m=>`<button class="${page===m[0]?'active':''}" onclick="go('${m[0]}')">${m[1]}</button>`).join('')}
async function go(p){page=p;nav();await render()}
async function render(){let h='';try{
if(page==='dashboard')h=`<h1>Dashboard</h1><p class="muted">Vista central de Kontalo.</p><div class="grid">
<div class="card"><span class="muted">Caja actual</span><div class="value">${money(12845000)}</div></div>
<div class="card"><span class="muted">Por cobrar</span><div class="value">${money(5115000)}</div></div>
<div class="card"><span class="muted">Inventario</span><div class="value">${money(5450000)}</div></div>
<div class="card"><span class="muted">Alertas</span><div class="value yellow">3</div></div></div>
<div class="card" style="margin-top:15px"><b>IA / Forecasting</b><div class="alert red">Revisión de facturas vencidas y riesgo de caja.</div><div class="alert green">Preparado para escenarios y análisis.</div></div>`;
else if(page==='cash')h=`<h1>Flujo de caja</h1><div class="grid"><div class="card"><span class="muted">Caja</span><div class="value">${money(12845000)}</div></div><div class="card"><span class="muted">Ingresos</span><div class="value green">${money(4670000)}</div></div><div class="card"><span class="muted">Egresos</span><div class="value red">${money(3210000)}</div></div><div class="card"><span class="muted">Proyección</span><div class="value">${money(13872600)}</div></div></div>`;
else if(page==='invoices'){let d=await api('/api/invoices');h=`<h1>Facturación</h1><button class="primary" onclick="newInvoice()">+ Nueva factura</button><div class="card"><table class="table"><tr><th>ID</th><th>Cliente</th><th>Importe</th><th>Vencimiento</th><th>Estado</th></tr>${d.map(x=>`<tr><td>${x.id}</td><td>${x.client}</td><td>${money(x.amount)}</td><td>${x.due}</td><td class="${x.status==='Vencida'?'red':x.status==='Pagada'?'green':''}">${x.status}</td></tr>`).join('')}</table></div>`}
else if(page==='stock'){let d=await api('/api/stock');h=`<h1>Stock</h1><button class="primary" onclick="newStock()">+ Producto</button><div class="card"><table class="table"><tr><th>SKU</th><th>Producto</th><th>Cantidad</th><th>Mínimo</th><th>Costo</th><th>Estado</th></tr>${d.map(x=>`<tr><td>${x.sku}</td><td>${x.name}</td><td>${x.qty}</td><td>${x.minimum}</td><td>${money(x.cost)}</td><td class="${x.qty<x.minimum?'yellow':'green'}">${x.qty<x.minimum?'Reponer':'OK'}</td></tr>`).join('')}</table></div>`}
else if(page==='budget'){let d=await api('/api/budgets');h=`<h1>Presupuesto</h1><div class="grid">${d.map(x=>`<div class="card"><b>${x.name}</b><div class="value">${x.budget?Math.round(x.spent/x.budget*100):0}%</div><span class="muted">${money(x.spent)} / ${money(x.budget)}</span></div>`).join('')}</div>`}
else if(page==='investments'){let d=await api('/api/investments');h=`<h1>Inversiones</h1><div class="card"><table class="table"><tr><th>Instrumento</th><th>Capital</th><th>Tasa</th><th>Rendimiento estimado</th></tr>${d.map(x=>`<tr><td>${x.name}</td><td>${money(x.amount)}</td><td>${x.rate}%</td><td class="green">${money(x.amount*x.rate/100)}</td></tr>`).join('')}</table></div>`}
else if(page==='markets'){let d=await api('/api/bcra');h=`<h1>Cotizaciones BCRA</h1><p class="muted">Consulta realizada desde el backend. No se expone ninguna clave al navegador.</p><div class="grid">${Object.entries(d).filter(([k])=>k!=='error').map(([k,v])=>`<div class="card"><span class="muted">${k}</span><div class="value">${v}</div><span class="muted">ARS · BCRA</span></div>`).join('')}</div>`}
else if(page==='commodities')h=`<h1>Commodities</h1><p class="muted">Módulo preparado para conectar proveedor externo desde el backend.</p><div class="grid">${['Oro','Plata','Petróleo','Soja','Trigo','Maíz'].map(x=>`<div class="card"><b>${x}</b><div class="value">—</div><span class="muted">Proveedor pendiente</span></div>`).join('')}</div>`;
else {let d=await api('/api/alerts');h=`<h1>Alertas</h1>${d.map(x=>`<div class="alert ${x.type}"><b>${x.title}</b><br>${x.message}</div>`).join('')||'<div class="alert green">No hay alertas.</div>'}`}
}catch(e){h=`<h1>Error</h1><div class="alert red">No se pudo cargar el módulo: ${e.message}</div>`}document.getElementById('main').innerHTML=h;nav()}
async function newInvoice(){const client=prompt('Cliente');const amount=Number(prompt('Importe ARS'));if(client&&amount>0){await api('/api/invoices',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({client,amount})});render()}}
async function newStock(){const name=prompt('Producto');const qty=Number(prompt('Cantidad'));const cost=Number(prompt('Costo unitario ARS'));const minimum=Number(prompt('Mínimo',10));if(name&&qty>=0&&cost>=0){await api('/api/stock',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({name,qty,cost,minimum})});render()}}
nav();render();
</script></body></html>'''

@APP.get('/')
def home(): return render_template_string(HTML)

@APP.get('/api/invoices')
def invoices():
    c=conn(); rows=c.execute('SELECT * FROM invoices ORDER BY id DESC').fetchall(); c.close(); return jsonify([dict(x) for x in rows])

@APP.post('/api/invoices')
def add_invoice():
    x=request.get_json(silent=True) or {}; client=str(x.get('client','')).strip(); amount=float(x.get('amount',0))
    if not client or amount<=0:return jsonify(error='Cliente e importe son obligatorios'),400
    c=conn(); c.execute('INSERT INTO invoices(client,amount,due,status) VALUES(?,?,?,?)',(client,amount,date.today().isoformat(),'Pendiente')); c.commit(); c.close(); return jsonify(ok=True)

@APP.get('/api/stock')
def stock():
    c=conn(); rows=c.execute('SELECT * FROM stock ORDER BY id DESC').fetchall(); c.close(); return jsonify([dict(x) for x in rows])

@APP.post('/api/stock')
def add_stock():
    x=request.get_json(silent=True) or {}; name=str(x.get('name','')).strip(); qty=int(x.get('qty',0)); minimum=int(x.get('minimum',10)); cost=float(x.get('cost',0))
    if not name or qty<0 or minimum<0 or cost<0:return jsonify(error='Datos inválidos'),400
    sku='SKU-'+str(abs(hash(name)))[:7]
    c=conn()
    try:c.execute('INSERT INTO stock(sku,name,qty,minimum,cost) VALUES(?,?,?,?,?)',(sku,name,qty,minimum,cost))
    except sqlite3.IntegrityError:c.execute('UPDATE stock SET qty=?,minimum=?,cost=? WHERE sku=?',(qty,minimum,cost,sku))
    c.commit();c.close();return jsonify(ok=True,sku=sku)

@APP.get('/api/budgets')
def budgets():
    c=conn(); rows=c.execute('SELECT * FROM budgets ORDER BY id').fetchall(); c.close(); return jsonify([dict(x) for x in rows])

@APP.get('/api/investments')
def investments():
    c=conn(); rows=c.execute('SELECT * FROM investments ORDER BY id').fetchall(); c.close(); return jsonify([dict(x) for x in rows])

@APP.get('/api/alerts')
def alerts():
    c=conn(); out=[]
    for x in c.execute("SELECT * FROM invoices WHERE status='Vencida'"):
        out.append({'type':'red','title':'Factura vencida','message':f"{x['client']} · ARS {x['amount']:,.0f}"})
    for x in c.execute('SELECT * FROM stock WHERE qty < minimum'):
        out.append({'type':'','title':'Stock bajo','message':f"{x['name']} · {x['qty']} unidades (mínimo {x['minimum']})"})
    c.close(); return jsonify(out)

@APP.get('/api/bcra')
def bcra():
    if requests is None:return jsonify({'USD':'Instalá requests','EUR':'Instalá requests','BRL':'Instalá requests'})
    try:
        r=requests.get('https://api.bcra.gob.ar/estadisticascambiarias/v1.0/Cotizaciones',timeout=10); r.raise_for_status(); data=r.json()
        rows=data.get('results',{}).get('detalle',[]); wanted={'USD','EUR','BRL','CLP','COP','GBP','CNY','UYU'}
        out={x.get('codigoMoneda'):x.get('tipoCotizacion') for x in rows if x.get('codigoMoneda') in wanted}
        return jsonify(out or {'USD':'Sin datos','EUR':'Sin datos','BRL':'Sin datos'})
    except Exception as e:return jsonify({'USD':'Sin conexión','EUR':'Sin conexión','BRL':'Sin conexión','error':str(e)})

@APP.get('/api/health')
def health():return jsonify(ok=True,app='Kontalo')

if __name__=='__main__':
    print('Kontalo: http://127.0.0.1:5000')
    APP.run(host='0.0.0.0',port=int(os.environ.get('PORT',5000)),debug=False)
