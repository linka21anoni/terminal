<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TERMINAL TRACK</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet-measure@3.1.0/dist/leaflet-measure.css" />
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&display=swap" rel="stylesheet">
<style>
    * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'JetBrains Mono', monospace; }
  body {
    background: #0a0e13;
    background-image:
      linear-gradient(rgba(0, 255, 200, 0.05) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0, 255, 200, 0.05) 1px, transparent 1px);
    background-size: 40px 40px;
    color: #e0e0e0;
    min-height: 100vh;
    overflow-x: hidden;
  }
  .loading { display: none; align-items: center; gap: 10px; color: #00ffc8; font-size: 12px; }
  .loading.active { display: flex; }
  .spinner { width: 16px; height: 16px; border: 2px solid #1a232b; border-top: 2px solid #00ffc8; border-radius: 50%; animation: spin 1s linear infinite; }
  @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  .auth-container { display: flex; align-items: center; justify-content: center; min-height: 100vh; padding: 20px; }
  .auth-box { background: rgba(16, 22, 28, 0.95); border: 1px solid #00ffc8; border-top: 3px solid #00ffc8; padding: 30px; border-radius: 4px; width: 100%; max-width: 400px; }
  .auth-box h2 { text-align: center; font-size: 24px; letter-spacing: 3px; margin-bottom: 25px; color: #00ffc8; text-shadow: 0 0 10px #00ffc8; }
  .auth-box input { margin-bottom: 15px; }
  .auth-box button { width: 100%; margin-bottom: 10px; }
  .auth-switch { text-align: center; margin-top: 15px; font-size: 12px; color: #6a7a87; }
  .auth-error { color: #ff4444; font-size: 12px; margin-bottom: 10px; text-align: center; }
  #app { display: none; }
  .sidebar { position: fixed; left: -280px; top: 0; width: 280px; height: 100%; background: #0a0e13; border-right: 2px solid #00ffc8; box-shadow: 0 0 20px rgba(0, 255, 200, 0.3); transition: left 0.3s ease; z-index: 1000; display: flex; flex-direction: column; }
  .sidebar.active { left: 0; }
  .sidebar-header { display: flex; justify-content: space-between; align-items: center; padding: 20px; border-bottom: 1px solid #1a232b; }
  .sidebar-logo { display: flex; align-items: center; gap: 10px; color: #00ffc8; font-size: 18px; font-weight: 700; text-shadow: 0 0 10px #00ffc8; }
  .sidebar-logo svg { width: 24px; height: 24px; stroke: #00ffc8; filter: drop-shadow(0 0 5px #00ffc8); }
  .close-btn { background: none; border: none; color: #e0e0e0; font-size: 24px; cursor: pointer; }
  .sidebar-menu { list-style: none; flex: 1; padding: 20px 0; overflow-y: auto; }
  .sidebar-menu li { margin-bottom: 5px; }
  .sidebar-menu a { display: flex; align-items: center; gap: 15px; padding: 14px 20px; color: #6a7a87; text-decoration: none; font-size: 13px; transition: all 0.2s; }
  .sidebar-menu a svg { width: 20px; height: 20px; stroke: #6a7a87; }
  .sidebar-menu a:hover { color: #00ffc8; background: rgba(0, 255, 200, 0.05); }
  .sidebar-menu a.active { background: rgba(0, 255, 200, 0.1); border-left: 3px solid #00ffc8; color: #00ffc8; text-shadow: 0 0 8px #00ffc8; }
  .sidebar-menu a.active svg { stroke: #00ffc8; filter: drop-shadow(0 0 5px #00ffc8); }
  .sidebar-footer { padding: 20px; border-top: 1px solid #1a232b; }
  .logout-btn { width: 45%; padding: 10px; background: #ff4444; border: none; border-radius: 5px; color: #fff; font-weight: 700; font-size: 13px; display: flex; align-items: center; justify-content: center; gap: 10px; cursor: pointer; }
  .logout-btn:hover { background: #ff2222; box-shadow: 0 0 15px rgba(255, 68, 68, 0.5); }
  .overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.7); opacity: 0; visibility: hidden; transition: all 0.3s; z-index: 999; }
  .overlay.active { opacity: 1; visibility: visible; }
  header { background: rgba(10, 14, 19, 0.8); backdrop-filter: blur(10px); padding: 15px 20px; border-bottom: 1px solid #00ffc8; display: flex; align-items: center; gap: 15px; position: sticky; top: 0; z-index: 100; }
  .menu-btn { border: 1px solid #00ffc8; padding: 8px 12px; border-radius: 4px; color: #00ffc8; background: transparent; cursor: pointer; font-size: 18px; }
  .menu-btn:hover { background: rgba(0, 255, 200, 0.1); }
  h1 { font-size: 20px; color: #e0e0e0; letter-spacing: 3px; font-weight: 700; }
  .main { padding: 20px; min-height: calc(100vh - 70px); }
  .page { display: none; }
  .page.active { display: block; }
  .card { background: rgba(16, 22, 28, 0.9); border: 1px solid #00ffc8; border-top: 3px solid #00ffc8; padding: 15px; border-radius: 4px; }
  .card-full { grid-column: 1 / -1; }
  .card-label { font-size: 11px; color: #6a7a87; margin-bottom: 8px; letter-spacing: 1px; }
  .card-value { font-size: 18px; font-weight: 700; color: #00ffc8; text-shadow: 0 0 8px #00ffc8; line-height: 1.4; }
  .card-value.white { color: #e0e0e0; text-shadow: none; }
  .card-value.green { color: #00ff88; text-shadow: 0 0 10px #00ff88; }
  .cards-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 15px; }
  button { padding: 12px; border-radius: 4px; border: 1px solid #2a3a47; font-size: 13px; font-family: 'JetBrains Mono', monospace; cursor: pointer; transition: all 0.2s; text-transform: uppercase; letter-spacing: 1px; }
  button.primary { background: #00ffc8; color: #0a0e13; font-weight: 700; border: none; }
  button.primary:hover { background: #00e6b4; }
  button.primary:disabled { opacity: 0.5; cursor: not-allowed; }
  button.secondary { background: transparent; border: 1px solid #00ffc8; color: #00ffc8; }
  button.secondary:hover { background: rgba(0, 255, 200, 0.1); }
  input { padding: 12px; border-radius: 4px; border: 1px solid #2a3a47; font-size: 13px; width: 100%; background: #0a0e13; color: #00ffc8; font-family: 'JetBrains Mono', monospace; }
  input::placeholder { color: #4a5a67; }
  .status-bar { border: 1px dashed #00ffc8; padding: 10px; border-radius: 4px; display: flex; justify-content: space-between; align-items: center; background: rgba(0, 255, 200, 0.05); margin-bottom: 20px; }
  .live-feed { border: 1px solid #00ff88; padding: 6px 12px; color: #00ff88; font-weight: 600; display: flex; align-items: center; gap: 8px; font-size: 12px; }
  .live-dot { width: 8px; height: 8px; background: #00ff88; border-radius: 50%; animation: pulse 1.5s infinite; }
  @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }
  .profile-card { display: flex; gap: 15px; align-items: center; padding: 20px; }
  .avatar { width: 80px; height: 80px; border: 2px solid #00ffc8; border-radius: 4px; background: #1a232b; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
  .avatar img { width: 70px; height: 70px; }
  .profile-info h3 { font-size: 20px; letter-spacing: 3px; color: #e0e0e0; margin-bottom: 8px; }
  .profile-info p { font-size: 14px; color: #00ffc8; margin-bottom: 4px; }
  .email-box { background: #1a232b; border-left: 3px solid #00ffc8; padding: 15px; margin-top: 15px; }
  .email-label { font-size: 11px; color: #6a7a87; margin-bottom: 5px; }
  .email-value { font-size: 14px; color: #e0e0e0; word-break: break-all; }
  .history-list { max-height: 300px; overflow-y: auto; margin-top: 10px; }
  .history-item { background: #0a0e13; border-left: 2px solid #00ffc8; padding: 10px; margin-bottom: 8px; font-size: 12px; }
  .history-time { color: #6a7a87; font-size: 11px; margin-bottom: 4px; }
  .history-action { color: #00ffc8; }
  .help-box { border: 1px dashed #00ffc8; padding: 15px; display: flex; flex-direction: column; gap: 15px; }
  .help-card { background: rgba(16, 22, 28, 0.9); border: 1px solid #1a232b; border-left: 3px solid #00ffc8; padding: 20px; }
  .help-title { font-size: 16px; color: #00ffc8; font-weight: 600; margin-bottom: 8px; }
  .help-desc { font-size: 13px; color: #6a7a87; line-height: 1.6; }
  #map, #trackMap { height: 350px; border-radius: 4px; border: 1px solid #2a3a47; margin-top: 15px; }
  .leaflet-container { background: #0a0e13; }
  .error { color: #ff4444; font-size: 12px; margin-top: 8px; }
  .status { color: #00ff88; font-size: 12px; margin-top: 8px; }
  .leaflet-control-measure { background: #0a0e13 !important; border: 1px solid #00ffc8 !important; border-radius: 4px !important; }
  .leaflet-control-measure a { background: #0a0e13 !important; color: #00ffc8 !important; }
  .leaflet-control-measure a:hover { background: rgba(0, 255, 200, 0.1) !important; }
  .leaflet-measure-tooltip { background: #0a0e13 !important; border: 1px solid #00ffc8 !important; color: #00ffc8 !important; }
  @media (max-width: 600px) { .cards-grid { grid-template-columns: 1fr; } }
</style>
</head>
<body>

<!-- LOGIN PAGE -->
<div id="auth" class="auth-container">
  <div class="auth-box" id="loginBox">
    <h2>TERMINAL TRACK</h2>
    <div class="auth-error" id="loginError"></div>
    <input type="password" id="loginCode" placeholder="MASUKKAN KODE AKSES">
    <button class="primary" onclick="login()">MASUK</button>
    <div class="auth-switch">MASUKKAN KODE AKSES UNTUK MASUK</div>
  </div>
</div>

<!-- MAIN APP -->
<div id="app">
  <div class="sidebar" id="sidebar">
    <div class="sidebar-header">
      <div class="sidebar-logo">
        <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l3 3-3 3m5 0h3M5 20h14a2 2 0 002-2V6a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/></svg>
         TERMINAL TRACK
      </div>
      <button class="close-btn" onclick="toggleSidebar()">×</button>
    </div>
    <ul class="sidebar-menu">
      <li><a href="#" class="active" data-page="dashboard" onclick="switchPage('dashboard')"><svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/></svg>DASHBOARD</a></li>
      <li><a href="#" data-page="profile" onclick="switchPage('profile')"><svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/></svg>PROFILE</a></li>
      <li><a href="#" data-page="tracker" onclick="switchPage('tracker')"><svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 3v2m6-2v2M9 19v2m6-2v2M5 9H3m2 6H3m18-6h-2m2 6h-2M7 19h10a2 2 0 002-2V7a2 2 0 00-2-2H7a2 2 0 00-2 2v10a2 2 0 002 2zM9 9h6v6H9V9z"/></svg>TRACKER</a></li>
      <li><a href="#" data-page="iplookup" onclick="switchPage('iplookup')"><svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9v-9m0-9v9"/></svg>IPLOOKUP</a></li>
      <li><a href="#" data-page="help" onclick="switchPage('help')"><svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.228 9c.549-1.165 2.03-2 3.772-2 2.21 0 4 1.343 4 3 0 1.4-1.278 2.575-3.006 2.907-.542.104-.994.54-.994 1.093m0 3h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/></svg>HELP</a></li>
    </ul>
    <div class="sidebar-footer">
      <button class="logout-btn" onclick="logout()">
        <svg fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"/></svg>
        LOGOUT
      </button>
    </div>
  </div>

  <div class="overlay" id="overlay" onclick="toggleSidebar()"></div>

  <div class="main-wrapper">
    <header>
      <div class="menu-btn" onclick="toggleSidebar()">☰</div>
      <h1 id="pageTitle">DASHBOARD</h1>
      <div style="margin-left:auto; color:#00ffc8;" id="headerUser">TERMINAL TRACK</div>
    </header>

    <div class="main">
      <!-- DASHBOARD PAGE -->
      <div class="page active" id="page-dashboard">
        <div class="status-bar">
          <div class="live-feed"><div class="live-dot"></div>LIVE FEED</div>
          <div id="liveTime">00:00:00 WIB</div>
        </div>
        <div class="cards-grid">
          <div class="card"><div class="card-label">STATUS</div><div class="card-value green" id="dashStatus">ACTIVE</div></div>
          <div class="card"><div class="card-label">UPTIME</div><div class="card-value" id="dashUptime">99.2%</div></div>
          <div class="card"><div class="card-label">CPU_USAGE</div><div class="card-value" id="dashCPU">0%</div></div>
          <div class="card"><div class="card-label">RAM_USAGE</div><div class="card-value" id="dashRAM">0 MB</div></div>
          <div class="card card-full"><div class="card-label">SESSION_TIME</div><div class="card-value" id="sessionTime">00:00:00</div></div>
        </div>
        <div class="card card-full">
          <div class="card-label">YOUR_LOCATION</div>
          <div id="dashMap"></div>
          <div class="status" id="dashLocationStatus" style="text-align:center; margin-top:10px;">Meminta izin lokasi...</div>
        </div>
      </div>

      <!-- PROFILE PAGE -->
      <div class="page" id="page-profile">
        <div class="card card-full profile-card">
          <div class="avatar"><img id="profileAvatar" src="" alt="avatar"></div>
          <div class="profile-info">
            <h3>TERMINAL TRACK</h3>
            <p id="profileUID">IP: MEMUAT...</p>
            <p id="usageTime">WAKTU PAKAI: 00:00:00</p>
          </div>
        </div>
        <div class="email-box">
          <div class="email-label">RIWAYAT PENGGUNAAN</div>
          <div class="history-list" id="historyList"></div>
          <button class="secondary" onclick="clearHistory()" style="margin-top:10px; width:100%;">HAPUS RIWAYAT</button>
        </div>
        <div class="email-box" style="margin-top:20px; border-left-color:#ff4444;">
          <div class="email-label" style="color:#ff4444;">PENGATURAN AKUN</div>
          <input type="password" id="oldCode" placeholder="KODE_LAMA" style="margin-top:10px;">
          <input type="password" id="newCode" placeholder="KODE_BARU" style="margin-top:10px;">
          <input type="password" id="confirmCode" placeholder="ULANGI_KODE_BARU" style="margin-top:10px;">
          <button class="primary" onclick="changeAccessCode()" style="margin-top:10px; width:100%;">UBAH KODE AKSES</button>
          <div class="error" id="settingsError"></div>
          <div class="status" id="settingsSuccess"></div>
        </div>
      </div>

      <!-- IPLOOKUP PAGE -->
      <div class="page" id="page-iplookup">
        <div class="card">
          <div class="card-label" style="color:#00ffc8; font-size:14px;">IP ADDRESS LOOKUP</div>
          <div style="display:flex; gap:10px; margin-top:10px;">
            <input type="text" id="ipInput" placeholder="Masukkan IP (kosongkan untuk IP Anda)">
            <button class="primary" onclick="lookupIP()" style="flex-shrink:0;">CHECK</button>
          </div>
          <div class="error" id="ipError"></div>
          <div class="status" id="ipStatus">SIAP_MENCARI</div>
        </div>
        <div id="ipResult" style="display:none; margin-top:15px;">
          <div class="card">
            <div class="card-label" style="color:#00ffc8;">RESULT</div>
            <div style="border:1px solid #2a3a47; padding:15px; margin-top:10px; border-radius:4px;">
              <div style="display:flex; align-items:center; gap:15px; margin-bottom:10px;">
                <img id="ipFlag" src="" style="width:40px; height:25px; border:1px solid #2a3a47;">
                <div>
                  <div id="ipAddress" style="color:#00ffc8; font-size:18px; font-weight:700;">0.0.0.0</div>
                  <div style="display:flex; gap:8px; margin-top:5px;">
                    <span style="background:#1a232b; padding:2px 8px; font-size:11px;">IPv4</span>
                    <span id="ipISP" style="border:1px solid #00ffc8; color:#00ffc8; padding:2px 8px; font-size:11px;">ISP</span>
                  </div>
                </div>
              </div>
              <div style="font-size:13px; color:#6a7a87;">
                <div>LAT: <span id="ipLat" style="color:#e0e0e0;">0</span></div>
                <div>LON: <span id="ipLon" style="color:#e0e0e0;">0</span></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- HELP PAGE -->
      <div class="page" id="page-help">
        <div class="help-box">
          <div class="help-card"><div class="help-title">1. DASHBOARD</div><div class="help-desc">Menampilkan status sistem, uptime, penggunaan CPU/RAM, dan lokasi Anda saat ini.</div></div>
          <div class="help-card"><div class="help-title">2. TRACKER</div><div class="help-desc">Masukkan nomor HP format 081234567890 lalu klik CARI LOKASI.</div></div>
          <div class="help-card"><div class="help-title">3. IPLOOKUP</div><div class="help-desc">Masukkan IP address untuk melihat lokasi, ISP, negara, dan koordinat.</div></div>
          <div class="help-card"><div class="help-title">4. PROFILE</div><div class="help-desc">Melihat riwayat penggunaan aplikasi dan ganti kode akses.</div></div>
        </div>
      </div>

      <!-- TRACKER PAGE -->
      <div class="page" id="page-tracker">
        <div class="card">
          <input type="tel" id="phoneInput" placeholder="081234567890 - Cari lokasi" style="margin-bottom:10px;">
          <div style="display:flex; gap:10px; flex-wrap:wrap; align-items:center;">
            <button class="primary" id="searchBtn" onclick="searchPhoneArea()">CARI LOKASI</button>
            <button class="secondary" onclick="getMyLocation()">LOKASI SAYA</button>
            <button class="secondary" onclick="clearTrack()">CLEAR</button>
            <div class="loading" id="loading"><div class="spinner"></div>SCANNING...</div>
          </div>
          <div class="error" id="trackError"></div>
          <div class="status" id="trackStatus">SYSTEM_READY // AWAITING_COMMAND</div>
        </div>
        <div id="trackMap"></div>
      </div>
    </div>
  </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/leaflet-measure@3.1.0/dist/leaflet-measure.min.js"></script>
<script>
// ====== CONFIG ======
const CURRENT_USER_KEY = "nodehistory_current_user";
const ACCESS_CODE_KEY = "nodehistory_access_code";
const HISTORY_KEY = "nodehistory_history";
const SESSION_START_KEY = "nodehistory_session_start";

// ====== GLOBAL STATE ======
let dashMap, trackMap;
let trackMarker;
let currentTileLayer;
let sessionInterval;

const phonePrefixes = [
  { prefix: "0811", operator: "Telkomsel", wilayah: "Aceh-Medan", lat: 3.5952, lng: 98.6722 },
  { prefix: "0812", operator: "Telkomsel", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0813", operator: "Telkomsel", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0814", operator: "Indosat", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0815", operator: "Indosat", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0816", operator: "Indosat", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0817", operator: "XL", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0818", operator: "XL", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0819", operator: "XL", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0821", operator: "Telkomsel", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0822", operator: "Telkomsel", wilayah: "Jabodetabek", lat: -6.2088, lng: 106.8456 },
  { prefix: "0823", operator: "Telkomsel", wilayah: "Sumatera-Kalimantan", lat: -0.7893, lng: 113.9213 },
  { prefix: "0851", operator: "Telkomsel", wilayah: "Nasional", lat: -2.5489, lng: 118.0149 },
  { prefix: "0852", operator: "Telkomsel", wilayah: "Nasional", lat: -2.5489, lng: 118.0149 },
  { prefix: "0853", operator: "Telkomsel", wilayah: "Nasional", lat: -2.5489, lng: 118.0149 },
  { prefix: "0855", operator: "Indosat", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0856", operator: "Indosat", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0857", operator: "Indosat", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0858", operator: "Indosat", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0877", operator: "XL", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0878", operator: "XL", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0881", operator: "Smartfren", wilayah: "Nasional", lat: -2.5489, lng: 118.0149 },
  { prefix: "0882", operator: "Smartfren", wilayah: "Nasional", lat: -2.5489, lng: 118.0149 },
  { prefix: "0895", operator: "Tri", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0896", operator: "Tri", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0897", operator: "Tri", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0898", operator: "Tri", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 },
  { prefix: "0899", operator: "Tri", wilayah: "Nasional", lat: -6.2088, lng: 106.8456 }
];

// ====== HISTORY & SESSION ======
function getHistory() { return JSON.parse(localStorage.getItem(HISTORY_KEY) || '[]'); }
function addHistory(action) {
  let history = getHistory();
  history.unshift({ action: action, time: new Date().toISOString() });
  if (history.length > 50) history.pop();
  localStorage.setItem(HISTORY_KEY, JSON.stringify(history));
  loadHistory();
}
function loadHistory() {
  const history = getHistory();
  const listEl = document.getElementById('historyList');
  if (!listEl) return;
  if (history.length === 0) {
    listEl.innerHTML = '<div style="color:#6a7a87; text-align:center; padding:20px;">BELUM ADA RIWAYAT</div>';
    return;
  }
  listEl.innerHTML = history.map(h => {
    const date = new Date(h.time);
    const timeStr = date.toLocaleString('id-ID');
    return `<div class="history-item"><div class="history-time">${timeStr}</div><div class="history-action">${h.action}</div></div>`;
  }).join('');
}
function clearHistory() { localStorage.removeItem(HISTORY_KEY); loadHistory(); }
function startSessionTimer() {
  if (!localStorage.getItem(SESSION_START_KEY)) { localStorage.setItem(SESSION_START_KEY, Date.now()); }
  const startTime = parseInt(localStorage.getItem(SESSION_START_KEY));
  sessionInterval = setInterval(() => {
    const elapsed = Date.now() - startTime;
    const hours = String(Math.floor(elapsed / 3600000)).padStart(2, '0');
    const minutes = String(Math.floor((elapsed % 3600000) / 60000)).padStart(2, '0');
    const seconds = String(Math.floor((elapsed % 60000) / 1000)).padStart(2, '0');
    const timeStr = `${hours}:${minutes}:${seconds}`;
    if (document.getElementById('sessionTime')) {
      document.getElementById('sessionTime').textContent = timeStr;
    }
    if (document.getElementById('usageTime')) {
      document.getElementById('usageTime').textContent = `WAKTU PAKAI: ${timeStr}`;
    }
  }, 1000);
}

function stopSessionTimer() {
  clearInterval(sessionInterval);
  localStorage.removeItem(SESSION_START_KEY);
}

// ====== AUTH FUNCTIONS ======
function getAccessCode() {
  return localStorage.getItem(ACCESS_CODE_KEY) || 'linka21';
}

function setAccessCode(code) {
  localStorage.setItem(ACCESS_CODE_KEY, code);
}

function getCurrentUser() {
  return JSON.parse(localStorage.getItem(CURRENT_USER_KEY) || 'null');
}

function setCurrentUser(user) {
  localStorage.setItem(CURRENT_USER_KEY, JSON.stringify(user));
}

function login() {
  const code = document.getElementById('loginCode').value.trim();
  const errorEl = document.getElementById('loginError');

  if (!code) {
    errorEl.textContent = 'MASUKKAN_KODE_AKSES';
    return;
  }

  const savedCode = getAccessCode();

  if (code!== savedCode) {
    errorEl.textContent = 'KODE_AKSES_SALAH';
    return;
  }

  let user = getCurrentUser();
  if (!user) {
    user = {
      id: Date.now(),
      username: 'TERMINAL TRACK',
      createdAt: new Date().toISOString()
    };
    setCurrentUser(user);
  }

  setCurrentUser(user);
  addHistory('LOGIN BERHASIL');
  startApp();
}

function logout() {
  addHistory('LOGOUT DARI APLIKASI');
  stopSessionTimer();
  localStorage.removeItem(CURRENT_USER_KEY);
  localStorage.removeItem(SESSION_START_KEY);
  window.location.reload(true);
}

function startApp() {
  const user = getCurrentUser();
  if (!user) return;

  document.getElementById('auth').style.display = 'none';
  document.getElementById('app').style.display = 'block';

  loadUserData();
  loadHistory();
  updateClock();
  updateSystemMonitor();
  startSessionTimer();

  setInterval(updateClock, 1000);
  setInterval(updateSystemMonitor, 2000);

  if (document.getElementById('dashMap')) {
    initDashMap();
  }
}

async function loadUserData() {
  const user = getCurrentUser();
  if (!user) return;

  try {
    const res = await fetch('https://api.ipify.org?format=json');
    const data = await res.json();
    document.getElementById('profileUID').textContent = `IP: ${data.ip}`;
    document.getElementById('profileAvatar').src = `https://api.dicebear.com/7.x/bottts/svg?seed=${data.ip}`;
  } catch (e) {
    document.getElementById('profileUID').textContent = `UID: #${user.id}`;
    document.getElementById('profileAvatar').src = `https://api.dicebear.com/7.x/bottts/svg?seed=${user.id}`;
  }
}

// ====== PENGATURAN AKUN ======
function changeAccessCode() {
  const oldCode = document.getElementById('oldCode').value.trim();
  const newCode = document.getElementById('newCode').value.trim();
  const confirmCode = document.getElementById('confirmCode').value.trim();
  const errorEl = document.getElementById('settingsError');
  const successEl = document.getElementById('settingsSuccess');

  errorEl.textContent = '';
  successEl.textContent = '';

  if (!oldCode ||!newCode ||!confirmCode) {
    errorEl.textContent = 'SEMUA_FIELD_WAJIB_DIISI';
    return;
  }

  if (oldCode!== getAccessCode()) {
    errorEl.textContent = 'KODE_LAMA_SALAH';
    return;
  }

  if (newCode!== confirmCode) {
    errorEl.textContent = 'KODE_BARU_TIDAK_COCOK';
    return;
  }

  if (newCode.length < 4) {
    errorEl.textContent = 'KODE_MIN_4_KARAKTER';
    return;
  }

  setAccessCode(newCode);
  addHistory('UBAH KODE AKSES');

  successEl.textContent = 'KODE_AKSES_BERHASIL_DIUBAH. LOGOUT SEKARANG UNTUK LOGIN ULANG.';

  document.getElementById('oldCode').value = '';
  document.getElementById('newCode').value = '';
  document.getElementById('confirmCode').value = '';

  setTimeout(() => {
    successEl.textContent = '';
  }, 4000);
}

// ====== SIDEBAR & PAGE NAVIGATION ======
function toggleSidebar() {
  document.getElementById('sidebar').classList.toggle('active');
  document.getElementById('overlay').classList.toggle('active');
}

function switchPage(page) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.sidebar-menu a').forEach(a => a.classList.remove('active'));

  document.getElementById(`page-${page}`).classList.add('active');
  document.querySelector(`[data-page="${page}"]`).classList.add('active');

  const titles = {
    dashboard: 'DASHBOARD',
    profile: 'PROFILE',
    tracker: 'TRACKER',
    iplookup: 'IPLOOKUP',
    help: 'HELP'
  };
  document.getElementById('pageTitle').textContent = titles[page];

  toggleSidebar();

  if (page === 'dashboard' &&!dashMap) {
    setTimeout(initDashMap, 100);
  }
  if (page === 'tracker' &&!trackMap) {
    setTimeout(initTrackMap, 100);
  }
}

// ====== DASHBOARD FUNCTIONS ======
function updateClock() {
  const now = new Date();
  const time = now.toLocaleTimeString('id-ID');
  document.getElementById('liveTime').textContent = time + ' WIB';
}

function updateSystemMonitor() {
  if (performance.memory) {
    const usedMB = (performance.memory.usedJSHeapSize / 1048576).toFixed(1);
    const totalMB = (performance.memory.jsHeapSizeLimit / 1048576).toFixed(0);
    document.getElementById('dashRAM').textContent = `${usedMB} MB / ${totalMB} MB`;
  } else {
    document.getElementById('dashRAM').textContent = 'N/A';
  }

  const start = performance.now();
  let count = 0;
  while (performance.now() - start < 10) { count++; }
  const cpu = Math.min(100, Math.floor((count / 50000) * 100));
  document.getElementById('dashCPU').textContent = cpu + '%';
}

function initDashMap() {
  if (dashMap) return;

  dashMap = L.map('dashMap').setView([-6.2088, 106.8456], 10);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(dashMap);

  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const lat = position.coords.latitude;
        const lng = position.coords.longitude;
        dashMap.setView([lat, lng], 13);
        L.marker([lat, lng]).addTo(dashMap).bindPopup('LOKASI_ANDA').openPopup();
        document.getElementById('dashLocationStatus').textContent = `LAT: ${lat.toFixed(4)} // LNG: ${lng.toFixed(4)}`;
        addHistory(`AKSES LOKASI: ${lat.toFixed(4)}, ${lng.toFixed(4)}`);
      },
      () => {
        document.getElementById('dashLocationStatus').textContent = 'IZIN_LOKASI_DITOLAK';
      }
    );
  }
}

// ====== IPLOOKUP FUNCTIONS ======
async function lookupIP() {
  const ip = document.getElementById('ipInput').value.trim();
  const errorEl = document.getElementById('ipError');
  const statusEl = document.getElementById('ipStatus');
  const resultDiv = document.getElementById('ipResult');

  errorEl.textContent = '';
  statusEl.textContent = 'MENCARI_DATA...';
  resultDiv.style.display = 'none';

  try {
    const url = ip? `http://ip-api.com/json/${ip}?fields=status,message,country,countryCode,regionName,city,lat,lon,isp,query,continent`
                   : `http://ip-api.com/json/?fields=status,message,country,countryCode,regionName,city,lat,lon,isp,query,continent`;

    const res = await fetch(url);
    const data = await res.json();

    if (data.status!== 'success') {
      errorEl.textContent = data.message || 'IP_TIDAK_DITEMUKAN';
      statusEl.textContent = 'GAGAL';
      return;
    }

    document.getElementById('ipAddress').textContent = data.query;
    document.getElementById('ipISP').textContent = data.isp;
    document.getElementById('ipLat').textContent = data.lat;
    document.getElementById('ipLon').textContent = data.lon;
    document.getElementById('ipFlag').src = `https://flagcdn.com/48x36/${data.countryCode.toLowerCase()}.png`;

    resultDiv.style.display = 'block';
    statusEl.textContent = 'DATA_DITEMUKAN';
    addHistory(`IPLOOKUP: ${data.query} - ${data.city}, ${data.country}`);

  } catch (e) {
    errorEl.textContent = 'GAGAL_CONNECT_KE_API';
    statusEl.textContent = 'ERROR';
  }
}

// ====== TRACKER FUNCTIONS ======
function initTrackMap() {
  if (trackMap) return;

  trackMap = L.map('trackMap', { zoomControl: true }).setView([-6.2088, 106.8456], 6);

  const streetLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  });

  const satelliteLayer = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Tiles © Esri'
  });

  currentTileLayer = streetLayer.addTo(trackMap);

  const toggleBtn = L.control({position: 'topright'});
  toggleBtn.onAdd = function () {
    const div = L.DomUtil.create('div', 'leaflet-bar');
    div.innerHTML = '<a href="#" id="mapToggle" style="background:#0a0e13;color:#00ffc8;padding:8px 12px;display:block;font-size:12px;font-weight:700;text-decoration:none;">SATELIT</a>';
    div.style.cursor = 'pointer';

    L.DomEvent.on(div, 'click', function(e) {
      L.DomEvent.stopPropagation(e);
      L.DomEvent.preventDefault(e);

      if (trackMap.hasLayer(streetLayer)) {
        trackMap.removeLayer(streetLayer);
        satelliteLayer.addTo(trackMap);
        currentTileLayer = satelliteLayer;
        div.querySelector('#mapToggle').textContent = 'PETA';
      } else {
        trackMap.removeLayer(satelliteLayer);
        streetLayer.addTo(trackMap);
        currentTileLayer = streetLayer;
        div.querySelector('#mapToggle').textContent = 'SATELIT';
      }
    });
    return div;
  };
  toggleBtn.addTo(trackMap);

  L.control.measure({
    position: 'topleft',
    primaryLengthUnit: 'kilometers',
    secondaryLengthUnit: 'meters',
    activeColor: '#00ffc8',
    completedColor: '#00ff88',
    localization: 'id'
  }).addTo(trackMap);
}

function searchPhoneArea() {
  const phone = document.getElementById('phoneInput').value.trim();
  const errorEl = document.getElementById('trackError');
  const statusEl = document.getElementById('trackStatus');
  const loadingEl = document.getElementById('loading');
  const searchBtn = document.getElementById('searchBtn');

  errorEl.textContent = '';

  if (!phone || phone.length < 10) {
    errorEl.textContent = 'NOMOR_TIDAK_VALID';
    return;
  }

  loadingEl.classList.add('active');
  searchBtn.disabled = true;
  statusEl.textContent = 'SCANNING_DATABASE_OPERATOR...';

  setTimeout(() => {
    const prefix = phone.substring(0, 4);
    const data = phonePrefixes.find(p => p.prefix === prefix);

    loadingEl.classList.remove('active');
    searchBtn.disabled = false;

    if (data) {
      statusEl.textContent = 'TARGET_LOCKED';

      if (trackMarker) trackMap.removeLayer(trackMarker);

      trackMap.setView([data.lat, data.lng], 12, { animate: true, duration: 1 });

      trackMarker = L.marker([data.lat, data.lng]).addTo(trackMap);

      const popupContent = `
        <div style="color:#0a0e13;">
          <strong>OPERATOR:</strong> ${data.operator}<br>
          <strong>WILAYAH:</strong> ${data.wilayah}<br>
          <strong>PREFIX:</strong> ${data.prefix}<br>
          <strong>KOORDINAT:</strong> ${data.lat}, ${data.lng}
        </div>
      `;

      trackMarker.bindPopup(popupContent).openPopup();

      L.circle([data.lat, data.lng], {
        color: '#00ffc8',
        fillColor: '#00ffc8',
        fillOpacity: 0.2,
        radius: 30000
      }).addTo(trackMap);

      navigator.geolocation.getCurrentPosition((pos) => {
        const userLat = pos.coords.latitude;
        const userLng = pos.coords.longitude;
        const dist = trackMap.distance([userLat, userLng], [data.lat, data.lng]);
        const km = (dist / 1000).toFixed(2);
        statusEl.textContent = `TARGET_LOCKED // JARAK: ${km} KM`;
      }, () => {});

      addHistory(`TRACKER: ${phone} - ${data.operator} ${data.wilayah}`);

    } else {
      statusEl.textContent = 'TARGET_NOT_FOUND';
      errorEl.textContent = 'PREFIX_TIDAK_DIKENALI_ATAU_INVALID';
    }
  }, 1200);
}

function getMyLocation() {
  const statusEl = document.getElementById('trackStatus');
  const errorEl = document.getElementById('trackError');

  errorEl.textContent = '';

  if (!navigator.geolocation) {
    errorEl.textContent = 'GEOLOCATION_TIDAK_DIDUKUNG_BROWSER';
    return;
  }

  statusEl.textContent = 'MENDAPATKAN_LOKASI_GPS...';

  navigator.geolocation.getCurrentPosition(
    (position) => {
      const lat = position.coords.latitude;
      const lng = position.coords.longitude;
      const accuracy = position.coords.accuracy;

      statusEl.textContent = 'LOKASI_DITEMUKAN';

      if (!trackMap) initTrackMap();

      trackMap.setView([lat, lng], 16, { animate: true, duration: 1 });

      if (trackMarker) trackMap.removeLayer(trackMarker);

      trackMarker = L.marker([lat, lng]).addTo(trackMap);

      const popupContent = `
        <div style="color:#0a0e13;">
          <strong>LOKASI_ANDA</strong><br>
          LAT: ${lat.toFixed(6)}<br>
          LNG: ${lng.toFixed(6)}<br>
          AKURASI: ±${Math.round(accuracy)} meter
        </div>
      `;

      trackMarker.bindPopup(popupContent).openPopup();
      addHistory(`LOKASI SAYA: ${lat.toFixed(6)}, ${lng.toFixed(6)}`);
    },
    (error) => {
      statusEl.textContent = 'GAGAL_MENDAPATKAN_LOKASI';
      errorEl.textContent = 'IZIN_LOKASI_DITOLAK';
    },
    { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 }
  );
}

function clearTrack() {
  document.getElementById('phoneInput').value = '';
  document.getElementById('trackError').textContent = '';
  document.getElementById('trackStatus').textContent = 'SYSTEM_READY // AWAITING_COMMAND';

  if (trackMarker && trackMap) {
    trackMap.removeLayer(trackMarker);
    trackMarker = null;
    trackMap.setView([-6.2088, 106.8456], 6);
  }
}

// ====== INIT ======
window.onload = () => {
  const user = getCurrentUser();
  if (user) {
    startApp();
  }
};
</script>
</body>
</html>
