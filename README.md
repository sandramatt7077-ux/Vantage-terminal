<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Vantage — Synthetic Indices Terminal</title>
<style>
  :root{
    --bg: #0a0d10;
    --panel: #10151a;
    --panel-2: #151b22;
    --line: #212a33;
    --line-soft: #1a2129;
    --text: #d8e0e6;
    --text-dim: #7c8a96;
    --text-faint: #4a5761;
    --up: #3ddc84;
    --up-dim: #1f6e46;
    --down: #ff5c6c;
    --down-dim: #7a2b33;
    --amber: #e8a23d;
    --signal-strong: #3ddc84;
    --signal-weak: #e8a23d;
    --signal-none: #4a5761;
    --mono: 'IBM Plex Mono', 'SF Mono', Consolas, monospace;
    --sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  html, body {
    height: 100%;
    background: var(--bg);
    color: var(--text);
    font-family: var(--sans);
    font-size: 14px;
    -webkit-font-smoothing: antialiased;
  }

  @media (prefers-reduced-motion: reduce) {
    * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
  }

  ::selection { background: var(--up-dim); color: #fff; }

  a { color: inherit; }

  button {
    font-family: var(--sans);
    cursor: pointer;
    border: none;
    background: none;
    color: inherit;
  }
  button:focus-visible, input:focus-visible, select:focus-visible {
    outline: 2px solid var(--up);
    outline-offset: 1px;
  }

  input, select {
    font-family: var(--mono);
    background: var(--panel-2);
    border: 1px solid var(--line);
    color: var(--text);
    padding: 8px 10px;
    border-radius: 3px;
    font-size: 13px;
    width: 100%;
  }
  input::placeholder { color: var(--text-faint); }

  .mono { font-family: var(--mono); }

  /* ===== Layout shell ===== */
  #app { display: flex; flex-direction: column; height: 100vh; }

  header.topbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    height: 52px;
    border-bottom: 1px solid var(--line);
    background: var(--panel);
    flex-shrink: 0;
  }

  .brand {
    display: flex;
    align-items: center;
    gap: 10px;
    font-weight: 600;
    letter-spacing: -0.01em;
    font-size: 15px;
  }
  .brand .mark {
    width: 18px; height: 18px;
    position: relative;
  }
  .brand .mark svg { width: 100%; height: 100%; display: block; }

  .conn-status {
    display: flex;
    align-items: center;
    gap: 14px;
    font-family: var(--mono);
    font-size: 12px;
    color: var(--text-dim);
  }
  .dot {
    width: 7px; height: 7px;
    border-radius: 50%;
    background: var(--text-faint);
    display: inline-block;
    margin-right: 6px;
    box-shadow: 0 0 0 0 rgba(61,220,132,0.5);
  }
  .dot.live { background: var(--up); animation: pulse-dot 2s infinite; }
  .dot.err { background: var(--down); }
  @keyframes pulse-dot {
    0% { box-shadow: 0 0 0 0 rgba(61,220,132,0.45); }
    70% { box-shadow: 0 0 0 6px rgba(61,220,132,0); }
    100% { box-shadow: 0 0 0 0 rgba(61,220,132,0); }
  }

  .balance-pill {
    font-family: var(--mono);
    font-size: 13px;
    padding: 5px 12px;
    border: 1px solid var(--line);
    border-radius: 3px;
    background: var(--panel-2);
  }
  .balance-pill .amt { color: var(--text); font-weight: 600; }

  .topbar-actions { display: flex; align-items: center; gap: 10px; }
  .btn {
    font-family: var(--mono);
    font-size: 12px;
    padding: 7px 14px;
    border: 1px solid var(--line);
    border-radius: 3px;
    color: var(--text-dim);
    transition: border-color .15s, color .15s;
  }
  .btn:hover { border-color: var(--text-dim); color: var(--text); }
  .btn.primary { border-color: var(--up-dim); color: var(--up); }
  .btn.primary:hover { border-color: var(--up); background: rgba(61,220,132,0.07); }

  /* ===== Main grid ===== */
  main.grid {
    flex: 1;
    display: grid;
    grid-template-columns: 240px 1fr 320px;
    grid-template-rows: 1fr auto;
    overflow: hidden;
  }

  /* Market list */
  aside.markets {
    grid-row: 1 / 3;
    border-right: 1px solid var(--line);
    background: var(--panel);
    overflow-y: auto;
    display: flex;
    flex-direction: column;
  }
  .markets-head {
    padding: 14px 16px 8px;
    font-size: 11px;
    letter-spacing: 0.03em;
    color: var(--text-faint);
    font-family: var(--mono);
    position: sticky;
    top: 0;
    background: var(--panel);
  }
  .market-item {
    padding: 10px 16px;
    display: flex;
    flex-direction: column;
    gap: 3px;
    cursor: pointer;
    border-left: 2px solid transparent;
    transition: background .1s;
  }
  .market-item:hover { background: var(--panel-2); }
  .market-item.active {
    background: var(--panel-2);
    border-left-color: var(--up);
  }
  .market-item .name { font-size: 13px; font-weight: 500; }
  .market-item .row2 {
    display: flex;
    justify-content: space-between;
    font-family: var(--mono);
    font-size: 12px;
    color: var(--text-dim);
  }
  .market-item .row2 .chg.up { color: var(--up); }
  .market-item .row2 .chg.down { color: var(--down); }

  /* Chart / center panel */
  section.chart-panel {
    grid-column: 2;
    grid-row: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    border-right: 1px solid var(--line);
  }
  .chart-head {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    padding: 14px 18px 10px;
    border-bottom: 1px solid var(--line-soft);
  }
  .chart-head .sym {
    font-size: 18px;
    font-weight: 600;
    letter-spacing: -0.01em;
  }
  .chart-head .price {
    font-family: var(--mono);
    font-size: 22px;
    font-weight: 600;
    margin-left: 14px;
  }
  .chart-head .price.up { color: var(--up); }
  .chart-head .price.down { color: var(--down); }
  .chart-head .meta {
    font-family: var(--mono);
    font-size: 12px;
    color: var(--text-dim);
    display: flex;
    gap: 16px;
  }

  #chartCanvas { width: 100%; flex: 1; display: block; }

  .chart-footer {
    display: flex;
    gap: 6px;
    padding: 8px 18px;
    border-top: 1px solid var(--line-soft);
    font-family: var(--mono);
    font-size: 11px;
  }
  .tf-btn {
    padding: 4px 10px;
    border-radius: 3px;
    color: var(--text-faint);
    border: 1px solid transparent;
  }
  .tf-btn:hover { color: var(--text-dim); }
  .tf-btn.active {
    color: var(--text);
    border-color: var(--line);
    background: var(--panel-2);
  }

  /* Analysis panel (bottom center) */
  section.analysis-panel {
    grid-column: 2;
    grid-row: 2;
    border-right: 1px solid var(--line);
    border-top: 1px solid var(--line);
    background: var(--panel);
    padding: 14px 18px 16px;
    max-height: 230px;
    overflow-y: auto;
  }
  .analysis-head {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
  }
  .analysis-head h2 {
    font-size: 12px;
    letter-spacing: 0.03em;
    color: var(--text-faint);
    font-weight: 500;
    font-family: var(--mono);
  }
  .sample-note {
    font-size: 11px;
    color: var(--text-faint);
    font-family: var(--mono);
  }

  .signal-row {
    display: flex;
    gap: 10px;
    margin-bottom: 12px;
  }
  .signal-card {
    flex: 1;
    background: var(--panel-2);
    border: 1px solid var(--line);
    border-radius: 4px;
    padding: 10px 12px;
    position: relative;
    overflow: hidden;
  }
  .signal-card.lead {
    border-color: var(--line);
  }
  .signal-card .label {
    font-family: var(--mono);
    font-size: 10px;
    color: var(--text-faint);
    letter-spacing: 0.02em;
    margin-bottom: 4px;
  }
  .signal-card .value {
    font-size: 15px;
    font-weight: 600;
  }
  .signal-card .value.up { color: var(--up); }
  .signal-card .value.down { color: var(--down); }
  .signal-card .sub {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--text-dim);
    margin-top: 2px;
  }

  .conf-bar-track {
    height: 4px;
    background: var(--line-soft);
    border-radius: 2px;
    margin-top: 8px;
    overflow: hidden;
  }
  .conf-bar-fill {
    height: 100%;
    border-radius: 2px;
    transition: width .3s ease;
  }

  .indicators-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 8px;
  }
  .ind-cell {
    background: var(--panel-2);
    border: 1px solid var(--line-soft);
    border-radius: 3px;
    padding: 7px 10px;
  }
  .ind-cell .k {
    font-family: var(--mono);
    font-size: 10px;
    color: var(--text-faint);
  }
  .ind-cell .v {
    font-family: var(--mono);
    font-size: 13px;
    margin-top: 2px;
  }
  .ind-cell .v.bull { color: var(--up); }
  .ind-cell .v.bear { color: var(--down); }
  .ind-cell .v.neutral { color: var(--text-dim); }

  .disclaimer-strip {
    margin-top: 12px;
    padding: 8px 10px;
    background: rgba(232,162,61,0.06);
    border: 1px solid rgba(232,162,61,0.25);
    border-radius: 3px;
    font-size: 11px;
    color: var(--amber);
    line-height: 1.5;
  }

  /* Trade ticket (right) */
  aside.ticket {
    grid-column: 3;
    grid-row: 1 / 3;
    background: var(--panel);
    overflow-y: auto;
    display: flex;
    flex-direction: column;
  }

  .ticket-section {
    padding: 16px 18px;
    border-bottom: 1px solid var(--line-soft);
  }
  .ticket-section h3 {
    font-family: var(--mono);
    font-size: 11px;
    letter-spacing: 0.03em;
    color: var(--text-faint);
    margin-bottom: 10px;
    font-weight: 500;
  }

  .field { margin-bottom: 10px; }
  .field label {
    display: block;
    font-size: 11px;
    color: var(--text-dim);
    margin-bottom: 5px;
  }

  .type-toggle {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 6px;
    margin-bottom: 10px;
  }
  .type-btn {
    padding: 10px;
    border: 1px solid var(--line);
    border-radius: 3px;
    font-family: var(--mono);
    font-size: 13px;
    font-weight: 600;
    text-align: center;
    color: var(--text-dim);
    transition: all .12s;
  }
  .type-btn.rise:hover, .type-btn.rise.active { border-color: var(--up); color: var(--up); background: rgba(61,220,132,0.06); }
  .type-btn.fall:hover, .type-btn.fall.active { border-color: var(--down); color: var(--down); background: rgba(255,92,108,0.06); }

  .contract-select {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  .duration-row {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 6px;
  }

  .stake-quick {
    display: flex;
    gap: 6px;
    margin-top: 6px;
  }
  .stake-quick button {
    flex: 1;
    padding: 5px;
    border: 1px solid var(--line-soft);
    border-radius: 3px;
    font-family: var(--mono);
    font-size: 11px;
    color: var(--text-faint);
  }
  .stake-quick button:hover { border-color: var(--text-dim); color: var(--text-dim); }

  .payout-preview {
    background: var(--panel-2);
    border: 1px solid var(--line);
    border-radius: 4px;
    padding: 12px;
    margin: 4px 0 12px;
  }
  .payout-row {
    display: flex;
    justify-content: space-between;
    font-family: var(--mono);
    font-size: 12px;
    padding: 3px 0;
    color: var(--text-dim);
  }
  .payout-row .v { color: var(--text); }
  .payout-row.hero .v { color: var(--up); font-size: 15px; font-weight: 600; }

  .confirm-btn {
    width: 100%;
    padding: 13px;
    border-radius: 4px;
    font-family: var(--mono);
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 0.02em;
    border: 1px solid var(--up-dim);
    color: #06120b;
    background: var(--up);
    transition: filter .12s;
  }
  .confirm-btn:hover { filter: brightness(1.08); }
  .confirm-btn:disabled {
    background: var(--panel-2);
    color: var(--text-faint);
    border-color: var(--line);
    cursor: not-allowed;
  }
  .confirm-btn.fall-mode { background: var(--down); border-color: var(--down-dim); }

  .risk-note {
    font-size: 10.5px;
    color: var(--text-faint);
    margin-top: 8px;
    line-height: 1.5;
  }

  /* Open positions + log */
  .positions-list { display: flex; flex-direction: column; gap: 6px; }
  .position-item {
    background: var(--panel-2);
    border: 1px solid var(--line-soft);
    border-radius: 3px;
    padding: 9px 10px;
    font-family: var(--mono);
    font-size: 11.5px;
  }
  .position-item .top-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 4px;
  }
  .position-item .sym { color: var(--text); font-weight: 600; }
  .position-item .pl { font-weight: 600; }
  .position-item .pl.pos { color: var(--up); }
  .position-item .pl.neg { color: var(--down); }
  .position-item .bottom-row {
    display: flex;
    justify-content: space-between;
    color: var(--text-faint);
    font-size: 10.5px;
  }
  .empty-state {
    color: var(--text-faint);
    font-size: 12px;
    text-align: center;
    padding: 20px 10px;
    font-family: var(--mono);
  }

  /* Setup / connect modal */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(4,6,8,0.75);
    backdrop-filter: blur(3px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
  }
  .modal {
    width: 440px;
    max-width: 92vw;
    background: var(--panel);
    border: 1px solid var(--line);
    border-radius: 6px;
    padding: 28px;
  }
  .modal h2 {
    font-size: 17px;
    margin-bottom: 6px;
    letter-spacing: -0.01em;
  }
  .modal .desc {
    font-size: 12.5px;
    color: var(--text-dim);
    line-height: 1.6;
    margin-bottom: 20px;
  }
  .modal .field label { color: var(--text-dim); }
  .modal .field input { margin-top: 2px; }
  .modal .helper-link {
    font-size: 11px;
    color: var(--text-faint);
    margin-top: 4px;
    display: block;
  }
  .modal .helper-link a { color: var(--up-dim); text-decoration: underline; }
  .modal .warn-box {
    background: rgba(232,162,61,0.06);
    border: 1px solid rgba(232,162,61,0.25);
    border-radius: 4px;
    padding: 10px 12px;
    font-size: 11.5px;
    color: var(--amber);
    line-height: 1.55;
    margin: 16px 0;
  }
  .modal .connect-btn {
    width: 100%;
    padding: 12px;
    background: var(--up);
    color: #06120b;
    border-radius: 4px;
    font-weight: 600;
    font-family: var(--mono);
    font-size: 13px;
    margin-top: 6px;
  }
  .modal .connect-btn:disabled { background: var(--panel-2); color: var(--text-faint); }
  .modal .err-msg {
    color: var(--down);
    font-size: 12px;
    margin-top: 10px;
    font-family: var(--mono);
  }

  /* Confirm trade modal */
  .confirm-modal .row {
    display: flex;
    justify-content: space-between;
    padding: 8px 0;
    border-bottom: 1px solid var(--line-soft);
    font-family: var(--mono);
    font-size: 13px;
  }
  .confirm-modal .row .k { color: var(--text-dim); }
  .confirm-modal .actions {
    display: flex;
    gap: 8px;
    margin-top: 18px;
  }
  .confirm-modal .actions button { flex: 1; padding: 11px; border-radius: 4px; font-family: var(--mono); font-weight: 600; font-size: 13px; }
  .confirm-modal .cancel { border: 1px solid var(--line); color: var(--text-dim); }
  .confirm-modal .go { background: var(--up); color: #06120b; }
  .confirm-modal .go.fall { background: var(--down); }

  .toast {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: var(--panel-2);
    border: 1px solid var(--line);
    padding: 10px 18px;
    border-radius: 5px;
    font-family: var(--mono);
    font-size: 12.5px;
    z-index: 200;
    box-shadow: 0 8px 24px rgba(0,0,0,0.4);
  }
  .toast.success { border-color: var(--up-dim); color: var(--up); }
  .toast.error { border-color: var(--down-dim); color: var(--down); }

  ::-webkit-scrollbar { width: 8px; height: 8px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--line); border-radius: 4px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--text-faint); }

  @media (max-width: 980px) {
    main.grid { grid-template-columns: 1fr; grid-template-rows: auto auto auto; overflow-y: auto; }
    aside.markets { grid-row: 1; max-height: 160px; border-right: none; border-bottom: 1px solid var(--line); }
    section.chart-panel { grid-column: 1; grid-row: 2; border-right: none; height: 360px; }
    section.analysis-panel { grid-column: 1; grid-row: 3; border-right: none; max-height: none; }
    aside.ticket { grid-column: 1; grid-row: 4; }
  }
</style>
</head>
<body>
<div id="app"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/pako.min.js"></script>
<script src="app.js"></script>
</body>
</html>
