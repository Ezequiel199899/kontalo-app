import os
import sqlite3
from datetime import date
from flask import Flask, jsonify, request, render_template_string

try:
    import requests
except ImportError:
    requests = None

app = Flask(__name__)

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
DB = os.path.join(BASE_DIR, "kontalo.db")


# ============================================================
# BASE DE DATOS
# ============================================================

def conn():
    c = sqlite3.connect(DB)
    c.row_factory = sqlite3.Row
    return c


def init_db():
    c = conn()

    c.executescript("""
        CREATE TABLE IF NOT EXISTS invoices (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            client TEXT NOT NULL,
            amount REAL NOT NULL,
            due TEXT NOT NULL,
            status TEXT NOT NULL
        );

        CREATE TABLE IF NOT EXISTS stock (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            sku TEXT UNIQUE NOT NULL,
            name TEXT NOT NULL,
            qty INTEGER NOT NULL,
            minimum INTEGER NOT NULL,
            cost REAL NOT NULL
        );

        CREATE TABLE IF NOT EXISTS budgets (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            budget REAL NOT NULL,
            spent REAL NOT NULL
        );

        CREATE TABLE IF NOT EXISTS investments (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            amount REAL NOT NULL,
            rate REAL NOT NULL
        );

        CREATE TABLE IF NOT EXISTS cash (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            kind TEXT NOT NULL,
            description TEXT NOT NULL,
            amount REAL NOT NULL,
            created TEXT NOT NULL
        );
    """)

    if c.execute("SELECT COUNT(*) FROM invoices").fetchone()[0] == 0:
        c.executemany(
            """
            INSERT INTO invoices(client, amount, due, status)
            VALUES (?, ?, ?, ?)
            """,
            [
                ("BuildCo Ltd", 1240000, "2026-09-05", "Pendiente"),
                ("Metro Supplies", 875000, "2026-08-18", "Vencida"),
                ("Alpha Group", 3120000, "2026-09-12", "Pagada"),
            ],
        )

    if c.execute("SELECT COUNT(*) FROM stock").fetchone()[0] == 0:
        c.executemany(
            """
            INSERT INTO stock(sku, name, qty, minimum, cost)
            VALUES (?, ?, ?, ?, ?)
            """,
            [
                ("SKU-001", "Taladro profesional", 24, 10, 185000),
                ("SKU-002", "Amoladora 900W", 7, 12, 132000),
                ("SKU-003", "Disco de corte", 80, 30, 4200),
            ],
        )

    if c.execute("SELECT COUNT(*) FROM budgets").fetchone()[0] == 0:
        c.executemany(
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

    if c.execute("SELECT COUNT(*) FROM investments").fetchone()[0] == 0:
        c.executemany(
            """
            INSERT INTO investments(name, amount, rate)
            VALUES (?, ?, ?)
            """,
            [
                ("Plazo fijo", 2000000, 32),
                ("FCI money market", 850000, 28),
            ],
        )

    if c.execute("SELECT COUNT(*) FROM cash").fetchone()[0] == 0:
        c.executemany(
            """
            INSERT INTO cash(kind, description, amount, created)
            VALUES (?, ?, ?, ?)
            """,
            [
                ("ingreso", "Cobro de clientes", 4670000, date.today().isoformat()),
                ("egreso", "Compras y operación", 3210000, date.today().isoformat()),
            ],
        )

    c.commit()
    c.close()


init_db()


# ============================================================
# FRONTEND EMBEBIDO
# ============================================================

HTML = r"""
<!doctype html>
<html lang="es">

<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">

<title>Kontalo</title>

<style>

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    background: #071019;
    color: #edf3f8;
    font-family: Arial, sans-serif;
}

header {
    height: 64px;
    background: #09131e;
    border-bottom: 1px solid #1d3040;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    position: sticky;
    top: 0;
    z-index: 10;
}

.logo {
    font-size: 24px;
    font-weight: bold;
}

.logo span {
    color: #00e5a0;
}

.status {
    font-size: 12px;
    color: #8fa2b4;
    border: 1px solid #1d3040;
    padding: 7px 10px;
    border-radius: 8px;
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
    background: transparent;
    color: #9db0c1;
    border: 0;
    padding: 11px;
    text-align: left;
    border-radius: 8px;
    margin-bottom: 4px;
    cursor: pointer;
}

.nav:hover,
.nav.active {
    background: #0d2d25;
    color: #00e5a0;
}

.main {
    padding: 25px;
    max-width: 1500px;
    width: 100%;
    margin: auto;
}

.title {
    font-size: 28px;
    font-weight: bold;
}

.subtitle {
    color: #8fa2b4;
    font-size: 13px;
    margin: 5px 0 20px;
}

.grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 12px;
}

.grid3 {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
}

.card {
    background: #0d1824;
    border: 1px solid #1d3040;
    border-radius: 12px;
    padding: 16px;
}

.label {
    color: #8fa2b4;
    font-size: 12px;
}

.value {
    font-size: 24px;
    font-weight: bold;
    margin: 8px 0;
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

.btn {
    background: #111f2d;
    color: white;
    border: 1px solid #1d3040;
    padding: 9px 12px;
    border-radius: 8px;
    cursor: pointer;
}

.btn.primary {
    background: #00e5a0;
    color: #06120e;
    font-weight: bold;
}

.toolbar {
    display: flex;
    gap: 8px;
    margin-bottom: 12px;
    flex-wrap: wrap;
}

.table {
    width: 100%;
    border-collapse: collapse;
}

.table th,
.table td {
    padding: 10px 7px;
    border-bottom: 1px solid #1d3040;
    font-size: 12px;
    text-align: left;
}

.table th {
    color: #8fa2b4;
}

.badge {
    display: inline-block;
    padding: 4px 8px;
    border-radius: 999px;
    background: #332914;
    color: #f6b73c;
    font-size: 10px;
}

.badge.green {
    background: #0d2d25;
    color: #00e5a0;
}

.badge.red {
    background: #321b22;
    color: #ff6b6b;
}

.alert {
    padding: 12px;
    margin: 8px 0;
    border-left: 3px solid #f6b73c;
    background: #101c27;
    border-radius: 0 8px 8px 0;
}

.alert.red {
    border-left-color: #ff6b6b;
}

.alert.green {
    border-left-color: #00e5a0;
}

.market {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
}

@media(max-width:900px) {

    .layout {
        grid-template-columns: 1fr;
    }

    .sidebar {
        display: flex;
        overflow-x: auto;
        min-height: auto;
    }

    .nav {
        min-width: max-content;
    }

    .grid,
    .grid3,
    .market {
        grid-template-columns: 1fr 1fr;
    }
}

@media(max-width:550px) {

    .grid,
    .grid3,
    .market {
        grid-template-columns: 1fr;
    }

    .main {
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

<div class="status">
    BCRA · Finanzas · Operaciones
</div>

</header>

<div class="layout">

<aside class="sidebar" id="navigation"></aside>

<main class="main" id="main"></main>

</div>

<script>

let page = "dashboard";

const menus = [
    ["dashboard", "⬡ Dashboard"],
    ["cashflow", "◎ Flujo de caja"],
    ["invoices", "◈ Facturación"],
    ["stock", "▦ Stock"],
    ["budget", "▤ Presupuesto"],
    ["investments", "◇ Inversiones"],
    ["markets", "◌ Cotizaciones BCRA"],
    ["commodities", "◆ Commodities"],
    ["alerts", "△ Alertas"]
];

function money(value) {

    return new Intl.NumberFormat(
        "es-AR",
        {
            style: "currency",
            currency: "ARS",
            maximumFractionDigits: 0
        }
    ).format(value);

}

function drawNavigation() {

    document.getElementById("navigation").innerHTML =
        menus.map(
            item => `
                <button
                    class="nav ${page === item[0] ? "active" : ""}"
                    onclick="go('${item[0]}')">
                    ${item[1]}
                </button>
            `
        ).join("");

}

async function api(url, options) {

    const response = await fetch(url, options);

    if (!response.ok) {
        throw new Error("Error HTTP " + response.status);
    }

    return response.json();

}

function card(title, value, description = "") {

    return `
        <div class="card">

            <div class="label">
                ${title}
            </div>

            <div class="value">
                ${value}
            </div>

            <div class="small">
                ${description}
            </div>

        </div>
    `;

}

async function go(newPage) {

    page = newPage;

    drawNavigation();

    await render();

}

async function render() {

    let html = "";

    if (page === "dashboard") {

        html = `

            <div class="title">
                Dashboard
            </div>

            <div class="subtitle">
                Visión general de la empresa.
            </div>

            <div class="grid">

                ${card(
                    "Caja actual",
                    money(12845000),
                    "Disponible"
                )}

                ${card(
                    "Por cobrar",
                    money(5115000),
                    "Facturas pendientes"
                )}

                ${card(
                    "Inventario",
                    money(5450000),
                    "Valor estimado"
                )}

                ${card(
                    "Alertas",
                    "3",
                    "Requieren atención"
                )}

            </div>

            <div class="grid3" style="margin-top:18px">

                <div class="card">

                    <b>
                        🤖 IA financiera
                    </b>

                    <div class="alert red">
                        Hay facturas vencidas que pueden afectar la caja.
                    </div>

                    <div class="alert green">
                        Kontalo puede proyectar escenarios de caja.
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
                        Escenario base a 30 días
                    </div>

                </div>

                <div class="card">

                    <b>
                        🏗 Arquitectura
                    </b>

                    <p class="small">
                        Flask API
                    </p>

                    <p class="small">
                        SQLite / PostgreSQL preparado
                    </p>

                    <p class="small">
                        BCRA
                    </p>

                </div>

            </div>

        `;

    }

    else if (page === "cashflow") {

        html = `

            <div class="title">
                Flujo de caja
            </div>

            <div class="subtitle">
                Control de ingresos, egresos y proyección.
            </div>

            <div class="grid">

                ${card(
                    "Caja",
                    money(12845000)
                )}

                ${card(
                    "Ingresos",
                    money(4670000),
                    "Período actual"
                )}

                ${card(
                    "Egresos",
                    money(3210000),
                    "Período actual"
                )}

                ${card(
                    "Proyección",
                    money(13872600),
                    "30 días"
                )}

            </div>

            <div class="card" style="margin-top:18px">

                <b>
                    Escenarios
                </b>

                <table class="table">

                    <tr>
                        <th>Escenario</th>
                        <th>Proyección</th>
                        <th>Lectura</th>
                    </tr>

                    <tr>
                        <td>Pesimista</td>
                        <td class="red">
                            ${money(12331200)}
                        </td>
                        <td>
                            Menores cobranzas
                        </td>
                    </tr>

                    <tr>
                        <td>Base</td>
                        <td class="green">
                            ${money(13872600)}
                        </td>
                        <td>
                            Comportamiento esperado
                        </td>
                    </tr>

                    <tr>
                        <td>Optimista</td>
                        <td class="green">
                            ${money(14771750)}
                        </td>
                        <td>
                            Cobranzas aceleradas
                        </td>
                    </tr>

                </table>

            </div>

        `;

    }

    else if (page === "invoices") {

        const data = await api("/api/invoices");

        html = `

            <div class="title">
                Facturación
            </div>

            <div class="subtitle">
                Facturas y cuentas por cobrar.
            </div>

            <div class="toolbar">

                <button
                    class="btn primary"
                    onclick="addInvoice()">
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

                    ${data.map(
                        x => `

                            <tr>

                                <td>
                                    ${x.id}
                                </td>

                                <td>
                                    ${x.client}
                                </td>

                                <td>
                                    ${money(x.amount)}
                                </td>

                                <td>
                                    ${x.due}
                                </td>

                                <td>

                                    <span
                                        class="badge ${
                                            x.status === "Pagada"
                                                ? "green"
                                                : x.status === "Vencida"
                                                    ? "red"
                                                    : ""
                                        }">

                                        ${x.status}

                                    </span>

                                </td>

                            </tr>

                        `
                    ).join("")}

                </table>

            </div>

        `;

    }

    else if (page === "stock") {

        const data = await api("/api/stock");

        html = `

            <div class="title">
                Stock
            </div>

            <div class="subtitle">
                Inventario y reposición.
            </div>

            <div class="toolbar">

                <button
                    class="btn primary"
                    onclick="addStock()">
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

                    ${data.map(
                        x => `

                            <tr>

                                <td>
                                    ${x.sku}
                                </td>

                                <td>
                                    ${x.name}
                                </td>

                                <td>
                                    ${x.qty}
                                </td>

                                <td>
                                    ${x.minimum}
                                </td>

                                <td>
                                    ${money(x.cost)}
                                </td>

                                <td>

                                    ${
                                        x.qty < x.minimum

                                        ? `
                                            <span class="badge">
                                                Reponer
                                            </span>
                                          `

                                        : `
                                            <span class="badge green">
                                                OK
                                            </span>
                                          `
                                    }

                                </td>

                            </tr>

                        `
                    ).join("")}

                </table>

            </div>

        `;

    }

    else if (page === "budget") {

        const data = await api("/api/budgets");

        html = `

            <div class="title">
                Presupuesto
            </div>

            <div class="subtitle">
                Presupuesto contra ejecución.
            </div>

            <div class="grid3">

                ${data.map(
                    x => {

                        const percent =
                            x.budget > 0
                                ? Math.round(
                                    x.spent / x.budget * 100
                                )
                                : 0;

                        return `

                            <div class="card">

                                <b>
                                    ${x.name}
                                </b>

                                <div class="value">
                                    ${percent}%
                                </div>

                                <div class="small">
                                    ${money(x.spent)}
                                    /
                                    ${money(x.budget)}
  