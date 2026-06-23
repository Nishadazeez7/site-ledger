!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Site Ledger — Project Finance Dashboard</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@600;700;800&family=IBM+Plex+Mono:wght@400;500;600;700&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --ink:#101b2b;
    --panel:#16263b;
    --panel-2:#1c3148;
    --line: rgba(226,163,59,0.16);
    --line-strong: rgba(226,163,59,0.32);
    --paper:#f2eddc;
    --paper-dim:#cfc8ab;
    --amber:#e2a33b;
    --amber-soft: rgba(226,163,59,0.14);
    --rust:#c1502e;
    --rust-soft: rgba(193,80,46,0.16);
    --green:#7a9b66;
    --green-soft: rgba(122,155,102,0.16);
    --steel:#7e93ab;
    --shadow: 0 14px 30px -16px rgba(0,0,0,0.55);
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{
    background:
      linear-gradient(var(--line) 1px, transparent 1px) 0 0/42px 42px,
      linear-gradient(90deg, var(--line) 1px, transparent 1px) 0 0/42px 42px,
      var(--ink);
    color:var(--paper);
    font-family:'Inter', sans-serif;
    min-height:100vh;
    padding: 28px 20px 60px;
  }
  .wrap{max-width:1280px;margin:0 auto;}
 
  /* ===== Header / Title block ===== */
  .titleblock{
    display:flex; justify-content:space-between; align-items:flex-end;
    border-bottom: 2px solid var(--amber);
    padding-bottom:18px; margin-bottom: 22px;
    flex-wrap:wrap; gap:16px;
  }
  .titleblock .eyebrow{
    font-family:'IBM Plex Mono'; font-size:11px; letter-spacing:.18em;
    color:var(--amber); text-transform:uppercase; margin-bottom:6px;
  }
  h1{
    font-family:'Big Shoulders Display'; font-weight:800;
    font-size:clamp(34px, 5vw, 54px); line-height:0.92; margin:0;
    letter-spacing:0.01em; text-transform:uppercase;
    color: var(--paper);
  }
  h1 span{color:var(--amber);}
  .stamp{
    font-family:'IBM Plex Mono'; font-size:12px; color:var(--ink);
    background:var(--amber); padding:10px 16px; border-radius:3px;
    transform: rotate(-2deg);
    box-shadow: var(--shadow);
    text-align:center; line-height:1.5; font-weight:600;
    white-space:nowrap;
  }
  .stamp b{display:block; font-size:15px; letter-spacing:.04em;}
 
  /* ===== Section labels ===== */
  .section-label{
    display:flex; align-items:center; gap:12px;
    font-family:'IBM Plex Mono'; font-size:12px; letter-spacing:.14em;
    text-transform:uppercase; color:var(--steel);
    margin: 38px 0 14px;
  }
  .section-label::after{
    content:""; flex:1; height:1px;
    background: repeating-linear-gradient(90deg, var(--line-strong) 0 6px, transparent 6px 12px);
  }
 
  /* ===== KPI row ===== */
  .kpi-row{
    display:grid; grid-template-columns:repeat(4,1fr); gap:14px;
  }
  .kpi{
    background: var(--panel);
    border:1px solid var(--line-strong);
    border-radius:6px;
    padding:18px 18px 16px;
    position:relative;
    box-shadow: var(--shadow);
  }
  .kpi::before{
    content:""; position:absolute; top:0; left:0; right:0; height:3px;
    background: var(--accent, var(--amber));
    border-radius:6px 6px 0 0;
  }
  .kpi .lbl{font-family:'IBM Plex Mono'; font-size:11px; letter-spacing:.1em; color:var(--steel); text-transform:uppercase;}
  .kpi .val{font-family:'IBM Plex Mono'; font-weight:700; font-size:26px; margin-top:8px; color:var(--paper);}
  .kpi .sub{font-size:12px; color:var(--paper-dim); margin-top:4px;}
  .kpi.income{--accent:var(--green);}
  .kpi.expense{--accent:var(--rust);}
  .kpi.withdrawal{--accent:var(--steel);}
  .kpi.balance{--accent:var(--amber);}
 
  /* ===== Grid layout ===== */
  .grid-2{display:grid; grid-template-columns: 1.3fr 1fr; gap:16px;}
  .grid-3{display:grid; grid-template-columns: repeat(3,1fr); gap:16px;}
  .panel{
    background:var(--panel);
    border:1px solid var(--line-strong);
    border-radius:6px;
    padding:18px 18px 8px;
    box-shadow: var(--shadow);
  }
  .panel h3{
    font-family:'Big Shoulders Display'; font-weight:700; text-transform:uppercase;
    letter-spacing:.02em; font-size:18px; margin:0 0 14px; color:var(--paper);
    display:flex; justify-content:space-between; align-items:baseline;
  }
  .panel h3 small{font-family:'IBM Plex Mono'; font-size:11px; color:var(--steel); text-transform:none; letter-spacing:0;}
  canvas{max-width:100%;}
 
  /* ===== Tables ===== */
  table{width:100%; border-collapse:collapse; font-size:13px;}
  thead th{
    text-align:left; font-family:'IBM Plex Mono'; font-size:10.5px;
    letter-spacing:.08em; text-transform:uppercase; color:var(--steel);
    padding:8px 6px; border-bottom:1px solid var(--line-strong);
  }
  tbody td{padding:9px 6px; border-bottom:1px solid var(--line); color:var(--paper-dim);}
  tbody tr:hover td{background: var(--amber-soft); color:var(--paper);}
  td.num, th.num{text-align:right; font-family:'IBM Plex Mono';}
  td.name{color:var(--paper); font-weight:500;}
  .pill{
    display:inline-block; padding:2px 8px; border-radius:20px; font-size:10.5px;
    font-family:'IBM Plex Mono'; letter-spacing:.04em;
  }
  .pill.exp{background:var(--rust-soft); color:#e8957c;}
  .pill.inc{background:var(--green-soft); color:#a9c897;}
  .pill.wd{background: rgba(126,147,171,0.18); color:var(--steel);}
  .pos{color:#a9c897;}
  .neg{color:#e8957c;}
 
  /* recent activity */
  .feed-row{
    display:grid; grid-template-columns: 70px 1fr auto; gap:10px;
    align-items:center; padding:9px 0; border-bottom:1px solid var(--line);
  }
  .feed-row .date{font-family:'IBM Plex Mono'; font-size:11px; color:var(--steel);}
  .feed-row .desc{font-size:13px; color:var(--paper);}
  .feed-row .meta{font-size:11px; color:var(--paper-dim); font-family:'IBM Plex Mono';}
  .feed-row .amt{font-family:'IBM Plex Mono'; font-weight:600; text-align:right; white-space:nowrap;}
 
  footer{
    margin-top:40px; padding-top:16px; border-top:1px solid var(--line-strong);
    font-family:'IBM Plex Mono'; font-size:11px; color:var(--steel);
    display:flex; justify-content:space-between; flex-wrap:wrap; gap:8px;
  }
 
  @media (max-width: 900px){
    .kpi-row{grid-template-columns:repeat(2,1fr);}
    .grid-2{grid-template-columns:1fr;}
    .grid-3{grid-template-columns:1fr;}
  }
</style>
</head>
<body>
<div class="wrap">
 
  <div class="titleblock">
    <div>
      <div class="eyebrow">Multi-Project Cash Book · Jan – Jun 2026</div>
      <h1>Site <span>Ledger</span></h1>
    </div>
    <div class="stamp">RECORDS VERIFIED<b>12 Active Projects</b></div>
  </div>
 
  <div class="kpi-row">
    <div class="kpi income">
      <div class="lbl">Total Income</div>
      <div class="val">₹6,90,150</div>
      <div class="sub">Across all projects, advances + receipts</div>
    </div>
    <div class="kpi expense">
      <div class="lbl">Total Expense</div>
      <div class="val">₹5,40,535</div>
      <div class="sub">Material, labour, travel &amp; site costs</div>
    </div>
    <div class="kpi withdrawal">
      <div class="lbl">Partner Withdrawals</div>
      <div class="val">₹1,40,000</div>
      <div class="sub">Profit share drawn — Nishad &amp; Shanid</div>
    </div>
    <div class="kpi balance">
      <div class="lbl">Net Balance</div>
      <div class="val">₹9,615</div>
      <div class="sub">Cash remaining after withdrawals</div>
    </div>
  </div>
 
  <div class="section-label">Cash Flow</div>
  <div class="grid-2">
    <div class="panel">
      <h3>Monthly Income vs Expense <small>₹ per month, Jan–Jun 2026</small></h3>
      <canvas id="trendChart" height="230"></canvas>
    </div>
    <div class="panel">
      <h3>Where Money Goes <small>Expense by category</small></h3>
      <canvas id="catChart" height="230"></canvas>
    </div>
  </div>
 
  <div class="section-label">Project Performance</div>
  <div class="grid-2">
    <div class="panel">
      <h3>Profit / Loss by Project <small>Income − Expense − Withdrawal</small></h3>
      <canvas id="projChart" height="260"></canvas>
    </div>
    <div class="panel">
      <h3>Project Ledger <small>sorted by profit</small></h3>
      <table>
        <thead><tr><th>Project</th><th class="num">Expense</th><th class="num">Income</th><th class="num">Profit</th></tr></thead>
        <tbody id="projTable"></tbody>
      </table>
    </div>
  </div>
 
  <div class="section-label">Partners &amp; Activity</div>
  <div class="grid-2">
    <div class="panel">
      <h3>Partner Settlement <small>who paid / who holds balance</small></h3>
      <table>
        <thead><tr><th>Partner</th><th class="num">Expense</th><th class="num">Income</th><th class="num">Withdrawn</th><th class="num">Balance</th></tr></thead>
        <tbody id="partnerTable"></tbody>
      </table>
      <h3 style="margin-top:22px;">Top Sub-Projects by Profit <small>work-package level</small></h3>
      <table>
        <thead><tr><th>Sub-Project</th><th class="num">Expense</th><th class="num">Income</th><th class="num">Profit</th></tr></thead>
        <tbody id="subTable"></tbody>
      </table>
    </div>
    <div class="panel">
      <h3>Recent Entries <small>latest 12 logged</small></h3>
      <div id="feed"></div>
    </div>
  </div>
 
  <footer>
    <span>SITE LEDGER · Generated from live cash book data</span>
    <span>Figures in INR (₹) · Withdrawal entries scoped to Sharco partnership account</span>
  </footer>
</div>
 
<script>
Chart.defaults.color = '#cfc8ab';
Chart.defaults.font.family = "'Inter', sans-serif";
Chart.defaults.borderColor = 'rgba(226,163,59,0.12)';
 
const fmt = n => '₹' + Math.round(n).toLocaleString('en-IN');
 
/* ---------- MONTHLY TREND (from transaction log) ---------- */
const months = ['Jan','Feb','Mar','Apr','May','Jun'];
const monthlyExpense = [6500, 72050, 70025, 109974, 202878, 79109];
const monthlyIncome  = [9300, 104500, 98000, 122200, 219500, 136650];
 
new Chart(document.getElementById('trendChart'), {
  type:'line',
  data:{
    labels: months,
    datasets:[
      {label:'Income', data:monthlyIncome, borderColor:'#7a9b66', backgroundColor:'rgba(122,155,102,0.18)', fill:true, tension:.35, pointRadius:3, borderWidth:2.5},
      {label:'Expense', data:monthlyExpense, borderColor:'#c1502e', backgroundColor:'rgba(193,80,46,0.14)', fill:true, tension:.35, pointRadius:3, borderWidth:2.5}
    ]
  },
  options:{
    plugins:{legend:{labels:{boxWidth:12, font:{family:"'IBM Plex Mono'", size:11}}}},
    scales:{
      y:{ticks:{callback:v=>'₹'+(v/1000)+'k', font:{family:"'IBM Plex Mono'", size:10}}, grid:{color:'rgba(226,163,59,0.08)'}},
      x:{grid:{display:false}, ticks:{font:{family:"'IBM Plex Mono'", size:11}}}
    }
  }
});
 
/* ---------- CATEGORY BREAKDOWN ---------- */
const catLabels = ['Material Purchase','Labour','Rent','Design','Travel','Food & Drinks','Utilities','Other','Rental','Bills/Misc/Health'];
const catData =    [264650, 191074, 24000, 20300, 10719, 10592, 9050, 5270, 1220, 1160];
new Chart(document.getElementById('catChart'), {
  type:'bar',
  data:{
    labels: catLabels,
    datasets:[{
      data: catData,
      backgroundColor:'#e2a33b',
      borderRadius:3,
      barThickness:14
    }]
  },
  options:{
    indexAxis:'y',
    plugins:{legend:{display:false}},
    scales:{
      x:{ticks:{callback:v=>'₹'+(v/1000)+'k', font:{family:"'IBM Plex Mono'", size:10}}, grid:{color:'rgba(226,163,59,0.08)'}},
      y:{grid:{display:false}, ticks:{font:{size:11}}}
    }
  }
});
 
/* ---------- PROJECT DATA (authoritative pivot) ---------- */
const projects = [
  {name:"Caliph",      expense:257336.90, income:333150.00, withdrawal:0,       profit:75813.10},
  {name:"Poonoor",     expense:132995.53, income:170000.00, withdrawal:0,       profit:37004.47},
  {name:"MES",         expense:61908.00,  income:96000.00,  withdrawal:0,       profit:34092.00},
  {name:"Ibrahim",     expense:500.00,    income:15000.00,  withdrawal:0,       profit:14500.00},
  {name:"Pharmacy Adivaram", expense:0,   income:10000.00,  withdrawal:0,       profit:10000.00},
  {name:"Ajmal",       expense:7035.00,   income:17000.00,  withdrawal:0,       profit:9965.00},
  {name:"Nihad's House", expense:12400.00, income:16800.00, withdrawal:0,       profit:4400.00},
  {name:"Adhil",       expense:23451.00,  income:26200.00,  withdrawal:0,       profit:2749.00},
  {name:"Ameen",       expense:5500.00,   income:5000.00,   withdrawal:0,       profit:-500.00},
  {name:"Siraj's House", expense:250.00,  income:0,         withdrawal:0,       profit:-250.00},
  {name:"Biriyom's",   expense:535.00,    income:0,         withdrawal:0,       profit:-535.00},
  {name:"Sharco",      expense:38624.00,  income:1000.00,   withdrawal:140000.00, profit:-177624.00},
];
const projSorted = [...projects].sort((a,b)=>b.profit-a.profit);
 
new Chart(document.getElementById('projChart'), {
  type:'bar',
  data:{
    labels: projSorted.map(p=>p.name),
    datasets:[{
      data: projSorted.map(p=>p.profit),
      backgroundColor: projSorted.map(p=>p.profit>=0 ? '#7a9b66' : '#c1502e'),
      borderRadius:3,
      barThickness:16
    }]
  },
  options:{
    indexAxis:'y',
    plugins:{legend:{display:false}, tooltip:{callbacks:{label:c=>fmt(c.raw)}}},
    scales:{
      x:{ticks:{callback:v=>'₹'+(v/1000)+'k', font:{family:"'IBM Plex Mono'", size:10}}, grid:{color:'rgba(226,163,59,0.08)'}},
      y:{grid:{display:false}, ticks:{font:{size:11}}}
    }
  }
});
 
const projTableBody = document.getElementById('projTable');
projSorted.forEach(p=>{
  projTableBody.innerHTML += `<tr>
    <td class="name">${p.name}</td>
    <td class="num">${fmt(p.expense)}</td>
    <td class="num">${fmt(p.income)}</td>
    <td class="num ${p.profit>=0?'pos':'neg'}">${p.profit>=0?'':'−'}${fmt(Math.abs(p.profit))}</td>
  </tr>`;
});
 
/* ---------- PARTNER SETTLEMENT ---------- */
const partners = [
  {name:'Nishad', expense:412626.43, income:529500.00, withdrawn:107000.00, balance:9873.57},
  {name:'Shanid', expense:127909.00, income:160650.00, withdrawn:33000.00, balance:-259.00},
];
const partnerTableBody = document.getElementById('partnerTable');
partners.forEach(p=>{
  partnerTableBody.innerHTML += `<tr>
    <td class="name">${p.name}</td>
    <td class="num">${fmt(p.expense)}</td>
    <td class="num">${fmt(p.income)}</td>
    <td class="num">${fmt(p.withdrawn)}</td>
    <td class="num ${p.balance>=0?'pos':'neg'}">${p.balance>=0?'':'−'}${fmt(Math.abs(p.balance))}</td>
  </tr>`;
});
 
/* ---------- SUB-PROJECTS ---------- */
const subs = [
  {name:'Office (Caliph)', expense:257336.90, income:333150.00, profit:75813.10},
  {name:'Flooring', expense:85359.00, income:122200.00, profit:36841.00},
  {name:'Bedroom (Poonoor)', expense:138495.53, income:175000.00, profit:36504.47},
  {name:'Fitlife', expense:7000.00, income:17000.00, profit:10000.00},
  {name:'Permit', expense:500.00, income:15000.00, profit:14500.00},
  {name:'Toilet', expense:5900.00, income:7500.00, profit:1600.00},
];
const subSorted = [...subs].sort((a,b)=>b.profit-a.profit).slice(0,6);
const subTableBody = document.getElementById('subTable');
subSorted.forEach(s=>{
  subTableBody.innerHTML += `<tr>
    <td class="name">${s.name}</td>
    <td class="num">${fmt(s.expense)}</td>
    <td class="num">${fmt(s.income)}</td>
    <td class="num ${s.profit>=0?'pos':'neg'}">${s.profit>=0?'':'−'}${fmt(Math.abs(s.profit))}</td>
  </tr>`;
});
 
/* ---------- RECENT ACTIVITY ---------- */
const recent = [
  {date:'Jun 18', proj:'Sharco', type:'wd', desc:'Profit share — Nishad', amt:33000},
  {date:'Jun 18', proj:'Sharco', type:'wd', desc:'Profit share — Nishad', amt:15000},
  {date:'Jun 17', proj:'Caliph', type:'exp', desc:'Misc — Wadood', amt:500},
  {date:'Jun 16', proj:'Caliph', type:'exp', desc:'Advance — Jaseem', amt:2500},
  {date:'Jun 16', proj:'Ameen', type:'inc', desc:'Design fee received', amt:5000},
  {date:'Jun 16', proj:'Sharco', type:'exp', desc:'Wifi — utilities', amt:350},
  {date:'Jun 16', proj:'Caliph', type:'inc', desc:'Advance — Shanid', amt:2500},
  {date:'Jun 15', proj:'Sharco', type:'exp', desc:'Adzum', amt:3500},
  {date:'Jun 13', proj:'Caliph', type:'exp', desc:'Chai House', amt:460},
  {date:'Jun 13', proj:'Caliph', type:'inc', desc:'Advance — Shanid', amt:5000},
  {date:'Jun 10', proj:'Caliph', type:'inc', desc:'Advance — Nishad', amt:75000},
  {date:'Jun 10', proj:'Caliph', type:'inc', desc:'Advance — Shanid', amt:20000},
];
const feed = document.getElementById('feed');
const pillLabel = {exp:'EXP', inc:'INC', wd:'WD'};
recent.forEach(r=>{
  feed.innerHTML += `<div class="feed-row">
    <div class="date">${r.date}</div>
    <div>
      <div class="desc">${r.desc}</div>
      <div class="meta"><span class="pill ${r.type}">${pillLabel[r.type]}</span> &nbsp;${r.proj}</div>
    </div>
    <div class="amt ${r.type==='exp'?'neg':'pos'}">${r.type==='exp'?'−':'+'}${fmt(r.amt)}</div>
  </div>`;
});
</script>
</body>
</html>
