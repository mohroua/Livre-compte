<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<meta name="theme-color" content="#070912">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>Livre de Compte Fraternel</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent;margin:0;padding:0}
:root{
  --bg:#070912;--s1:#0d1120;--s2:#131928;--s3:#1a2235;
  --border:rgba(255,255,255,0.07);--border2:rgba(255,255,255,0.12);
  --blue:#4f8eff;--gold:#f4b942;--green:#00d68f;--red:#ff4d6a;--wa:#25d366;
  --text:#e8edf5;--muted:#5a6580;
  --ui:'Syne',sans-serif;--mono:'DM Mono',monospace;
  --r:14px;--r2:18px;
}
html,body{height:100%;background:var(--bg);color:var(--text);font-family:var(--ui);overscroll-behavior:none}
input,button{font-family:var(--ui)}
input[type=number]::-webkit-inner-spin-button{-webkit-appearance:none}
input:focus{outline:2px solid var(--blue);outline-offset:-1px}
::-webkit-scrollbar{width:3px}
::-webkit-scrollbar-thumb{background:var(--s3);border-radius:4px}
button{cursor:pointer;border:none;background:none;color:inherit}

/* Layout */
#app{max-width:520px;margin:0 auto;min-height:100vh;display:flex;flex-direction:column;position:relative}
#header{padding:20px 18px 12px;display:flex;justify-content:space-between;align-items:center;flex-shrink:0}
#content{flex:1;overflow-y:auto;padding:0 14px 120px}
#tabnav{flex-shrink:0;display:flex;margin:0 14px 16px;background:var(--s1);border-radius:14px;padding:4px;gap:3px}

/* Header */
.hdr-left .sub{font-size:10px;font-weight:700;letter-spacing:3px;color:var(--muted);text-transform:uppercase}
.hdr-left .title{font-size:19px;font-weight:800;margin-top:2px}
.hdr-right{display:flex;gap:8px;align-items:center}
.saved-badge{font-size:10px;color:var(--green);font-family:var(--mono);display:flex;align-items:center;gap:4px;opacity:0;transition:opacity .4s}
.saved-badge.show{opacity:1}

/* Icon Buttons */
.ibtn{width:36px;height:36px;border-radius:10px;background:var(--s2);border:1px solid var(--border);
  display:flex;align-items:center;justify-content:center;color:var(--muted);transition:all .2s;flex-shrink:0}
.ibtn:hover{border-color:var(--border2);color:var(--text)}
.ibtn svg{pointer-events:none}

/* Balance Card */
#balance-card{
  margin:0 14px 18px;background:linear-gradient(140deg,#111d36 0%,#0d1628 60%,#080f1e 100%);
  border-radius:20px;padding:22px 22px 18px;border:1px solid rgba(79,142,255,.12);position:relative;overflow:hidden
}
#balance-card::before{
  content:'';position:absolute;top:-30px;right:-30px;width:100px;height:100px;
  border-radius:50%;filter:blur(36px);transition:background .5s;pointer-events:none
}
#balance-card.pos::before{background:rgba(0,214,143,.1)}
#balance-card.neg::before{background:rgba(255,77,106,.1)}
#balance-card.zero::before{background:rgba(79,142,255,.07)}
.bc-label{font-size:10px;font-weight:700;letter-spacing:3px;color:var(--muted);text-transform:uppercase;margin-bottom:6px}
#bc-amount{font-family:var(--mono);font-size:34px;font-weight:500;letter-spacing:-1px;transition:color .4s}
#bc-amount .unit{font-size:13px;color:var(--muted);font-weight:400;margin-left:6px}
#bc-status{margin-top:7px;font-size:12px;display:flex;align-items:center;gap:6px;transition:color .4s}
.bc-divider{border:none;border-top:1px solid var(--border);margin:16px 0 14px}
.bc-stats{display:flex;gap:0}
.bc-stat{flex:1;padding-left:14px;border-left:1px solid var(--border);margin-left:14px}
.bc-stat:first-child{padding-left:0;border-left:none;margin-left:0}
.bc-stat .slabel{font-size:9px;color:var(--muted);letter-spacing:2px;font-weight:700;text-transform:uppercase;line-height:1.4}
.bc-stat .sval{font-family:var(--mono);font-size:17px;margin-top:4px}
.bc-stat .sunit{font-size:9px;color:var(--muted)}

/* Tabs */
.tab-btn{flex:1;padding:9px 6px;border-radius:11px;font-size:11px;font-weight:700;
  display:flex;align-items:center;justify-content:center;gap:5px;transition:all .2s;
  color:var(--muted);background:transparent}
.tab-btn.active{background:var(--blue);color:#fff;box-shadow:0 4px 12px rgba(79,142,255,.3)}
.tab-btn svg{flex-shrink:0}

/* Section label */
.sec-label{font-size:10px;font-weight:700;color:var(--muted);letter-spacing:3px;text-transform:uppercase;margin-bottom:12px}

/* Transaction Row */
.tx-row{background:var(--s2);border-radius:14px;padding:13px 14px;margin-bottom:8px;
  border:1px solid var(--border);display:flex;align-items:center;gap:11px;
  animation:fadeIn .25s ease}
.tx-icon{width:38px;height:38px;border-radius:11px;flex-shrink:0;display:flex;align-items:center;justify-content:center}
.tx-icon.me{background:rgba(0,214,143,.1);color:var(--green)}
.tx-icon.bro{background:rgba(255,77,106,.1);color:var(--red)}
.tx-body{flex:1;min-width:0}
.tx-name{font-weight:700;font-size:13px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.tx-meta{font-size:11px;color:var(--muted);margin-top:2px}
.tx-amount{font-family:var(--mono);font-size:15px;font-weight:500;flex-shrink:0;margin-right:6px}
.tx-amount.me{color:var(--green)}
.tx-amount.bro{color:var(--red)}
.tx-action{width:30px;height:30px;border-radius:8px;display:flex;align-items:center;justify-content:center;flex-shrink:0;font-size:12px}
.tx-wa{background:rgba(37,211,102,.08);border:1px solid rgba(37,211,102,.15);color:var(--wa)}
.tx-del{background:rgba(255,77,106,.08);border:1px solid rgba(255,77,106,.15);color:var(--red)}

/* Empty State */
.empty{text-align:center;padding:50px 0;color:var(--muted)}
.empty .emoji{font-size:36px;margin-bottom:10px}
.empty .etitle{font-size:14px}
.empty .esub{font-size:12px;margin-top:6px;opacity:.6}

/* Chart containers */
.chart-card{background:var(--s2);border-radius:16px;padding:18px 10px 12px;border:1px solid var(--border);margin-bottom:16px}
.chart-legend{display:flex;justify-content:center;gap:20px;margin-top:10px}
.legend-item{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted)}
.legend-dot{width:8px;height:8px;border-radius:3px}

/* Monthly summary rows */
.month-row{background:var(--s2);border-radius:12px;padding:12px 14px;margin-bottom:8px;
  border:1px solid var(--border);display:flex;align-items:center}
.month-row .mkey{font-weight:700;font-size:13px}
.month-row .mdetail{font-size:11px;color:var(--muted);margin-top:2px;font-family:var(--mono)}
.month-row .mnet{font-family:var(--mono);font-size:15px;font-weight:500}

/* FAB */
#fab{position:fixed;bottom:28px;right:calc(50% - 260px + 14px);width:54px;height:54px;
  border-radius:17px;background:var(--blue);display:flex;align-items:center;justify-content:center;
  color:#fff;box-shadow:0 8px 28px rgba(79,142,255,.45);z-index:50;transition:transform .15s,box-shadow .15s}
#fab:hover{transform:scale(1.08);box-shadow:0 12px 32px rgba(79,142,255,.55)}
@media(max-width:520px){#fab{right:20px}}

/* Modals */
.overlay{position:fixed;inset:0;background:rgba(0,0,0,.7);z-index:200;animation:fadeIn .2s ease;display:none}
.overlay.open{display:flex}
#add-modal .sheet{
  background:var(--s1);width:100%;max-width:520px;margin:0 auto;
  border-radius:22px 22px 0 0;padding:22px;border-top:1px solid var(--border);
  animation:slideUp .25s ease;align-self:flex-end
}
#sync-modal .dialog{
  background:var(--s1);border-radius:20px;padding:24px;width:100%;max-width:440px;
  border:1px solid var(--border);margin:auto
}
.modal-handle{width:36px;height:4px;border-radius:2px;background:var(--s3);margin:0 auto 20px}
.modal-hdr{display:flex;justify-content:space-between;align-items:center;margin-bottom:22px}
.modal-title{font-size:17px;font-weight:800}
.close-btn{width:30px;height:30px;border-radius:9px;background:var(--s2);border:1px solid var(--border);
  display:flex;align-items:center;justify-content:center;color:var(--muted)}

/* Form */
.field{margin-bottom:14px}
.field label{font-size:10px;font-weight:700;color:var(--muted);letter-spacing:3px;text-transform:uppercase;display:block;margin-bottom:8px}
.field input{width:100%;padding:14px 16px;background:var(--s2);border:1px solid var(--border);
  border-radius:13px;color:var(--text);font-size:16px}
#inp-amount{font-size:22px;font-family:var(--mono)}
.add-btns{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.add-btn{padding:16px;border-radius:15px;font-size:13px;font-weight:800;
  display:flex;align-items:center;justify-content:center;gap:8px;transition:opacity .15s}
.add-btn:hover{opacity:.85}
.add-me{background:rgba(0,214,143,.1);border:1px solid rgba(0,214,143,.25);color:var(--green)}
.add-bro{background:rgba(255,77,106,.1);border:1px solid rgba(255,77,106,.25);color:var(--red)}

/* Info boxes in sync modal */
.info-box{border-radius:12px;padding:14px 16px;margin-bottom:12px;font-size:12px;line-height:1.65}
.info-box .ibtitle{font-weight:700;font-size:12px;margin-bottom:6px;display:flex;align-items:center;gap:6px}
.info-box .ibtext{color:var(--muted)}
.info-box .ibtext b{color:var(--text)}
.info-box .ibtext code{background:var(--s2);padding:1px 5px;border-radius:4px;font-size:11px;font-family:var(--mono)}
.ib-blue{background:rgba(79,142,255,.08);border:1px solid rgba(79,142,255,.2)}
.ib-blue .ibtitle{color:var(--blue)}
.ib-gold{background:rgba(244,185,66,.08);border:1px solid rgba(244,185,66,.2)}
.ib-gold .ibtitle{color:var(--gold)}
.ib-green{background:rgba(0,214,143,.08);border:1px solid rgba(0,214,143,.2)}
.ib-green .ibtitle{color:var(--green)}

/* Export buttons */
.export-row{display:flex;gap:8px;justify-content:flex-end;margin-bottom:14px}
.exp-btn{padding:6px 12px;border-radius:8px;font-size:11px;font-weight:700;display:flex;align-items:center;gap:5px}
.exp-csv{background:rgba(0,214,143,.1);border:1px solid rgba(0,214,143,.2);color:var(--green)}
.exp-pdf{background:rgba(244,185,66,.1);border:1px solid rgba(244,185,66,.2);color:var(--gold)}

/* See more btn */
.see-more{width:100%;padding:13px;background:var(--s2);border:1px solid var(--border);
  border-radius:12px;color:var(--blue);font-size:12px;font-weight:700;margin-top:4px}

/* Animations */
@keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}}
@keyframes fadeIn{from{opacity:0}to{opacity:1}}

/* Print */
@media print{
  .no-print{display:none!important}
  body{background:#fff!important;color:#000!important}
  *{color:#000!important;background:#fff!important;border-color:#ddd!important}
  #balance-card{border:1px solid #ccc!important}
  .tx-row{border:1px solid #eee!important;page-break-inside:avoid}
}
</style>
</head>
<body>
<div id="app">
  <!-- Header -->
  <div id="header" class="no-print">
    <div class="hdr-left">
      <div class="sub">Livre de Compte</div>
      <div class="title">Fraternel 🤝</div>
    </div>
    <div class="hdr-right">
      <span class="saved-badge" id="saved-badge">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6L9 17l-5-5"/></svg>
        Sauvegardé
      </span>
      <button class="ibtn" onclick="exportCSV()" title="Exporter CSV">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      </button>
      <button class="ibtn" onclick="window.print()" title="Imprimer PDF">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
      </button>
      <button class="ibtn" onclick="openSync()" title="Synchronisation">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12.55a11 11 0 0114.08 0"/><path d="M1.42 9a16 16 0 0121.16 0"/><path d="M8.53 16.11a6 6 0 016.95 0"/><circle cx="12" cy="20" r="1" fill="currentColor"/></svg>
      </button>
    </div>
  </div>

  <!-- Balance Card -->
  <div id="balance-card" class="zero">
    <div class="bc-label">Solde Actuel</div>
    <div id="bc-amount">0.00 <span class="unit">DZD</span></div>
    <div id="bc-status">— Comptes équilibrés</div>
    <hr class="bc-divider">
    <div class="bc-stats">
      <div class="bc-stat">
        <div class="slabel">Mes dépenses</div>
        <div class="sval" id="stat-me" style="color:var(--green)">0 <span class="sunit">DZD</span></div>
      </div>
      <div class="bc-stat">
        <div class="slabel">Ses dépenses</div>
        <div class="sval" id="stat-bro" style="color:var(--red)">0 <span class="sunit">DZD</span></div>
      </div>
      <div class="bc-stat">
        <div class="slabel">Transactions</div>
        <div class="sval" id="stat-count" style="color:var(--blue)">0</div>
      </div>
    </div>
  </div>

  <!-- Tab Nav -->
  <div id="tabnav" class="no-print">
    <button class="tab-btn active" data-tab="home" onclick="switchTab('home')">
      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
      Accueil
    </button>
    <button class="tab-btn" data-tab="stats" onclick="switchTab('stats')">
      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
      Stats
    </button>
    <button class="tab-btn" data-tab="history" onclick="switchTab('history')">
      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
      Historique
    </button>
  </div>

  <!-- Content -->
  <div id="content">
    <!-- HOME TAB -->
    <div id="tab-home">
      <div class="sec-label">Dernières transactions</div>
      <div id="home-list"></div>
    </div>

    <!-- STATS TAB -->
    <div id="tab-stats" style="display:none">
      <div class="sec-label">Dépenses mensuelles</div>
      <div class="chart-card">
        <canvas id="chart-bar" height="180"></canvas>
        <div class="chart-legend">
          <div class="legend-item"><div class="legend-dot" style="background:var(--green)"></div> Moi</div>
          <div class="legend-item"><div class="legend-dot" style="background:var(--red)"></div> Frère</div>
        </div>
      </div>
      <div class="sec-label">Évolution du solde net</div>
      <div class="chart-card">
        <canvas id="chart-area" height="140"></canvas>
      </div>
      <div class="sec-label">Résumé mensuel</div>
      <div id="month-summary"></div>
    </div>

    <!-- HISTORY TAB -->
    <div id="tab-history" style="display:none">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:12px">
        <div class="sec-label" style="margin-bottom:0">Historique complet</div>
        <div class="export-row" style="margin-bottom:0">
          <button class="exp-btn exp-csv" onclick="exportCSV()">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
            CSV
          </button>
          <button class="exp-btn exp-pdf" onclick="window.print()">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
            PDF
          </button>
        </div>
      </div>
      <div id="history-list"></div>
    </div>
  </div>

  <!-- FAB -->
  <button id="fab" class="no-print" onclick="openAdd()">
    <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2.5"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
  </button>
</div>

<!-- Add Transaction Modal -->
<div class="overlay" id="add-modal" onclick="closeModal(event,'add-modal')">
  <div class="sheet">
    <div class="modal-handle"></div>
    <div class="modal-hdr">
      <div class="modal-title">Nouvelle Transaction</div>
      <button class="close-btn" onclick="closeAdd()">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
      </button>
    </div>
    <div class="field">
      <label>Montant (DZD)</label>
      <input type="number" id="inp-amount" placeholder="0.00" inputmode="decimal" autocomplete="off">
    </div>
    <div class="field" style="margin-bottom:22px">
      <label>Description</label>
      <input type="text" id="inp-reason" placeholder="Ex: Café, Transport, Restaurant...">
    </div>
    <div class="add-btns">
      <button class="add-btn add-me" onclick="addEntry('me')">
        <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="7" y1="17" x2="17" y2="7"/><polyline points="7 7 17 7 17 17"/></svg>
        J'ai payé
      </button>
      <button class="add-btn add-bro" onclick="addEntry('bro')">
        <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="17" y1="7" x2="7" y2="17"/><polyline points="17 17 7 17 7 7"/></svg>
        Il a payé
      </button>
    </div>
  </div>
</div>

<!-- Sync Modal -->
<div class="overlay" id="sync-modal" onclick="closeModal(event,'sync-modal')" style="align-items:center;justify-content:center;padding:16px">
  <div class="dialog">
    <div class="modal-hdr">
      <div class="modal-title" style="display:flex;align-items:center;gap:8px">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="var(--blue)" stroke-width="2"><path d="M5 12.55a11 11 0 0114.08 0"/><path d="M1.42 9a16 16 0 0121.16 0"/><path d="M8.53 16.11a6 6 0 016.95 0"/><circle cx="12" cy="20" r="1" fill="var(--blue)"/></svg>
        Synchronisation
      </div>
      <button class="close-btn" onclick="document.getElementById('sync-modal').classList.remove('open')">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
      </button>
    </div>
    <div class="info-box ib-blue">
      <div class="ibtitle">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
        Option 1 — Firebase (Recommandé)
      </div>
      <div class="ibtext">
        Firebase Realtime Database permet la sync en temps réel entre deux appareils gratuitement.<br><br>
        <b>Étapes :</b><br>
        1. Créer un projet sur <b>firebase.google.com</b><br>
        2. Activer "Realtime Database"<br>
        3. Remplacer <code>localStorage</code> par <code>firebase.database()</code><br>
        4. Partager les credentials avec ton frère
      </div>
    </div>
    <div class="info-box ib-gold">
      <div class="ibtitle">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 6L9 17l-5-5"/></svg>
        Option 2 — Google Sheets
      </div>
      <div class="ibtext">
        Via Google Apps Script, chaque transaction peut être envoyée à un Google Sheet partagé.<br><br>
        Avantages : Visualisation native, partage simple, historique automatique.
      </div>
    </div>
    <div class="info-box ib-green" style="margin-bottom:0">
      <div class="ibtitle">💾 Actuellement actif</div>
      <div class="ibtext">
        Tes données sont sauvegardées localement (localStorage) et persistent entre les sessions. Utilise le bouton <b>WhatsApp</b> sur chaque transaction pour synchroniser manuellement avec ton frère.
      </div>
    </div>
  </div>
</div>

<script>
const KEY = 'fraternal_ledger_v4';
let entries = [];
let barChart = null, areaChart = null;

/* ── Storage ── */
function save() {
  localStorage.setItem(KEY, JSON.stringify(entries));
  const b = document.getElementById('saved-badge');
  b.classList.add('show');
  clearTimeout(b._t);
  b._t = setTimeout(() => b.classList.remove('show'), 1800);
}
function load() {
  try { entries = JSON.parse(localStorage.getItem(KEY)) || []; } catch { entries = []; }
}

/* ── Computed ── */
function getBalance() { return entries.reduce((s,e) => s + (e.who==='me' ? e.amount : -e.amount), 0); }
function getMonthlyData() {
  const map = {};
  entries.forEach(e => {
    const d = new Date(e.date);
    const key = `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}`;
    const label = d.toLocaleString('fr-FR',{month:'short'}).slice(0,3).toUpperCase();
    if (!map[key]) map[key] = {key, label, moi:0, lui:0, net:0};
    if (e.who==='me') { map[key].moi += e.amount; map[key].net += e.amount; }
    else { map[key].lui += e.amount; map[key].net -= e.amount; }
  });
  return Object.values(map).sort((a,b) => a.key.localeCompare(b.key)).slice(-6);
}

/* ── Update Balance Card ── */
function updateCard() {
  const bal = getBalance();
  const card = document.getElementById('balance-card');
  const amtEl = document.getElementById('bc-amount');
  const stEl = document.getElementById('bc-status');
  card.className = bal > 0 ? 'pos' : bal < 0 ? 'neg' : 'zero';
  amtEl.innerHTML = `${bal > 0 ? '+' : ''}${bal.toFixed(2)} <span class="unit">DZD</span>`;
  amtEl.style.color = bal > 0 ? 'var(--green)' : bal < 0 ? 'var(--red)' : 'var(--text)';
  const icon = bal > 0 ? '↑' : bal < 0 ? '↓' : '—';
  const msg = bal > 0 ? 'Ton frère te doit de l\'argent' : bal < 0 ? 'Tu lui dois de l\'argent' : 'Comptes équilibrés ✓';
  stEl.textContent = `${icon} ${msg}`;
  stEl.style.color = bal > 0 ? 'var(--green)' : bal < 0 ? 'var(--red)' : 'var(--muted)';

  const me = entries.filter(e=>e.who==='me').reduce((s,e)=>s+e.amount,0);
  const bro = entries.filter(e=>e.who==='bro').reduce((s,e)=>s+e.amount,0);
  document.getElementById('stat-me').innerHTML = `${me.toFixed(0)} <span class="sunit">DZD</span>`;
  document.getElementById('stat-bro').innerHTML = `${bro.toFixed(0)} <span class="sunit">DZD</span>`;
  document.getElementById('stat-count').textContent = entries.length;
}

/* ── Render TX Row HTML ── */
function txHTML(e) {
  const isMe = e.who === 'me';
  const d = new Date(e.date);
  const label = d.toLocaleDateString('fr-FR',{day:'2-digit',month:'short'});
  const msg = encodeURIComponent(`*📒 Registre de Compte*\n\n📌 *${e.reason}*\n💰 Montant: *${isMe?'+':'-'}${e.amount.toFixed(2)} DZD*\n👤 Payé par: ${isMe?'Moi':'Mon frère'}\n\n📊 Solde: *${Math.abs(getBalance()).toFixed(2)} DZD*`);
  return `<div class="tx-row" id="tx-${e.id}">
    <div class="tx-icon ${e.who}">
      ${isMe
        ? `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="7" y1="17" x2="17" y2="7"/><polyline points="7 7 17 7 17 17"/></svg>`
        : `<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="17" y1="7" x2="7" y2="17"/><polyline points="17 17 7 17 7 7"/></svg>`}
    </div>
    <div class="tx-body">
      <div class="tx-name">${escHtml(e.reason)}</div>
      <div class="tx-meta">${label} · ${isMe?'Payé par moi':'Payé par frère'}</div>
    </div>
    <div class="tx-amount ${e.who}">${isMe?'+':'-'}${e.amount.toFixed(2)}</div>
    <a href="https://wa.me/?text=${msg}" class="tx-action tx-wa" target="_blank" title="Partager WhatsApp">
      <svg width="13" height="13" viewBox="0 0 24 24" fill="currentColor"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347"/></svg>
    </a>
    <button class="tx-action tx-del" onclick="deleteEntry(${e.id})" title="Supprimer">
      <svg width="13" ></svg>="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6m3 0V4a1 1 0 011-1h4a1 1 0 011 1v2"/></svg>
    </button>
  </div>`;
}

/* ── Render Home ── */
function renderHome() {
  const el = document.getElementById('home-list');
  if (!entries.length) {
    el.innerHTML = `<div class="empty"><div class="emoji">📋</div><div class="etitle">Aucune transaction pour l'instant</div><div class="esub">Appuyez sur + pour en ajouter une</div></div>`;
    return;
  }
  const slice = entries.slice(0, 7);
  el.innerHTML = slice.map(txHTML).join('');
  if (entries.length > 7) {
    el.innerHTML += `<button class="see-more" onclick="switchTab('history')">Voir tout (${entries.length} transactions) →</button>`;
  }
}

/* ── Render History ── */
function renderHistory() {
  const el = document.getElementById('history-list');
  if (!entries.length) {
    el.innerHTML = `<div class="empty"><div class="emoji">📋</div><div class="etitle">Aucune transaction</div></div>`;
    return;
  }
  el.innerHTML = entries.map(txHTML).join('');
}

/* ── Render Stats ── */
function renderStats() {
  const data = getMonthlyData();
  if (!data.length) {
    document.getElementById('month-summary').innerHTML = `<div class="empty"><div class="emoji">📊</div><div class="etitle">Pas assez de données</div><div class="esub">Ajoutez des transactions pour voir les statistiques</div></div>`;
    if (barChart) { barChart.destroy(); barChart = null; }
    if (areaChart) { areaChart.destroy(); areaChart = null; }
    document.getElementById('chart-bar').getContext('2d').clearRect(0,0,9999,9999);
    document.getElementById('chart-area').getContext('2d').clearRect(0,0,9999,9999);
    return;
  }

  const labels = data.map(d => d.label);
  const moiData = data.map(d => d.moi);
  const luiData = data.map(d => d.lui);
  const netData = data.map(d => d.net);

  const gridColor = 'rgba(255,255,255,0.04)';
  const tickColor = '#5a6580';
  const baseOpts = {
    plugins: { legend: { display: false }, tooltip: {
      backgroundColor: '#0d1120', borderColor: 'rgba(255,255,255,0.07)', borderWidth: 1,
      titleColor: '#5a6580', bodyColor: '#e8edf5',
      titleFont: { family: 'DM Mono', size: 10 }, bodyFont: { family: 'DM Mono', size: 11 },
      padding: 10, cornerRadius: 10
    }},
    scales: {
      x: { grid: { color: gridColor }, ticks: { color: tickColor, font: { family: 'DM Mono', size: 10 } }, border: { display: false } },
      y: { grid: { color: gridColor }, ticks: { color: tickColor, font: { family: 'DM Mono', size: 10 } }, border: { display: false } }
    }
  };

  if (barChart) barChart.destroy();
  barChart = new Chart(document.getElementById('chart-bar'), {
    type: 'bar',
    data: {
      labels,
      datasets: [
        { label: 'Moi', data: moiData, backgroundColor: 'rgba(0,214,143,0.8)', borderRadius: 6, barPercentage: 0.55 },
        { label: 'Frère', data: luiData, backgroundColor: 'rgba(255,77,106,0.8)', borderRadius: 6, barPercentage: 0.55 }
      ]
    },
    options: { ...baseOpts, responsive: true, maintainAspectRatio: false }
  });

  if (areaChart) areaChart.destroy();
  const ctx2 = document.getElementById('chart-area').getContext('2d');
  const grad = ctx2.createLinearGradient(0, 0, 0, 140);
  grad.addColorStop(0, 'rgba(79,142,255,0.25)');
  grad.addColorStop(1, 'rgba(79,142,255,0.01)');
  areaChart = new Chart(ctx2, {
    type: 'line',
    data: {
      labels,
      datasets: [{
        label: 'Solde net', data: netData,
        borderColor: '#4f8eff', backgroundColor: grad, fill: true,
        tension: 0.4, pointBackgroundColor: '#4f8eff', pointRadius: 4, pointBorderWidth: 0, borderWidth: 2
      }]
    },
    options: { ...baseOpts, responsive: true, maintainAspectRatio: false }
  });

  const summary = document.getElementById('month-summary');
  summary.innerHTML = data.slice().reverse().map(m => `
    <div class="month-row">
      <div style="flex:1">
        <div class="mkey">${m.key}</div>
        <div class="mdetail"><span style="color:var(--green)">+${m.moi.toFixed(0)}</span> / <span style="color:var(--red)">-${m.lui.toFixed(0)}</span> DZD</div>
      </div>
      <div class="mnet" style="color:${m.net >= 0 ? 'var(--green)' : 'var(--red)'}">${m.net > 0 ? '+' : ''}${m.net.toFixed(0)} DZD</div>
    </div>`).join('');
}

/* ── Full Render ── */
function render() {
  updateCard();
  renderHome();
  renderHistory();
  const activeTab = document.querySelector('.tab-btn.active')?.dataset?.tab;
  if (activeTab === 'stats') renderStats();
}

/* ── CRUD ── */
function addEntry(who) {
  const amtEl = document.getElementById('inp-amount');
  const rsnEl = document.getElementById('inp-reason');
  const amount = parseFloat(amtEl.value);
  if (!amount || amount <= 0) { amtEl.focus(); return; }
  const reason = rsnEl.value.trim() || 'Transaction';
  entries.unshift({ id: Date.now(), who, amount, reason, date: new Date().toISOString() });
  save(); render();
  amtEl.value = ''; rsnEl.value = '';
  closeAdd();
}
function deleteEntry(id) {
  entries = entries.filter(e => e.id !== id);
  save(); render();
}

/* ── Tabs ── */
let currentTab = 'home';
function switchTab(tab) {
  ['home','stats','history'].forEach(t => {
    document.getElementById(`tab-${t}`).style.display = t === tab ? '' : 'none';
  });
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.dataset.tab === tab));
  currentTab = tab;
  if (tab === 'stats') renderStats();
}

/* ── Modals ── */
function openAdd() {
  document.getElementById('add-modal').classList.add('open');
  setTimeout(() => document.getElementById('inp-amount').focus(), 100);
}
function closeAdd() { document.getElementById('add-modal').classList.remove('open'); }
function openSync() { document.getElementById('sync-modal').classList.add('open'); }
function closeModal(e, id) { if (e.target === e.currentTarget) document.getElementById(id).classList.remove('open'); }

/* ── Key events ── */
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') { closeAdd(); document.getElementById('sync-modal').classList.remove('open'); }
});
document.getElementById('inp-amount').addEventListener('keydown', e => {
  if (e.key === 'Enter') document.getElementById('inp-reason').focus();
});
document.getElementById('inp-reason').addEventListener('keydown', e => {
  if (e.key === 'Enter') addEntry('me');
});

/* ── Export CSV ── */
function exportCSV() {
  const rows = [['Date','Payé par','Montant (DZD)','Description'],
    ...entries.map(e => [
      new Date(e.date).toLocaleDateString('fr-FR'),
      e.who === 'me' ? 'Moi' : 'Frère',
      e.amount.toFixed(2),
      `"${e.reason.replace(/"/g,'""')}"`
    ])
  ].map(r => r.join(',')).join('\n');
  const a = Object.assign(document.createElement('a'), {
    href: URL.createObjectURL(new Blob(['\uFEFF'+rows], {type:'text/csv;charset=utf-8;'})),
    download: `registre_${new Date().toISOString().slice(0,10)}.csv`
  });
  a.click(); URL.revokeObjectURL(a.href);
}

function escHtml(s) {
  return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

/* ── Init ── */
load(); render();
</script>
</body>
</html>
