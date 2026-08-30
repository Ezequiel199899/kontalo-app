# ============================================================
# KONTALO - APP COMPLETA EN UN SOLO ARCHIVO
# ============================================================
#
# Incluye:
#   - Dashboard financiero
#   - Flujo de caja
#   - Facturación
#   - Stock
#   - Presupuesto
#   - Inversiones
#   - Cotizaciones BCRA
#   - Commodities
#   - Forecast financiero
#   - Alertas
#   - API REST
#   - SQLite local
#   - PostgreSQL mediante DATABASE_URL
#   - Frontend integrado
#
# INSTALACIÓN:
#
#   pip install flask requests
#
# EJECUCIÓN:
#
#   python app.py
#
# Luego:
#
#   http://localhost:5000
#
# PostgreSQL:
#
#   pip install flask requests psycopg2-binary
#
#   DATABASE_URL="postgresql://usuario:password@host:5432/kontalo" python app.py
#
# ============================================================

import os
import sqlite3
from datetime import date, datetime, timedelta

import requests
from flask import Flask, jsonify, request, render_template_string


# ============================================================
# CONFIGURACIÓN
# ============================================================

app = Flask(__name__)

PORT = int(os.getenv("PORT", "5000"))

DATABASE_URL = os.getenv("DATABASE_URL", "").strip()

SQLITE_DATABASE = os.getenv(
    "SQLITE_DATABASE",
    "kontalo.db"
)


# ============================================================
# BASE DE DATOS
# ============================================================

def get_connection():

    if DATABASE_URL:

        try:
            import psycopg2

            connection = psycopg2.connect(
                DATABASE_URL
            )

            return connection

        except ImportError:

            raise RuntimeError(
                "Para PostgreSQL instalá psycopg2-binary"
            )

    connection = sqlite3.connect(
        SQLITE_DATABASE
    )

    connection.row_factory = sqlite3.Row

    return connection


def execute(
    sql,
    params=(),
    fetch=False,
    many=False
):

    connection = get_connection()

    cursor = connection.cursor()

    try:

        if DATABASE_URL:

            sql = sql.replace(
                "?",
                "%s"
            )

        if many:

            cursor.executemany(
                sql,
                params
            )

        else:

            cursor.execute(
                sql,
                params
            )

        result = None

        if fetch:

            rows = cursor.fetchall()

            if DATABASE_URL:

                columns = [
                    column[0]
                    for column in cursor.description
                ]

                result = [
                    dict(
                        zip(
                            columns,
                            row
                        )
                    )
                    for row in rows
                ]

            else:

                result = [
                    dict(row)
                    for row in rows
                ]

        connection.commit()

        return result

    finally:

        cursor.close()

        connection.close()


# ============================================================
# CREACIÓN DE TABLAS
# ============================================================

def initialize_database():

    tables = [

        """
        CREATE TABLE IF NOT EXISTS invoices(

            id INTEGER PRIMARY KEY AUTOINCREMENT,

            client TEXT NOT NULL,

            amount REAL NOT NULL,

            issue_date TEXT NOT NULL,

            due_date TEXT NOT NULL,

            status TEXT NOT NULL DEFAULT 'Pendiente'

        )
        """,

        """
        CREATE TABLE IF NOT EXISTS products(

            id INTEGER PRIMARY KEY AUTOINCREMENT,

            sku TEXT UNIQUE,

            name TEXT NOT NULL,

            category TEXT DEFAULT 'General',

            qty INTEGER NOT NULL DEFAULT 0,

            min_qty INTEGER NOT NULL DEFAULT 0,

            cost REAL NOT NULL DEFAULT 0,

            price REAL NOT NULL DEFAULT 0

        )
        """,

        """
        CREATE TABLE IF NOT EXISTS cash_movements(

            id INTEGER PRIMARY KEY AUTOINCREMENT,

            movement_type TEXT NOT NULL,

            description TEXT NOT NULL,

            amount REAL NOT NULL,

            movement_date TEXT NOT NULL

        )
        """,

        """
        CREATE TABLE IF NOT EXISTS budgets(

            id INTEGER PRIMARY KEY AUTOINCREMENT,

            name TEXT NOT NULL,

            budget REAL NOT NULL DEFAULT 0,

            spent REAL NOT NULL DEFAULT 0

        )
        """,

        """
        CREATE TABLE IF NOT EXISTS investments(

            id INTEGER PRIMARY KEY AUTOINCREMENT,

            name TEXT NOT NULL,

            instrument TEXT NOT NULL,

            amount REAL NOT NULL DEFAULT 0,

            rate REAL NOT NULL DEFAULT 0,

            created_at TEXT NOT NULL

        )
        """
    ]

    for table in tables:

        try:

            execute(table)

        except Exception:

            if DATABASE_URL:

                table = table.replace(
                    "INTEGER PRIMARY KEY AUTOINCREMENT",
                    "SERIAL PRIMARY KEY"
                )

                execute(table)

            else:

                raise


    seed_database()


# ============================================================
# DATOS DEMO
# ============================================================

def seed_database():

    today = date.today()

    invoice_count = execute(
        "SELECT COUNT(*) AS count FROM invoices",
        fetch=True
    )[0]["count"]

    if invoice_count == 0:

        execute(
            """
            INSERT INTO invoices
            (
                client,
                amount,
                issue_date,
                due_date,
                status
            )
            VALUES(?,?,?,?,?)
            """,

            [
                (
                    "Cliente Demo",
                    1240000,
                    str(
                        today -
                        timedelta(days=10)
                    ),
                    str(
                        today +
                        timedelta(days=20)
                    ),
                    "Pendiente"
                ),

                (
                    "Cliente Vencido",
                    875000,
                    str(
                        today -
                        timedelta(days=40)
                    ),
                    str(
                        today -
                        timedelta(days=10)
                    ),
                    "Vencida"
                ),

                (
                    "Cliente Pagado",
                    3120000,
                    str(
                        today -
                        timedelta(days=20)
                    ),
                    str(
                        today -
                        timedelta(days=5)
                    ),
                    "Pagada"
                )
            ],
            many=True
        )


    product_count = execute(
        "SELECT COUNT(*) AS count FROM products",
        fetch=True
    )[0]["count"]

    if product_count == 0:

        execute(
            """
            INSERT INTO products
            (
                sku,
                name,
                category,
                qty,
                min_qty,
                cost,
                price
            )
            VALUES(?,?,?,?,?,?,?)
            """,

            [
                (
                    "SKU-001",
                    "Taladro Profesional",
                    "Herramientas",
                    24,
                    10,
                    185000,
                    265000
                ),

                (
                    "SKU-002",
                    "Amoladora 900W",
                    "Herramientas",
                    7,
                    12,
                    132000,
                    199000
                ),

                (
                    "SKU-003",
                    "Disco de Corte",
                    "Accesorios",
                    80,
                    30,
                    4200,
                    7200
                )
            ],
            many=True
        )


    movement_count = execute(
        """
        SELECT COUNT(*) AS count
        FROM cash_movements
        """,
        fetch=True
    )[0]["count"]

    if movement_count == 0:

        execute(
            """
            INSERT INTO cash_movements
            (
                movement_type,
                description,
                amount,
                movement_date
            )
            VALUES(?,?,?,?)
            """,

            [
                (
                    "ingreso",
                    "Cobro de cliente",
                    4670000,
                    str(today)
                ),

                (
                    "egreso",
                    "Compra de mercadería",
                    1800000,
                    str(today)
                ),

                (
                    "egreso",
                    "Gastos operativos",
                    1410000,
                    str(today)
                )
            ],

            many=True
        )


    budget_count = execute(
        """
        SELECT COUNT(*) AS count
        FROM budgets
        """,
        fetch=True
    )[0]["count"]

    if budget_count == 0:

        execute(
            """
            INSERT INTO budgets
            (
                name,
                budget,
                spent
            )
            VALUES(?,?,?)
            """,

            [
                (
                    "Marketing",
                    500000,
                    420000
                ),

                (
                    "Compras",
                    3500000,
                    2980000
                ),

                (
                    "Operación",
                    1800000,
                    1710000
                )
            ],

            many=True
        )


    investment_count = execute(
        """
        SELECT COUNT(*) AS count
        FROM investments
        """,
        fetch=True
    )[0]["count"]

    if investment_count == 0:

        execute(
            """
            INSERT INTO investments
            (
                name,
                instrument,
                amount,
                rate,
                created_at
            )
            VALUES(?,?,?,?,?)
            """,

            [
                (
                    "Reserva",
                    "Money Market",
                    850000,
                    28,
                    str(today)
                ),

                (
                    "Capital",
                    "Plazo fijo",
                    2000000,
                    32,
                    str(today)
                )
            ],

            many=True
        )


# ============================================================
# UTILIDADES
# ============================================================

def number(value):

    try:

        return float(value)

    except Exception:

        return 0.0


def current_date():

    return str(
        date.today()
    )


# ============================================================
# FORECAST
# ============================================================

def calculate_forecast():

    movements = execute(
        """
        SELECT
            movement_type,
            amount
        FROM cash_movements
        """,
        fetch=True
    )

    income = sum(
        number(row["amount"])
        for row in movements
        if row["movement_type"] == "ingreso"
    )

    expenses = sum(
        number(row["amount"])
        for row in movements
        if row["movement_type"] == "egreso"
    )

    invoices = execute(
        """
        SELECT
            amount,
            status
        FROM invoices
        """,
        fetch=True
    )

    receivable = sum(
        number(row["amount"])
        for row in invoices
        if row["status"] != "Pagada"
    )

    initial_cash = 12845000

    net = (
        income -
        expenses
    )

    current_cash = (
        initial_cash +
        net
    )

    pessimistic = (
        current_cash +
        receivable * 0.45
    )

    base = (
        current_cash +
        receivable * 0.70
    )

    optimistic = (
        current_cash +
        receivable * 0.90
    )

    return {

        "income": income,

        "expenses": expenses,

        "net": net,

        "receivable": receivable,

        "current_cash": current_cash,

        "pessimistic": pessimistic,

        "base": base,

        "optimistic": optimistic
    }


# ============================================================
# DASHBOARD
# ============================================================

@app.get("/api/dashboard")
def dashboard():

    forecast = calculate_forecast()

    products = execute(
        """
        SELECT
            qty,
            cost
        FROM products
        """,
        fetch=True
    )

    inventory = sum(
        number(row["qty"]) *
        number(row["cost"])
        for row in products
    )

    investments = execute(
        """
        SELECT amount
        FROM investments
        """,
        fetch=True
    )

    investment_total = sum(
        number(row["amount"])
        for row in investments
    )

    budgets = execute(
        """
        SELECT
            budget,
            spent
        FROM budgets
        """,
        fetch=True
    )

    budget_total = sum(
        number(row["budget"])
        for row in budgets
    )

    spent_total = sum(
        number(row["spent"])
        for row in budgets
    )

    return jsonify({

        "cash":
            forecast["current_cash"],

        "receivable":
            forecast["receivable"],

        "inventory":
            inventory,

        "investments":
            investment_total,

        "budget":
            budget_total,

        "spent":
            spent_total,

        "forecast":
            forecast
    })


# ============================================================
# FACTURACIÓN
# ============================================================

@app.get("/api/invoices")
def get_invoices():

    return jsonify(
        execute(
            """
            SELECT *
            FROM invoices
            ORDER BY id DESC
            """,
            fetch=True
        )
    )


@app.post("/api/invoices")
def create_invoice():

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    client = str(
        data.get(
            "client",
            ""
        )
    ).strip()

    amount = number(
        data.get(
            "amount"
        )
    )

    if not client:

        return jsonify({
            "ok": False,
            "error": "El cliente es obligatorio"
        }), 400

    if amount <= 0:

        return jsonify({
            "ok": False,
            "error": "El importe debe ser mayor que cero"
        }), 400

    execute(
        """
        INSERT INTO invoices
        (
            client,
            amount,
            issue_date,
            due_date,
            status
        )
        VALUES(?,?,?,?,?)
        """,

        (
            client,

            amount,

            current_date(),

            data.get(
                "due_date",
                str(
                    date.today() +
                    timedelta(days=30)
                )
            ),

            "Pendiente"
        )
    )

    return jsonify({
        "ok": True
    })


@app.put("/api/invoices/<int:invoice_id>")
def update_invoice(
    invoice_id
):

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    status = data.get(
        "status"
    )

    allowed = {
        "Pendiente",
        "Pagada",
        "Vencida",
        "Anulada"
    }

    if status not in allowed:

        return jsonify({
            "ok": False,
            "error": "Estado inválido"
        }), 400

    execute(
        """
        UPDATE invoices
        SET status=?
        WHERE id=?
        """,

        (
            status,
            invoice_id
        )
    )

    return jsonify({
        "ok": True
    })


# ============================================================
# STOCK
# ============================================================

@app.get("/api/stock")
def get_stock():

    return jsonify(
        execute(
            """
            SELECT *
            FROM products
            ORDER BY id DESC
            """,
            fetch=True
        )
    )


@app.post("/api/stock")
def create_stock():

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    name = str(
        data.get(
            "name",
            ""
        )
    ).strip()

    if not name:

        return jsonify({
            "ok": False,
            "error": "El nombre es obligatorio"
        }), 400

    sku = data.get(
        "sku"
    )

    if not sku:

        sku = (
            "SKU-" +
            str(
                int(
                    datetime.now().timestamp()
                )
            )
        )

    try:

        execute(
            """
            INSERT INTO products
            (
                sku,
                name,
                category,
                qty,
                min_qty,
                cost,
                price
            )
            VALUES(?,?,?,?,?,?,?)
            """,

            (
                sku,

                name,

                data.get(
                    "category",
                    "General"
                ),

                int(
                    number(
                        data.get(
                            "qty"
                        )
                    )
                ),

                int(
                    number(
                        data.get(
                            "min_qty",
                            10
                        )
                    )
                ),

                number(
                    data.get(
                        "cost"
                    )
                ),

                number(
                    data.get(
                        "price"
                    )
                )
            )
        )

    except Exception:

        return jsonify({
            "ok": False,
            "error": "No se pudo crear el producto. Revisá el SKU."
        }), 400

    return jsonify({
        "ok": True
    })


@app.put("/api/stock/<int:product_id>")
def update_stock(
    product_id
):

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    quantity = int(
        number(
            data.get(
                "qty"
            )
        )
    )

    execute(
        """
        UPDATE products
        SET qty=?
        WHERE id=?
        """,

        (
            quantity,
            product_id
        )
    )

    return jsonify({
        "ok": True
    })


# ============================================================
# FLUJO DE CAJA
# ============================================================

@app.get("/api/cashflow")
def get_cashflow():

    return jsonify({

        "movements":
            execute(
                """
                SELECT *
                FROM cash_movements
                ORDER BY id DESC
                """,
                fetch=True
            ),

        "forecast":
            calculate_forecast()
    })


@app.post("/api/cashflow")
def create_cashflow():

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    movement_type = data.get(
        "movement_type"
    )

    amount = number(
        data.get(
            "amount"
        )
    )

    if movement_type not in {
        "ingreso",
        "egreso"
    }:

        return jsonify({
            "ok": False,
            "error": "Tipo de movimiento inválido"
        }), 400

    if amount <= 0:

        return jsonify({
            "ok": False,
            "error": "El importe debe ser mayor que cero"
        }), 400

    execute(
        """
        INSERT INTO cash_movements
        (
            movement_type,
            description,
            amount,
            movement_date
        )
        VALUES(?,?,?,?)
        """,

        (
            movement_type,

            data.get(
                "description",
                "Movimiento"
            ),

            amount,

            data.get(
                "movement_date",
                current_date()
            )
        )
    )

    return jsonify({
        "ok": True
    })


# ============================================================
# PRESUPUESTOS
# ============================================================

@app.get("/api/budgets")
def get_budgets():

    return jsonify(
        execute(
            """
            SELECT *
            FROM budgets
            ORDER BY id
            """,
            fetch=True
        )
    )


@app.post("/api/budgets")
def create_budget():

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    name = str(
        data.get(
            "name",
            "Nuevo presupuesto"
        )
    ).strip()

    execute(
        """
        INSERT INTO budgets
        (
            name,
            budget,
            spent
        )
        VALUES(?,?,?)
        """,

        (
            name,

            number(
                data.get(
                    "budget"
                )
            ),

            number(
                data.get(
                    "spent"
                )
            )
        )
    )

    return jsonify({
        "ok": True
    })


# ============================================================
# INVERSIONES
# ============================================================

@app.get("/api/investments")
def get_investments():

    rows = execute(
        """
        SELECT *
        FROM investments
        ORDER BY id DESC
        """,
        fetch=True
    )

    for row in rows:

        row[
            "estimated_annual_return"
        ] = (
            number(
                row["amount"]
            ) *
            number(
                row["rate"]
            ) /
            100
        )

    return jsonify(rows)


@app.post("/api/investments")
def create_investment():

    data = (
        request.get_json(
            silent=True
        )
        or {}
    )

    execute(
        """
        INSERT INTO investments
        (
            name,
            instrument,
            amount,
            rate,
            created_at
        )
        VALUES(?,?,?,?,?)
        """,

        (
            data.get(
                "name",
                "Inversión"
            ),

            data.get(
                "instrument",
                "Otro"
            ),

            number(
                data.get(
                    "amount"
                )
            ),

            number(
                data.get(
                    "rate"
                )
            ),

            current_date()
        )
    )

    return jsonify({
        "ok": True
    })


# ============================================================
# FORECAST
# ============================================================

@app.get("/api/forecast")
def api_forecast():

    return jsonify(
        calculate_forecast()
    )


# ============================================================
# ALERTAS
# ============================================================

@app.get("/api/alerts")
def get_alerts():

    alerts = []

    overdue = execute(
        """
        SELECT
            client,
            amount
        FROM invoices
        WHERE status='Vencida'
        """,
        fetch=True
    )

    for invoice in overdue:

        alerts.append({

            "type":
                "danger",

            "title":
                "Factura vencida",

            "message":
                (
                    f'{invoice["client"]} · '
                    f'ARS {number(invoice["amount"]):,.0f}'
                )
        })

    low_stock = execute(
        """
        SELECT
            name,
            qty,
            min_qty
        FROM products
        WHERE qty < min_qty
        """,
        fetch=True
    )

    for product in low_stock:

        alerts.append({

            "type":
                "warning",

            "title":
                "Stock bajo",

            "message":
                (
                    f'{product["name"]} · '
                    f'{product["qty"]} unidades'
                )
        })

    return jsonify(
        alerts
    )


# ============================================================
# BCRA
# ============================================================

@app.get("/api/bcra")
def bcra():

    url = (
        "https://api.bcra.gob.ar/"
        "estadisticascambiarias/v1.0/"
        "Cotizaciones"
    )

    try:

        response = requests.get(
            url,
            timeout=10
        )

        response.raise_for_status()

        return jsonify({

            "ok":
                True,

            "source":
                "BCRA",

            "data":
                response.json()
        })

    except Exception as error:

        return jsonify({

            "ok":
                False,

            "source":
                "BCRA",

            "error":
                str(error),

            "data":
                None
        })


# ============================================================
# COMMODITIES
# ============================================================

@app.get("/api/commodities")
def commodities():

    return jsonify({

        "ok":
            True,

        "provider":
            "not_configured",

        "items": [

            {
                "name":
                    "Oro",

                "symbol":
                    "XAU"
            },

            {
                "name":
                    "Plata",

                "symbol":
                    "XAG"
            },

            {
                "name":
                    "Petróleo",

                "symbol":
                    "CL"
            },

            {
                "name":
                    "Soja",

                "symbol":
                    "ZS"
            },

            {
                "name":
                    "Maíz",

                "symbol":
                    "ZC"
            },

            {
                "name":
                    "Trigo",

                "symbol":
                    "ZW"
            }
        ]
    })


# ============================================================
# HEALTH CHECK
# ============================================================

@app.get("/health")
def health():

    return jsonify({

        "status":
            "ok",

        "application":
            "Kontalo",

        "database":
            (
                "PostgreSQL"
                if DATABASE_URL
                else "SQLite"
            )
    })


# ============================================================
# FRONTEND
# ============================================================

HTML = r"""
<!DOCTYPE html>

<html lang="es">

<head>

<meta charset="UTF-8">

<meta
name="viewport"
content="width=device-width,initial-scale=1.0"
>

<title>Kontalo</title>

<style>

*{
box-sizing:border-box;
}

body{

margin:0;

background:#071019;

color:#edf3f8;

font-family:
Arial,
Helvetica,
sans-serif;

}

header{

height:64px;

display:flex;

align-items:center;

justify-content:space-between;

padding:0 22px;

background:#09131e;

border-bottom:
1px solid #1d3040;

}

.logo{

font-size:25px;

font-weight:800;

}

.logo b{

color:#00e5a0;

}

.status{

font-size:12px;

color:#8fa2b4;

}

.layout{

display:grid;

grid-template-columns:220px 1fr;

min-height:
calc(100vh - 64px);

}

.sidebar{

background:#09131e;

border-right:
1px solid #1d3040;

padding:12px;

}

.nav{

width:100%;

display:block;

border:0;

background:transparent;

color:#9db0c1;

padding:11px;

margin:3px 0;

border-radius:8px;

text-align:left;

cursor:pointer;

}

.nav:hover,
.nav.active{

background:#0d2d25;

color:#00e5a0;

}

main{

padding:25px;

max-width:1450px;

width:100%;

margin:auto;

}

.title{

font-size:28px;

font-weight:800;

}

.subtitle{

font-size:13px;

color:#8fa2b4;

margin:
6px 0 20px;

}

.grid{

display:grid;

grid-template-columns:
repeat(4,1fr);

gap:12px;

}

.grid3{

display:grid;

grid-template-columns:
repeat(3,1fr);

gap:12px;

}

.market{

display:grid;

grid-template-columns:
repeat(4,1fr);

gap:12px;

}

.card{

background:#0d1824;

border:
1px solid #1d3040;

border-radius:12px;

padding:16px;

}

.label{

font-size:12px;

color:#8fa2b4;

}

.value{

font-size:23px;

font-weight:800;

margin-top:7px;

}

.small{

font-size:12px;

color:#8fa2b4;

}

.green{

color:#00e5a0;

}

.red{

color:#ff6b6b;

}

.yellow{

color:#f6b73c;

}

.alert{

padding:11px;

margin:8px 0;

background:#111d28;

border-left:
3px solid #f6b73c;

border-radius:0 8px 8px 0;

}

.alert.ok{

border-left-color:#00e5a0;

}

.alert.danger{

border-left-color:#ff6b6b;

}

.table{

width:100%;

border-collapse:collapse;

}

.table th,
.table td{

padding:9px;

border-bottom:
1px solid #1d3040;

font-size:12px;

text-align:left;

}

.table th{

color:#8fa2b4;

}

.action{

border:
1px solid #1d3040;

background:#111f2d;

color:#edf3f8;

padding:9px 13px;

border-radius:8px;

cursor:pointer;

}

.primary{

background:#00e5a0;

color:#06120e;

font-weight:800;

}

.toolbar{

display:flex;

gap:8px;

margin-bottom:12px;

}

pre