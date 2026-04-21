[index.html](https://github.com/user-attachments/files/26915641/index.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>113年休閒農業區產業分析儀表板</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;700;900&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<style>
  :root {
    --ink: #1a1a0e;
    --cream: #f5f0e8;
    --sage: #6b8c5a;
    --sage-light: #8aad75;
    --sage-dark: #3d5e2e;
    --earth: #9c6b3c;
    --rice: #e8dfc8;
    --mist: #d4e0d0;
    --gold: #c8a84b;
    --sky: #7aaebf;
    --red-accent: #c04a2e;
  }
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: 'Noto Serif TC', serif; background: var(--cream); color: var(--ink); overflow-x: hidden; }
  body::before {
    content:''; position:fixed; inset:0;
    background-image:url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='%23000' fill-opacity='0.015'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/svg%3E");
    pointer-events:none; z-index:0;
  }
  header { background:var(--sage-dark); color:var(--cream); position:relative; overflow:hidden; z-index:10; }
  header::after { content:''; position:absolute; bottom:0;left:0;right:0; height:3px; background:linear-gradient(90deg,var(--gold),var(--sage-light),var(--gold)); }
  .header-inner { display:flex; align-items:flex-end; gap:40px; padding:32px 48px 24px; position:relative; }
  .header-deco { position:absolute;right:40px;bottom:0;opacity:0.07;font-size:160px;line-height:1;user-select:none;pointer-events:none; }
  .header-tag { font-family:'DM Mono',monospace; font-size:10px; letter-spacing:3px; text-transform:uppercase; color:var(--sage-light); margin-bottom:8px; }
  h1 { font-size:clamp(22px,3.2vw,42px); font-weight:900; line-height:1.1; letter-spacing:-1px; }
  .header-sub { font-size:12px; color:rgba(245,240,232,0.5); margin-top:5px; }
  .header-stats { display:flex; gap:36px; margin-left:auto; flex-shrink:0; }
  .hstat { text-align:right; }
  .hstat-num { font-size:28px;font-weight:900;color:var(--gold);letter-spacing:-1px;font-family:'DM Mono',monospace;display:block; }
  .hstat-label { font-size:9px;color:rgba(245,240,232,0.45);letter-spacing:1px;margin-top:2px; }
  .nav-tabs { background:var(--ink);display:flex;overflow-x:auto;scrollbar-width:none;position:sticky;top:0;z-index:100; }
  .nav-tabs::-webkit-scrollbar{display:none;}
  .tab-btn { padding:12px 24px;font-family:'Noto Serif TC',serif;font-size:13px;color:rgba(245,240,232,0.4);background:none;border:none;cursor:pointer;white-space:nowrap;border-bottom:2px solid transparent;transition:all 0.2s; }
  .tab-btn:hover{color:var(--cream);}
  .tab-btn.active{color:var(--gold);border-bottom-color:var(--gold);}
  .tab-panel{display:none;}
  .tab-panel.active{display:block;}

  /* ===== MAP TAB ===== */
  .map-layout { display:grid; grid-template-columns:320px 1fr; height:calc(100vh - 118px); min-height:580px; }
  .map-sidebar { background:white; overflow-y:auto; border-right:1px solid var(--mist); display:flex; flex-direction:column; }
  .sidebar-header { background:var(--sage-dark); color:var(--cream); padding:18px 20px 14px; flex-shrink:0; }
  .sidebar-header h2 { font-size:14px; font-weight:900; }
  .sidebar-header p { font-size:10px; color:rgba(245,240,232,0.55); margin-top:3px; }
  .api-status { display:flex;align-items:center;gap:6px;padding:8px 20px;background:var(--rice);border-bottom:1px solid var(--mist);flex-shrink:0; }
  .status-dot { width:7px;height:7px;border-radius:50%;background:#bbb;flex-shrink:0;transition:background 0.3s; }
  .status-dot.loading{background:var(--gold);animation:pulse 1s infinite;}
  .status-dot.ok{background:var(--sage);}
  .status-dot.err{background:var(--red-accent);}
  @keyframes pulse{0%,100%{opacity:1}50%{opacity:0.3}}
  .status-text { font-family:'DM Mono',monospace; font-size:9px; color:#888; }
  .county-list{flex:1;overflow-y:auto;}
  .county-item { padding:12px 20px; border-bottom:1px solid var(--rice); cursor:pointer; transition:all 0.15s; display:flex; align-items:center; gap:10px; }
  .county-item:hover,.county-item.active { background:var(--cream); border-left:3px solid var(--sage); padding-left:17px; }
  .county-item.active{background:var(--mist);}
  .ci-color{width:11px;height:11px;border-radius:2px;flex-shrink:0;}
  .ci-name{font-size:13px;font-weight:700;}
  .ci-stats{font-size:9px;color:#888;margin-top:2px;font-family:'DM Mono',monospace;}
  .ci-rev{font-family:'DM Mono',monospace;font-size:11px;font-weight:700;color:var(--sage-dark);margin-left:auto;flex-shrink:0;}
  .sidebar-legend{padding:14px 20px;border-top:1px solid var(--mist);flex-shrink:0;background:var(--cream);}
  .legend-title{font-size:9px;letter-spacing:1.5px;color:#999;text-transform:uppercase;margin-bottom:6px;font-family:'DM Mono',monospace;}
  .legend-scale{display:flex;height:8px;border-radius:2px;overflow:hidden;margin-bottom:3px;}
  .legend-labels{display:flex;justify-content:space-between;font-size:8px;font-family:'DM Mono',monospace;color:#aaa;}
  #taiwanMap{width:100%;height:100%;background:#e8f4f0;}
  .map-info-box { position:absolute;top:14px;right:14px;background:white;border-radius:3px;padding:14px 16px;box-shadow:0 4px 20px rgba(0,0,0,0.15);z-index:500;min-width:190px;border-top:3px solid var(--sage);pointer-events:none;transition:opacity 0.2s;opacity:0; }
  .map-info-box.visible{opacity:1;}
  .info-county{font-size:15px;font-weight:900;margin-bottom:8px;}
  .info-row{display:flex;justify-content:space-between;font-size:11px;margin-bottom:3px;}
  .info-key{color:#888;}
  .info-val{font-family:'DM Mono',monospace;font-weight:700;color:var(--sage-dark);}

  /* ===== MAIN CONTENT ===== */
  main{padding:28px 44px;}
  .stat-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;margin-bottom:22px;}
  .stat-card{background:white;padding:20px;border-radius:2px;box-shadow:0 1px 4px rgba(0,0,0,0.06);border-left:4px solid var(--sage);}
  .stat-card.earth{border-left-color:var(--earth);}
  .stat-card.gold{border-left-color:var(--gold);}
  .stat-card.sky{border-left-color:var(--sky);}
  .stat-icon{font-size:24px;margin-bottom:7px;display:block;}
  .stat-val{font-size:28px;font-weight:900;font-family:'DM Mono',monospace;letter-spacing:-1px;line-height:1;}
  .stat-val span{font-size:13px;font-weight:400;color:#888;margin-left:2px;}
  .stat-label{font-size:10px;color:#888;margin-top:4px;}
  .grid-2{display:grid;grid-template-columns:1fr 1fr;gap:20px;}
  .grid-2-1{display:grid;grid-template-columns:2fr 1fr;gap:20px;}
  .grid-1-2{display:grid;grid-template-columns:1fr 2fr;gap:20px;}
  .card{background:white;border-radius:2px;padding:22px;box-shadow:0 1px 4px rgba(0,0,0,0.06);position:relative;overflow:hidden;animation:fadeIn 0.4s ease both;}
  .card::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:var(--sage);}
  .card.earth::before{background:var(--earth);}
  .card.gold::before{background:var(--gold);}
  .card.sky::before{background:var(--sky);}
  .card-title{font-size:9px;letter-spacing:2px;text-transform:uppercase;color:var(--sage-dark);margin-bottom:16px;font-weight:700;font-family:'DM Mono',monospace;}
  .bar-chart{display:flex;flex-direction:column;gap:9px;}
  .bar-row{display:flex;align-items:center;gap:9px;}
  .bar-label{width:68px;font-size:10px;text-align:right;color:var(--ink);flex-shrink:0;}
  .bar-wrap{flex:1;background:var(--rice);height:18px;border-radius:1px;overflow:hidden;}
  .bar-fill{height:100%;border-radius:1px;transition:width 1s cubic-bezier(0.16,1,0.3,1);}
  .bar-val{width:76px;font-size:9px;font-family:'DM Mono',monospace;color:#777;flex-shrink:0;}
  .donut-wrap{display:flex;align-items:center;gap:20px;}
  .donut-legend{display:flex;flex-direction:column;gap:6px;flex:1;}
  .legend-item{display:flex;align-items:center;gap:6px;font-size:10px;}
  .legend-dot{width:8px;height:8px;border-radius:50%;flex-shrink:0;}
  .legend-val{font-family:'DM Mono',monospace;font-size:9px;color:#888;margin-left:auto;}
  .monthly-bars{display:flex;align-items:flex-end;gap:7px;height:120px;padding-top:14px;}
  .month-bar-wrap{flex:1;display:flex;flex-direction:column;align-items:center;gap:4px;height:100%;justify-content:flex-end;}
  .month-bar{width:100%;border-radius:2px 2px 0 0;transition:height 1s cubic-bezier(0.16,1,0.3,1);}
  .month-label{font-size:9px;color:#888;}
  .month-val{font-family:'DM Mono',monospace;font-size:8px;color:var(--sage-dark);}
  .zone-list{display:flex;flex-direction:column;gap:7px;}
  .zone-item{display:grid;grid-template-columns:22px 1fr auto;align-items:center;gap:8px;padding:9px 12px;background:var(--cream);border-radius:2px;border-left:3px solid var(--sage);transition:transform 0.15s;}
  .zone-item:hover{transform:translateX(3px);}
  .zone-rank{font-family:'DM Mono',monospace;font-size:9px;color:var(--sage);font-weight:700;}
  .zone-name{font-size:12px;font-weight:700;}
  .zone-county{font-size:9px;color:#888;margin-top:1px;}
  .zone-rev{font-family:'DM Mono',monospace;font-size:11px;font-weight:700;color:var(--sage-dark);}
  .zone-vis{font-size:8px;color:#999;margin-top:1px;}
  .zone-progress{grid-column:1/-1;height:2px;background:var(--rice);border-radius:1px;overflow:hidden;}
  .zone-progress-fill{height:100%;background:linear-gradient(90deg,var(--sage),var(--sage-light));transition:width 1s cubic-bezier(0.16,1,0.3,1);}
  .rev-comp{display:flex;flex-direction:column;gap:5px;}
  .rev-row{display:flex;align-items:center;gap:7px;}
  .rev-name{font-size:10px;width:86px;color:var(--ink);flex-shrink:0;}
  .rev-bar-wrap{flex:1;background:var(--rice);height:12px;border-radius:1px;overflow:hidden;}
  .rev-bar-fill{height:100%;border-radius:1px;}
  .rev-pct{font-family:'DM Mono',monospace;font-size:8px;color:#888;width:28px;text-align:right;flex-shrink:0;}
  @keyframes fadeIn{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
  .leaflet-popup-content-wrapper{border-radius:3px;box-shadow:0 4px 20px rgba(0,0,0,0.2);border-top:3px solid var(--sage);}
  @media(max-width:900px){
    .header-inner{flex-direction:column;align-items:flex-start;}
    .header-stats{margin-left:0;flex-wrap:wrap;}
    .map-layout{grid-template-columns:1fr;height:auto;}
    .map-sidebar{height:260px;}
    #taiwanMap{height:480px;}
    main{padding:16px;}
    .stat-grid{grid-template-columns:1fr 1fr;}
    .grid-2,.grid-2-1,.grid-1-2{grid-template-columns:1fr;}
  }
</style>
</head>
<body>

<header>
  <div class="header-inner">
    <div>
      <div class="header-tag">民國113年 · 農業部農村發展及水土保持署</div>
      <h1>休閒農業區<br>產業分析儀表板</h1>
      <div class="header-sub">串接官方圖資管理系統 dofs.ardswc.gov.tw · 全國業者調查資料</div>
    </div>
    <div class="header-stats">
      <div class="hstat"><span class="hstat-num">2,182</span><div class="hstat-label">業者總數</div></div>
      <div class="hstat"><span class="hstat-num">104</span><div class="hstat-label">休閒農業區</div></div>
      <div class="hstat"><span class="hstat-num">443</span><div class="hstat-label">億元年營業額</div></div>
    </div>
    <div class="header-deco">🌾</div>
  </div>
</header>

<nav class="nav-tabs">
  <button class="tab-btn active" onclick="switchTab('map',this)">🗾 台灣地圖</button>
  <button class="tab-btn" onclick="switchTab('overview',this)">📊 總覽分析</button>
  <button class="tab-btn" onclick="switchTab('zones',this)">🌿 農業區排行</button>
  <button class="tab-btn" onclick="switchTab('visitors',this)">👥 遊客分析</button>
  <button class="tab-btn" onclick="switchTab('business',this)">💼 業態分佈</button>
</nav>

<!-- MAP TAB -->
<div class="tab-panel active" id="tab-map">
  <div class="map-layout">
    <div class="map-sidebar">
      <div class="sidebar-header">
        <h2>🏡 縣市別統計面板</h2>
        <p>資料來源：農業部水保署開放 API + 業者調查</p>
      </div>
      <div class="api-status" id="apiStatus">
        <div class="status-dot loading" id="statusDot"></div>
        <span class="status-text" id="statusText">正在串接 API 資料…</span>
      </div>
      <div class="county-list" id="countyList"></div>
      <div class="sidebar-legend">
        <div class="legend-title">年度營業額熱度</div>
        <div class="legend-scale" id="legendScale"></div>
        <div class="legend-labels"><span>低</span><span>中</span><span>高</span></div>
      </div>
    </div>
    <div style="position:relative;">
      <div id="taiwanMap"></div>
      <div class="map-info-box" id="mapInfoBox">
        <div class="info-county" id="infoCounty">滑鼠移至縣市</div>
        <div class="info-row"><span class="info-key">劃定農業區數</span><span class="info-val" id="infoDesg">—</span></div>
        <div class="info-row"><span class="info-key">登記農場數</span><span class="info-val" id="infoFarm">—</span></div>
        <div class="info-row"><span class="info-key">調查業者數</span><span class="info-val" id="infoBiz">—</span></div>
        <div class="info-row"><span class="info-key">上半年遊客</span><span class="info-val" id="infoVis">—</span></div>
        <div class="info-row"><span class="info-key">年度營業額</span><span class="info-val" id="infoRev">—</span></div>
      </div>
    </div>
  </div>
</div>

<!-- OVERVIEW TAB -->
<div class="tab-panel" id="tab-overview">
<main>
  <div class="stat-grid">
    <div class="stat-card"><span class="stat-icon">🏡</span><div class="stat-val">2,182<span>家</span></div><div class="stat-label">全國業者總數</div></div>
    <div class="stat-card earth"><span class="stat-icon">🌾</span><div class="stat-val">443<span>億</span></div><div class="stat-label">年度總營業額</div></div>
    <div class="stat-card gold"><span class="stat-icon">👥</span><div class="stat-val">1,206<span>萬</span></div><div class="stat-label">上半年遊客人次</div></div>
    <div class="stat-card sky"><span class="stat-icon">💰</span><div class="stat-val">369<span>元/人</span></div><div class="stat-label">平均遊客消費</div></div>
  </div>
  <div class="grid-2-1" style="margin-bottom:20px;">
    <div class="card"><div class="card-title">縣市業者數 × 年度營業額</div><div class="bar-chart" id="countyBarChart"></div></div>
    <div class="card gold"><div class="card-title">上半年月份遊客趨勢</div><div class="monthly-bars" id="monthlyBars"></div></div>
  </div>
  <div class="grid-2">
    <div class="card earth"><div class="card-title">業態分佈</div><div class="donut-wrap"><svg width="140" height="140" viewBox="0 0 140 140" id="donutSvg"></svg><div class="donut-legend" id="donutLegend"></div></div></div>
    <div class="card sky"><div class="card-title">收益結構（112年）</div><div class="rev-comp" id="revComp"></div></div>
  </div>
</main>
</div>

<!-- ZONES TAB -->
<div class="tab-panel" id="tab-zones">
<main>
  <div class="grid-1-2">
    <div class="card"><div class="card-title">TOP 15 農業區排行</div><div class="zone-list" id="zoneList"></div></div>
    <div class="card earth"><div class="card-title">遊客量 vs 營業額 氣泡圖</div><div style="position:relative;height:380px;"><canvas id="bubbleCanvas" width="580" height="380"></canvas></div></div>
  </div>
</main>
</div>

<!-- VISITORS TAB -->
<div class="tab-panel" id="tab-visitors">
<main>
  <div class="card" style="margin-bottom:20px;"><div class="card-title">113年1–6月 各月遊客人次（全國加總）</div><div id="monthlyBarLarge" class="monthly-bars" style="height:160px;"></div></div>
  <div class="grid-2">
    <div class="card sky"><div class="card-title">縣市遊客人次排行</div><div class="bar-chart" id="countyVisitorBar"></div></div>
    <div class="card"><div class="card-title">縣市年度營業額排行</div><div class="bar-chart" id="countyRevBar"></div></div>
  </div>
</main>
</div>

<!-- BUSINESS TAB -->
<div class="tab-panel" id="tab-business">
<main>
  <div class="card" style="margin-bottom:20px;"><div class="card-title">業態子類別佔比</div><div class="bar-chart" id="catBarChart"></div></div>
  <div class="grid-2">
    <div class="card earth"><div class="card-title">業態說明</div><div style="display:flex;flex-direction:column;gap:8px;margin-top:4px;" id="catDesc"></div></div>
    <div class="card gold"><div class="card-title">官方 API — 歷年劃定區數統計</div><div class="bar-chart" id="apiYearChart" style="margin-top:4px;"></div></div>
  </div>
</main>
</div>

<script>
const COUNTY_DATA = {"南投縣":{"revenue":30563.2,"visitors":976387,"count":158},"台中市":{"revenue":16199.0,"visitors":639830,"count":122},"台北市":{"revenue":7517.2,"visitors":260946,"count":85},"台南市":{"revenue":119186.4,"visitors":2261972,"count":239},"台東縣":{"revenue":26123.9,"visitors":732677,"count":155},"嘉義縣":{"revenue":43002.8,"visitors":393872,"count":83},"宜蘭縣":{"revenue":80084.6,"visitors":1949912,"count":383},"屏東縣":{"revenue":3519.2,"visitors":81367,"count":56},"彰化縣":{"revenue":8013.6,"visitors":264298,"count":78},"新北市":{"revenue":3267.2,"visitors":108803,"count":16},"新竹縣":{"revenue":4906.7,"visitors":261199,"count":63},"桃園市":{"revenue":10278.9,"visitors":370321,"count":109},"花蓮縣":{"revenue":21643.4,"visitors":963147,"count":106},"苗栗縣":{"revenue":44230.1,"visitors":2148030,"count":208},"雲林縣":{"revenue":18314.7,"visitors":481437,"count":107},"高雄市":{"revenue":4717.6,"visitors":130324,"count":61}};
const TOP_ZONES=[{"county":"台南市","zone":"溪南","count":65,"revenue":70605.7,"visitors":788418},{"county":"台南市","zone":"梅嶺","count":41,"revenue":32769.8,"visitors":966216},{"county":"宜蘭縣","zone":"枕頭山","count":34,"revenue":23157.5,"visitors":480545},{"county":"苗栗縣","zone":"壢西坪","count":24,"revenue":13056.5,"visitors":153940},{"county":"雲林縣","zone":"金湖","count":31,"revenue":11504.1,"visitors":354014},{"county":"宜蘭縣","zone":"林業","count":2,"revenue":10967.9,"visitors":397374},{"county":"台東縣","zone":"初鹿","count":16,"revenue":10131.7,"visitors":357380},{"county":"南投縣","zone":"糯米橋","count":31,"revenue":7254.2,"visitors":94575},{"county":"南投縣","zone":"自強愛國","count":14,"revenue":7072.3,"visitors":428464},{"county":"苗栗縣","zone":"黃金小鎮","count":42,"revenue":6270.7,"visitors":223432},{"county":"宜蘭縣","zone":"梅花湖","count":42,"revenue":6036.1,"visitors":257810},{"county":"苗栗縣","zone":"南江","count":25,"revenue":5809.0,"visitors":58342},{"county":"台北市","zone":"貓空","count":51,"revenue":5567.2,"visitors":186881},{"county":"花蓮縣","zone":"馬太鞍","count":11,"revenue":5337.2,"visitors":271407},{"county":"宜蘭縣","zone":"新港澳","count":7,"revenue":5949.1,"visitors":87675}];
const MONTHLY={"1月":2024707,"2月":2394548,"3月":2059794,"4月":1870836,"5月":1821191,"6月":1887833};
const CAT_COUNTS={"民宿":423,"風味餐飲":435,"展售中心":36,"農事體驗":264,"生態導覽":87,"農特產品":553,"農村文化":120,"其他":41};
const COLORS=['#6b8c5a','#9c6b3c','#c8a84b','#7aaebf','#c04a2e','#7d5a8c','#3d8c7a','#8c5a5a'];

// County name normalization (API uses 臺, common uses 台)
function normName(s){ return s ? s.replace(/臺/g,'台') : ''; }

let apiData={}, geoLayer=null, leafMap=null;

function switchTab(id,btn){
  document.querySelectorAll('.tab-panel').forEach(p=>p.classList.remove('active'));
  document.querySelectorAll('.tab-btn').forEach(b=>b.classList.remove('active'));
  document.getElementById('tab-'+id).classList.add('active');
  btn.classList.add('active');
  if(id==='map')setTimeout(()=>{if(leafMap)leafMap.invalidateSize();},60);
  if(id==='zones')setTimeout(drawBubble,120);
  setTimeout(animateBars,120);
}

function revenueColor(val,maxVal){
  const t=Math.pow(Math.max(val,0)/maxVal,0.55);
  const r=Math.round(245-t*(245-61)), g=Math.round(240-t*(240-94)), b=Math.round(232-t*(232-46));
  return `rgb(${r},${g},${b})`;
}

function buildLegend(){
  const el=document.getElementById('legendScale');
  let h='';
  for(let i=0;i<20;i++){const t=i/19;const r=Math.round(245-t*(245-61)),g=Math.round(240-t*(240-94)),b=Math.round(232-t*(232-46));h+=`<div style="flex:1;background:rgb(${r},${g},${b})"></div>`;}
  el.innerHTML=h;
}

async function fetchApiData(){
  const dot=document.getElementById('statusDot'), txt=document.getElementById('statusText');
  try{
    const res=await fetch('https://data.ardswc.gov.tw/api/SWCBStatusReport/14',{method:'POST',headers:{'Content-Type':'application/json'},body:'{}'});
    if(!res.ok)throw new Error('HTTP '+res.status);
    const json=await res.json();
    json.forEach(row=>{
      const n=normName(row.County);
      if(!apiData[n]||row.StatisticsYear>apiData[n].year)
        apiData[n]={year:row.StatisticsYear,desg:row.Desg_Cnt||0,farm:row.Farm_Reg||0};
    });
    dot.className='status-dot ok';
    txt.textContent=`已串接 API（${json.length}筆）· 水保署開放資料`;
    buildCountySidebar();
    if(geoLayer)updateMapColors();
    buildApiYearChart(json);
  }catch(e){
    dot.className='status-dot err';
    txt.textContent='API 跨網域限制（CORS），改用本地資料';
    buildCountySidebar();
  }
}

function initMap(){
  leafMap=L.map('taiwanMap',{center:[23.8,121.0],zoom:7,zoomControl:true,attributionControl:true});
  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png',{attribution:'© OpenStreetMap © CARTO',subdomains:'abcd',maxZoom:18}).addTo(leafMap);

  // Try to load Taiwan county GeoJSON
  const urls=[
    'https://raw.githubusercontent.com/g0v/twgeojson/master/json/twCounty2010.geo.json',
    'https://cdn.jsdelivr.net/gh/g0v/twgeojson@master/json/twCounty2010.geo.json'
  ];
  function tryLoad(i){
    if(i>=urls.length){showFallbackMap();return;}
    fetch(urls[i]).then(r=>r.ok?r.json():Promise.reject()).then(geojson=>{
      geoLayer=L.geoJSON(geojson,{style:styleFeature,onEachFeature:onEachFeature}).addTo(leafMap);
      leafMap.fitBounds(geoLayer.getBounds(),{padding:[20,20]});
      updateMapColors();
    }).catch(()=>tryLoad(i+1));
  }
  tryLoad(0);
}

function showFallbackMap(){
  // Add a note overlay if GeoJSON fails
  const div=L.control({position:'bottomleft'});
  div.onAdd=()=>{
    const d=L.DomUtil.create('div');
    d.style.cssText='background:white;padding:8px 12px;border-radius:3px;font-size:11px;color:#888;border-top:2px solid #c04a2e;';
    d.textContent='縣市邊界資料載入中，請確認網路連線';
    return d;
  };
  div.addTo(leafMap);
}

function getLocalData(mapName){
  const n=normName(mapName);
  for(const [k,v] of Object.entries(COUNTY_DATA)){
    if(normName(k)===n||n.includes(normName(k).slice(0,-1))||normName(k).includes(n.slice(0,-1)))
      return{key:k,...v};
  }
  return null;
}

function styleFeature(f){
  const name=f.properties.name||f.properties.COUNTYNAME||'';
  const maxRev=Math.max(...Object.values(COUNTY_DATA).map(d=>d.revenue));
  const cd=getLocalData(name);
  return{fillColor:revenueColor(cd?cd.revenue:0,maxRev),weight:1.5,opacity:1,color:'#6b8c5a',fillOpacity:0.83};
}

function onEachFeature(f,layer){
  const name=f.properties.name||f.properties.COUNTYNAME||'未知';
  const cd=getLocalData(name);
  const apiInfo=apiData[normName(name)]||{};

  layer.on({
    mouseover:e=>{
      e.target.setStyle({weight:2.5,color:'#3d5e2e',fillOpacity:0.95});
      const box=document.getElementById('mapInfoBox');
      box.classList.add('visible');
      document.getElementById('infoCounty').textContent=name;
      document.getElementById('infoDesg').textContent=apiInfo.desg!==undefined?apiInfo.desg+'區':'查詢中';
      document.getElementById('infoFarm').textContent=apiInfo.farm!==undefined?apiInfo.farm+'家':'查詢中';
      document.getElementById('infoBiz').textContent=cd?cd.count+' 家':'—';
      document.getElementById('infoVis').textContent=cd?(cd.visitors/10000).toFixed(1)+' 萬人':'—';
      document.getElementById('infoRev').textContent=cd?(cd.revenue/10000).toFixed(1)+' 億元':'—';
      if(cd)document.querySelectorAll('.county-item').forEach(el=>el.classList.toggle('active',el.dataset.key===cd.key));
    },
    mouseout:e=>{
      if(geoLayer)geoLayer.resetStyle(e.target);
      document.getElementById('mapInfoBox').classList.remove('visible');
      document.querySelectorAll('.county-item').forEach(el=>el.classList.remove('active'));
    },
    click:e=>leafMap.fitBounds(e.target.getBounds(),{padding:[40,40]})
  });

  const pop=cd
    ?`<div style="min-width:155px;font-family:'Noto Serif TC',serif;">
        <div style="font-weight:900;font-size:14px;margin-bottom:7px;">${name}</div>
        <div style="font-size:10px;color:#666;margin-bottom:3px;">業者數：<b>${cd.count} 家</b></div>
        <div style="font-size:10px;color:#666;margin-bottom:3px;">上半年遊客：<b>${(cd.visitors/10000).toFixed(1)} 萬人</b></div>
        <div style="font-size:10px;color:#666;">年營業額：<b style="color:#3d5e2e;">${(cd.revenue/10000).toFixed(2)} 億元</b></div>
        ${apiInfo.desg!==undefined?`<div style="font-size:10px;color:#666;margin-top:3px;">劃定農業區：<b>${apiInfo.desg}區</b></div>`:''}
      </div>`
    :`<div>${name}</div>`;
  layer.bindPopup(pop);
}

function updateMapColors(){
  if(!geoLayer)return;
  const maxRev=Math.max(...Object.values(COUNTY_DATA).map(d=>d.revenue));
  geoLayer.eachLayer(layer=>{
    const name=layer.feature.properties.name||layer.feature.properties.COUNTYNAME||'';
    const cd=getLocalData(name);
    layer.setStyle({fillColor:revenueColor(cd?cd.revenue:0,maxRev),weight:1.5,color:'#6b8c5a',fillOpacity:0.83});
  });
}

function buildCountySidebar(){
  const el=document.getElementById('countyList');
  const maxRev=Math.max(...Object.values(COUNTY_DATA).map(d=>d.revenue));
  const sorted=Object.entries(COUNTY_DATA).sort((a,b)=>b[1].revenue-a[1].revenue);
  el.innerHTML=sorted.map(([county,d])=>{
    const ai=apiData[normName(county)]||{};
    const desgTxt=ai.desg!==undefined?` · ${ai.desg}個農業區`:'';
    return`<div class="county-item" data-key="${county}" onclick="focusCounty('${county}')">
      <div class="ci-color" style="background:${revenueColor(d.revenue,maxRev)}"></div>
      <div><div class="ci-name">${county}</div><div class="ci-stats">${d.count}家業者${desgTxt}</div></div>
      <div class="ci-rev">${(d.revenue/10000).toFixed(1)}億</div>
    </div>`;
  }).join('');
}

function focusCounty(county){
  if(!geoLayer||!leafMap)return;
  geoLayer.eachLayer(layer=>{
    const name=layer.feature.properties.name||layer.feature.properties.COUNTYNAME||'';
    const cd=getLocalData(name);
    if(cd&&cd.key===county){leafMap.fitBounds(layer.getBounds(),{padding:[40,40]});layer.openPopup();}
  });
}

// Charts
function buildCountyBar(){
  const el=document.getElementById('countyBarChart');
  const sorted=Object.entries(COUNTY_DATA).sort((a,b)=>b[1].revenue-a[1].revenue).slice(0,12);
  const maxRev=sorted[0][1].revenue;
  el.innerHTML=sorted.map(([c,d],i)=>`<div class="bar-row"><div class="bar-label">${c}</div><div class="bar-wrap"><div class="bar-fill" style="width:0%;background:${COLORS[i%COLORS.length]}" data-target="${(d.revenue/maxRev*100).toFixed(1)}"></div></div><div class="bar-val">${(d.revenue/10000).toFixed(1)}億</div></div>`).join('');
}

function buildMonthlyBars(id){
  const el=document.getElementById(id);
  const max=Math.max(...Object.values(MONTHLY));
  el.innerHTML=Object.entries(MONTHLY).map(([m,v],i)=>`<div class="month-bar-wrap"><div class="month-val">${(v/10000).toFixed(0)}萬</div><div class="month-bar" style="height:0%;background:${i===1?'#c8a84b':i%2===0?'#6b8c5a':'#8aad75'}" data-target="${(v/max*100).toFixed(1)}%"></div><div class="month-label">${m}</div></div>`).join('');
}

function buildDonut(){
  const svg=document.getElementById('donutSvg'),legend=document.getElementById('donutLegend');
  const entries=Object.entries(CAT_COUNTS),total=entries.reduce((s,[,v])=>s+v,0);
  const cx=70,cy=70,r=58,ri=34;let angle=-Math.PI/2,paths='';
  entries.forEach(([label,val],i)=>{
    const frac=val/total,a2=angle+frac*2*Math.PI;
    const x1=cx+r*Math.cos(angle),y1=cy+r*Math.sin(angle),x2=cx+r*Math.cos(a2),y2=cy+r*Math.sin(a2);
    const xi1=cx+ri*Math.cos(angle),yi1=cy+ri*Math.sin(angle),xi2=cx+ri*Math.cos(a2),yi2=cy+ri*Math.sin(a2);
    const lg=frac>0.5?1:0;
    paths+=`<path d="M${xi1},${yi1} L${x1},${y1} A${r},${r} 0 ${lg} 1 ${x2},${y2} L${xi2},${yi2} A${ri},${ri} 0 ${lg} 0 ${xi1},${yi1} Z" fill="${COLORS[i]}" opacity="0.9"/>`;
    angle=a2;
  });
  svg.innerHTML=paths+`<text x="${cx}" y="${cy+4}" text-anchor="middle" font-size="12" font-weight="bold" fill="#1a1a0e" font-family="Noto Serif TC">${total}家</text>`;
  legend.innerHTML=entries.map(([l,v],i)=>`<div class="legend-item"><div class="legend-dot" style="background:${COLORS[i]}"></div><span>${l}</span><span class="legend-val">${v}</span></div>`).join('');
}

function buildRevComp(){
  const el=document.getElementById('revComp');
  const data=[['餐飲',32,'#6b8c5a'],['住宿',28,'#9c6b3c'],['農特產伴手禮',18,'#c8a84b'],['體驗活動',12,'#7aaebf'],['門票',6,'#c04a2e'],['生鮮農產品',4,'#7d5a8c']];
  el.innerHTML=data.map(([n,p,c])=>`<div class="rev-row"><div class="rev-name">${n}</div><div class="rev-bar-wrap"><div class="rev-bar-fill" style="width:0%;background:${c}" data-target="${p}%"></div></div><div class="rev-pct">${p}%</div></div>`).join('');
}

function buildZoneList(){
  const el=document.getElementById('zoneList'),maxRev=TOP_ZONES[0].revenue;
  el.innerHTML=TOP_ZONES.map((z,i)=>`<div class="zone-item"><div class="zone-rank">${String(i+1).padStart(2,'0')}</div><div><div class="zone-name">${z.zone}</div><div class="zone-county">${z.county}·${z.count}家</div></div><div><div class="zone-rev">${(z.revenue/10000).toFixed(2)}億</div><div class="zone-vis">${(z.visitors/10000).toFixed(1)}萬人</div></div><div class="zone-progress"><div class="zone-progress-fill" style="width:0%" data-target="${(z.revenue/maxRev*100).toFixed(1)}%"></div></div></div>`).join('');
}

function buildCatBar(){
  const el=document.getElementById('catBarChart');
  const sorted=Object.entries(CAT_COUNTS).sort((a,b)=>b[1]-a[1]),max=sorted[0][1];
  el.innerHTML=sorted.map(([cat,count],i)=>`<div class="bar-row"><div class="bar-label">${cat}</div><div class="bar-wrap"><div class="bar-fill" style="width:0%;background:${COLORS[i%COLORS.length]}" data-target="${(count/max*100).toFixed(1)}"></div></div><div class="bar-val">${count}家(${(count/1959*100).toFixed(1)}%)</div></div>`).join('');
}

function buildCatDesc(){
  const el=document.getElementById('catDesc');
  const d=[['6.1','民宿','🏘️','住宿服務，融入農村生活'],['6.2','風味餐飲','🍽️','在地特色風味餐飲'],['6.4','農事體驗','🌱','農耕體驗，親身參與'],['6.5','生態導覽','🦋','自然生態解說教育'],['6.6','農特產品','🛒','農特產品及伴手禮'],['6.7','農村文化','🎭','農村文化傳統工藝']];
  el.innerHTML=d.map(([c,n,i,desc])=>`<div style="display:flex;align-items:flex-start;gap:9px;padding:8px;background:var(--cream);border-radius:2px;"><span style="font-size:16px;flex-shrink:0;">${i}</span><div><div style="font-weight:700;font-size:11px;">${c} ${n}</div><div style="font-size:9px;color:#777;margin-top:1px;">${desc}</div></div></div>`).join('');
}

function buildCountyBars(){
  const sv=Object.entries(COUNTY_DATA).sort((a,b)=>b[1].visitors-a[1].visitors).slice(0,12);
  const sr=Object.entries(COUNTY_DATA).sort((a,b)=>b[1].revenue-a[1].revenue).slice(0,12);
  const mv=sv[0][1].visitors,mr=sr[0][1].revenue;
  document.getElementById('countyVisitorBar').innerHTML=sv.map(([c,d],i)=>`<div class="bar-row"><div class="bar-label">${c}</div><div class="bar-wrap"><div class="bar-fill" style="width:0%;background:${COLORS[i%COLORS.length]}" data-target="${(d.visitors/mv*100).toFixed(1)}"></div></div><div class="bar-val">${(d.visitors/10000).toFixed(0)}萬人</div></div>`).join('');
  document.getElementById('countyRevBar').innerHTML=sr.map(([c,d],i)=>`<div class="bar-row"><div class="bar-label">${c}</div><div class="bar-wrap"><div class="bar-fill" style="width:0%;background:${COLORS[i%COLORS.length]}" data-target="${(d.revenue/mr*100).toFixed(1)}"></div></div><div class="bar-val">${(d.revenue/10000).toFixed(1)}億</div></div>`).join('');
}

function buildApiYearChart(json){
  const el=document.getElementById('apiYearChart');if(!el)return;
  const byYear={};
  json.forEach(r=>{if(!byYear[r.StatisticsYear])byYear[r.StatisticsYear]={desg:0,farm:0};byYear[r.StatisticsYear].desg+=(r.Desg_Cnt||0);byYear[r.StatisticsYear].farm+=(r.Farm_Reg||0);});
  const years=Object.keys(byYear).sort().slice(-8),maxD=Math.max(...years.map(y=>byYear[y].desg));
  el.innerHTML=years.map(y=>`<div class="bar-row"><div class="bar-label" style="width:32px;">${y}</div><div class="bar-wrap"><div class="bar-fill" style="width:0%;background:#6b8c5a" data-target="${(byYear[y].desg/maxD*100).toFixed(1)}"></div></div><div class="bar-val">${byYear[y].desg}區/${byYear[y].farm}場</div></div>`).join('');
  setTimeout(animateBars,100);
}

function drawBubble(){
  const canvas=document.getElementById('bubbleCanvas');if(!canvas)return;
  const ctx=canvas.getContext('2d'),W=canvas.width,H=canvas.height,pad=55;
  ctx.clearRect(0,0,W,H);ctx.fillStyle='#fafaf7';ctx.fillRect(0,0,W,H);
  const maxVis=Math.max(...TOP_ZONES.map(z=>z.visitors)),maxRev=Math.max(...TOP_ZONES.map(z=>z.revenue)),maxCount=Math.max(...TOP_ZONES.map(z=>z.count));
  ctx.strokeStyle='#e8dfc8';ctx.lineWidth=1;
  for(let i=0;i<=4;i++){const x=pad+(W-pad*2)*i/4,y=pad+(H-pad*2)*i/4;ctx.beginPath();ctx.moveTo(x,pad);ctx.lineTo(x,H-pad);ctx.stroke();ctx.beginPath();ctx.moveTo(pad,y);ctx.lineTo(W-pad,y);ctx.stroke();}
  ctx.fillStyle='#999';ctx.font='10px DM Mono';ctx.textAlign='center';
  ctx.fillText('遊客人數 →',W/2,H-7);
  ctx.save();ctx.translate(13,H/2);ctx.rotate(-Math.PI/2);ctx.fillText('↑ 營業額',0,0);ctx.restore();
  TOP_ZONES.forEach((z,i)=>{
    const x=pad+(z.visitors/maxVis)*(W-pad*2),y=H-pad-(z.revenue/maxRev)*(H-pad*2),r=4+(z.count/maxCount)*20;
    ctx.beginPath();ctx.arc(x,y,r,0,Math.PI*2);ctx.fillStyle=COLORS[i%COLORS.length]+'bb';ctx.fill();
    ctx.strokeStyle=COLORS[i%COLORS.length];ctx.lineWidth=1.5;ctx.stroke();
    ctx.fillStyle='#1a1a0e';ctx.font='bold 9px Noto Serif TC';ctx.textAlign='center';ctx.fillText(z.zone,x,y-r-3);
  });
}

function animateBars(){
  document.querySelectorAll('.bar-fill[data-target],.rev-bar-fill[data-target],.zone-progress-fill[data-target]').forEach(el=>{
    const t=el.getAttribute('data-target');
    setTimeout(()=>{el.style.width=t.includes('%')?t:t+'%';},80);
  });
  document.querySelectorAll('.month-bar[data-target]').forEach(el=>{
    setTimeout(()=>{el.style.height=el.getAttribute('data-target');},80);
  });
}

function init(){
  buildLegend();
  buildCountySidebar();
  buildCountyBar();
  buildMonthlyBars('monthlyBars');
  buildMonthlyBars('monthlyBarLarge');
  buildDonut();
  buildRevComp();
  buildZoneList();
  buildCatBar();
  buildCatDesc();
  buildCountyBars();
  setTimeout(animateBars,150);
  initMap();
  fetchApiData();
}

window.addEventListener('load',init);
</script>
</body>
</html>
