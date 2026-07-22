import { useState, useEffect, useCallback } from "react";
import { AreaChart, Area, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, LineChart, Line } from "recharts";

// ── DESIGN TOKENS ─────────────────────────────────────────────────────────────
const G = {
  bg:"#0A0F1A", surface:"#111827", border:"#1E2D3D",
  accent:"#00E5A0", accentDim:"#0D2A1F", accentHover:"#00FFB3",
  text:"#E8EDF5", muted:"#8899AA", faint:"#4A5A6A",
  danger:"#FF6B6B", warn:"#F59E0B",
  fontDisplay:"'Space Grotesk', sans-serif",
  fontBody:"'Inter', system-ui, sans-serif",
};

// ── YOUR REAL API ─────────────────────────────────────────────────────────────
const API_BASE = "https://contabilidad-de-datos.onrender.com";
async function fetchForecast(values) {
  const res = await fetch(`${API_BASE}/forecast`, {
    method:"POST", headers:{"Content-Type":"application/json"},
    body: JSON.stringify({ values }),
  });
  if (!res.ok) throw new Error(`API ${res.status}`);
  return res.json();
}

// ── FX + COMMODITIES (Frankfurter — no key needed) ────────────────────────────
// Frankfurter has EUR as base. We'll get USD rates relative to EUR,
// then invert to get "how many X per 1 USD"
async function fetchFXRates() {
  const res = await fetch("https://api.frankfurter.dev/v1/latest?from=USD&to=EUR,ARS,BRL,MXN,COP,CLP,GBP,CNY");
  if (!res.ok) throw new Error("FX API error");
  const data = await res.json();
  return data.rates; // { EUR: 0.88, ARS: 1440, BRL: 5.17, ... }
}

// Commodities: use open proxy via fawazahmed0 CDN (no key, 200+ pairs incl metals)
// Gold (XAU) and Silver (XAG) vs USD
async function fetchCommodities() {
  const [goldRes, silverRes, oilRes] = await Promise.allSettled([
    fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xau.json"),
    fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/xag.json"),
    fetch("https://cdn.jsdelivr.net/npm/@fawazahmed0/currency-api@latest/v1/currencies/usd.json"),
  ]);
  const gold   = goldRes.status   === "fulfilled" ? await goldRes.value.json()   : null;
  const silver = silverRes.status === "fulfilled" ? await silverRes.value.json() : null;
  const oil    = oilRes.status    === "fulfilled" ? await oilRes.value.json()    : null;
  return {
    goldUSD:   gold   ? +(1 / gold.xau.usd).toFixed(2)   : null,
    silverUSD: silver ? +(1 / silver.xag.usd).toFixed(2) : null,
    // oil not in this API – we'll use a fixed recent ref
    oilUSD: oil ? null : null,
  };
}

// ── STYLES ────────────────────────────────────────────────────────────────────
const css = `
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap');
  *, *::before, *::after { box-sizing:border-box; margin:0; padding:0; }
  body { background:${G.bg}; color:${G.text}; font-family:${G.fontBody}; }
  ::selection { background:${G.accent}; color:${G.bg}; }
  input, button, select { font-family:${G.fontBody}; }

  @keyframes fadeIn { from{opacity:0;transform:translateY(8px)} to{opacity:1;transform:translateY(0)} }
  @keyframes spin   { to{transform:rotate(360deg)} }
  @keyframes pulse  { 0%,100%{opacity:1} 50%{opacity:.4} }
  @keyframes ticker { 0%{transform:translateX(0)} 100%{transform:translateX(-50%)} }

  .fade-in { animation:fadeIn .3s ease forwards; }

  .btn { border:none; border-radius:7px; cursor:pointer; font-weight:600; font-size:.9rem; transition:all .15s; display:inline-flex; align-items:center; gap:.5rem; }
  .btn-primary { background:${G.accent}; color:${G.bg}; padding:.75rem 1.5rem; }
  .btn-primary:hover:not(:disabled) { background:${G.accentHover}; transform:translateY(-1px); }
  .btn-primary:disabled { opacity:.5; cursor:not-allowed; }
  .btn-ghost { background:transparent; color:${G.text}; border:1px solid ${G.border}; padding:.6rem 1.2rem; }
  .btn-ghost:hover { border-color:${G.accent}; color:${G.accent}; }
  .btn-danger { background:transparent; color:${G.danger}; border:1px solid ${G.danger}22; padding:.5rem 1rem; font-size:.8rem; }
  .btn-danger:hover { background:${G.danger}11; }

  .input { background:#0A0F1A; border:1px solid ${G.border}; border-radius:7px; color:${G.text}; font-size:.9rem; padding:.75rem 1rem; width:100%; transition:border-color .2s; outline:none; }
  .input:focus { border-color:${G.accent}; }
  .input::placeholder { color:${G.faint}; }

  .card { background:${G.surface}; border:1px solid ${G.border}; border-radius:12px; }
  .nav-item { color:${G.muted}; font-size:.875rem; padding:.6rem .9rem; border-radius:7px; cursor:pointer; display:flex; align-items:center; gap:.6rem; transition:all .15s; }
  .nav-item:hover { background:${G.accentDim}; color:${G.accent}; }
  .nav-item.active { background:${G.accentDim}; color:${G.accent}; font-weight:600; }

  .badge { font-size:.7rem; font-weight:700; padding:.2rem .55rem; border-radius:999px; }
  .badge-green  { background:${G.accentDim}; color:${G.accent}; }
  .badge-red    { background:#FF6B6B18; color:${G.danger}; }
  .badge-yellow { background:#F59E0B18; color:${G.warn}; }

  .spinner { width:18px; height:18px; border:2px solid ${G.border}; border-top-color:${G.accent}; border-radius:50%; animation:spin .7s linear infinite; flex-shrink:0; }

  .tab { padding:.5rem 1rem; border-radius:6px; font-size:.85rem; cursor:pointer; color:${G.muted}; background:transparent; border:none; font-weight:500; transition:all .15s; }
  .tab.active { background:${G.accentDim}; color:${G.accent}; font-weight:600; }

  .alert-item  { border-left:3px solid; padding:.9rem 1rem; border-radius:0 8px 8px 0; margin-bottom:.6rem; }
  .alert-danger { border-color:${G.danger}; background:#FF6B6B08; }
  .alert-warn   { border-color:${G.warn};   background:#F59E0B08; }
  .alert-ok     { border-color:${G.accent}; background:${G.accentDim}22; }

  /* Live ticker */
  .ticker-wrap { overflow:hidden; background:${G.surface}; border-bottom:1px solid ${G.border}; padding:.5rem 0; }
  .ticker-track { display:flex; gap:0; white-space:nowrap; animation:ticker 40s linear infinite; }
  .ticker-track:hover { animation-play-state:paused; }
  .ticker-item { display:inline-flex; align-items:center; gap:.5rem; padding:0 2rem; font-size:.8rem; border-right:1px solid ${G.border}; }

  /* FX cards */
  .fx-card { background:${G.surface}; border:1px solid ${G.border}; border-radius:10px; padding:1rem 1.2rem; transition:border-color .2s; }
  .fx-card:hover { border-color:${G.accent}44; }

  ::-webkit-scrollbar { width:4px; }
  ::-webkit-scrollbar-thumb { background:${G.border}; border-radius:4px; }

  @media(max-width:768px){
    .sidebar { display:none !important; }
    .kpi-grid { grid-template-columns:1fr 1fr !important; }
    .bottom-grid { grid-template-columns:1fr !important; }
    .fx-grid { grid-template-columns:1fr 1fr !important; }
  }
`;

// ── STATIC MOCK DATA ──────────────────────────────────────────────────────────
const COMPANIES = [
  { id:1, name:"El Clavo Hardware",  sector:"Retail",        plan:"Growth",  seed:[210,195,230,215,240,225], currency:"ARS" },
  { id:2, name:"Soto Import SA",     sector:"Import/Export", plan:"Scale",   seed:[450,480,510,490,530,505], currency:"USD" },
  { id:3, name:"Dubois Consulting",  sector:"Services",      plan:"Starter", seed:[80,95,88,102,91,110],     currency:"BRL" },
];

const MONTHS = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];

const INVOICES = [
  { id:"INV-001", client:"BuildCo Ltd",    amount:12400, date:"Jun 10", status:"paid"    },
  { id:"INV-002", client:"Metro Supplies", amount:8750,  date:"Jun 08", status:"pending" },
  { id:"INV-003", client:"Alpha Group",    amount:31200, date:"Jun 05", status:"paid"    },
  { id:"INV-004", client:"City Works",     amount:5600,  date:"Jun 01", status:"overdue" },
  { id:"INV-005", client:"TechParts SA",   amount:19800, date:"May 28", status:"paid"    },
  { id:"INV-006", client:"Harbor Imports", amount:44100, date:"May 25", status:"pending" },
];

const ALERTS = [
  { type:"danger", title:"Cash flow risk detected",     desc:"Projected cash drops below threshold in May.",      time:"2h ago" },
  { type:"warn",   title:"Invoice overdue — City Works", desc:"INV-004 for $5,600 is 14 days overdue.",          time:"1d ago" },
  { type:"ok",     title:"Monthly target reached",      desc:"June inflow exceeded forecast by 12.4%.",           time:"2d ago" },
  { type:"ok",     title:"FX rate opportunity",         desc:"Favorable USD spread — optimal for import orders.", time:"3d ago" },
];

const fmt = n => n>=1e6 ? `$${(n/1e6).toFixed(1)}M` : n>=1000 ? `$${(n/1000).toFixed(0)}K` : `$${n}`;

// ── SHARED UI ─────────────────────────────────────────────────────────────────
function Logo({ size=1.3 }) {
  return (
    <span style={{ fontFamily:G.fontDisplay, fontWeight:700, fontSize:`${size}rem`, letterSpacing:"-0.02em" }}>
      <span style={{ color:G.accent }}>kon</span>talo
    </span>
  );
}
function Spin() { return <div className="spinner"/>; }

function KPI({ label, value, delta, up, loading }) {
  return (
    <div className="card" style={{ padding:"1.25rem 1.4rem" }}>
      <div style={{ fontSize:".7rem", color:G.faint, textTransform:"uppercase", letterSpacing:".08em", marginBottom:".5rem" }}>{label}</div>
      <div style={{ fontFamily:G.fontDisplay, fontSize:"1.75rem", fontWeight:700, lineHeight:1, marginBottom:".35rem" }}>
        {loading ? <Spin/> : value}
      </div>
      {delta && <div style={{ fontSize:".78rem", color:up?G.accent:G.danger }}>{up?"▲":"▼"} {delta}</div>}
    </div>
  );
}

const ChartTip = ({ active, payload, label }) => {
  if (!active||!payload?.length) return null;
  return (
    <div style={{ background:G.surface, border:`1px solid ${G.border}`, borderRadius:8, padding:".7rem 1rem", fontSize:".8rem" }}>
      <div style={{ color:G.muted, marginBottom:".3rem" }}>{label}</div>
      {payload.map(p=>(
        <div key={p.name} style={{ color:p.color||G.accent, fontWeight:600 }}>
          {p.name==="projected"?"Projected":"Actual"}: {fmt(p.value)}
        </div>
      ))}
    </div>
  );
};

// ── LIVE TICKER ───────────────────────────────────────────────────────────────
function LiveTicker({ fx, commodities, loading }) {
  const items = [
    fx?.EUR   && { label:"EUR/USD", val:(1/fx.EUR).toFixed(4), up:true },
    fx?.ARS   && { label:"USD/ARS", val:fx.ARS.toFixed(2),     up:false },
    fx?.BRL   && { label:"USD/BRL", val:fx.BRL.toFixed(4),     up:false },
    fx?.MXN   && { label:"USD/MXN", val:fx.MXN.toFixed(4),     up:false },
    fx?.GBP   && { label:"GBP/USD", val:(1/fx.GBP).toFixed(4), up:true  },
    fx?.CNY   && { label:"USD/CNY", val:fx.CNY.toFixed(4),     up:false },
    commodities?.goldUSD   && { label:"GOLD oz", val:`$${commodities.goldUSD.toLocaleString()}`, up:true },
    commodities?.silverUSD && { label:"SILVER oz", val:`$${commodities.silverUSD.toFixed(2)}`, up:false },
    { label:"CRUDE OIL", val:"$73.40", up:true },   // fallback static
    { label:"SOYBEANS bu", val:"$10.82", up:false },
    { label:"WHEAT bu", val:"$5.64", up:true },
    { label:"CORN bu", val:"$4.38", up:false },
  ].filter(Boolean);

  if (loading) return (
    <div className="ticker-wrap" style={{ display:"flex", alignItems:"center", gap:".75rem", padding:".5rem 1.5rem", fontSize:".8rem", color:G.faint }}>
      <Spin/> Loading live market rates…
    </div>
  );

  // duplicate for seamless loop
  const all = [...items, ...items];

  return (
    <div className="ticker-wrap">
      <div className="ticker-track">
        {all.map((item,i) => (
          <span key={i} className="ticker-item">
            <span style={{ color:G.faint }}>{item.label}</span>
            <span style={{ color:item.up?G.accent:G.danger, fontWeight:600 }}>{item.val}</span>
            <span style={{ color:item.up?G.accent:G.danger, fontSize:".65rem" }}>{item.up?"▲":"▼"}</span>
          </span>
        ))}
      </div>
    </div>
  );
}

// ── FX & MARKETS PAGE ─────────────────────────────────────────────────────────
function MarketsPage({ fx, commodities, fxLoading, fxError, fxDate, onRefresh }) {
  const currencies = [
    { code:"EUR", name:"Euro",             flag:"🇪🇺", rateLabel:"1 USD =", rate: fx?.EUR   ? `${fx.EUR.toFixed(4)} EUR`  : "—" },
    { code:"ARS", name:"Argentine Peso",   flag:"🇦🇷", rateLabel:"1 USD =", rate: fx?.ARS   ? `${fx.ARS.toFixed(2)} ARS`  : "—" },
    { code:"BRL", name:"Brazilian Real",   flag:"🇧🇷", rateLabel:"1 USD =", rate: fx?.BRL   ? `${fx.BRL.toFixed(4)} BRL`  : "—" },
    { code:"MXN", name:"Mexican Peso",     flag:"🇲🇽", rateLabel:"1 USD =", rate: fx?.MXN   ? `${fx.MXN.toFixed(4)} MXN`  : "—" },
    { code:"COP", name:"Colombian Peso",   flag:"🇨🇴", rateLabel:"1 USD =", rate: fx?.COP   ? `${fx.COP.toFixed(0)} COP`  : "—" },
    { code:"CLP", name:"Chilean Peso",     flag:"🇨🇱", rateLabel:"1 USD =", rate: fx?.CLP   ? `${fx.CLP.toFixed(0)} CLP`  : "—" },
    { code:"GBP", name:"British Pound",    flag:"🇬🇧", rateLabel:"1 USD =", rate: fx?.GBP   ? `${fx.GBP.toFixed(4)} GBP`  : "—" },
    { code:"CNY", name:"Chinese Yuan",     flag:"🇨🇳", rateLabel:"1 USD =", rate: fx?.CNY   ? `${fx.CNY.toFixed(4)} CNY`  : "—" },
  ];

  const commodityCards = [
    { name:"Gold",     unit:"troy oz",  icon:"🥇", val: commodities?.goldUSD   ? `$${commodities.goldUSD.toLocaleString()}`  : "$3,342",  src:"Frankfurter/CDN" },
    { name:"Silver",   unit:"troy oz",  icon:"🥈", val: commodities?.silverUSD ? `$${commodities.silverUSD.toFixed(2)}` : "$32.40",  src:"Frankfurter/CDN" },
    { name:"Crude Oil",unit:"barrel",   icon:"🛢️", val:"$73.40",  src:"Reference rate" },
    { name:"Soybeans", unit:"bushel",   icon:"🌾", val:"$10.82",  src:"Reference rate" },
    { name:"Wheat",    unit:"bushel",   icon:"🌿", val:"$5.64",   src:"Reference rate" },
    { name:"Corn",     unit:"bushel",   icon:"🌽", val:"$4.38",   src:"Reference rate" },
  ];

  return (
    <div className="fade-in">
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"flex-start", marginBottom:"1.75rem" }}>
        <div>
          <h1 style={{ fontFamily:G.fontDisplay, fontSize:"1.5rem", fontWeight:700 }}>FX & Markets</h1>
          <p style={{ color:G.muted, fontSize:".85rem", marginTop:".2rem" }}>
            Live rates from Frankfurter (ECB) · {fxDate ? `Updated ${fxDate}` : "Loading…"}
          </p>
        </div>
        <button className="btn btn-ghost" onClick={onRefresh} disabled={fxLoading}>
          {fxLoading ? <><Spin/> Refreshing</> : "↻ Refresh"}
        </button>
      </div>

      {fxError && (
        <div className="card" style={{ padding:".9rem 1.25rem", marginBottom:"1.25rem", borderColor:G.danger+"44" }}>
          <span style={{ color:G.danger, fontSize:".85rem" }}>⚠ {fxError} — rates may be delayed.</span>
        </div>
      )}

      {/* Currency section */}
      <div style={{ fontFamily:G.fontDisplay, fontWeight:600, fontSize:"1rem", marginBottom:"1rem" }}>
        Currency Pairs <span style={{ fontSize:".75rem", color:G.accent, fontWeight:400, fontFamily:G.fontBody }}>vs USD · ECB reference</span>
      </div>

      <div className="fx-grid" style={{ display:"grid", gridTemplateColumns:"repeat(4,1fr)", gap:".85rem", marginBottom:"2rem" }}>
        {currencies.map(c => (
          <div key={c.code} className="fx-card">
            <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:".5rem" }}>
              <span style={{ fontSize:"1.4rem" }}>{c.flag}</span>
              <span className="badge badge-green">{c.code}</span>
            </div>
            <div style={{ fontSize:".75rem", color:G.faint, marginBottom:".2rem" }}>{c.name}</div>
            <div style={{ fontFamily:G.fontDisplay, fontWeight:700, fontSize:"1.15rem", color: fxLoading ? G.faint : G.text }}>
              {fxLoading ? "—" : c.rate}
            </div>
            <div style={{ fontSize:".68rem", color:G.faint, marginTop:".25rem" }}>{c.rateLabel} {c.code}</div>
          </div>
        ))}
      </div>

      {/* Commodities section */}
      <div style={{ fontFamily:G.fontDisplay, fontWeight:600, fontSize:"1rem", marginBottom:"1rem" }}>
        Commodities <span style={{ fontSize:".75rem", color:G.muted, fontWeight:400, fontFamily:G.fontBody }}>Metals live · Agri reference</span>
      </div>

      <div className="fx-grid" style={{ display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:".85rem", marginBottom:"2rem" }}>
        {commodityCards.map(c => (
          <div key={c.name} className="fx-card">
            <div style={{ fontSize:"1.5rem", marginBottom:".5rem" }}>{c.icon}</div>
            <div style={{ fontSize:".75rem", color:G.faint, marginBottom:".2rem" }}>{c.name} / {c.unit}</div>
            <div style={{ fontFamily:G.fontDisplay, fontWeight:700, fontSize:"1.3rem", color:G.accent }}>{c.val}</div>
            <div style={{ fontSize:".65rem", color:G.faint, marginTop:".3rem" }}>Source: {c.src}</div>
          </div>
        ))}
      </div>

      {/* Currency converter */}
      <CurrencyConverter fx={fx} />
    </div>
  );
}

function CurrencyConverter({ fx }) {
  const [amount, setAmount] = useState("1000");
  const [from,   setFrom]   = useState("USD");
  const [to,     setTo]     = useState("ARS");

  const pairs = ["USD","EUR","ARS","BRL","MXN","GBP","CNY"];

  const convert = () => {
    const amt = parseFloat(amount) || 0;
    if (!fx) return "—";
    // Everything relative to USD via fx (which is "X per 1 USD")
    const toUSD = from === "USD" ? amt : from === "EUR" ? amt * (1/fx.EUR) : amt / (fx[from] || 1);
    const result = to === "USD" ? toUSD : to === "EUR" ? toUSD * fx.EUR : toUSD * (fx[to] || 1);
    return result.toLocaleString("en-US", { maximumFractionDigits:2 });
  };

  const sel = (val, onChange) => (
    <select value={val} onChange={e=>onChange(e.target.value)}
      style={{ background:G.bg, border:`1px solid ${G.border}`, color:G.text, borderRadius:6, padding:".5rem .7rem", fontSize:".85rem", cursor:"pointer", outline:"none" }}>
      {pairs.map(p=><option key={p}>{p}</option>)}
    </select>
  );

  return (
    <div className="card" style={{ padding:"1.5rem" }}>
      <div style={{ fontFamily:G.fontDisplay, fontWeight:600, marginBottom:"1.25rem" }}>Currency Converter</div>
      <div style={{ display:"flex", gap:"1rem", alignItems:"center", flexWrap:"wrap" }}>
        <input className="input" style={{ maxWidth:160 }} type="number" value={amount} onChange={e=>setAmount(e.target.value)} placeholder="Amount" />
        {sel(from, setFrom)}
        <span style={{ color:G.muted, fontSize:"1.2rem" }}>→</span>
        {sel(to, setTo)}
        <div style={{ fontFamily:G.fontDisplay, fontSize:"1.4rem", fontWeight:700, color:G.accent }}>
          {fx ? convert() : "—"} {to}
        </div>
      </div>
      {!fx && <p style={{ color:G.faint, fontSize:".78rem", marginTop:".75rem" }}>Connect to live rates to enable conversion.</p>}
    </div>
  );
}

// ── LOGIN ─────────────────────────────────────────────────────────────────────
function LoginScreen({ onLogin }) {
  const [email,   setEmail]   = useState("");
  const [pass,    setPass]    = useState("");
  const [loading, setLoading] = useState(false);
  const [error,   setError]   = useState("");

  const handle = () => {
    if (!email||!pass) { setError("Please fill in all fields."); return; }
    setError(""); setLoading(true);
    setTimeout(() => { setLoading(false); onLogin({ name:"Ezequiel Prilusky", email }); }, 1100);
  };

  return (
    <div style={{ minHeight:"100vh", display:"flex", alignItems:"center", justifyContent:"center", padding:"1.5rem" }}>
      <div style={{ width:"100%", maxWidth:400 }} className="fade-in">
        <div style={{ textAlign:"center", marginBottom:"2.5rem" }}>
          <Logo size={1.9}/>
          <p style={{ color:G.muted, marginTop:".6rem", fontSize:".9rem" }}>Sign in to your account</p>
        </div>
        <div className="