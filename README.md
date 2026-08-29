
   from flask import Flask, jsonify, request, render_template_string
import sqlite3
import requests
import os

app = Flask(__name__)

DB = "kontalo.db"


def get_db():
    connection = sqlite3.connect(DB)
    connection.row_factory = sqlite3.Row
    return connection


def init_db():
    db = get_db()

    db.execute("""
        CREATE TABLE IF NOT EXISTS invoices (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            client TEXT NOT NULL,
            amount REAL NOT NULL,
            due_date TEXT,
            status TEXT NOT NULL DEFAULT 'Pendiente'
        )
    """)

    db.execute("""
        CREATE TABLE IF NOT EXISTS stock (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            sku TEXT,
            name TEXT NOT NULL,
            quantity INTEGER NOT NULL DEFAULT 0,
            minimum INTEGER NOT NULL DEFAULT 0,
            cost REAL NOT NULL DEFAULT 0
        )
    """)

    db.execute("""
        CREATE TABLE IF NOT EXISTS budgets (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            budget REAL NOT NULL,
            spent REAL NOT NULL DEFAULT 0
        )
    """)

    db.execute("""
        CREATE TABLE IF NOT EXISTS investments (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            amount REAL NOT NULL,
            rate REAL NOT NULL DEFAULT 0
        )
    """)

    if db.execute("SELECT COUNT(*) FROM invoices").fetchone()[0] == 0:
        db.executemany(
            """
            INSERT INTO invoices(client, amount, due_date, status)
            VALUES (?, ?, ?, ?)
            """,
            [
                ("BuildCo Ltd", 1240000, "2026-09-05", "Pendiente"),
                ("Metro Supplies", 875000, "2026-08-18", "Vencida"),
                ("Alpha Group", 3120000, "2026-09-12", "Pagada"),
            ],
        )

    if db.execute("SELECT COUNT(*) FROM stock").fetchone()[0] == 0:
        db.executemany(
            """
            INSERT INTO stock(sku, name, quantity, minimum, cost)
            VALUES (?, ?, ?, ?, ?)
            """,
            [
                ("SKU-001", "Taladro profesional", 24, 10, 185000),
                ("SKU-002", "Amoladora 900W", 7, 12, 132000),
                ("SKU-003", "Disco de corte", 80, 30, 4200),
            ],
        )

    if db.execute("SELECT COUNT(*) FROM budgets").fetchone()[0] == 0:
        db.executemany(
            """
            INSERT INTO budgets(name, budget, spent)
            VALUES (?, ?, ?)
            """,
            [
                ("Marketing", 500000, 420000),
                ("Compras", 3500000, 2980000),
                ("Operación", 1800000, 1710000),
            ],
        )

    if db.execute("SELECT COUNT(*) FROM investments").fetchone()[0] == 0:
        db.executemany(
            """
            INSERT INTO investments(name, amount, rate)
            VALUES (?, ?, ?)
            """,
            [
                ("Plazo fijo", 2000000, 32),
                ("FCI money market", 850000, 28),
            ],
        )

    db.commit()
    db.close()


@app.get("/")
def home():
    with open("index.html", "r", encoding="utf-8") as file:
        return render_template_string(file.read())


@app.get("/api/invoices")
def invoices():
    db = get_db()
    rows = db.execute(
        "SELECT * FROM invoices ORDER BY id DESC"
    ).fetchall()
    db.close()

    return jsonify([dict(row) for row in rows])


@app.post("/api/invoices")
def create_invoice():
    data = request.get_json() or {}

    client = data.get("client")
    amount = float(data.get("amount", 0))
    due_date = data.get("due_date", "")

    if not client or amount <= 0:
        return jsonify({"error": "Datos inválidos"}), 400

    db = get_db()

    db.execute(
        """
        INSERT INTO invoices(client, amount, due_date, status)
        VALUES (?, ?, ?, 'Pendiente')
        """,
        (client, amount, due_date),
    )

    db.commit()
    db.close()

    return jsonify({"success": True})


@app.get("/api/stock")
def stock():
    db = get_db()

    rows = db.execute(
        "SELECT * FROM stock ORDER BY id DESC"
    ).fetchall()

    db.close()

    return jsonify([dict(row) for row in rows])


@app.post("/api/stock")
def create_stock():
    data = request.get_json() or {}

    name = data.get("name")
    sku = data.get("sku", "")
    quantity = int(data.get("quantity", 0))
    minimum = int(data.get("minimum", 0))
    cost = float(data.get("cost", 0))

    if not name:
        return jsonify({"error": "Falta el nombre"}), 400

    db = get_db()

    db.execute(
        """
        INSERT INTO stock(sku, name, quantity, minimum, cost)
        VALUES (?, ?, ?, ?, ?)
        """,
        (sku, name, quantity, minimum, cost),
    )

    db.commit()
    db.close()

    return jsonify({"success": True})


@app.get("/api/budgets")
def budgets():
    db = get_db()

    rows = db.execute(
        "SELECT * FROM budgets ORDER BY id"
    ).fetchall()

    db.close()

    return jsonify([dict(row) for row in rows])


@app.get("/api/investments")
def investments():
    db = get_db()

    rows = db.execute(
        "SELECT * FROM investments ORDER BY id"
    ).fetchall()

    db.close()

    return jsonify([dict(row) for row in rows])


@app.get("/api/alerts")
def alerts():
    db = get_db()
    alerts_list = []

    invoices = db.execute(
        "SELECT * FROM invoices WHERE status = 'Vencida'"
    ).fetchall()

    for invoice in invoices:
        alerts_list.append({
            "type": "danger",
            "title": "Factura vencida",
            "message": f"{invoice['client']} · ${invoice['amount']:,.0f}"
        })

    stock_items = db.execute(
        "SELECT * FROM stock WHERE quantity < minimum"
    ).fetchall()

    for item in stock_items:
        alerts_list.append({
            "type": "warning",
            "title": "Stock bajo",
            "message": (
                f"{item['name']} · "
                f"{item['quantity']} unidades "
                f"(mínimo {item['minimum']})"
            )
        })

    db.close()

    return jsonify(alerts_list)


@app.get("/api/bcra")
def bcra():
    """
    El frontend NO llama directamente al BCRA.
    Flask hace la consulta desde el servidor.
    No hay API key en el frontend.
    """

    url = (
        "https://api.bcra.gob.ar/"
        "estadisticascambiarias/v1.0/Cotizaciones"
    )

    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()

        data = response.json()

        details = data.get("results", {}).get("detalle", [])

        currencies = {
            "USD",
            "EUR",
            "BRL",
            "CLP",
            "COP",
            "GBP",
            "CNY",
            "UYU",
        }

        result = {}

        for item in details:
            code = item.get("codigoMoneda")

            if code in currencies:
                result[code] = item.get("tipoCotizacion")

        return jsonify({
            "source": "BCRA",
            "data": result
        })

    except requests.RequestException as error:
        return jsonify({
            "source": "BCRA",
            "error": "No se pudo consultar el BCRA",
            "details": str(error)
        }), 503


@app.get("/api/health")
def health():
    return jsonify({
        "status": "ok",
        "application": "Kontalo"
    })


init_db()


if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))

    app.run(
        host="0.0.0.0",
        port=port,
        debug=False
    )<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Kontalo</title>

<style>

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    background: #071019;
    color: #edf3f8;
    font-family: Arial, Helvetica, sans-serif;
}

header {
    height: 64px;
    background: #09131e;
    border-bottom: 1px solid #1d3040;

    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 0 22px;
}

.logo {
    font-size: 23px;
    font-weight: 800;
}

.logo span {
    color: #00e5a0;
}

.status {
    color: #8fa2b4;
    font-size: 12px;
}

.layout {
    display: grid;
    grid-template-columns: 220px 1fr;
    min-height: calc(100vh - 64px);
}

.sidebar {
    background: #09131e;
    border-right: 1px solid #1d3040;
    padding: 15px;
}

.nav {
    width: 100%;
    border: 0;
    background: transparent;
    color: #8fa2b4;

    text-align: left;

    padding: 11px;
    margin: 3px 0;

    border-radius: 8px;

    cursor: pointer;
}

.nav:hover,
.nav.active {
    background: #0d2d25;
    color: #00e5a0;
}

main {
    padding: 25px;
    max-width: 1500px;
    width: 100%;
    margin: auto;
}

.title {
    font-size: 28px;
    font-weight: 800;
}

.subtitle {
    color: #8fa2b4;
    font-size: 13px;
    margin-top: 5px;
    margin-bottom: 22px;
}

.grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 12px;
}

.grid-3 {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
    margin-top: 18px;
}

.card {
    background: #0d1824;
    border: 1px solid #1d3040;
    border-radius: 12px;
    padding: 17px;
}

.label {
    color: #8fa2b4;
    font-size: 12px;
}

.value {
    font-size: 25px;
    font-weight: 800;
    margin: 7px 0;
}

.small {
    color: #8fa2b4;
    font-size: 12px;
}

.green {
    color: #00e5a0;
}

.red {
    color: #ff6b6b;
}

.yellow {
    color: #f6b73c;
}

.toolbar {
    display: flex;
    gap: 8px;
    margin-bottom: 12px;
}

button {
    font-family: inherit;
}

.btn {
    background: #111f2d;
    color: #edf3f8;

    border: 1px solid #1d3040;
    border-radius: 8px;

    padding: 9px 12px;

    cursor: pointer;
}

.btn-primary {
    background: #00e5a0;
    color: #06120e;

    border-color: #00e5a0;

    font-weight: 800;
}

.table {
    width: 100%;
    border-collapse: collapse;
}

.table th,
.table td {
    border-bottom: 1px solid #1d3040;

    padding: 11px 8px;

    text-align: left;

    font-size: 12px;
}

.table th {
    color: #8fa2b4;
}

.badge {
    display: inline-block;

    padding: 4px 8px;

    border-radius: 999px;

    font-size: 10px;

    background: #332914;
    color: #f6b73c;
}

.badge.green {
    background: #0d2d25;
    color: #00e5a0;
}

.badge.red {
    background: #321b22;
    color: #ff6b6b;
}

.market {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
}

.alert {
    background: #101c27;

    padding: 12px;

    margin: 8px 0;

    border-left: 3px solid #f6b73c;

    border-radius: 0 8px 8px 0;
}

.alert.danger {
    border-left-color: #ff6b6b;
}

.alert.success {
    border-left-color: #00e5a0;
}

input {
    background: #08121c;
    color: white;

    border: 1px solid #1d3040;

    padding: 9px;

    border-radius: 8px;
}

.form {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;

    margin-bottom: 14px;
}

@media(max-width:900px) {

    .layout {
        grid-template-columns: 1fr;
    }

    .sidebar {
        display: flex;
        overflow-x: auto;
    }

    .nav {
        min-width: max-content;
    }

    .grid,
    .grid-3,
    .market,
    .form {
        grid-template-columns: 1fr 1fr;
    }
}

@media(max-width:550px) {

    .grid,
    .grid-3,
    .market,
    .form {
        grid-template-columns: 1fr;
    }

    main {
        padding: 14px;
    }
}

</style>
</head>

<body>

<header>

<div class="logo">
    kontalo<span>.</span>
</div>

<div class="status" id="status">
    Kontalo · sistema
</div>

</header>


<div class="layout">

<aside class="sidebar">

<button class="nav active" onclick="showPage('dashboard', this)">
    ⬡ Dashboard
</button>

<button class="nav" onclick="showPage('cashflow', this)">
    ◎ Flujo de caja
</button>

<button class="nav" onclick="showPage('invoices', this)">
    ◈ Facturación
</button>

<button class="nav" onclick="showPage('stock', this)">
    ▦ Stock
</button>

<button class="nav" onclick="showPage('budget', this)">
    ▤ Presupuesto
</button>

<button class="nav" onclick="showPage('investments', this)">
    ◇ Inversiones
</button>

<button class="nav" onclick="showPage('markets', this)">
    ◌ Cotizaciones BCRA
</button>

<button class="nav" onclick="showPage('commodities', this)">
    ◆ Commodities
</button>

<button class="nav" onclick="showPage('alerts', this)">
    △ Alertas
</button>

</aside>


<main id="main">

</main>

</div>


<script>

const money = value => {

    return new Intl.NumberFormat(
        "es-AR",
        {
            style: "currency",
            currency: "ARS",
            maximumFractionDigits: 0
        }
    ).format(value);

};


async function api(url, options = {}) {

    const response = await fetch(url, options);

    if (!response.ok) {

        throw new Error(
            "Error HTTP " + response.status
        );

    }

    return response.json();

}


function showPage(page, element) {

    document
        .querySelectorAll(".nav")
        .forEach(button => {

            button.classList.remove("active");

        });

    if (element) {
        element.classList.add("active");
    }

    if (page === "dashboard") dashboard();

    if (page === "cashflow") cashflow();

    if (page === "invoices") invoices();

    if (page === "stock") stock();

    if (page === "budget") budget();

    if (page === "investments") investments();

    if (page === "markets") markets();

    if (page === "commodities") commodities();

    if (page === "alerts") alerts();

}


function dashboard() {

    document.getElementById("main").innerHTML = `

        <div class="title">
            Dashboard
        </div>

        <div class="subtitle">
            Visión financiera de tu empresa en un solo lugar.
        </div>

        <div class="grid">

            <div class="card">
                <div class="label">
                    Caja actual
                </div>

                <div class="value">
                    ${money(12845000)}
                </div>

                <div class="small">
                    Posición registrada
                </div>
            </div>


            <div class="card">

                <div class="label">
                    Cuentas por cobrar
                </div>

                <div class="value">
                    ${money(5115000)}
                </div>

                <div class="small">
                    Facturas pendientes
                </div>

            </div>


            <div class="card">

                <div class="label">
                    Inventario
                </div>

                <div class="value">
                    ${money(5450000)}
                </div>

                <div class="small">
                    Valor estimado
                </div>

            </div>


            <div class="card">

                <div class="label">
                    Alertas
                </div>

                <div class="value yellow">
                    3
                </div>

                <div class="small">
                    Requieren atención
                </div>

            </div>

        </div>


        <div class="grid-3">

            <div class="card">

                <b>
                    🤖 IA financiera
                </b>

                <div class="alert danger">

                    Hay facturas vencidas que
                    pueden afectar la caja.

                </div>

                <div class="alert success">

                    La IA podrá detectar
                    variaciones y generar
                    escenarios.

                </div>

            </div>


            <div class="card">

                <b>
                    📊 Proyección
                </b>

                <div class="value green">

                    ${money(13872600)}

                </div>

                <div class="small">

                    Escenario base · 30 días

                </div>

            </div>


            <div class="card">

                <b>
                    🏗 Arquitectura
                </b>

                <p class="small">
                    Frontend
                </p>

                <p class="small">
                    Flask / API
                </p>

                <p class="small">
                    PostgreSQL
                </p>

                <p class="small">
                    BCRA
                </p>

                <p class="small">
                    IA / Forecasting
                </p>

            </div>

        </div>

    `;

}


async function invoices() {

    const data = await api("/api/invoices");

    document.getElementById("main").innerHTML = `

        <div class="title">
            Facturación
        </div>

        <div class="subtitle">
            Clientes, facturas y cuentas por cobrar.
        </div>


        <div class="toolbar">

            <button
                class="btn btn-primary"
                onclick="newInvoice()"
            >
                + Nueva factura
            </button>

        </div>


        <div class="card">

            <table class="table">

                <thead>

                    <tr>

                        <th>ID</th>
                        <th>Cliente</th>
                        <th>Importe</th>
                        <th>Vencimiento</th>
                        <th>Estado</th>

                    </tr>

                </thead>

                <tbody>

                    ${data.map(invoice => `

                        <tr>

                            <td>
                                ${invoice.id}
                            </td>

                            <td>
                                ${invoice.client}
                            </td>

                            <td>
                                ${money(invoice.amount)}
                            </td>

                            <td>
                                ${invoice.due_date || "-"}
                            </td>

                            <td>

                                <span class="
                                    badge
                                    ${
                                        invoice.status === "Pagada"
                                        ? "green"
                                        : invoice.status === "Vencida"
                                        ? "red"
                                        : ""
                                    }
                                ">

                                    ${invoice.status}

                                </span>

                            </td>

                        </tr>

                    `).join("")}

                </tbody>

            </table>

        </div>

    `;

}


async function newInvoice() {

    const client = prompt(
        "Nombre del cliente:"
    );

    if (!client) return;


    const amount = Number(
        prompt("Importe:")
    );

    if (!amount || amount <= 0) return;


    const due = prompt(
        "Fecha de vencimiento (YYYY-MM-DD):"
    );


    await api(
        "/api/invoices",
        {
            method: "POST",

            headers: {
                "Content-Type":
                    "application/json"
            },

            body: JSON.stringify({
                client,
                amount,
                due_date: due
            })
        }
    );


    invoices();

}


async function stock() {

    const data = await api("/api/stock");

    document.getElementById("main").innerHTML = `

        <div class="title">
            Stock
        </div>

        <div class="subtitle">
            Inventario y alertas de reposición.
        </div>


        <div class="toolbar">

            <button
                class="btn btn-primary"
                onclick="newProduct()"
            >
                + Producto
            </button>

        </div>


        <div class="card">

            <table class="table">

                <thead>

                    <tr>

                        <th>SKU</th>
                        <th>Producto</th>
                        <th>Unidades</th>
                        <th>Mínimo</th>
                        <th>Costo</th>
                        <th>Estado</th>

                    </tr>

                </thead>

                <tbody>

                    ${data.map(item => `

                        <tr>

                            <td>
                                ${item.sku}
                            </td>

                            <td>
                                ${item.name}
                            </td>

                            <td>
                                ${item.quantity}
                            </td>

                            <td>
                                ${item.minimum}
                            </td>

                            <td>
                                ${money(item.cost)}
                            </td>

                            <td>

                                ${
                                    item.quantity <
                                    item.minimum

                                    ?

                                    `<span class="badge">
                                        Reponer
                                    </span>`

                                    :

                                    `<span class="badge green">
                                        OK
                                    </span>`
                                }

                            </td>

                        </tr>

                    `).join("")}

                </tbody>

            </table>

        </div>

    `;

}


async function newProduct() {

    const name = prompt(
        "Nombre del producto:"
    );

    if (!name) return;


    const sku = prompt(
        "SKU:"
    );


    const quantity = Number(
        prompt("Cantidad:")
    );


    const minimum = Number(
        prompt("Stock mínimo:")
    );


    const cost = Number(
        prompt("Costo unitario:")
    );


    await api(
        "/api/stock",
        {
            method: "POST",

            headers: {
                "Content-Type":
                    "application/json"
            },

            body: JSON.stringify({
                name,
                sku,
                quantity,
                minimum,
                cost
            })
        }
    );


    stock();

}


async function budget() {

    const data = await api(
        "/api/budgets"
    );


    document.getElementById("main").innerHTML = `

        <div class="title">
            Presupuesto
        </div>

        <div class="subtitle">
            Presupuesto contra gasto real.
        </div>


        <div class="grid-3">

            ${data.map(item => {

                const percentage =
                    Math.round(
                        item.spent /
                        item.budget *
                        100
                    );

                return `

                    <div class="card">

                        <b>
                            ${item.name}
                        </b>

                        <div class="value">

                            ${percentage}%

                        </div>

                        <div class="small">

                            ${money(item.spent)}
                            /
                            ${money(item.budget)}

                        </div>

                    </div>

                `;

            }).join("")}

        </div>

    `;

}


async function investments() {

    const data = await api(
        "/api/investments"
    );


    document.getElementById("main").innerHTML = `

        <div class="title">
            Inversiones
        </div>

        <div class="subtitle">
            Registro de inversiones y rendimiento estimado.
        </div>


        <div class="card">

            <table class="table">

                <thead>

                    <tr>

                        <th>
                            Instrumento
                        </th>

                        <th>
                            Capital
                        </th>

                        <th>
                            Tasa
                        </th>

                        <th>
                            Rendimiento estimado
                        </th>

                    </tr>

                </thead>

                <tbody>

                    ${data.map(item => `

                        <tr>

                            <td>
                                ${item.name}
                            </td>

                            <td>
                                ${money(item.amount)}
                            </td>

                            <td>
                                ${item.rate}%
                            </td>

                            <td class="green">

                                ${money(
                                    item.amount *
                                    item.rate /
                                    100
                                )}

                            </td>

                        </tr>

                    `).join("")}

                </tbody>

            </table>

        </div>

    `;

}


async function markets() {

    const response = await api(
        "/api/bcra"
    );


    const data =
        response.data || {};


    document.getElementById("main").innerHTML = `

        <div class="title">
            Cotizaciones BCRA
        </div>

        <div class="subtitle">

            Cotizaciones obtenidas por el
            backend desde la API pública
            del Banco Central.

        </div>


        <div class="market">

            ${Object.entries(data).map(
                ([currency, value]) => `

                    <div class="card">

                        <div class="label">
                            ${currency}
                        </div>

                        <div class="value">

                            ${value}

                        </div>

                        <div class="small">

                            ARS · BCRA

                        </div>

                    </div>

                `
            ).join("")}

        </div>

    `;

}


function cashflow() {

    document.getElementById("main").innerHTML = `

        <div class="title">
            Flujo de caja
        </div>

        <div class="subtitle">
            Control y proyección de liquidez.
        </div>


        <div class="grid">

            <div class="card">

                <div class="label">
                    Caja inicial
                </div>

                <div class="value">
                    ${money(12845000)}
                </div>

            </div>


            <div class="card">

                <div class="label">
                    Ingresos
                </div>

                <div class="value green">
                    ${money(4670000)}
                </div>

            </div>


            <div class="card">

                <div class="label">
                    Egresos
                </div>

                <div class="value re