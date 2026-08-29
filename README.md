#!/bin/bash

# ============================================================
# KONTALO V2
# Frontend + Spring Boot + PostgreSQL + Flask/Python + BCRA
# ============================================================

set -e

PROJECT="Kontalo-V2"

echo "Creando proyecto $PROJECT..."

rm -rf "$PROJECT"

mkdir -p "$PROJECT/frontend"
mkdir -p "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/config"
mkdir -p "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller"
mkdir -p "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity"
mkdir -p "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository"
mkdir -p "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/service"
mkdir -p "$PROJECT/backend-spring/src/main/resources"
mkdir -p "$PROJECT/backend-flask"

# ============================================================
# docker-compose.yml
# ============================================================

cat > "$PROJECT/docker-compose.yml" <<'EOF'
services:

  postgres:
    image: postgres:15
    container_name: kontalo-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: contabilidad
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  flask:
    build:
      context: ./backend-flask
    container_name: kontalo-flask
    restart: unless-stopped
    environment:
      PORT: 5000
      AI_API_KEY: ${AI_API_KEY:-}
    ports:
      - "5000:5000"
    depends_on:
      - postgres

  spring:
    build:
      context: ./backend-spring
    container_name: kontalo-spring
    restart: unless-stopped
    environment:
      DB_URL: jdbc:postgresql://postgres:5432/contabilidad
      DB_USER: postgres
      DB_PASSWORD: postgres
      FLASK_BASE_URL: http://flask:5000
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - flask

  frontend:
    image: nginx:alpine
    container_name: kontalo-frontend
    restart: unless-stopped
    ports:
      - "3000:80"
    volumes:
      - ./frontend:/usr/share/nginx/html:ro
    depends_on:
      - spring

volumes:
  postgres_data:
EOF

# ============================================================
# .env.example
# ============================================================

cat > "$PROJECT/.env.example" <<'EOF'
# ============================================================
# KONTALO VARIABLES
# ============================================================

# Solo para el backend.
# NUNCA colocar esta clave en frontend/index.html.
AI_API_KEY=
EOF

# ============================================================
# .gitignore
# ============================================================

cat > "$PROJECT/.gitignore" <<'EOF'
.env
*.env

target/
*.class

__pycache__/
*.pyc
.venv/
venv/

.idea/
.vscode/

.DS_Store
EOF

# ============================================================
# README
# ============================================================

cat > "$PROJECT/README.md" <<'EOF'
# KONTALO V2

Plataforma financiera para pequeñas y medianas empresas.

## Arquitectura

Frontend
   |
   v
Spring Boot
   |
   +---- PostgreSQL
   |
   +---- BCRA
   |
   +---- Flask/Python
             |
             +---- IA
             +---- Forecasting
             +---- Commodities

## Servicios

Frontend:
http://localhost:3000

Spring Boot:
http://localhost:8080

Swagger:
http://localhost:8080/swagger-ui/index.html

Flask:
http://localhost:5000

PostgreSQL:
localhost:5432

## Seguridad

Nunca colocar API keys en el frontend.

Las claves externas deben estar en variables de entorno del backend.
EOF

# ============================================================
# FRONTEND
# ============================================================

cat > "$PROJECT/frontend/index.html" <<'EOF'
<!DOCTYPE html>

<html lang="es">

<head>

<meta charset="UTF-8">

<meta
name="viewport"
content="width=device-width, initial-scale=1.0"
>

<title>Kontalo</title>

<style>

:root{

--bg:#071019;
--panel:#0d1824;
--panel2:#111f2d;
--border:#1e3040;

--text:#edf4fa;
--muted:#8ea2b4;

--green:#00e5a0;
--greenDark:#0c3028;

--red:#ff6262;
--yellow:#f5b942;

}

*{
box-sizing:border-box;
}

body{

margin:0;

background:var(--bg);

color:var(--text);

font-family:

Inter,
system-ui,
-apple-system,
BlinkMacSystemFont,
"Segoe UI",
sans-serif;

}

header{

height:64px;

display:flex;

align-items:center;

justify-content:space-between;

padding:0 22px;

background:#09131e;

border-bottom:1px solid var(--border);

position:sticky;

top:0;

z-index:10;

}

.logo{

font-size:23px;

font-weight:800;

}

.logo span{

color:var(--green);

}

.status{

font-size:12px;

color:var(--muted);

border:1px solid var(--border);

padding:7px 10px;

border-radius:8px;

}

.layout{

display:grid;

grid-template-columns:220px 1fr;

min-height:calc(100vh - 64px);

}

.sidebar{

background:#09131e;

border-right:1px solid var(--border);

padding:14px;

}

.nav{

width:100%;

border:0;

background:transparent;

color:var(--muted);

text-align:left;

padding:11px;

margin-bottom:3px;

border-radius:8px;

cursor:pointer;

}

.nav:hover,
.nav.active{

background:var(--greenDark);

color:var(--green);

}

main{

width:100%;

max-width:1450px;

margin:auto;

padding:25px;

}

h1{

margin:0;

font-size:28px;

}

.subtitle{

margin-top:5px;

margin-bottom:20px;

font-size:13px;

color:var(--muted);

}

.grid4{

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

.card{

background:var(--panel);

border:1px solid var(--border);

border-radius:12px;

padding:16px;

}

.label{

font-size:12px;

color:var(--muted);

}

.value{

font-size:24px;

font-weight:750;

margin-top:7px;

}

.small{

font-size:12px;

color:var(--muted);

}

.green{

color:var(--green);

}

.red{

color:var(--red);

}

.yellow{

color:var(--yellow);

}

.section{

margin-top:16px;

}

.section h2{

font-size:15px;

margin-top:0;

}

button{

font-family:inherit;

}

.btn{

background:var(--panel2);

color:var(--text);

border:1px solid var(--border);

border-radius:8px;

padding:9px 12px;

cursor:pointer;

}

.btn:hover{

border-color:var(--green);

}

.btn.primary{

background:var(--green);

color:#06120e;

border-color:var(--green);

font-weight:700;

}

.toolbar{

display:flex;

gap:8px;

flex-wrap:wrap;

margin-bottom:12px;

}

table{

width:100%;

border-collapse:collapse;

}

th,
td{

padding:10px 7px;

border-bottom:1px solid var(--border);

font-size:12px;

text-align:left;

}

th{

color:var(--muted);

}

.badge{

display:inline-block;

padding:4px 8px;

border-radius:999px;

font-size:10px;

font-weight:700;

}

.ok{

background:var(--greenDark);

color:var(--green);

}

.warning{

background:#332914;

color:var(--yellow);

}

.danger{

background:#321a20;

color:var(--red);

}

.alert{

padding:12px;

margin:8px 0;

background:#101c27;

border-left:3px solid var(--yellow);

border-radius:0 8px 8px 0;

}

.alert.greenAlert{

border-color:var(--green);

}

.alert.redAlert{

border-color:var(--red);

}

.market{

display:grid;

grid-template-columns:

repeat(4,1fr);

gap:12px;

}

@media(max-width:950px){

.layout{

grid-template-columns:1fr;

}

.sidebar{

display:flex;

overflow:auto;

gap:5px;

}

.nav{

min-width:max-content;

}

.grid4,
.grid3,
.market{

grid-template-columns:

repeat(2,1fr);

}

}

@media(max-width:600px){

main{

padding:14px;

}

.grid4,
.grid3,
.market{

grid-template-columns:1fr;

}

}

</style>

</head>

<body>

<header>

<div class="logo">
kontalo<span>.</span>
</div>

<div
class="status"
id="apiStatus"
>
Sistema local
</div>

</header>

<div class="layout">

<aside
class="sidebar"
id="sidebar"
>
</aside>

<main
id="app"
>
</main>

</div>

<script>

const API = "/api";

let currentPage = "dashboard";

let bcraData = {};

const databaseDemo = {

cash:12845000,

invoices:[

{
id:"INV-001",
client:"BuildCo",
amount:1240000,
due:"2026-09-05",
status:"Pendiente"
},

{
id:"INV-002",
client:"Metro Supplies",
amount:875000,
due:"2026-08-18",
status:"Vencida"
},

{
id:"INV-003",
client:"Alpha Group",
amount:3120000,
due:"2026-09-12",
status:"Pagada"
}

],

stock:[

{
sku:"SKU-001",
name:"Taladro profesional",
qty:24,
minimum:10,
cost:185000
},

{
sku:"SKU-002",
name:"Amoladora 900W",
qty:7,
minimum:12,
cost:132000
},

{
sku:"SKU-003",
name:"Disco de corte",
qty:80,
minimum:30,
cost:4200
}

],

budget:[

{
category:"Marketing",
budget:500000,
spent:420000
},

{
category:"Compras",
budget:3500000,
spent:2980000
},

{
category:"Operaciones",
budget:1800000,
spent:1710000
}

],

investments:[

{
name:"Plazo fijo",
capital:2000000,
rate:32
},

{
name:"FCI",
capital:850000,
rate:28
}

]

};

function money(value){

return new Intl.NumberFormat(
"es-AR",
{
style:"currency",
currency:"ARS",
maximumFractionDigits:0
}
).format(value);

}

const menu=[

["dashboard","⬡","Dashboard"],

["cashflow","◎","Flujo de caja"],

["invoices","◈","Facturación"],

["stock","▦","Stock"],

["budget","▤","Presupuesto"],

["investments","◇","Inversiones"],

["markets","◌","Cotizaciones"],

["commodities","◆","Commodities"],

["alerts","△","Alertas"]

];

function drawMenu(){

document.getElementById("sidebar").innerHTML=

menu.map(item=>`

<button
class="nav ${currentPage===item[0]?"active":""}"
onclick="navigate('${item[0]}')"
>

${item[1]} ${item[2]}

</button>

`).join("");

}

function navigate(page){

currentPage=page;

drawMenu();

render();

}

function metric(label,value,description=""){

return `

<div class="card">

<div class="label">${label}</div>

<div class="value">${value}</div>

<div class="small">${description}</div>

</div>

`;

}

function dashboard(){

const receivable=

databaseDemo.invoices

.filter(x=>x.status!=="Pagada")

.reduce((sum,x)=>sum+x.amount,0);

const inventory=

databaseDemo.stock

.reduce((sum,x)=>sum+x.qty*x.cost,0);

const alerts=

databaseDemo.stock.filter(x=>x.qty<x.minimum).length

+

databaseDemo.invoices.filter(x=>x.status==="Vencida").length;

return `

<h1>Dashboard</h1>

<div class="subtitle">

Visión general de la empresa.

</div>

<div class="grid4">

${metric(
"Caja actual",
money(databaseDemo.cash),
"Saldo disponible"
)}

${metric(
"Cuentas por cobrar",
money(receivable),
"Facturas pendientes"
)}

${metric(
"Inventario",
money(inventory),
"Valor a costo"
)}

${metric(
"Alertas",
alerts,
"Eventos a revisar"
)}

</div>

<div class="grid3 section">

<div class="card">

<h2>🤖 IA financiera</h2>

<div class="alert redAlert">

Hay facturas vencidas.

</div>

<div class="alert">

Hay productos por debajo del stock mínimo.

</div>

<div class="alert greenAlert">

La cobranza pendiente puede mejorar la liquidez.

</div>

<div class="small">

La capa Python analizará los datos reales
para generar previsiones y recomendaciones.

</div>

</div>

<div class="card">

<h2>📊 Proyección</h2>

<div class="value green">

${money(databaseDemo.cash*1.08)}

</div>

<div class="small">

Escenario base a 30 días.

</div>

</div>

<div class="card">

<h2>🏗 Arquitectura</h2>

<p class="small">
Frontend → Spring Boot
</p>

<p class="small">
Spring Boot → PostgreSQL
</p>

<p class="small">
Spring Boot → Flask/Python
</p>

<p class="small">
Spring Boot → BCRA
</p>

</div>

</div>

`;

}

function cashflow(){

return `

<h1>Flujo de caja</h1>

<div class="subtitle">
Control y proyección de liquidez.
</div>

<div class="grid4">

${metric(
"Caja inicial",
money(databaseDemo.cash)
)}

${metric(
"Ingresos próximos",
money(4360000)
)}

${metric(
"Egresos estimados",
money(3210000)
)}

${metric(
"Caja proyectada",
money(databaseDemo.cash+1150000)
)}

</div>

<div class="card section">

<h2>Escenarios</h2>

<table>

<tr>
<th>Escenario</th>
<th>Proyección</th>
<th>Interpretación</th>
</tr>

<tr>
<td>Pesimista</td>
<td class="red">
${money(databaseDemo.cash*0.96)}
</td>
<td>Demoras de cobranza</td>
</tr>

<tr>
<td>Base</td>
<td class="green">
${money(databaseDemo.cash*1.08)}
</td>
<td>Comportamiento esperado</td>
</tr>

<tr>
<td>Optimista</td>
<td class="green">
${money(databaseDemo.cash*1.15)}
</td>
<td>Cobranza acelerada</td>
</tr>

</table>

</div>

`;

}

function invoices(){

return `

<h1>Facturación</h1>

<div class="subtitle">

Clientes, facturas y cuentas por cobrar.

</div>

<div class="toolbar">

<button
class="btn primary"
onclick="addInvoice()"
>
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

${databaseDemo.invoices.map(x=>`

<tr>

<td>${x.id}</td>

<td>${x.client}</td>

<td>${money(x.amount)}</td>

<td>${x.due}</td>

<td>

<span
class="badge
${x.status==="Pagada"
?"ok"
:x.status==="Vencida"
?"danger"
:"warning"}"
>

${x.status}

</span>

</td>

</tr>

`).join("")}

</table>

</div>

`;

}

function addInvoice(){

const client=prompt("Cliente:");

if(!client)return;

const amount=Number(
prompt("Importe en ARS:")
);

if(!amount)return;

databaseDemo.invoices.push({

id:
"INV-"+Date.now().toString().slice(-6),

client,

amount,

due:
new Date().toISOString().slice(0,10),

status:"Pendiente"

});

render();

}

function stock(){

return `

<h1>Stock</h1>

<div class="subtitle">

Inventario, costos y reposición.

</div>

<div class="toolbar">

<button
class="btn primary"
onclick="addProduct()"
>
+ Producto
</button>

</div>

<div class="card">

<table>

<tr>

<th>SKU</th>

<th>Producto</th>

<th>Cantidad</th>

<th>Mínimo</th>

<th>Costo</th>

<th>Estado</th>

</tr>

${databaseDemo.stock.map(x=>`

<tr>

<td>${x.sku}</td>

<td>${x.name}</td>

<td>${x.qty}</td>

<td>${x.minimum}</td>

<td>${money(x.cost)}</td>

<td>

<span
class="badge ${x.qty<x.minimum
?"warning"
:"ok"}"
>

${x.qty<x.minimum
?"Reponer"
:"OK"}

</span>

</td>

</tr>

`).join("")}

</table>

</div>

`;

}

function addProduct(){

const name=prompt("Producto:");

if(!name)return;

const quantity=Number(
prompt("Cantidad:")
);

const cost=Number(
prompt("Costo unitario:")
);

databaseDemo.stock.push({

sku:
"SKU-"+Date.now().toString().slice(-6),

name,

qty:quantity,

minimum:10,

cost

});

render();

}

function budget(){

return `

<h1>Presupuesto</h1>

<div class="subtitle">

Presupuesto contra ejecución.

</div>

<div class="grid3">

${databaseDemo.budget.map(x=>{

const percentage=
Math.round(
x.spent/x.budget*100
);

return `

<div class="card">

<div class="label">
${x.category}
</div>

<div class="value">
${percentage}%
</div>

<div class="small">

${money(x.spent)}
/
${money(x.budget)}

</div>

</div>

`;

}).join("")}

</div>

`;

}

function investments(){

return `

<h1>Inversiones</h1>

<div class="subtitle">

Seguimiento de inversiones empresariales.

</div>

<div class="toolbar">

<button
class="btn primary"
onclick="addInvestment()"
>
+ Inversión
</button>

</div>

<div class="card">

<table>

<tr>

<th>Instrumento</th>
<th>Capital</th>
<th>Tasa</th>
<th>Estimación anual</th>

</tr>

${databaseDemo.investments.map(x=>`

<tr>

<td>${x.name}</td>

<td>${money(x.capital)}</td>

<td>${x.rate}%</td>

<td class="green">

${money(
x.capital*x.rate/100
)}

</td>

</tr>

`).join("")}

</table>

</div>

`;

}

function addInvestment(){

const name=prompt("Instrumento:");

if(!name)return;

const capital=
Number(prompt("Capital:"));

const rate=
Number(prompt("Tasa %:"));

databaseDemo.investments.push({

name,

capital,

rate

});

render();

}

function markets(){

const currencies=[
"USD",
"EUR",
"BRL",
"CLP",
"COP",
"GBP",
"CNY",
"UYU"
];

return `

<h1>Cotizaciones</h1>

<div class="subtitle">

Cotizaciones obtenidas mediante el backend.

</div>

<div class="toolbar">

<button
class="btn primary"
onclick="loadBCRA()"
>
↻ Actualizar BCRA
</button>

</div>

<div class="market">

${currencies.map(currency=>`

<div class="card">

<div class="label">
${currency}
</div>

<div class="value">

${bcraData[currency] ?? "—"}

</div>

<div class="small">

ARS por unidad

</div>

</div>

`).join("")}

</div>

`;

}

async function loadBCRA(){

try{

const response=
await fetch(
API+"/mercado/bcra"
);

if(!response.ok)
throw new Error();

const data=
await response.json();

const details=
data?.results?.detalle || [];

details.forEach(item=>{

if(
item.codigoMoneda &&
item.tipoCotizacion
){

bcraData[
item.codigoMoneda
]=Number(
item.tipoCotizacion
).toLocaleString(
"es-AR",
{
minimumFractionDigits:2,
maximumFractionDigits:4
}
);

}

});

document.getElementById(
"apiStatus"
).textContent=
"BCRA conectado";

render();

}catch(error){

document.getElementById(
"apiStatus"
).textContent=
"BCRA no disponible";

}

}

function commodities(){

const items=[
"Oro",
"Plata",
"Petróleo",
"Soja",
"Trigo",
"Maíz"
];

return `

<h1>Commodities</h1>

<div class="subtitle">

Los proveedores de commodities se conectan
desde el backend.

</div>

<div class="market">

${items.map(x=>`

<div class="card">

<div class="label">
${x}
</div>

<div class="value">
—
</div>

<div class="small">
Proveedor externo
</div>

</div>

`).join("")}

</div>

`;

}

function alerts(){

const alerts=[];

databaseDemo.invoices
.filter(x=>x.status==="Vencida")
.forEach(x=>{

alerts.push(`

<div class="alert redAlert">

<b>Factura vencida</b>

<br>

${x.client}

·

${money(x.amount)}

</div>

`);

});

databaseDemo.stock
.filter(x=>x.qty<x.minimum)
.forEach(x=>{

alerts.push(`

<div class="alert">

<b>Stock bajo</b>

<br>

${x.name}

·

${x.qty} unidades

</div>

`);

});

return `

<h1>Alertas</h1>

<div class="subtitle">

Eventos importantes para la empresa.

</div>

${alerts.join("") ||

`

<div class="alert greenAlert">

No hay alertas críticas.

</div>

`}

`;

}

function render(){

const pages={

dashboard,

cashflow,

invoices,

stock,

budget,

investments,

markets,

commodities,

alerts

};

document.getElementById(
"app"
).innerHTML=
pages[currentPage]();

}

drawMenu();

render();

loadBCRA();

</script>

</body>

</html>
EOF

# ============================================================
# SPRING BOOT pom.xml
# ============================================================

cat > "$PROJECT/backend-spring/pom.xml" <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>

<project
xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
http://maven.apache.org/POM/4.0.0
https://maven.apache.org/xsd/maven-4.0.0.xsd">

<modelVersion>4.0.0</modelVersion>

<parent>

<groupId>
org.springframework.boot
</groupId>

<artifactId>
spring-boot-starter-parent
</artifactId>

<version>
3.3.5
</version>

<relativePath/>

</parent>

<groupId>
com.contabilidad
</groupId>

<artifactId>
kontalo-api
</artifactId>

<version>
0.0.1-SNAPSHOT
</version>

<properties>

<java.version>
17
</java.version>

</properties>

<dependencies>

<dependency>

<groupId>
org.springframework.boot
</groupId>

<artifactId>
spring-boot-starter-web
</artifactId>

</dependency>

<dependency>

<groupId>
org.springframework.boot
</groupId>

<artifactId>
spring-boot-starter-data-jpa
</artifactId>

</dependency>

<dependency>

<groupId>
org.postgresql
</groupId>

<artifactId>
postgresql
</artifactId>

<scope>
runtime
</scope>

</dependency>

<dependency>

<groupId>
org.springdoc
</groupId>

<artifactId>
springdoc-openapi-starter-webmvc-ui
</artifactId>

<version>
2.6.0
</version>

</dependency>

</dependencies>

<build>

<plugins>

<plugin>

<groupId>
org.springframework.boot
</groupId>

<artifactId>
spring-boot-maven-plugin
</artifactId>

</plugin>

</plugins>

</build>

</project>
EOF

# ============================================================
# Spring Boot application
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/KontaloApplication.java" <<'EOF'
package com.contabilidad.api;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KontaloApplication {

    public static void main(String[] args) {

        SpringApplication.run(
            KontaloApplication.class,
            args
        );

    }

}
EOF

# ============================================================
# application.properties
# ============================================================

cat > "$PROJECT/backend-spring/src/main/resources/application.properties" <<'EOF'
server.port=8080

spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/contabilidad}

spring.datasource.username=${DB_USER:postgres}

spring.datasource.password=${DB_PASSWORD:postgres}

spring.jpa.hibernate.ddl-auto=update

spring.jpa.open-in-view=false

flask.base-url=${FLASK_BASE_URL:http://localhost:5000}

springdoc.swagger-ui.path=/swagger-ui.html
EOF

# ============================================================
# RestTemplate
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/config/RestTemplateConfig.java" <<'EOF'
package com.contabilidad.api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {

        return new RestTemplate();

    }

}
EOF

# ============================================================
# FACTURA
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity/Factura.java" <<'EOF'
package com.contabilidad.api.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name="facturas")
public class Factura {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String cliente;

    private BigDecimal importe;

    private LocalDate vencimiento;

    private String estado;

    public Long getId() {
        return id;
    }

    public String getCliente() {
        return cliente;
    }

    public void setCliente(String cliente) {
        this.cliente = cliente;
    }

    public BigDecimal getImporte() {
        return importe;
    }

    public void setImporte(BigDecimal importe) {
        this.importe = importe;
    }

    public LocalDate getVencimiento() {
        return vencimiento;
    }

    public void setVencimiento(LocalDate vencimiento) {
        this.vencimiento = vencimiento;
    }

    public String getEstado() {
        return estado;
    }

    public void setEstado(String estado) {
        this.estado = estado;
    }

}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository/FacturaRepository.java" <<'EOF'
package com.contabilidad.api.repository;

import com.contabilidad.api.entity.Factura;

import org.springframework.data.jpa.repository.JpaRepository;

public interface FacturaRepository
extends JpaRepository<Factura, Long> {
}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/FacturaController.java" <<'EOF'
package com.contabilidad.api.controller;

import com.contabilidad.api.entity.Factura;
import com.contabilidad.api.repository.FacturaRepository;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/facturas")
public class FacturaController {

    private final FacturaRepository repository;

    public FacturaController(
        FacturaRepository repository
    ) {

        this.repository = repository;

    }

    @GetMapping
    public List<Factura> all() {

        return repository.findAll();

    }

    @GetMapping("/{id}")
    public Factura one(
        @PathVariable Long id
    ) {

        return repository
            .findById(id)
            .orElseThrow();

    }

    @PostMapping
    public Factura create(
        @RequestBody Factura factura
    ) {

        return repository.save(factura);

    }

    @PutMapping("/{id}")
    public Factura update(
        @PathVariable Long id,
        @RequestBody Factura factura
    ) {

        factura.setIdForUpdate(id);

        return repository.save(factura);

    }

    @DeleteMapping("/{id}")
    public void delete(
        @PathVariable Long id
    ) {

        repository.deleteById(id);

    }

}
EOF

# Fix helper method through replacing entity/controller with compatible update implementation.
python3 - <<'PY'
from pathlib import Path
p=Path("Kontalo-V2/backend-spring/src/main/java/com/contabilidad/api/entity/Factura.java")
s=p.read_text()
s=s.replace("    public String getCliente()", """    public void setIdForUpdate(Long id) {
        this.id = id;
    }

    public String getCliente()""")
p.write_text(s)
PY

# ============================================================
# PRODUCTO / STOCK
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity/Producto.java" <<'EOF'
package com.contabilidad.api.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;

@Entity
@Table(name="productos")
public class Producto {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String sku;

    private String nombre;

    private Integer cantidad;

    private Integer minimo;

    private BigDecimal costo;

    public Long getId() {
        return id;
    }

    public void setIdForUpdate(Long id) {
        this.id = id;
    }

    public String getSku() {
        return sku;
    }

    public void setSku(String sku) {
        this.sku = sku;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public Integer getCantidad() {
        return cantidad;
    }

    public void setCantidad(Integer cantidad) {
        this.cantidad = cantidad;
    }

    public Integer getMinimo() {
        return minimo;
    }

    public void setMinimo(Integer minimo) {
        this.minimo = minimo;
    }

    public BigDecimal getCosto() {
        return costo;
    }

    public void setCosto(BigDecimal costo) {
        this.costo = costo;
    }

}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository/ProductoRepository.java" <<'EOF'
package com.contabilidad.api.repository;

import com.contabilidad.api.entity.Producto;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductoRepository
extends JpaRepository<Producto, Long> {
}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/ProductoController.java" <<'EOF'
package com.contabilidad.api.controller;

import com.contabilidad.api.entity.Producto;
import com.contabilidad.api.repository.ProductoRepository;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/productos")
public class ProductoController {

    private final ProductoRepository repository;

    public ProductoController(
        ProductoRepository repository
    ) {
        this.repository = repository;
    }

    @GetMapping
    public List<Producto> all() {
        return repository.findAll();
    }

    @GetMapping("/{id}")
    public Producto one(
        @PathVariable Long id
    ) {
        return repository.findById(id).orElseThrow();
    }

    @PostMapping
    public Producto create(
        @RequestBody Producto producto
    ) {
        return repository.save(producto);
    }

    @PutMapping("/{id}")
    public Producto update(
        @PathVariable Long id,
        @RequestBody Producto producto
    ) {
        producto.setIdForUpdate(id);
        return repository.save(producto);
    }

    @DeleteMapping("/{id}")
    public void delete(
        @PathVariable Long id
    ) {
        repository.deleteById(id);
    }

}
EOF

# ============================================================
# MOVIMIENTOS DE CAJA
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity/MovimientoCaja.java" <<'EOF'
package com.contabilidad.api.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name="movimientos_caja")
public class MovimientoCaja {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private LocalDate fecha;

    private String tipo;

    private String categoria;

    private BigDecimal importe;

    private String descripcion;

    public Long getId() {
        return id;
    }

    public void setIdForUpdate(Long id) {
        this.id = id;
    }

    public LocalDate getFecha() {
        return fecha;
    }

    public void setFecha(LocalDate fecha) {
        this.fecha = fecha;
    }

    public String getTipo() {
        return tipo;
    }

    public void setTipo(String tipo) {
        this.tipo = tipo;
    }

    public String getCategoria() {
        return categoria;
    }

    public void setCategoria(String categoria) {
        this.categoria = categoria;
    }

    public BigDecimal getImporte() {
        return importe;
    }

    public void setImporte(BigDecimal importe) {
        this.importe = importe;
    }

    public String getDescripcion() {
        return descripcion;
    }

    public void setDescripcion(String descripcion) {
        this.descripcion = descripcion;
    }

}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository/MovimientoCajaRepository.java" <<'EOF'
package com.contabilidad.api.repository;

import com.contabilidad.api.entity.MovimientoCaja;

import org.springframework.data.jpa.repository.JpaRepository;

public interface MovimientoCajaRepository
extends JpaRepository<MovimientoCaja, Long> {
}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/MovimientoCajaController.java" <<'EOF'
package com.contabilidad.api.controller;

import com.contabilidad.api.entity.MovimientoCaja;
import com.contabilidad.api.repository.MovimientoCajaRepository;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/caja")
public class MovimientoCajaController {

    private final MovimientoCajaRepository repository;

    public MovimientoCajaController(
        MovimientoCajaRepository repository
    ) {
        this.repository = repository;
    }

    @GetMapping
    public List<MovimientoCaja> all() {
        return repository.findAll();
    }

    @GetMapping("/{id}")
    public MovimientoCaja one(
        @PathVariable Long id
    ) {
        return repository.findById(id).orElseThrow();
    }

    @PostMapping
    public MovimientoCaja create(
        @RequestBody MovimientoCaja movimiento
    ) {
        return repository.save(movimiento);
    }

    @PutMapping("/{id}")
    public MovimientoCaja update(
        @PathVariable Long id,
        @RequestBody MovimientoCaja movimiento
    ) {
        movimiento.setIdForUpdate(id);
        return repository.save(movimiento);
    }

    @DeleteMapping("/{id}")
    public void delete(
        @PathVariable Long id
    ) {
        repository.deleteById(id);
    }

}
EOF

# ============================================================
# PRESUPUESTO
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity/Presupuesto.java" <<'EOF'
package com.contabilidad.api.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;

@Entity
@Table(name="presupuestos")
public class Presupuesto {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String categoria;

    private BigDecimal presupuesto;

    private BigDecimal ejecutado;

    public Long getId() {
        return id;
    }

    public void setIdForUpdate(Long id) {
        this.id = id;
    }

    public String getCategoria() {
        return categoria;
    }

    public void setCategoria(String categoria) {
        this.categoria = categoria;
    }

    public BigDecimal getPresupuesto() {
        return presupuesto;
    }

    public void setPresupuesto(BigDecimal presupuesto) {
        this.presupuesto = presupuesto;
    }

    public BigDecimal getEjecutado() {
        return ejecutado;
    }

    public void setEjecutado(BigDecimal ejecutado) {
        this.ejecutado = ejecutado;
    }

}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository/PresupuestoRepository.java" <<'EOF'
package com.contabilidad.api.repository;

import com.contabilidad.api.entity.Presupuesto;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PresupuestoRepository
extends JpaRepository<Presupuesto, Long> {
}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/PresupuestoController.java" <<'EOF'
package com.contabilidad.api.controller;

import com.contabilidad.api.entity.Presupuesto;
import com.contabilidad.api.repository.PresupuestoRepository;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/presupuestos")
public class PresupuestoController {

    private final PresupuestoRepository repository;

    public PresupuestoController(
        PresupuestoRepository repository
    ) {
        this.repository = repository;
    }

    @GetMapping
    public List<Presupuesto> all() {
        return repository.findAll();
    }

    @GetMapping("/{id}")
    public Presupuesto one(
        @PathVariable Long id
    ) {
        return repository.findById(id).orElseThrow();
    }

    @PostMapping
    public Presupuesto create(
        @RequestBody Presupuesto presupuesto
    ) {
        return repository.save(presupuesto);
    }

    @PutMapping("/{id}")
    public Presupuesto update(
        @PathVariable Long id,
        @RequestBody Presupuesto presupuesto
    ) {
        presupuesto.setIdForUpdate(id);
        return repository.save(presupuesto);
    }

    @DeleteMapping("/{id}")
    public void delete(
        @PathVariable Long id
    ) {
        repository.deleteById(id);
    }

}
EOF

# ============================================================
# INVERSIONES
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/entity/Inversion.java" <<'EOF'
package com.contabilidad.api.entity;

import jakarta.persistence.*;

import java.math.BigDecimal;

@Entity
@Table(name="inversiones")
public class Inversion {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String instrumento;

    private BigDecimal capital;

    private BigDecimal tasa;

    public Long getId() {
        return id;
    }

    public void setIdForUpdate(Long id) {
        this.id = id;
    }

    public String getInstrumento() {
        return instrumento;
    }

    public void setInstrumento(String instrumento) {
        this.instrumento = instrumento;
    }

    public BigDecimal getCapital() {
        return capital;
    }

    public void setCapital(BigDecimal capital) {
        this.capital = capital;
    }

    public BigDecimal getTasa() {
        return tasa;
    }

    public void setTasa(BigDecimal tasa) {
        this.tasa = tasa;
    }

}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/repository/InversionRepository.java" <<'EOF'
package com.contabilidad.api.repository;

import com.contabilidad.api.entity.Inversion;

import org.springframework.data.jpa.repository.JpaRepository;

public interface InversionRepository
extends JpaRepository<Inversion, Long> {
}
EOF

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/InversionController.java" <<'EOF'
package com.contabilidad.api.controller;

import com.contabilidad.api.entity.Inversion;
import com.contabilidad.api.repository.InversionRepository;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/inversiones")
public class InversionController {

    private final InversionRepository repository;

    public InversionController(
        InversionRepository repository
    ) {
        this.repository = repository;
    }

    @GetMapping
    public List<Inversion> all() {
        return repository.findAll();
    }

    @GetMapping("/{id}")
    public Inversion one(
        @PathVariable Long id
    ) {
        return repository.findById(id).orElseThrow();
    }

    @PostMapping
    public Inversion create(
        @RequestBody Inversion inversion
    ) {
        return repository.save(inversion);
    }

    @PutMapping("/{id}")
    public Inversion update(
        @PathVariable Long id,
        @RequestBody Inversion inversion
    ) {
        inversion.setIdForUpdate(id);
        return repository.save(inversion);
    }

    @DeleteMapping("/{id}")
    public void delete(
        @PathVariable Long id
    ) {
        repository.deleteById(id);
    }

}
EOF

# ============================================================
# MERCADOS / BCRA / FLASK
# ============================================================

cat > "$PROJECT/backend-spring/src/main/java/com/contabilidad/api/controller/MercadoController.java" <<'EOF'
package com.contabilidad.api.controller;

import org.springframework.beans.factory.annotation.Value;

import org.springframework.http.ResponseEntity;

import org.springframework.web.bind.annotation.*;

import org.springframework.web.client.RestTemplate;

import java.util.Map;

@RestController
@RequestMapping("/api")
public class MercadoController {

    @Value("${flask.base-url:http://localhost:5000}")
    private String flaskBaseUrl;

    private final RestTemplate restTemplate;

    public MercadoController(
        RestTemplate restTemplate
    ) {
        this.restTemplate = restTemplate;
    }

    @GetMapping("/mercado/bcra")
    public ResponseEntity<String> bcra() {

        String url =
            "https://api.bcra.gob.ar/" +
            "estadisticascambiarias/v1.0/Cotizaciones";

        String response =
            restTemplate.getForObject(
                url,
                String.class
            );

        return ResponseEntity.ok(response);
    }

    @GetMapping("/divisas")
    public ResponseEntity<Map> divisas(

        @RequestParam(
            defaultValue="USD"
        )
        String base,

        @RequestParam(
            defaultValue="EUR"
        )
        String destino

    ) {

        String url =
            flaskBaseUrl +
            "/divisas?base=" +
            base +
            "&destino=" +
            destino;

        Map response =
            restTemplate.getForObject(
                url,
                Map.class
            );

        return ResponseEntity.ok(response);
    }

    @PostMapping("/ai/consultar")
    public ResponseEntity<Map> ai(

        @RequestBody
        Map<String,Object> body

    ) {

        Map response =
            restTemplate.postForObject(
                flaskBaseUrl +
                "/ai/consultar",
                body,
                Map.class
            );

        return ResponseEntity.ok(response);
    }

    @PostMapping("/forecast")
    public ResponseEntity<Map> forecast(

        @RequestBody
        Map<String,Object> body

    ) {

        Map response =
            restTemplate.postForObject(
                flaskBaseUrl +
                "/forecast",
                body,
                Map.class
            );

        return ResponseEntity.ok(response);
    }

    @GetMapping("/materias-primas")
    public ResponseEntity<Map> commodities(

        @RequestParam
        String nombre

    ) {

        Map response =
            restTemplate.getForObject(
                flaskBaseUrl +
                "/materias-primas?nombre=" +
                nombre,
                Map.class
            );

        return ResponseEntity.ok(response);
    }

}
EOF

# ============================================================
# SPRING DOCKERFILE
# ============================================================

cat > "$PROJECT/backend-spring/Dockerfile" <<'EOF'
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .

RUN mvn -q dependency:go-offline

COPY src ./src

RUN mvn -q -DskipTests package

FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=build \
/app/target/kontalo-api-0.0.1-SNAPSHOT.jar \
app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]
EOF

# ============================================================
# FLASK / PYTHON
# ============================================================

cat > "$PROJECT/backend-flask/requirements.txt" <<'EOF'
Flask==3.0.3
requests==2.32.3
numpy==2.1.2
pandas==2.2.3
scikit-learn==1.5.2
EOF

cat > "$PROJECT/backend-flask/app.py" <<'EOF'
import os

from flask import Flask, jsonify, request

app = Flask(__name__)


@app.get("/health")
def health():

    return jsonify({
        "status": "ok",
        "service": "kontalo-python"
    })


@app.get("/divisas")
def divisas():

    base = request.args.get(
        "base",
        "USD"
    )

    destino = request.args.get(
        "destino",
        "EUR"
    )

    return jsonify({

        "base": base,

        "destino": destino,

        "status":
        "provider-ready"

    })


@app.get("/materias-primas")
def materias_primas():

    nombre = request.args.get(
        "nombre",
        "Oro"
    )

    return jsonify({

        "commodity": nombre,

        "status":
        "provider-ready",

        "message":
        "Conectar proveedor de commodities desde backend."

    })


@app.post("/ai/consultar")
def consultar_ia():

    body =
        request.get_json(
            silent=True
        ) or {}

    api_key =
        os.getenv(
            "AI_API_KEY"
        )

    if not api_key:

        return jsonify({

            "status":
            "provider-not-configured",

            "message":
            "Configurar AI_API_KEY como secreto del backend.",

            "input":
            body

        })


    # Aquí se conecta el proveedor de IA elegido.
    # La clave nunca se envía desde el frontend.

    return jsonify({

        "status":
        "provider-configured",

        "message":
        "Punto de integración de IA listo.",

        "input":
        body

    })


@app.post("/forecast")
def forecast():

    body =
        request.get_json(
            silent=True
        ) or {}

    values =
        body.get(
            "values",
            []
        )

    if not values:

        return jsonify({

            "status":
            "no-data",

            "forecast":
            []

        })


    # Forecasting inicial sencillo.
    # Después puede reemplazarse por un modelo más avanzado.

    try:

        numeric = [
            float(x)
            for x in values
        ]

        average =
            sum(numeric) / len(numeric)

        forecast_values = [

            round(
                average,
                2
            )

            for _ in range(7)

        ]

        return jsonify({

            "status":
            "ok",

            "method":
            "baseline-average",

            "forecast":
            forecast_values

        })

    except Exception as error:

        return jsonify({

            "status":
            "error",

            "message":
            str(error)

        }), 400


if __name__ == "__main__":

    app.run(

        host="0.0.0.0",

        port=int(
            os.getenv(
                "PORT",
                "5000"
            )
        )

    )
EOF

# ============================================================
# FLASK DOCKERFILE
# ============================================================

cat > "$PROJECT/backend-flask/Dockerfile" <<'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python","app.py"]
EOF

# ============================================================
# FIN
# ============================================================

echo ""
echo "============================================================"
echo " KONTALO V2 CREADO"
echo "============================================================"
echo ""
echo "Carpeta:"
echo "  $PROJECT"
echo ""
echo "Para levantarlo:"
echo ""
echo "  cd $PROJECT"
echo "  cp .env.example .env"
echo "  docker compose up --build"
echo ""
echo "Frontend:"
echo "  http://localhost:3000"
echo ""
echo "Spring Boot:"
echo "  http://localhost:8080"
echo ""
echo "Swagger:"
echo "  http://localhost:8080/swagger-ui/index.html"
echo ""
echo "Flask:"
echo "  http://localhost:5000/health"
echo ""
echo "PostgreSQL:"
echo "  localhost:5432"
echo ""
echo "============================================================"
