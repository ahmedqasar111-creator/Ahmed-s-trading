<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>tradeFlex — Real-time Trading Dashboard</title>
  <meta name="description" content="tradeFlex demo: responsive trading UI with live-sim market data, chart, order entry and book." />  <!-- Chart.js CDN -->  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>  <!-- Google Fonts -->  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&display=swap" rel="stylesheet">  <style>
    :root{
      --bg:#0f1724; /* dark navy */
      --panel:#0b1220; /* panel */
      --muted:#94a3b8;
      --accent:#06b6d4; /* cyan */
      --accent-2:#7c3aed; /* purple */
      --success:#10b981;
      --danger:#ef4444;
      --glass: rgba(255,255,255,0.04);
      --radius:14px;
      font-family: 'Inter', system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
      color-scheme: dark;
    }

    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#061020);color:#e6eef7}

    .app{max-width:1200px;margin:28px auto;padding:20px;gap:18px;display:grid;grid-template-columns: 320px 1fr;grid-auto-rows:min-content}

    /* Left sidebar */
    .sidebar{background:linear-gradient(180deg,var(--panel), rgba(20,28,40,0.6));padding:18px;border-radius:var(--radius);backdrop-filter: blur(8px);box-shadow:0 6px 20px rgba(2,6,23,0.6)}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:44px;height:44px;border-radius:10px;display:grid;place-items:center;background:linear-gradient(135deg,var(--accent),var(--accent-2));font-weight:800}
    .brand h1{font-size:18px;margin:0}
    .brand p{margin:0;font-size:12px;color:var(--muted)}

    .balance{margin-top:18px;padding:12px;background:var(--glass);border-radius:12px}
    .balance .val{font-size:20px;font-weight:700}
    .balance .label{color:var(--muted);font-size:12px}

    .market-list{margin-top:16px}
    .market-item{display:flex;justify-content:space-between;padding:10px;border-radius:10px;cursor:pointer}
    .market-item:hover{background:rgba(255,255,255,0.02)}
    .symbol{display:flex;gap:10px;align-items:center}
    .sym-badge{width:36px;height:36;border-radius:9px;background:linear-gradient(135deg,#0b1220, rgba(255,255,255,0.02));display:grid;place-items:center;font-weight:700}
    .price{font-weight:700}
    .up{color:var(--success)}
    .down{color:var(--danger)}

    .sidebar .section-title{margin-top:18px;color:var(--muted);font-size:12px}

    /* Main area */
    .main{display:flex;flex-direction:column;gap:16px;margin-left:18px}
    .topbar{display:flex;justify-content:space-between;align-items:center;gap:12px}
    .search{flex:1;margin-left:12px}
    .controls{display:flex;gap:8px;align-items:center}

    .card{background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:12px;box-shadow:0 6px 20px rgba(2,6,23,0.25)}

    /* Grid layout for chart + panels */
    .grid{display:grid;grid-template-columns: 1fr 340px;gap:16px}

    /* Chart */
    .chart-wrap{height:360px;padding:12px}
    canvas{width:100% !important;height:100% !important}

    /* Order form */
    .order-panel{display:flex;flex-direction:column;gap:12px}
    .tabs{display:flex;gap:8px}
    .tab{padding:8px 12px;border-radius:8px;cursor:pointer}
    .tab.active{background:linear-gradient(90deg, rgba(255,255,255,0.03), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03)}

    .order-row{display:flex;gap:8px}
    .input, .select{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:10px;border-radius:10px;color:inherit;flex:1}
    .big{font-size:20px;font-weight:700}
    .btn{padding:10px 14px;border-radius:10px;border:0;cursor:pointer;font-weight:700}
    .btn.buy{background:linear-gradient(90deg,var(--accent),#00cfe8);color:#012;}
    .btn.sell{background:linear-gradient(90deg,#ff6b6b,#ff3b3b);}

    /* Order book & trades */
    .book{display:flex;flex-direction:column;gap:10px}
    .book table{width:100%;border-collapse:collapse;font-size:13px}
    .book th, .book td{padding:6px;text-align:right}
    .book th{color:var(--muted);font-weight:600}
    .asks td{background:linear-gradient(180deg,rgba(255,0,0,0.02),transparent)}
    .bids td{background:linear-gradient(180deg,rgba(0,255,0,0.02),transparent)}

    /* footer small widgets */
    .widgets{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
    .widget{padding:12px;border-radius:10px;background:rgba(255,255,255,0.01)}

    /* responsive */
    @media (max-width:980px){
      .app{grid-template-columns:1fr;}
      .grid{grid-template-columns:1fr}
      .sidebar{order:2}
      .main{order:1;margin-left:0}
    }

  </style></head>
<body>  <div class="app">
    <aside class="sidebar card" aria-label="Sidebar">
      <div class="brand">
        <div class="logo">TF</div>
        <div>
          <h1>tradeFlex</h1>
          <p>Fast. Flexible. Intuitive.</p>
        </div>
      </div><div class="balance" role="status" aria-live="polite">
    <div class="label">Account Balance</div>
    <div class="val" id="balance">$25,000.00</div>
    <div style="font-size:12px;color:var(--muted)">Portfolio: <span id="portfolioVal">$25,000.00</span></div>
  </div>

  <div class="section-title">Markets</div>
  <div class="market-list" id="marketList" role="list">
    <!-- items inserted by JS -->
  </div>

  <div class="section-title">Quick actions</div>
  <div style="display:flex;gap:8px;margin-top:8px">
    <button class="btn buy" style="flex:1" id="quickBuy">Buy BTC</button>
    <button class="btn sell" style="flex:1" id="quickSell">Sell BTC</button>
  </div>

  <div class="section-title">Watchlist</div>
  <div style="margin-top:8px;color:var(--muted);font-size:13px">Add your favorite pairs to the watchlist by clicking the ⭐ next to the symbol.</div>
</aside>

<main class="main">
  <div class="topbar card" role="region" aria-label="Topbar">
    <div style="display:flex;align-items:center;gap:12px">
      <div style="font-weight:700">BTC / USD</div>
      <div style="color:var(--muted);font-size:13px">Spot • Taker fee 0.20%</div>
    </div>

    <div class="controls">
      <div style="text-align:right">
        <div style="font-weight:800" id="midPrice">$34,210.50</div>
        <div style="font-size:12px;color:var(--muted)" id="priceChange">+1.24%</div>
      </div>
      <div style="width:1px;height:36px;background:rgba(255,255,255,0.03);margin:0 8px"></div>
      <div style="display:flex;gap:8px">
        <button class="btn" title="Deposit">Deposit</button>
        <button class="btn" title="Settings">⚙</button>
      </div>
    </div>
  </div>

  <div class="grid">
    <section class="card chart-panel">
      <div class="chart-wrap">
        <canvas id="priceChart" aria-label="Price chart"></canvas>
      </div>
      <div style="display:flex;gap:8px;margin-top:8px;align-items:center;justify-content:space-between">
        <div style="display:flex;gap:8px">
          <button class="tab active" data-interval="1">1m</button>
          <button class="tab" data-interval="5">5m</button>
          <button class="tab" data-interval="15">15m</button>
          <button class="tab" data-interval="60">1h</button>
        </div>

        <div style="color:var(--muted);font-size:13px">Simulated live market</div>
      </div>
    </section>

    <aside class="card order-panel" aria-label="Order panel">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div style="font-weight:700">Order Entry</div>
        <div style="color:var(--muted);font-size:13px">Available: <strong id="availableBalance">$25,000</strong></div>
      </div>

      <div class="tabs" role="tablist">
        <div class="tab active" data-type="market" role="tab">Market</div>
        <div class="tab" data-type="limit" role="tab">Limit</div>
        <div class="tab" data-type="stop" role="tab">Stop</div>
      </div>

      <div style="display:flex;gap:8px;align-items:center">
        <input class="input" id="qty" placeholder="Quantity (e.g., 0.01)" />
        <input class="input" id="price" placeholder="Price (limit only)" />
      </div>

      <div class="order-row">
        <button class="btn buy" id="buyBtn">Buy</button>
        <button class="btn sell" id="sellBtn">Sell</button>
        <button class="btn" id="fillSample">Fill Sample</button>
      </div>

      <div class="book">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div style="font-weight:700">Order Book</div>
          <div style="color:var(--muted);font-size:13px">Depth: top 10</div>
        </div>
        <table aria-label="Order book" id="orderBook">
          <thead>
            <tr><th>Price</th><th>Amount</th><th>Total</th></tr>
          </thead>
          <tbody id="bookBody">
            <!-- rows from JS -->
          </tbody>
        </table>
      </div>
    </aside>
  </div>

  <div class="widgets">
    <div class="widget card">
      <div style="font-weight:700">Recent Trades</div>
      <div id="recentTrades" style="margin-top:8px;font-size:13px;color:var(--muted)">
        <!-- recent trades -->
      </div>
    </div>

    <div class="widget card">
      <div style="font-weight:700">Open Orders</div>
      <div id="openOrders" style="margin-top:8px;font-size:13px;color:var(--muted)">No open orders</div>
    </div>

    <div class="widget card">
      <div style="font-weight:700">Performance</div>
      <div style="margin-top:8px;font-size:13px;color:var(--muted)">24h P&L: <span id="pnl">$0.00</span></div>
    </div>
  </div>

</main>

  </div>  <script>
    /***********************
     * tradeFlex — Demo Frontend
     * Single-file HTML/CSS/JS
     * - Simulates live ticks
     * - Interactive chart (Chart.js)
     * - Order entry simulation
     ***********************/

    // --- initial data ---
    const symbols = [
      {sym:'BTC/USD', price:34210.50},
      {sym:'ETH/USD', price:1834.22},
      {sym:'SOL/USD', price:98.31},
      {sym:'AVAX/USD', price:39.44},
      {sym:'LINK/USD', price:15.23}
    ];

    // state
    let activeSymbol = symbols[0];
    let balance = 25000.00;
    let portfolio = 0; // USD amount invested
    const openOrders = [];
    const recentTrades = [];

    // --- helpers ---
    function fmt(n){return typeof n==='number' ? n.toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}) : n}

    // populate market list
    const marketList = document.getElementById('marketList');
    function renderMarketList(){
      marketList.innerHTML = '';
      symbols.forEach((s,idx)=>{
        const div = document.createElement('div');
        div.className='market-item';
        div.setAttribute('role','listitem');
        div.innerHTML = `
          <div class="symbol">
            <div class="sym-badge">${s.sym.split('/')[0]}</div>
            <div>
              <div style="font-weight:700">${s.sym}</div>
              <div style="font-size:12px;color:var(--muted)">Spot</div>
            </div>
          </div>
          <div class="price ${s.price>=activeSymbol.price?'up':'down'}">${fmt(s.price)}</div>
        `;
        div.onclick = ()=>{ setActiveSymbol(idx); };
        marketList.appendChild(div);
      });
    }

    // set active symbol
    function setActiveSymbol(i){
      activeSymbol = symbols[i];
      document.querySelector('.topbar [style*="font-weight:700"]').textContent = activeSymbol.sym;
      updateMidPrice();
      resetChart();
      renderMarketList();
    }

    // price & change UI
    function updateMidPrice(){
      const mid = activeSymbol.price;
      document.getElementById('midPrice').textContent = `$${fmt(mid)}`;
      // compute a simulated 24h change (for demo only)
      const change = ((Math.random()-0.45)*2).toFixed(2);
      const changeEl = document.getElementById('priceChange');
      changeEl.textContent = (change>0?'+':'')+change+'%';
      changeEl.style.color = change>0 ? 'var(--success)' : 'var(--danger)';
    }

    // --- Chart setup ---
    const ctx = document.getElementById('priceChart').getContext('2d');
    const chartData = {labels:[],datasets:[{label:'Price',data:[],tension:0.2,pointRadius:0, borderWidth:2}]};
    const priceChart = new Chart(ctx,{type:'line',data:chartData,options:{responsive:true,plugins:{legend:{display:false}},scales:{x:{display:false},y:{grid:{color:'rgba(255,255,255,0.03)'}}}});

    function resetChart(){
      chartData.labels.length=0;chartData.datasets[0].data.length=0;
      // initialize with small history
      const base = activeSymbol.price;
      for(let i=30;i>0;i--){
        chartData.labels.push('');
        chartData.datasets[0].data.push((base*(1 + (Math.random()-0.5)/200)).toFixed(2));
      }
      priceChart.update();
    }

    // simulate ticks
    function tick(){
      // mutate active symbol slightly
      const s = activeSymbol;
      const vol = (Math.random()-0.48)*0.002; // small noise
      s.price = Number((s.price * (1+vol)).toFixed(2));

      // update UI
      updateMidPrice();
      // push to chart
      chartData.labels.push('');
      chartData.datasets[0].data.push(s.price);
      if(chartData.labels.length>80){chartData.labels.shift();chartData.datasets[0].data.shift();}
      priceChart.update('none');

      // recent trades
      const trade = {price:s.price,qty:(Math.random()*0.5).toFixed(4),side:Math.random()>0.5?'buy':'sell',time:new Date().toLocaleTimeString()};
      recentTrades.unshift(trade);
      if(recentTrades.length>10) recentTrades.pop();
      renderRecentTrades();

      // update order book simulation
      renderOrderBook();
    }

    // render recent trades
    function renderRecentTrades(){
      const el = document.getElementById('recentTrades');
      el.innerHTML='';
      recentTrades.forEach(t=>{
        const d = document.createElement('div');
        d.style.display='flex';d.style.justifyContent='space-between';d.style.marginBottom='6px';
        d.innerHTML = `<div style="font-size:13px;">${t.time} • ${t.side.toUpperCase()}</div><div style="font-weight:700">${fmt(t.price)}</div>`;
        el.appendChild(d);
      });
    }

    // order book - top 10 bids/asks
    function renderOrderBook(){
      const tbody = document.getElementById('bookBody');
      tbody.innerHTML='';
      const mid = activeSymbol.price;
      // asks (higher prices)
      for(let i=5;i>0;i--){
        const price = Number((mid * (1 + 0.0008 * i + Math.random()/10000)).toFixed(2));
        const amt = (Math.random()*2).toFixed(4);
        const total = (price*amt).toFixed(2);
        const tr = document.createElement('tr');
        tr.className='asks';
        tr.innerHTML = `<td style="color:var(--danger)">${fmt(price)}</td><td>${amt}</td><td>${fmt(total)}</td>`;
        tbody.appendChild(tr);
      }
      // bids
      for(let i=1;i<=5;i++){
        const price = Number((mid * (1 - 0.0008 * i + Math.random()/10000)).toFixed(2));
        const amt = (Math.random()*3).toFixed(4);
        const total = (price*amt).toFixed(2);
        const tr = document.createElement('tr');
        tr.className='bids';
        tr.innerHTML = `<td style="color:var(--success)">${fmt(price)}</td><td>${amt}</td><td>${fmt(total)}</td>`;
        tbody.appendChild(tr);
      }
    }

    // order handling
    document.getElementById('buyBtn').addEventListener('click',()=>placeOrder('buy'));
    document.getElementById('sellBtn').addEventListener('click',()=>placeOrder('sell'));
    document.getElementById('fillSample').addEventListener('click',()=>{document.getElementById('qty').value='0.01';document.getElementById('price').value=activeSymbol.price});

    function placeOrder(side){
      const qty = parseFloat(document.getElementById('qty').value);
      const priceInput = parseFloat(document.getElementById('price').value);
      if(!qty || qty<=0){alert('Enter a valid quantity');return}

      // determine order type (market vs limit)
      const activeTab = document.querySelector('.order-panel .tabs .tab.active') || document.querySelector('.order-panel .tab.active');
      const type = activeTab ? activeTab.getAttribute('data-type') || activeTab.getAttribute('data-interval') : 'market';
      const executedPrice = (type==='market' || !priceInput) ? activeSymbol.price : priceInput;

      // simple risk check
      const cost = executedPrice * qty;
      if(side==='buy' && cost>balance){alert('Insufficient balance');return}

      // create order
      const order = {id:Date.now(),symbol:activeSymbol.sym,side,type,qty,price:executedPrice,status:'filled',time:new Date().toLocaleTimeString()};
      openOrders.push(order);

      // execute
      if(side==='buy'){
        balance -= cost;portfolio += cost;document.getElementById('pnl').textContent = '$0.00';
      } else {
        balance += cost;portfolio -= cost; // demo simple
      }
      updateAccountUI();

      recentTrades.unshift({price:executedPrice,qty:qty,side,time:new Date().toLocaleTimeString()});if(recentTrades.length>10)recentTrades.pop();renderRecentTrades();
      renderOpenOrders();
    }

    function renderOpenOrders(){
      const el = document.getElementById('openOrders');
      if(openOrders.length===0){el.textContent='No open orders';return}
      el.innerHTML='';
      openOrders.slice(-10).reverse().forEach(o=>{
        const d = document.createElement('div');
        d.style.display='flex';d.style.justifyContent='space-between';d.style.marginBottom='6px';
        d.innerHTML = `<div>${o.side.toUpperCase()} ${o.qty} ${o.symbol}</div><div style="font-weight:700">${fmt(o.price)}</div>`;
        el.appendChild(d);
      });
    }

    function updateAccountUI(){
      document.getElementById('balance').textContent = `$${fmt(balance)}`;
      document.getElementById('availableBalance').textContent = `$${fmt(balance)}`;
      document.getElementById('portfolioVal').textContent = `$${fmt(portfolio)}`;
    }

    // tab interactions
    document.querySelectorAll('.tabs .tab').forEach(t=>t.addEventListener('click',e=>{
      document.querySelectorAll('.tabs .tab').forEach(x=>x.classList.remove('active'));
      t.classList.add('active');
    }));
    document.querySelectorAll('.chart-panel .tab').forEach(t=>t.addEventListener('click',e=>{
      document.querySelectorAll('.chart-panel .tab').forEach(x=>x.classList.remove('active'));
      t.classList.add('active');
      // interval could change simulation frequency or data length; for demo we ignore
    }));

    // quick actions
    document.getElementById('quickBuy').addEventListener('click',()=>{document.getElementById('qty').value='0.005';document.getElementById('price').value='';placeOrder('buy')});
    document.getElementById('quickSell').addEventListener('click',()=>{document.getElementById('qty').value='0.005';document.getElementById('price').value='';placeOrder('sell')});

    // initialization
    renderMarketList();
    setActiveSymbol(0);
    resetChart();
    renderOrderBook();
    renderRecentTrades();
    updateAccountUI();

    // start ticks
    setInterval(tick, 1200);

    // accessibility: keyboard focus for market items
    marketList.addEventListener('keydown', (e)=>{
      if(e.key==='ArrowDown' || e.key==='ArrowUp'){
        e.preventDefault();
        const items = Array.from(marketList.children);
        const idx = items.findIndex(it=>it===document.activeElement);
        let next = 0;
        if(e.key==='ArrowDown') next = Math.min(items.length-1, idx+1);
        else next = Math.max(0, idx-1);
        items[next].focus();
      }
    });

  </script></body>
</html>

      
