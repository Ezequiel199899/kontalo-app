<!DOCTYPE html><html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kontalo</title><style>
*{box-sizing:border-box}
body{
    margin:0;
    background:#071019;
    color:#edf3f8;
    font-family:Arial,sans-serif
}
header{
    height:64px;
    background:#09131e;
    border-bottom:1px solid #1d3040;
    display:flex;
    align-items:center;
    justify-content:space-between;
    padding:0 20px;
}
.logo{
    font-size:24px;
    font-weight:800;
}
.logo span{color:#00e5a0}
.status{
    color:#8fa2b4;
    font-size:12px;
}
.layout{
    display:grid;
    grid-template-columns:210px 1fr;
    min-height:calc(100vh - 64px);
}
aside{
    background:#09131e;
    border-right:1px solid #1d3040;
    padding:12px;
}
nav button{
    width:100%;
    border:0;
    background:transparent;
    color:#9db0c1;
    padding:11px;
    border-radius:8px;
    text-align:left;
    cursor:pointer;
    margin:2px 0;
}
nav button:hover,
nav button.active{
    background:#0d2d25;
    color:#00e5a0;
}
main{
    padding:25px;
    max-width:1450px;
    width:100%;
    margin:auto;
}
h1{
    margin:0 0 5px;
    font-size:28px;
}
.sub{
    color:#8fa2b4;
    font-size:13px;
    margin-bottom:20px;
}
.grid{
    display:grid;
    grid-template-columns:repeat(4,1fr);
    gap:12px;
}
.grid3{
    display:grid;
    grid-template-columns:repeat(3,1fr);
    gap:12px;
}
.card{
    background:#0d1824;
    border:1px solid #1d3040;
    border-radius:12px;
    padding:16px;
}
.label{
    color:#8fa2b4;
    font-size:12px;
}
.value{
    font-size:24px;
    font-weight:800;
    margin:8px 0;
}
.small{
    color:#8fa2b4;
    font-size:12px;
}
.ok{color:#00e5a0}
.bad{color:#ff6b6b}
.warn{color:#f6b73c}
table{
    width:100%;
    border-collapse:collapse;
}
th,td{
    padding:10px 7px;
    border-bottom:1px solid #1d3040;
    text-align:left;
    font-size:12px;
}
th{color:#8fa2b4}
button.action{
    border:1px solid #1d3040;
    background:#111f2d;
    color:#fff;
    padding:9px 12px;
    border-radius:8px;
    cursor:pointer;
}
button.primary{
    background:#00e5a0;
    color:#06120e;
    font-weight:800;
}
.toolbar{
    display:flex;
    gap:8px;
    margin-bottom:12px;
}
.badge{
    font-size:10px;
    padding:4px 8px;
    border-radius:999px;
    background:#332914;
    color:#f6b73c;
}
.badge.green{
    background:#0d2d25;
    color:#00e5a0;
}
.badge.red{
    background:#321b22;
    color:#ff6b6b;
}
.alert{
    padding:12px;
    margin:8px 0;
    background:#101c27;
    border-left:3px solid #f6b73c;
    border-radius:0 8px 8px 0;
}
.alert.bad{border-left-color:#ff6b6b}
.alert.ok{border-left-color:#00e5a0}
.market{
    display:grid;
    grid-template-columns:repeat(4,1fr);
    gap:12px;
}
.loading{
    color:#8fa2b4;
    padding:20px 0;
}
@media(max-width:900px){
    .layout{grid-template-columns:1fr}
    aside{overflow:auto}
    nav{
        display:flex;
        gap:5px;
        overflow:auto;
    }
    nav button{
        white-space:nowrap;
        min-width:max-content;
    }
    .grid,.grid3,.market{
        grid-template-columns:1fr 1fr;
    }
}
@media(max-width:550px){
    main{padding:14px}
    .grid,.grid3,.market{
        grid-template-columns:1fr;
    }
}
</style></head><body><header>
    <div class="logo">kontalo<span>.</span></div>
    <div class="status" id="connectionStatus">
        Conectando al backend...
    </div>
</header><div class="layout"><aside>
<nav id="nav"></nav>
</aside><main id="main">
<div class="loading">Cargando Kontalo...</div>
</main></div><script>

/*
===========================================================
KONTALO FRONTEND
===========================================================

IMPORTANTE:

CAMBIÁ ESTA URL POR LA URL PÚBLICA DONDE ESTÉ CORRIENDO
TU app.py.

Ejemplo:

const API_BASE =
"https://tu-backend.onrender.com";

NO pongas aquí API keys.

===========================================================
*/

const API_BASE =
"https://TU-BACKEND.onrender.com";


const menus = [
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

let page = "dashboard";


function money(value){

    return new Intl.NumberFormat(
        "es-AR",
        {
            style:"currency",
            currency:"ARS",
            maximumFractionDigits:0
        }
    ).format(Number(value)||0);

}


async function api(endpoint,options={}){

    const response =
        await fetch(API_BASE + endpoint,{
            ...options,
            headers:{
                "Content-Type":"application/json",
                ...(options.headers||{})
            }
        });

    if(!response.ok){

        throw new Error(
            "Error HTTP " + response.status
        );

    }

    return await response.json();

}


function renderNav(){

    document.getElementById("nav").innerHTML =
        menus.map(item => {

            return `
            <button
                class="${page===item[0]?"active":""}"
                onclick="go('${item[0]}')">
                ${item[1]}
            </button>
            `;

        }).join("");

}


function card(title,value,description=""){

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


async function go(newPage){

    page = newPage;

    renderNav();

    await render();

}


async function render(){

    const main =
        document.getElementById("main");

    try{

        if(page==="dashboard"){

            main.innerHTML = `

            <h1>Dashboard</h1>

            <div class="sub">
                Control financiero y operativo de tu empresa.
            </div>

            <div class="grid">

                ${card(
                    "Caja actual",
                    money(12845000),
                    "Saldo disponible"
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

            <div class="grid3"
                 style="margin-top:18px">

                <div class="card">

                    <b>🤖 IA financiera</b>

                    <div class="alert bad">
                        Hay facturas vencidas que pueden afectar la caja.
                    </div>

                    <div class="alert ok">
                        Kontalo puede analizar los datos y proyectar escenarios.
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
                        GitHub Pages → Flask API
                    </p>

                    <p class="small">
                        Flask → base de datos
                    </p>

                    <p class="small">
                        Flask → BCRA
                    </p>

                </div>

            </div>

            `;

        }


        else if(page==="cashflow"){

            main.innerHTML = `

            <h1>Flujo de caja</h1>

            <div class="sub">
                Ingresos, egresos y proyecciones.
            </div>

            <div class="grid">

                ${card(
                    "Caja inicial",
                    money(12845000)
                )}

                ${card(
                    "Ingresos",
                    money(4670000)
                )}

                ${card(
                    "Egresos",
                    money(3210000)
                )}

                ${card(
                    "Caja proyectada",
                    money(14260000)
                )}

            </div>

            <div class="card"
                 style="margin-top:18px">

                <b>Escenarios</b>

                <table>

                    <tr>
                        <th>Escenario</th>
                        <th>Proyección</th>
                        <th>Lectura</th>
                    </tr>

                    <tr>
                        <td>Pesimista</td>
                        <td class="bad">
                            ${money(12331200)}
                        </td>
                        <td>
                            Menores cobranzas
                        </td>
                    </tr>

                    <tr>
                        <td>Base</td>
                        <td class="ok">
                            ${money(13872600)}
                        </td>
                        <td>
                            Esperado
                        </td>
                    </tr>

                    <tr>
                        <td>Optimista</td>
                        <td class="ok">
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


        else if(page==="invoices"){

            const data =
                await api("/api/invoices");

            main.innerHTML = `

            <h1>Facturación</h1>

            <div class="sub">
                Cuentas por cobrar y vencimientos.
            </div>

            <div class="toolbar">

                <button
                    class="action primary"
                    onclick="addInvoice()">
                    + Nueva factura
                </button>

            </div>

            <div class="card">

                <table>

                    <tr>
                        <th>ID</th>
                        <th>Cliente</th>
                        <th>Importe</th>
                        <th>Vencimiento</th>
                        <th>Estado</th>
                    </tr>

                    ${
                        data.map(x => `

                        <tr>

                            <td>
                                ${x.id}
                            </td>

                            <td>
                                ${escapeHtml(x.client)}
                            </td>

                            <td>
                                ${money(x.amount)}
                            </td>

                            <td>
                                ${x.due}
                            </td>

                            <td>

                                <span class="badge ${
                                    x.status==="Pagada"
                                    ?"green":
                                    x.status==="Vencida"
                                    ?"red":""
                                }">

                                    ${x.status}

                                </span>

                            </td>

                        </tr>

                        `).join("")
                    }

                </table>

            </div>

            `;

        }


        else if(page==="stock"){

            const data =
                await api("/api/stock");

            main.innerHTML = `

            <h1>Stock</h1>

            <div class="sub">
                Inventario y reposición.
            </div>

            <div class="toolbar">

                <button
                    class="action primary"
                    onclick="addStock()">
                    + Producto
                </button>

            </div>

            <div class="card">

                <table>

                    <tr>
                        <th>SKU</th>
                        <th>Producto</th>
                        <th>Unidades</th>
                        <th>Mínimo</th>
                        <th>Costo</th>
                        <th>Estado</th>
                    </tr>

                    ${
                        data.map(x => `

                        <tr>

                            <td>
                                ${x.sku}
                            </td>

                            <td>
                                ${escapeHtml(x.name)}
                            </td>

                            <td>
                                ${x.qty}
                            </td>

                            <td>
                                ${x.min}
                            </td>

                            <td>
                                ${money(x.cost)}
                            </td>

                            <td>

                                ${
                                    x.qty < x.min

                                    ?

                                    '<span class="badge">Reponer</span>'

                                    :

                                    '<span class="badge green">OK</span>'
                                }

                            </td>

                        </tr>

                        `).join("")
                    }

                </table>

            </div>

            `;

        }


        else if(page==="budget"){

            const data =
                await api("/api/budgets");

            main.innerHTML = `

            <h1>Presupuesto</h1>

            <div class="sub">
                Presupuesto contra ejecución.
            </div>

            <div class="grid3">

                ${
                    data.map(x => {

                        const percent =
                            Math.round(
                                x.spent /
                                x.budget *
                                100
                            );

                        return `

                        <div class="card">

                            <b>
                                ${escapeHtml(x.name)}
                            </b>

                            <div class="value">
                                ${percent}%
                            </div>

                            <div class="small">
                                ${money(x.spent)}
                                /
                                ${money(x.budget)}
                            </div>

                        </div>

                        `;

                    }).join("")
                }

            </div>

            `;

        }


        else if(page==="investments"){

            const data =
                await api("/api/investments");

            main.innerHTML = `

            <h1>Inversiones</h1>

            <div class="sub">
                Capital y rendimiento estimado.
            </div>

            <div class="card">

                <table>

                    <tr>
                        <th>Instrumento</th>
                        <th>Capital</th>
                        <th>Tasa</th>
                        <th>Rendimiento anual</th>
                    </tr>

                    ${
                        data.map(x => `

                        <tr>

                            <td>
                                ${escapeHtml(x.name)}
                            </td>

                            <td>
                                ${money(x.amount)}
                            </td>

                            <td>
                                ${x.rate}%
                            </td>

                            <td class="ok">
                                ${money(
                                    x.amount *
                                    x.rate /
                                    100
                                )}
                            </td>

                        </tr>

                        `).join("")
                    }

                </table>

            </div>

            `;

        }


        else if(page==="markets"){

            const data =
                await api("/api/bcra");

            main.innerHTML = `

            <h1>Cotizaciones BCRA</h1>

            <div class="sub">
                Cotizaciones consultadas por el backend.
            </div>

            <div class="market">

                ${
                    Object.entries(data)
                    .map(([code,value]) => `

                    <div class="card">

                        <div class="label">
                            ${code}
                        </div>

                        <div class="value">
                            ${value ?? "—"}
                        </div>

                        <div class="small">
                            BCRA
                        </div>

                    </div>

                    `).join("")
                }

            </div>

            `;

        }


        else if(page==="commodities"){

            main.innerHTML = `

            <h1>Commodities</h1>

            <div class="sub">
                Módulo preparado para conectar proveedores externos.
            </div>

            <div class="market">

                ${
                    [
                        "Oro",
                        "Plata",
                        "Petróleo",
                        "Soja",
                        "Trigo",
                        "Maíz"
                    ].map(name => `

                    <div class="card">

                        <b>${name}</b>

                        <div class="value">
                            —
                        </div>

                        <div class="small">
                            Proveedor pendiente
                        </div>

                    </div>

                    `).join("")
                }

            </div>

            `;

        }


        else if(page==="alerts"){

            const data =
                await api("/api/alerts");

            main.innerHTML = `

            <h1>Alertas</h1>

            <div class="sub">
                Eventos financieros e inventario.
            </div>

            ${
                data.length

                ?

                data.map(x => `

                    <div class="alert ${x.type}">

                        <b>
                            ${escapeHtml(x.title)}
                        </b>

                        <br>

                        ${escapeHtml(x.message)}

                    </div>

                `).join("")

                :

                `<div class="alert ok">
                    No hay alertas.
                </div>`
            }

            `;

        }

    }

    catch(error){

        main.innerHTML = `

        <h1>Error de conexión</h1>

        <div class="alert bad">

            No se pudo conectar con el backend.

            <br><br>

            <b>
                ${escapeHtml(error.message)}
            </b>

            <br><br>

            Revisá que API_BASE apunte a la URL pública de tu app.py.

        </div>

        `;

    }

}


function escapeHtml(value){

    return String(value ?? "")
    