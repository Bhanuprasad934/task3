<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>World Explorer — Plain HTML/CSS/JS</title>
  <meta name="description" content="Search countries, filter, sort, visualize data and try a couple of public APIs (dog image, random quote)." />

  <!-- Chart.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

  <style>
    :root{
      --bg:#f6f8fb; --card:#ffffff; --muted:#6b7280; --accent:#2563eb; --glass: rgba(255,255,255,0.6);
      --radius:14px; --shadow: 0 6px 18px rgba(16,24,40,0.06);
      color-scheme: light;
    }
    [data-theme="dark"]{
      --bg:#0b1220; --card:#0f1724; --muted:#9aa4b2; --accent:#4f46e5; --glass: rgba(255,255,255,0.03);
      color-scheme: dark;
    }

    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial; background:linear-gradient(180deg,var(--bg), #e9eef8 120%); color:var(--fg,#0b1220)}

    .container{max-width:1200px;margin:28px auto;padding:18px}
    header{display:flex;align-items:center;gap:12px;margin-bottom:14px}
    .brand{display:flex;flex-direction:column}
    h1{margin:0;font-size:20px}
    p.lead{margin:0;color:var(--muted);font-size:13px}

    .top-controls{display:grid;grid-template-columns:1fr 360px;gap:14px;margin-bottom:14px}

    .card{background:var(--card);border-radius:var(--radius);box-shadow:var(--shadow);padding:14px}
    .controls{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
    .search{flex:1;display:flex;align-items:center;gap:8px}
    .search input{flex:1;padding:10px 12px;border-radius:10px;border:1px solid rgba(15,23,42,0.06);font-size:14px}

    select,input[type=number]{padding:8px;border-radius:8px;border:1px solid rgba(15,23,42,0.06)}
    button{background:transparent;border:1px solid rgba(15,23,42,0.06);padding:8px 10px;border-radius:10px;cursor:pointer}
    button.primary{background:linear-gradient(90deg,var(--accent),#06b6d4);color:white;border:0}

    .two-col{display:grid;grid-template-columns:1fr 380px;gap:14px}
    .results{display:grid;gap:10px}
    .grid-cards{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:12px}
    .country{padding:12px;border-radius:12px;border:1px solid rgba(15,23,42,0.04);display:flex;flex-direction:column;gap:8px;cursor:pointer;transition:transform .15s}
    .country:hover{transform:translateY(-4px)}
    .country .meta{display:flex;justify-content:space-between;font-size:12px;color:var(--muted)}
    .compact .country{padding:8px;font-size:13px}

    .controls-row{display:flex;gap:8px;align-items:center}

    .right-panel{display:flex;flex-direction:column;gap:10px}
    .img-box{width:100%;height:180px;background:linear-gradient(180deg, #eef2ff, #fff);border-radius:10px;display:grid;place-items:center;overflow:hidden}
    .img-box img{width:100%;height:100%;object-fit:cover}

    .table-wrap{overflow:auto;border-radius:12px}
    table{width:100%;border-collapse:collapse}
    th,td{padding:10px;text-align:left;border-bottom:1px solid rgba(15,23,42,0.03)}

    footer{margin-top:18px;text-align:center;font-size:12px;color:var(--muted)}

    /* small screens */
    @media (max-width:900px){
      .top-controls{grid-template-columns:1fr}
      .two-col{grid-template-columns:1fr}
      .right-panel{order:2}
    }

    /* tiny helpers */
    .muted{color:var(--muted)}
    .flex{display:flex;gap:8px;align-items:center}
    .spacer{flex:1}

    /* theme toggle */
    .theme-toggle{display:inline-flex;align-items:center;gap:8px;padding:6px;border-radius:999px}

    /* loading */
    .loading{opacity:0.6;filter:grayscale(0.2);}

  </style>
</head>
<body>
  <div class="container" id="app">
    <header>
      <div style="width:44px;height:44px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:grid;place-items:center;color:white;font-weight:700">WE</div>
      <div class="brand">
        <h1>World Explorer — Plain HTML/CSS/JS</h1>
        <p class="lead">Search countries, filter, sort, visualize and call a couple public APIs.</p>
      </div>
      <div class="spacer"></div>
      <div class="theme-toggle card" role="region" aria-label="Theme">
        <label style="font-size:13px;margin-right:6px">Theme</label>
        <button id="themeBtn">Toggle</button>
      </div>
    </header>

    <div class="top-controls">
      <div class="card">
        <div class="controls">
          <div class="search">
            <input id="q" placeholder="Search country by name..." aria-label="Search countries">
            <button id="clearQ">Clear</button>
          </div>

          <div class="controls-row">
            <select id="regionSel" aria-label="Region">
              <option value="all">All regions</option>
            </select>
            <select id="sortSel" aria-label="Sort">
              <option value="population_desc">Population ↓</option>
              <option value="population_asc">Population ↑</option>
              <option value="area_desc">Area ↓</option>
              <option value="area_asc">Area ↑</option>
              <option value="name_asc">Name A→Z</option>
              <option value="name_desc">Name Z→A</option>
            </select>

            <label class="muted">Top N</label>
            <input id="topN" type="number" min="3" max="50" value="10" style="width:70px">

            <button id="viewToggle">Table</button>
            <button id="compactToggle">Compact</button>
          </div>
        </div>
      </div>

      <div>
        <div class="card">
          <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px">
            <div style="font-weight:600">Quick Insights</div>
            <div style="font-size:12px;color:var(--muted)">Top populations</div>
          </div>
          <canvas id="barChart" width="400" height="220"></canvas>
        </div>
      </div>
    </div>

    <div class="two-col">
      <div>
        <div class="card results" id="resultsCard">
          <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px">
            <div style="font-weight:600">Results <span id="count" class="muted">—</span></div>
            <div class="muted">Click a card for details</div>
          </div>

          <div id="grid" class="grid-cards"></div>
          <div id="tableWrap" class="table-wrap" style="display:none;margin-top:8px">
            <table id="table"><thead><tr><th>Name</th><th>Region</th><th>Capital</th><th>Population</th><th>Area km²</th></tr></thead><tbody></tbody></table>
          </div>

        </div>
      </div>

      <aside class="right-panel">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
            <div style="font-weight:600">API Playground</div>
            <div class="flex">
              <button id="dogBtn">Dog</button>
              <button id="quoteBtn">Quote</button>
            </div>
          </div>

          <div class="img-box" id="dogBox">Click Dog to fetch image</div>
          <div style="margin-top:8px;font-size:13px;color:var(--muted)" id="quoteBox">Click Quote to fetch random quote</div>
        </div>

        <div class="card">
          <div style="font-weight:600;margin-bottom:8px">Selected Country</div>
          <div id="detail">No country selected</div>
        </div>
      </aside>
    </div>

    <footer class="muted">Built with plain HTML/CSS/JS. Public APIs used: restcountries.com, dog.ceo, api.quotable.io</footer>
  </div>

<script>
// Small helper utilities
const $ = (sel, ctx=document) => ctx.querySelector(sel);
const $$ = (sel, ctx=document) => Array.from(ctx.querySelectorAll(sel));

// DOM refs
const qEl = $('#q');
const clearQ = $('#clearQ');
const regionSel = $('#regionSel');
const sortSel = $('#sortSel');
const topN = $('#topN');
const viewToggle = $('#viewToggle');
const compactToggle = $('#compactToggle');
const gridEl = $('#grid');
const tableWrap = $('#tableWrap');
const tableBody = $('#table tbody');
const countEl = $('#count');
const barCanvas = $('#barChart');
const dogBtn = $('#dogBtn');
const quoteBtn = $('#quoteBtn');
const dogBox = $('#dogBox');
const quoteBox = $('#quoteBox');
const detail = $('#detail');
const themeBtn = $('#themeBtn');

let countries = [];
let filtered = [];
let compact = false;
let view = 'cards';
let chart = null;
let debounceTimer = null;

// Theme persistence
const root = document.documentElement;
function applyTheme(dark){
  if(dark) root.setAttribute('data-theme','dark'); else root.removeAttribute('data-theme');
  localStorage.setItem('we_theme', dark ? 'dark' : 'light');
}
applyTheme(localStorage.getItem('we_theme') === 'dark');
themeBtn.addEventListener('click', ()=> applyTheme(root.getAttribute('data-theme')==='dark' ? false : true));

// Fetch countries
async function loadCountries(){
  try{
    gridEl.innerHTML = '<div class="loading" style="padding:40px;text-align:center">Loading countries…</div>';
    const res = await fetch('https://restcountries.com/v3.1/all');
    if(!res.ok) throw new Error('countries fetch failed');
    countries = await res.json();
    countries.sort((a,b)=> (a.name.common||'').localeCompare(b.name.common||''));
    populateRegions();
    doFilter();
  }catch(e){
    gridEl.innerHTML = '<div style="padding:40px;text-align:center;color:#b91c1c">Could not load countries.</div>';
    console.error(e);
  }
}

function populateRegions(){
  const regions = new Set(countries.map(c=>c.region).filter(Boolean));
  regions.forEach(r=>{
    const opt = document.createElement('option'); opt.value = r; opt.textContent = r; regionSel.appendChild(opt);
  });
}

// Debounced filter trigger
qEl.addEventListener('input', ()=> debounce(doFilter, 350));
clearQ.addEventListener('click', ()=> { qEl.value=''; doFilter(); });
regionSel.addEventListener('change', doFilter);
sortSel.addEventListener('change', doFilter);
topN.addEventListener('change', ()=> { updateCharts(); });

viewToggle.addEventListener('click', ()=>{ view = view==='cards' ? 'table' : 'cards'; renderView(); });
compactToggle.addEventListener('click', ()=>{ compact = !compact; document.getElementById('resultsCard').classList.toggle('compact', compact); renderResults(); });

// Filtering & sorting
function doFilter(){
  const q = qEl.value.trim().toLowerCase();
  const region = regionSel.value;
  filtered = countries.filter(c=>{
    if(region !== 'all' && c.region !== region) return false;
    if(q && ! ( (c.name.common||'').toLowerCase().includes(q) || (c.altSpellings||[]).join(' ').toLowerCase().includes(q) ) ) return false;
    return true;
  });
  // sorting
  const s = sortSel.value;
  filtered.sort((a,b)=>{
    if(s.startsWith('population')){
      const dir = s.endsWith('asc')?1:-1; return dir*((a.population||0)-(b.population||0));
    }
    if(s.startsWith('area')){const dir = s.endsWith('asc')?1:-1; return dir*((a.area||0)-(b.area||0));}
    if(s.startsWith('name')){const dir = s.endsWith('asc')?1:-1; return dir*((a.name.common||'').localeCompare(b.name.common||''));}
    return 0;
  });
  renderResults();
  updateCharts();
}

function debounce(fn,ms=300){ clearTimeout(debounceTimer); debounceTimer = setTimeout(fn,ms); }

// Render
function renderResults(){
  countEl.textContent = filtered.length;
  if(view === 'cards'){
    tableWrap.style.display='none';
    gridEl.style.display='grid';
    gridEl.innerHTML='';
    filtered.forEach(c=>{
      const el = document.createElement('div'); el.className='country';
      el.innerHTML = `
        <div style="display:flex;gap:10px;align-items:center">
          <img src="${c.flags?.png||''}" alt="flag" style="width:44px;height:28px;object-fit:cover;border-radius:6px;border:1px solid rgba(0,0,0,0.04)">
          <div style="font-weight:600">${escapeHtml(c.name.common)}</div>
        </div>
        <div class="meta">
          <div>${c.region||'—'}</div>
          <div>${(c.capital||['—'])[0]}</div>
        </div>
        <div class="meta">
          <div>Pop: ${numberWithCommas(c.population||0)}</div>
          <div>Area: ${numberWithCommas(Math.round(c.area||0))} km²</div>
        </div>
      `;
      el.addEventListener('click', ()=> showDetail(c));
      gridEl.appendChild(el);
    });
  } else {
    gridEl.style.display='none';
    tableWrap.style.display='block';
    tableBody.innerHTML='';
    filtered.forEach(c=>{
      const tr = document.createElement('tr');
      tr.innerHTML = <td>${escapeHtml(c.name.common)}</td><td>${c.region||'—'}</td><td>${(c.capital||['—'])[0]}</td><td>${numberWithCommas(c.population||0)}</td><td>${numberWithCommas(Math.round(c.area||0))}</td>;
      tr.addEventListener('click', ()=> showDetail(c));
      tableBody.appendChild(tr);
    });
  }
}

function renderView(){
  if(view==='cards'){ viewToggle.textContent='Table'; tableWrap.style.display='none'; gridEl.style.display='grid'; }
  else { viewToggle.textContent='Cards'; gridEl.style.display='none'; tableWrap.style.display='block'; }
  renderResults();
}

function showDetail(c){
  detail.innerHTML = `
    <div style="font-weight:700;margin-bottom:6px">${escapeHtml(c.name.common)} <span style='font-size:12px;color:var(--muted)'>${c.cca3}</span></div>
    <div class='muted' style='font-size:13px'>Official: ${escapeHtml(c.name.official||'—')}</div>
    <div style='margin-top:8px;font-size:13px'>Region: ${c.region||'—'} <br> Capital: ${(c.capital||['—'])[0]} <br> Population: ${numberWithCommas(c.population||0)} <br> Area: ${numberWithCommas(Math.round(c.area||0))} km²</div>
    <div style='margin-top:8px;font-size:13px;color:var(--muted)'>Languages: ${(c.languages? Object.values(c.languages).join(', '):'—')}</div>
    <div style='margin-top:8px'><img src='${c.flags?.png||''}' alt='flag' style='width:100%;border-radius:8px;margin-top:8px'></div>
  `;
  detail.scrollIntoView({behavior:'smooth'});
}

// Charts
function updateCharts(){
  const n = Math.max(3, Math.min(50, Number(topN.value||10)));
  const arr = filtered.slice().sort((a,b)=>(b.population||0)-(a.population||0)).slice(0,n);
  const labels = arr.map(i=>i.name.common);
  const data = arr.map(i=>i.population||0);
  if(chart){ chart.data.labels = labels; chart.data.datasets[0].data = data; chart.update(); return; }
  chart = new Chart(barCanvas.getContext('2d'),{
    type:'bar',
    data:{labels, datasets:[{label:'Population', data, borderRadius:6}]},
    options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false}}}
  });
}

// API playground
dogBtn.addEventListener('click', async ()=>{
  dogBox.innerHTML = 'Loading…';
  try{
    const r = await fetch('https://dog.ceo/api/breeds/image/random');
    const j = await r.json();
    dogBox.innerHTML = <img src='${j.message}' alt='dog'>;
  }catch(e){ dogBox.innerHTML = 'Could not fetch dog.' }
});

quoteBtn.addEventListener('click', async ()=>{
  quoteBox.textContent = 'Loading…';
  try{
    const r = await fetch('https://api.quotable.io/random');
    const j = await r.json();
    quoteBox.innerHTML = <blockquote style="margin:0;font-style:italic">“${escapeHtml(j.content)}”</blockquote><div style="margin-top:6px;text-align:right">— ${escapeHtml(j.author)}</div>;
  }catch(e){ quoteBox.textContent='Could not fetch quote.' }
});

// small helpers
function numberWithCommas(x){ return String(x).replace(/\B(?=(\d{3})+(?!\d))/g, ','); }
function escapeHtml(s){ if(!s) return ''; return String(s).replace(/[&<>"']/g, c=>({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;',"":'&#96;' })[c]); }

// init
loadCountries();

// initial view state
renderView();

</script>
</body>
</html>
