
  import os,sqlite3,datetime as dt,requests
from flask import Flask,request,jsonify,render_template_string

app=Flask(__name__)
DB=os.getenv("KONTALO_DB","kontalo.db")
PORT=int(os.getenv("PORT","5000"))

def db():
    c=sqlite3.connect(DB)
    c.row_factory=sqlite3.Row
    return c

def init_db():
    c=db()
    c.executescript("""
    CREATE TABLE IF NOT EXISTS invoices(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        client TEXT NOT NULL,
        amount REAL NOT NULL,
        issue_date TEXT NOT NULL,
        due_date TEXT NOT NULL,
        status TEXT NOT NULL DEFAULT 'Pendiente'
    );
    CREATE TABLE IF NOT EXISTS products(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        sku TEXT UNIQUE,
        name TEXT NOT NULL,
        category TEXT DEFAULT 'General',
        qty INTEGER DEFAULT 0,
        min_qty INTEGER DEFAULT 0,
        cost REAL DEFAULT 0,
        price REAL DEFAULT 0
    );
    CREATE TABLE IF NOT EXISTS movements(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        kind TEXT NOT NULL,
        description TEXT NOT NULL,
        amount REAL NOT NULL,
        movement_date TEXT NOT NULL
    );
    CREATE TABLE IF NOT EXISTS budgets(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        budget REAL DEFAULT 0,
        spent REAL DEFAULT 0
    );
    CREATE TABLE IF NOT EXISTS investments(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        instrument TEXT NOT NULL,
        amount REAL DEFAULT 0,
        rate REAL DEFAULT 0,
        created_at TEXT NOT NULL
    );
    """)
    if c.execute("SELECT COUNT(*) FROM invoices").fetchone()[0]==0:
        d=dt.date.today()
        c.executemany(
            "INSERT INTO invoices(client,amount,issue_date,due_date,status) VALUES(?,?,?,?,?)",
            [
                ("Cliente Demo",1250000,str(d),str(d+dt.timedelta(days=30)),"Pendiente"),
                ("Cliente Vencido",875000,str(d-dt.timedelta(days=40)),str(d-dt.timedelta(days=10)),"Vencida"),
                ("Cliente Pagado",3120000,str(d-dt.timedelta(days=20)),str(d-dt.timedelta(days=5)),"Pagada")
            ])
    if c.execute("SELECT COUNT(*) FROM products").fetchone()[0]==0:
        c.executemany(
            "INSERT INTO products(sku,name,category,qty,min_qty,cost,price) VALUES(?,?,?,?,?,?,?)",
            [
                ("SKU-001","Taladro profesional","Herramientas",24,10,185000,265000),
                ("SKU-002","Amoladora 900W","Herramientas",7,12,132000,199000),
                ("SKU-003","Disco de corte","Accesorios",80,30,4200,7200)
            ])
    if c.execute("SELECT COUNT(*) FROM movements").fetchone()[0]==0:
        d=str(dt.date.today())
        c.executemany(
            "INSERT INTO movements(kind,description,amount,movement_date) VALUES(?,?,?,?)",
            [
                ("ingreso","Cobro de cliente",4670000,d),
                ("egreso","Compra de mercadería",1800000,d),
                ("egreso","Gastos operativos",1410000,d)
            ])
    if c.execute("SELECT COUNT(*) FROM budgets").fetchone()[0]==0:
        c.executemany(
            "INSERT INTO budgets(name,budget,spent) VALUES(?,?,?)",
            [
                ("Marketing",500000,420000),
                ("Compras",3500000,2980000),
                ("Operación",1800000,1710000)
            ])
    if c.execute("SELECT COUNT(*) FROM investments").fetchone()[0]==0:
        d=str(dt.date.today())
        c.executemany(
            "INSERT INTO investments(name,instrument,amount,rate,created_at) VALUES(?,?,?,?,?)",
            [
                ("Reserva","Money Market",850000,28,d),
                ("Capital","Plazo fijo",2000000,32,d)
            ])
    c.commit()
    c.close()

def rows(sql,args=()):
    c=db()
    r=[dict(x) for x in c.execute(sql,args).fetchall()]
    c.close()
    return r

def execute(sql,args=()):
    c=db()
    cur=c.execute(sql,args)
    c.commit()
    result=cur.lastrowid
    c.close()
    return result

def num(x):
    try:return float(x)
    except:return 0.0

def forecast():
    m=rows("SELECT kind,amount FROM movements")
    income=sum(num(x["amount"]) for x in m if x["kind"]=="ingreso")
    expense=sum(num(x["amount"]) for x in m if x["kind"]=="egreso")
    invoices=rows("SELECT amount,status FROM invoices")
    receivable=sum(num(x["amount"]) for x in invoices if x["status"]!="Pagada")
    cash=12845000+income-expense
    return {
        "cash":cash,
        "income":income,
        "expenses":expense,
        "net":income-expense,
        "receivable":receivable,
        "pessimistic":cash+receivable*.45,
        "base":cash+receivable*.70,
        "optimistic":cash+receivable*.90
    }

@app.get("/health")
def health():
    return jsonify({"status":"ok","application":"Kontalo"})

@app.get("/api/dashboard")
def dashboard():
    f=forecast()
    products=rows("SELECT qty,cost FROM products")
    investments=rows("SELECT amount FROM investments")
    budgets=rows("SELECT budget,spent FROM budgets")
    inventory=sum(num(x["qty"])*num(x["cost"]) for x in products)
    invtotal=sum(num(x["amount"]) for x in investments)
    return jsonify({
        "cash":f["cash"],
        "receivable":f["receivable"],
        "inventory":inventory,
        "investments":invtotal,
        "budget":sum(num(x["budget"]) for x in budgets),
        "spent":sum(num(x["spent"]) for x in budgets),
        "forecast":f
    })

@app.route("/api/invoices",methods=["GET","POST"])
def invoices():
    if request.method=="GET":
        return jsonify(rows("SELECT * FROM invoices ORDER BY id DESC"))
    x=request.get_json() or {}
    client=str(x.get("client","")).strip()
    amount=num(x.get("amount"))
    if not client or amount<=0:
        return jsonify({"ok":False,"error":"Cliente e importe son obligatorios"}),400
    execute(
        "INSERT INTO invoices(client,amount,issue_date,due_date,status) VALUES(?,?,?,?,?)",
        (
            client,
            amount,
            str(dt.date.today()),
            x.get("due_date",str(dt.date.today()+dt.timedelta(days=30))),
            "Pendiente"
        ))
    return jsonify({"ok":True})

@app.put("/api/invoices/<int:i>")
def update_invoice(i):
    x=request.get_json() or {}
    if x.get("status") not in ["Pendiente","Pagada","Vencida","Anulada"]:
       