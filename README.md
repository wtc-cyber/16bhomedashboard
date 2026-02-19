<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>HK Smart Home Dashboard</title>
    
    <!-- 外部資源 -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;700&family=Roboto+Mono:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.tailwindcss.com"></script>

    <!-- ========================================================================================== -->
    <!--                                        CSS 樣式區域                                         -->
    <!-- ========================================================================================== -->
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: rgba(30, 41, 59, 0.7);
            --text-main: #f1f5f9;
            --text-dim: #94a3b8;
            --accent: #38bdf8;
            --danger: #ef4444;
            --font-main: 'Noto Sans TC', sans-serif;
            --font-mono: 'Roboto Mono', monospace;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            font-family: var(--font-main);
            overflow: hidden;
            height: 100vh;
            width: 100vw;
            margin: 0;
            font-size: 14px;
        }

        /* 錯誤訊息顯示區 (Debug) */
        #error-console {
            display: none;
            position: fixed;
            top: 0; left: 0; right: 0;
            background: rgba(220, 38, 38, 0.9);
            color: white;
            padding: 10px;
            font-family: monospace;
            font-size: 12px;
            z-index: 99999;
            white-space: pre-wrap;
            pointer-events: none;
        }

        #app-container {
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            padding: 1rem;
            box-sizing: border-box;
            transition: transform 1s ease-in-out;
        }

        header, footer { flex-shrink: 0; }

        main {
            flex: 1;
            min-height: 0;
            display: grid;
            grid-template-columns: 1fr;
            grid-template-rows: 40% 35% 25%;
            gap: 0.75rem;
            margin-top: 0.5rem;
            margin-bottom: 0.5rem;
        }

        @media (min-width: 768px) {
            main {
                grid-template-columns: repeat(12, minmax(0, 1fr));
                grid-template-rows: 1fr; 
                gap: 1rem;
                margin-top: 1rem;
                margin-bottom: 1rem;
            }
        }

        .glass-panel {
            background: var(--card-bg);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            height: 100%;
            position: relative;
        }

        .scroll-content {
            overflow-y: auto;
            flex: 1;
            scrollbar-width: none; 
            -ms-overflow-style: none;
        }
        .scroll-content::-webkit-scrollbar { display: none; }

        .control-btn {
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            padding: 0.4rem 0.8rem;
            border-radius: 0.5rem;
            color: var(--text-dim);
            transition: all 0.2s;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 0.4rem;
            font-size: 0.75rem;
        }
        .control-btn:hover { background: rgba(255, 255, 255, 0.2); color: white; }

        /* Settings Overlay UI */
        .setting-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 0.75rem;
            border-bottom: 1px solid rgba(255,255,255,0.1);
            padding-bottom: 0.4rem;
        }
        .setting-input {
            background: rgba(0,0,0,0.3);
            border: 1px solid rgba(255,255,255,0.2);
            color: white;
            padding: 0.2rem 0.4rem;
            border-radius: 0.25rem;
            width: 100px;
            text-align: center;
            font-size: 0.9rem;
        }

        /* Hardware Protection */
        body.deep-sleep { background-color: #000000 !important; }
        body.deep-sleep #app-container > *:not(#header-section) { display: none !important; }
        body.deep-sleep #header-section {
            position: absolute; top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            width: 100%; text-align: center;
            display: block !important;
        }
        body.deep-sleep .hide-in-sleep { display: none; }
        body.deep-sleep #date { display: none; }
        body.deep-sleep #warning-container { display: none; }
        
        .ticker-wrap { width: 100%; overflow: hidden; white-space: nowrap; }
        .ticker { display: inline-block; animation: ticker 80s linear infinite; }
        @keyframes ticker { 0% { transform: translate3d(100%, 0, 0); } 100% { transform: translate3d(-100%, 0, 0); } }

        #alert-overlay, #setup-overlay, #settings-overlay {
            position: fixed; inset: 0; z-index: 50;
            display: none; 
            justify-content: center; align-items: center;
        }
        #alert-overlay { background: var(--danger); z-index: 9999; flex-direction: column; animation: pulse-red 2s infinite; }
        #setup-overlay { background: rgba(0,0,0,0.9); z-index: 10000; display: flex; }
        #settings-overlay { background: rgba(0,0,0,0.85); backdrop-filter: blur(8px); }

        @keyframes pulse-red { 0% { background: #7f1d1d; } 50% { background: #ef4444; } 100% { background: #7f1d1d; } }

        /* Weather Forecast Grid Fixed */
        .forecast-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 0.5rem;
            text-align: center;
            padding-top: 1rem;
            border-top: 1px solid rgba(255,255,255,0.1);
            width: 100%;
            flex-shrink: 0; 
            margin-top: auto; 
            background: rgba(0,0,0,0.1);
            padding-bottom: 0.5rem;
        }
        
        /* Flex fix for weather card content */
        .weather-card-content {
            display: flex;
            flex-direction: column;
            height: 100%;
            justify-content: space-between;
            overflow: hidden; 
        }

        /* NEW: Internal Weather Warning Styles - Fixed */
        #inner-warning-container {
            margin-top: 1rem;
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            overflow-y: auto;
            flex: 1; 
            min-height: 0;
        }
        
        .warning-item {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            background: rgba(220, 38, 38, 0.2);
            border: 1px solid rgba(220, 38, 38, 0.4);
            padding: 0.5rem 0.75rem;
            border-radius: 0.5rem;
            font-size: 0.9rem;
            color: #fca5a5;
            font-weight: bold;
            flex-shrink: 0;
        }

        .warning-icon {
            height: 36px;
            width: auto;
            display: block;
            margin-right: 0.5rem;
        }
        
        /* Bus Card Styles */
        .bus-info-header {
            border-top: 1px solid rgba(255,255,255,0.2);
            border-bottom: 1px solid rgba(255,255,255,0.2);
            padding: 0.25rem 0;
            margin-bottom: 0.25rem;
        }

        /* Text Clamping Utility */
        .line-clamp-2 {
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            overflow: hidden;
        }
    </style>
</head>
<body>

    <!-- 除錯訊息顯示 -->
    <div id="error-console"></div>

    <!-- ========================================================================================== -->
    <!--                                       HTML 結構區域                                         -->
    <!-- ========================================================================================== -->

    <!-- Setup Overlay -->
    <div id="setup-overlay">
        <div class="text-center px-4">
            <h1 class="text-3xl font-bold mb-4">Smart Dashboard</h1>
            <p class="mb-6 text-gray-400">Tap to initialize System</p>
            <button id="start-btn" class="bg-blue-600 hover:bg-blue-500 text-white px-8 py-3 rounded-full text-xl font-bold transition shadow-lg shadow-blue-500/50">
                Start / 啟動
            </button>
            <p id="firebase-warning" class="hidden text-red-400 text-sm mt-4 font-bold border border-red-500 p-2 rounded">
                ⚠ 注意：Firebase 未設定，待辦事項無法同步。<br>請在代碼中填入 API Key。
            </p>
        </div>
    </div>

    <!-- Local Settings Overlay (Backup) -->
    <div id="settings-overlay">
        <div class="glass-panel p-8 w-96 max-w-full relative">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-white">Local Settings</h2>
                <button id="close-settings" class="text-gray-400 hover:text-white"><i data-lucide="x"></i></button>
            </div>
            <p class="text-yellow-400 text-xs mb-4">Tip: Use the Admin Panel for easier configuration.</p>
            <div id="settings-form-content">
                <button id="local-reset-btn" class="w-full bg-red-600 hover:bg-red-500 text-white py-2 rounded font-bold">Reset Config</button>
            </div>
        </div>
    </div>

    <!-- Alert Overlay -->
    <div id="alert-overlay">
        <i data-lucide="siren" class="w-32 h-32 text-white mb-4 animate-bounce"></i>
        <p id="alert-message" class="text-4xl text-white font-bold mb-8 text-center px-10 leading-tight">...</p>
        <button id="ack-btn" class="bg-white text-red-600 px-10 py-4 rounded-full text-2xl font-bold shadow-lg">
            Acknowledge
        </button>
    </div>

    <!-- Main Container -->
    <div id="app-container">
        
        <!-- HEADER -->
        <header id="header-section" class="flex justify-between items-end pb-2 border-b border-white/10">
            <div class="flex items-end gap-4">
                <h1 id="clock" class="clock-display text-5xl font-mono font-bold tracking-tighter text-white leading-none">00:00:00</h1>
                <div class="flex flex-col mb-1">
                    <div class="flex items-center gap-2">
                        <p id="date" class="date-display text-lg text-slate-300 font-light">Loading...</p>
                    </div>
                </div>
            </div>
            
            <div class="flex flex-col items-end gap-1 hide-in-sleep">
                <div class="flex gap-1">
                    <button id="lang-btn" class="control-btn"><i data-lucide="globe" class="w-3 h-3"></i></button>
                    <button id="fullscreen-btn" class="control-btn"><i data-lucide="maximize" class="w-3 h-3"></i></button>
                    <button id="settings-btn" class="control-btn"><i data-lucide="settings" class="w-3 h-3"></i></button>
                </div>
                <div class="flex items-center gap-1 text-yellow-400 text-xs mt-1">
                    <i data-lucide="sun" class="w-3 h-3"></i>
                    <span id="holiday-status">Checking...</span>
                </div>
            </div>
        </header>

        <!-- MAIN -->
        <main>
            <!-- COL 1: Weather (Now Full Height) -->
            <div class="md:col-span-4 flex flex-col gap-4 h-full min-h-0">
                <article class="glass-panel p-4 flex-1 relative min-h-0">
                    <div class="absolute top-0 right-0 p-4 opacity-10"><i data-lucide="cloud-rain" class="w-24 h-24"></i></div>
                    <div class="weather-card-content">
                        <!-- Top Info -->
                        <div class="flex-1 flex flex-col justify-start min-h-0 overflow-y-auto scroll-hide">
                            <div class="flex-shrink-0">
                                <div class="flex justify-between items-start mb-1">
                                    <h2 class="text-slate-400 text-xs uppercase tracking-wider" data-i18n="weather_title">HK Weather</h2>
                                    <span id="weather-source" class="text-[10px] text-green-400 border border-green-800 bg-green-900/30 px-2 py-0.5 rounded">...</span>
                                </div>
                                <div class="flex items-end gap-3">
                                    <span id="temp" class="text-6xl font-bold text-white">--.-°</span>
                                    <div class="pb-2">
                                        <p id="humidity" class="text-blue-300 text-base">Hum: --%</p>
                                        <p id="rain-prob" class="text-slate-400 text-sm">Rain: --mm</p>
                                    </div>
                                </div>
                                <p id="weather-desc" class="text-lg mt-2 font-medium">Initializing...</p>
                                
                                <!-- Inner Warning Container -->
                                <div id="inner-warning-container"></div>
                            </div>
                        </div>
                        
                        <!-- Forecast Grid -->
                        <div id="forecast-grid" class="forecast-grid"></div>
                    </div>
                </article>
            </div>

            <!-- COL 2: Schedule & Todos -->
            <div class="md:col-span-4 flex flex-col gap-3 h-full min-h-0">
                <article id="schedule-panel" class="glass-panel p-4 h-1/2 min-h-0 flex flex-col border-t-4 border-accent transition-all duration-500">
                    <div class="flex justify-between items-center mb-2 flex-shrink-0">
                        <h3 class="text-slate-400 uppercase text-xs tracking-wider" data-i18n="schedule_title">Upcoming</h3>
                        <span class="text-[10px] bg-slate-700 px-2 py-0.5 rounded text-white">Sync</span>
                    </div>
                    <div id="schedule-list" class="space-y-2 scroll-content"></div>
                </article>
                
                <article id="todo-panel" class="glass-panel p-4 h-1/2 min-h-0 flex flex-col transition-all duration-500">
                    <div class="flex justify-between items-center mb-2 flex-shrink-0">
                        <h3 class="text-slate-400 uppercase text-xs tracking-wider" data-i18n="todo_title">To-Do</h3>
                    </div>
                    <div id="todo-list" class="space-y-2 scroll-content"></div>
                </article>
            </div>

            <!-- COL 3: Action Tips (Top) + Bus (Bottom) -->
            <div class="md:col-span-4 flex flex-col gap-3 h-full min-h-0">
                
                <!-- NEW LOCATION: Action Tips (Takes ~30-35% height) -->
                <article class="glass-panel p-4 h-1/3 min-h-0 flex-shrink-0 flex flex-col">
                    <div class="flex items-center gap-2 mb-2 text-accent flex-shrink-0">
                        <i data-lucide="zap" class="w-4 h-4"></i>
                        <h3 class="font-bold text-sm" data-i18n="suggestion_title">Action Tips</h3>
                    </div>
                    <p id="action-tip" class="text-base leading-snug overflow-y-auto pr-1 text-white/90 scroll-content">Analyzing...</p>
                </article>

                <!-- Bus Arrivals (Takes remaining height) -->
                <article class="glass-panel p-4 flex-1 flex flex-col relative min-h-0">
                    <!-- Title -->
                    <div class="flex items-center gap-2 mb-1 flex-shrink-0">
                        <i data-lucide="bus" class="text-yellow-500 w-4 h-4"></i>
                        <h3 class="font-bold text-lg" data-i18n="bus_title">KMB Arrivals</h3>
                    </div>
                    
                    <!-- New Bus Header Layout -->
                    <div class="bus-info-header flex-shrink-0">
                        <div class="flex items-center justify-between">
                            <!-- Shrink Route to text-2xl -->
                            <span id="bus-route-title" class="text-2xl font-bold text-accent">--</span>
                            <!-- Shrink Stop Name to text-base -->
                            <span id="bus-stop-name-display" class="text-base font-bold text-right truncate max-w-[150px]">--</span>
                        </div>
                    </div>

                    <!-- List Area -->
                    <div id="bus-list" class="space-y-2 scroll-content">
                        <div class="text-center text-slate-500 mt-6 text-sm">Initializing...</div>
                    </div>
                    
                    <!-- Footer Status -->
                    <div class="mt-auto pt-2 flex justify-between text-[10px] text-slate-500 border-t border-slate-700/50 flex-shrink-0">
                        <span id="bus-status">Auto-Track</span>
                    </div>
                </article>
            </div>
        </main>

        <footer class="pt-2 border-t border-slate-800">
            <div class="flex items-center gap-3">
                <span class="bg-red-600 text-white text-[10px] font-bold px-2 py-0.5 rounded flex-shrink-0">NEWS</span>
                <div class="ticker-wrap"><div id="news-ticker" class="ticker text-base text-slate-300">Loading News...</div></div>
            </div>
        </footer>
    </div>

    <!-- ========================================================================================== -->
    <!--                                       JS 程式碼區域                                         -->
    <!-- ========================================================================================== -->
    
    <!-- 全局錯誤捕捉 (Debug Mode) -->
    <script>
        window.onerror = function(message, source, lineno, colno, error) {
            const consoleBox = document.getElementById('error-console');
            consoleBox.style.display = 'block';
            consoleBox.innerText += `❌ ${message} at line ${lineno}\n`;
        };
        window.addEventListener('unhandledrejection', function(event) {
            const consoleBox = document.getElementById('error-console');
            consoleBox.style.display = 'block';
            consoleBox.innerText += `❌ Promise Error: ${event.reason}\n`;
        });
    </script>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, getDocs, addDoc, updateDoc, deleteDoc, onSnapshot, query, orderBy, limit, where, setDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // -----------------------------------------------------------------------------------------
        // ⚠️⚠️⚠️ 請在此處填入你的 FIREBASE 設定 (API KEY) ⚠️⚠️⚠️
        // 如果你沒有填寫，程式將無法連接到資料庫。
        // -----------------------------------------------------------------------------------------
        const LOCAL_FIREBASE_CONFIG = { 
        apiKey: "AIzaSyB7AE6WUyGkVGnA3YjKwd9B10uu-LCowo8",
  authDomain: "homedashboardv2.firebaseapp.com",
  projectId: "homedashboardv2",
  storageBucket: "homedashboardv2.firebasestorage.app",
  messagingSenderId: "240636392334",
  appId: "1:240636392334:web:764203fa6e887876cfaadf",
  measurementId: "G-ZX95TQ0KT6"
};
        // -----------------------------------------------------------------------------------------

        const DEFAULT_CONFIG = {
            lang: 'zh-HK',
            sleepMode: { startHour: 5, startMin: 59, endHour: 6, endMin: 0 },
            busConfig: { route: '273B', targetStopName: '御景峰', seq: null, dir: null },
            weatherStation: 'GPS'
        };

        const I18N = {
            'zh-HK': {
                weather_title: '香港天氣', suggestion_title: '即時行動建議', schedule_title: '即將行程', todo_title: '待辦事項', bus_title: '巴士到站',
                dtip_rain: '現正下雨，外出請務必帶備雨具。', dtip_heavy_rain: '暴雨警告生效中！請留在安全室內。', dtip_very_cold: '天氣嚴寒 (低於12°C)，請穿著厚羽絨及保暖衣物。', dtip_cold: '天氣寒冷 (12-17°C)，建議穿著厚外套或大衣。', dtip_cool: '天氣清涼 (18-23°C)，建議穿著薄外套或風衣。', dtip_warm: '天氣溫暖 (24-27°C)，穿著透氣衣物，舒適宜人。', dtip_hot: '天氣炎熱 (28-32°C)，請注意防曬及補充水分。', dtip_very_hot: '酷熱天氣 (33°C+)，請避免長時間戶外活動，慎防中暑。', dtip_humid: '相對濕度高 (>85%)，建議開啟抽濕機防止牆身出水。', dtip_dry: '天氣乾燥 (濕度<45%)，請多喝水及塗抹潤膚霜。', dtip_default: '天氣狀況良好，適合進行戶外活動。',
                bus_no_data: '暫時沒有班次', bus_stop_not_found: '找不到符合的車站', bus_searching: '搜尋中...',
                date_format: { weekday: 'short', year: 'numeric', month: 'numeric', day: 'numeric' },
                holiday_none: '暫無公眾假期', holiday_none_en: 'No Public Holiday'
            },
            'en': {
                weather_title: 'HK Weather', suggestion_title: 'Action Tips', schedule_title: 'Upcoming', todo_title: 'To-Do List', bus_title: 'Bus Arrivals',
                dtip_rain: 'Raining. Bring Umbrella.', dtip_heavy_rain: 'Heavy Rain Warning!', dtip_very_cold: 'Very Cold (<12°C). Wear heavy coat.', dtip_cold: 'Cold (12-17°C). Wear coat.', dtip_cool: 'Cool (18-23°C). Wear jacket.', dtip_warm: 'Warm (24-27°C). Breathable wear.', dtip_hot: 'Hot (28-32°C). Sunscreen needed.', dtip_very_hot: 'Very Hot (33°C+). Avoid outdoors.', dtip_humid: 'Humid (>85%). Dehumidify.', dtip_dry: 'Dry (<45%). Moisturize.', dtip_default: 'Comfortable weather.',
                bus_no_data: 'No Departures', bus_stop_not_found: 'Stop Not Found', bus_searching: 'Searching...',
                date_format: { weekday: 'short', year: 'numeric', month: 'short', day: 'numeric' },
                holiday_none: 'No Public Holiday', holiday_none_en: 'No Public Holiday'
            }
        };

        const HKO_STATIONS = [
            { name: "香港天文台 (HQ)", lat: 22.3023, lon: 114.1742 }, { name: "上水 Sheung Shui", lat: 22.5019, lon: 114.1287 }, { name: "大埔 Tai Po", lat: 22.4460, lon: 114.1789 }, { name: "沙田 Sha Tin", lat: 22.4025, lon: 114.2100 }, { name: "屯門 Tuen Mun", lat: 22.3855, lon: 113.9642 }, { name: "將軍澳 Tseung Kwan O", lat: 22.3160, lon: 114.2494 }, { name: "西貢 Sai Kung", lat: 22.3836, lon: 114.2709 }, { name: "長洲 Cheung Chau", lat: 22.2011, lon: 114.0267 }, { name: "赤鱲角 Airport", lat: 22.3094, lon: 113.9220 }, { name: "啟德 Kai Tak", lat: 22.3072, lon: 114.2125 }, { name: "黃竹坑 Wong Chuk Hang", lat: 22.2479, lon: 114.1736 }
        ];

        let APP_CONFIG = JSON.parse(localStorage.getItem('dash_config')) || DEFAULT_CONFIG;
        let app, auth, db, appId;
        try {
            const config = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : LOCAL_FIREBASE_CONFIG;
            if (config.apiKey === "YOUR_API_KEY") {
                console.error("Firebase Config Missing");
                document.getElementById('firebase-warning').classList.remove('hidden');
            } else {
                app = initializeApp(config); auth = getAuth(app); db = getFirestore(app);
                appId = typeof __app_id !== 'undefined' ? __app_id : 'default-local-app-id';
            }
        } catch (e) { console.error("Firebase Init Error", e); }
        
        const getCollection = (name) => db ? collection(db, 'artifacts', appId, 'public', 'data', name) : null;

        /* --- 0. AUDIO MANAGER --- */
        class AudioManager {
            constructor() {
                this.ctx = null;
                this.isAlerting = false;
                this.currentId = null;
                this.cycleCount = 0;
                this.timers = [];
            }
            init() { if (!this.ctx) this.ctx = new (window.AudioContext || window.webkitAudioContext)(); if (this.ctx.state === 'suspended') this.ctx.resume(); }
            startAlert(id, text) { if (this.currentId === id && this.isAlerting) return; this.stop(); this.init(); this.currentId = id; this.isAlerting = true; this.cycleCount = 0; this.runCycle(text); }
            runCycle(text) {
                if (!this.isAlerting) return;
                document.getElementById('alert-overlay').style.display = 'flex';
                const cycleStartTime = Date.now();
                const activeDuration = 60 * 1000;
                const playLoop = () => {
                    if (!this.isAlerting) return;
                    if (Date.now() - cycleStartTime > activeDuration) {
                        this.cycleCount++;
                        if (this.cycleCount < 3) {
                            document.getElementById('alert-overlay').style.display = 'none';
                            const t = setTimeout(() => this.runCycle(text), 5 * 60 * 1000);
                            this.timers.push(t);
                        } else { this.stop(); document.getElementById('alert-overlay').style.display = 'none'; }
                        return;
                    }
                    this.playMelody();
                    const t_speak = setTimeout(() => this.speak(text), 6000);
                    const t_next = setTimeout(playLoop, 15000);
                    this.timers.push(t_speak, t_next);
                };
                playLoop();
            }
            playMelody() {
                if (!this.ctx) this.init();
                if (this.ctx.state === 'suspended') this.ctx.resume();
                const now = this.ctx.currentTime;
                const notes = [{ f: 739.99, t: 0.0, d: 2.5 }, { f: 587.33, t: 0.5, d: 2.5 }, { f: 440.00, t: 1.0, d: 2.5 }, { f: 587.33, t: 1.5, d: 2.5 }, { f: 659.25, t: 2.0, d: 2.5 }, { f: 880.00, t: 2.5, d: 3.5 }];
                notes.forEach(n => {
                    const osc = this.ctx.createOscillator(); const gain = this.ctx.createGain();
                    osc.connect(gain); gain.connect(this.ctx.destination);
                    osc.type = 'sine'; osc.frequency.value = n.f;
                    gain.gain.setValueAtTime(0, now + n.t);
                    gain.gain.linearRampToValueAtTime(0.3, now + n.t + 0.03);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + n.t + n.d);
                    osc.start(now + n.t); osc.stop(now + n.t + n.d);
                });
            }
            speak(text) {
                if (!window.speechSynthesis) return;
                window.speechSynthesis.cancel();
                const u = new SpeechSynthesisUtterance(text);
                u.rate = 0.75;
                const hasEnglish = /[a-zA-Z]/.test(text);
                const targetLang = hasEnglish ? 'en-US' : 'zh-HK';
                const voices = window.speechSynthesis.getVoices();
                let v = null;
                if (targetLang === 'zh-HK') {
                    v = voices.find(voice => voice.lang === 'zh-HK');
                    if (!v) v = voices.find(voice => (voice.name.toLowerCase().includes('cantonese') || voice.lang.includes('HK')) && !voice.name.toLowerCase().includes('mandarin') && !voice.lang.includes('zh-CN'));
                } else { v = voices.find(voice => voice.lang.startsWith('en')); }
                if (v) u.voice = v; u.lang = targetLang; window.speechSynthesis.speak(u);
            }
            stop() { this.isAlerting = false; this.currentId = null; this.cycleCount = 0; this.timers.forEach(t => clearTimeout(t)); this.timers = []; if (this.ctx) this.ctx.suspend(); if (window.speechSynthesis) window.speechSynthesis.cancel(); }
        }

        /* --- NEW: HOLIDAY MANAGER --- */
        class HolidayManager {
            constructor() { this.checkHoliday(); setInterval(() => this.checkHoliday(), 3600000); } // Check every hour
            async checkHoliday() {
                const now = new Date();
                const year = now.getFullYear();
                const dateString = now.toISOString().split('T')[0];
                const el = document.getElementById('holiday-status');
                const lang = APP_CONFIG.lang || 'zh-HK';
                
                try {
                    const res = await fetch(`https://date.nager.at/api/v3/PublicHolidays/${year}/HK`);
                    const data = await res.json();
                    const holiday = data.find(h => h.date === dateString);
                    
                    if (holiday) {
                        const name = (lang === 'zh-HK') ? holiday.localName : holiday.name;
                        el.textContent = `★ ${name}`;
                        el.parentElement.classList.add('text-red-400');
                        el.parentElement.classList.remove('text-yellow-400');
                    } else {
                        el.textContent = I18N[lang].holiday_none;
                        el.parentElement.classList.add('text-yellow-400');
                        el.parentElement.classList.remove('text-red-400');
                    }
                } catch(e) { 
                    console.error("Holiday API Error", e);
                    el.textContent = "..."; 
                }
            }
        }

        /* --- 1. SYSTEM MANAGER --- */
        class SystemManager {
            constructor() {
                this.initClock(); this.initBurnIn(); this.initControls(); this.updateLanguage();
                if(db) this.listenRemoteConfig();
                document.addEventListener('visibilitychange', () => { if (document.visibilityState === 'visible') this.requestWakeLock(); });
            }
            listenRemoteConfig() {
                const configRef = doc(getCollection('settings'), 'config');
                onSnapshot(configRef, (doc) => {
                    if (doc.exists()) {
                        const remote = doc.data(); APP_CONFIG = { ...APP_CONFIG, ...remote };
                        if (remote.busConfig && remote.busConfig.targetStopName !== APP_CONFIG.busConfig.targetStopName) { APP_CONFIG.busConfig.seq = null; APP_CONFIG.busConfig.dir = null; }
                        localStorage.setItem('dash_config', JSON.stringify(APP_CONFIG)); this.updateLanguage();
                        if(window.busMgr) window.busMgr.init(); if(window.weatherMgr) window.weatherMgr.locateAndFetch(); if(window.holidayMgr) window.holidayMgr.checkHoliday();
                    }
                });
            }
            async requestWakeLock() { try { await navigator.wakeLock.request('screen'); } catch (e) {} }
            initControls() {
                document.getElementById('lang-btn').onclick = () => {
                    APP_CONFIG.lang = APP_CONFIG.lang === 'zh-HK' ? 'en' : 'zh-HK'; localStorage.setItem('dash_config', JSON.stringify(APP_CONFIG)); this.updateLanguage();
                    window.weatherMgr?.generateAction(window.weatherMgr.lastTemp, window.weatherMgr.lastRain, window.weatherMgr.lastHum, APP_CONFIG.lang);
                    window.busMgr?.fetchData(); window.holidayMgr?.checkHoliday();
                };
                document.getElementById('fullscreen-btn').onclick = () => { if (!document.fullscreenElement) document.documentElement.requestFullscreen(); else document.exitFullscreen(); };
                document.getElementById('settings-btn').onclick = () => document.getElementById('settings-overlay').style.display = 'flex';
                document.getElementById('close-settings').onclick = () => document.getElementById('settings-overlay').style.display = 'none';
                document.getElementById('local-reset-btn').onclick = () => { localStorage.clear(); window.location.reload(); };
            }
            updateLanguage() {
                const lang = APP_CONFIG.lang || 'zh-HK'; const dict = I18N[lang];
                document.querySelectorAll('[data-i18n]').forEach(el => { const key = el.getAttribute('data-i18n'); if (dict[key]) el.textContent = dict[key]; });
            }
            initClock() {
                setInterval(() => {
                    const now = new Date(); document.getElementById('clock').textContent = now.toLocaleTimeString('en-GB', { hour12: false });
                    const lang = APP_CONFIG.lang || 'zh-HK'; const locale = lang === 'zh-HK' ? 'zh-HK' : 'en-GB';
                    document.getElementById('date').textContent = now.toLocaleDateString(locale, I18N[lang].date_format);
                    const min = now.getHours() * 60 + now.getMinutes();
                    const start = APP_CONFIG.sleepMode.startHour * 60 + APP_CONFIG.sleepMode.startMin;
                    const end = APP_CONFIG.sleepMode.endHour * 60 + APP_CONFIG.sleepMode.endMin;
                    let sleep = start < end ? (min >= start && min < end) : (min >= start || min < end);
                    if (sleep) document.body.classList.add('deep-sleep'); else document.body.classList.remove('deep-sleep');
                }, 1000);
            }
            initBurnIn() { setInterval(() => { const x = Math.floor(Math.random() * 6) - 3; const y = Math.floor(Math.random() * 6) - 3; document.getElementById('app-container').style.transform = `translate(${x}px, ${y}px)`; }, 300000); }
        }

        /* --- 2. BUS MANAGER --- */
        class BusManager {
            constructor() { this.init(); }
            async init() {
                this.config = APP_CONFIG.busConfig;
                if (!this.config.seq || this.config.seq === null) { await this.scanRoute(); } else { this.fetchData(); }
                if(this.timer) clearInterval(this.timer); this.timer = setInterval(() => this.fetchData(), 30000); 
            }
            async scanRoute() {
                const { route, targetStopName } = this.config; const container = document.getElementById('bus-list'); const lang = APP_CONFIG.lang || 'zh-HK';
                container.innerHTML = `<div class="text-center text-accent animate-pulse mt-10">${I18N[lang].bus_searching}</div>`;
                document.getElementById('bus-route-title').textContent = route; document.getElementById('bus-stop-name-display').textContent = `...`;
                try {
                    const directions = ['outbound', 'inbound']; let found = false;
                    for (const dir of directions) {
                        if (found) break; const res = await fetch(`https://data.etabus.gov.hk/v1/transport/kmb/route-stop/${route}/${dir}/1`); const data = await res.json(); const stops = data.data;
                        for (let i = 0; i < stops.length; i++) {
                            const stopEntry = stops[i]; const detailRes = await fetch(`https://data.etabus.gov.hk/v1/transport/kmb/stop/${stopEntry.stop}`); const detail = await detailRes.json();
                            const nameTC = detail.data.name_tc; const nameEN = detail.data.name_en; const key = targetStopName.toUpperCase();
                            if (nameTC.includes(key) || nameEN.toUpperCase().includes(key)) {
                                found = true; this.config.seq = stopEntry.seq; this.config.dir = (dir === 'outbound') ? 'O' : 'I'; this.config.foundName = (APP_CONFIG.lang === 'zh-HK') ? nameTC : nameEN;
                                APP_CONFIG.busConfig = this.config; localStorage.setItem('dash_config', JSON.stringify(APP_CONFIG));
                                document.getElementById('bus-status').textContent = "Locating Done"; this.fetchData(); break;
                            }
                        }
                    }
                    if (!found) container.innerHTML = `<div class="text-center text-red-400 mt-10">${I18N[lang].bus_stop_not_found}</div>`;
                } catch(e) { container.innerHTML = `<div class="text-center text-red-400 mt-10">Scan Error</div>`; }
            }
            async fetchData() {
                const { route, seq, dir, foundName } = this.config; if (!seq) return; 
                const container = document.getElementById('bus-list'); const lang = APP_CONFIG.lang || 'zh-HK';
                document.getElementById('bus-route-title').textContent = route; document.getElementById('bus-stop-name-display').textContent = foundName;
                try {
                    const res = await fetch(`https://data.etabus.gov.hk/v1/transport/kmb/route-eta/${route}/1`); const json = await res.json();
                    const buses = json.data.filter(b => parseInt(b.seq) === parseInt(seq)).filter(b => b.dir === dir).filter(b => !!b.eta).sort((a, b) => new Date(a.eta) - new Date(b.eta)).slice(0, 3);
                    container.innerHTML = "";
                    if (buses.length === 0) { container.innerHTML = `<div class="text-center text-slate-500 mt-10">${I18N[lang].bus_no_data}</div>`; return; }
                    buses.forEach((bus, index) => {
                        const diffMins = Math.floor((new Date(bus.eta) - new Date()) / 60000); if (diffMins < -2) return; 
                        const timeStr = new Date(bus.eta).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' });
                        const div = document.createElement('div'); div.className = "flex justify-between items-center border-b border-slate-700 pb-2";
                        div.innerHTML = `<div class="text-xl font-mono text-white">${timeStr}</div><div class="text-right"><span class="text-2xl ${diffMins <= 5 ? 'text-red-400 animate-pulse' : 'text-green-400'} font-mono font-bold">${diffMins <= 0 ? 'Arr' : diffMins + 'm'}</span></div>`;
                        container.appendChild(div);
                    });
                } catch(e) { console.error("Bus Fetch Error", e); }
            }
        }

        /* --- 3. WEATHER MANAGER --- */
        class WeatherManager {
            constructor() {
                this.lastTemp = 25; this.lastRain = 0; this.lastHum = 70;
                this.locateAndFetch(); setInterval(() => this.locateAndFetch(), 60000); 
                window.weatherMgr = this;
            }
            locateAndFetch() {
                const stationOverride = APP_CONFIG.weatherStation; document.getElementById('weather-desc').textContent = "Locating...";
                
                // Chrome Fallback
                const fallback = () => {
                    console.log("GPS Blocked/Unavailable, using HQ Fallback");
                    this.fetchData(HKO_STATIONS[0]); 
                };

                if (stationOverride && stationOverride !== 'GPS') { 
                    const st = HKO_STATIONS.find(s => s.name.includes(stationOverride)) || HKO_STATIONS[0]; 
                    this.fetchData(st); return; 
                }
                
                // 5s Timeout
                const timeout = setTimeout(fallback, 5000);

                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(
                        (pos) => { clearTimeout(timeout); this.fetchData(this.findNearestStation(pos.coords.latitude, pos.coords.longitude)); },
                        (err) => { clearTimeout(timeout); fallback(); }
                    );
                } else { clearTimeout(timeout); fallback(); }
            }
            findNearestStation(lat, lon) {
                let minDist = Infinity; let nearest = HKO_STATIONS[0];
                HKO_STATIONS.forEach(st => {
                    const R = 6371; const dLat = (st.lat - lat) * Math.PI / 180; const dLon = (st.lon - lon) * Math.PI / 180;
                    const a = Math.sin(dLat/2) * Math.sin(dLat/2) + Math.cos(lat * Math.PI / 180) * Math.cos(st.lat * Math.PI / 180) * Math.sin(dLon/2) * Math.sin(dLon/2);
                    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a)); const d = R * c; if (d < minDist) { minDist = d; nearest = st; }
                }); return nearest;
            }
            async fetchData(station) {
                const lang = APP_CONFIG.lang || 'zh-HK'; const langParam = lang === 'zh-HK' ? 'tc' : 'en';
                document.getElementById('weather-source').textContent = `${station.name}`; document.getElementById('weather-desc').textContent = "Loading...";
                try {
                    const res = await fetch(`https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=rhrread&lang=${langParam}`); const data = await res.json();
                    let tempVal = data.temperature.data[0].value; 
                    const match = data.temperature.data.find(t => station.name.includes(t.place) || t.place.includes(station.name.split(' ')[0])); if (match) tempVal = match.value;
                    this.lastTemp = tempVal; this.lastRain = data.rainfall.data[0]?.max || 0; this.lastHum = data.humidity.data[0].value;
                    document.getElementById('temp').textContent = `${parseFloat(tempVal).toFixed(1)}°`; document.getElementById('humidity').textContent = `Hum: ${data.humidity.data[0].value}%`;
                    document.getElementById('weather-desc').textContent = "";
                    this.generateAction(this.lastTemp, this.lastRain, this.lastHum, lang);
                    
                    const wContainer = document.getElementById('inner-warning-container'); 
                    wContainer.innerHTML = ''; // Clear container to prevent duplication
                    
                    const warnRes = await fetch(`https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=warnsum&lang=en`); const warnData = await warnRes.json();
                    
                    // Set to track processed warning codes to prevent duplicates
                    const processedCodes = new Set();

                    if (warnData) {
                        for (const [key, value] of Object.entries(warnData)) {
                            if (value.actionCode && value.actionCode !== "CANCEL") {
                                const code = value.code; 
                                
                                // Prevent duplicates
                                if (processedCodes.has(code)) continue;
                                processedCodes.add(code);

                                if (code) {
                                    const div = document.createElement('div');
                                    let colorClass = "w-black"; 
                                    if (code.includes('RED') || code.includes('FIREY') || code.includes('TC3')) colorClass = "w-amber";
                                    if (code.includes('BLACK') || code.includes('FIRER') || code.includes('TC8') || code.includes('TC9') || code.includes('TC10')) colorClass = "w-red";
                                    if (code.includes('COLD') || code.includes('FROST') || code.includes('TC1')) colorClass = "w-blue";
                                    
                                    div.className = `warning-item ${colorClass}`;
                                    const name = (lang === 'zh-HK') ? value.name.replace('Signal', '信號').replace('Warning', '警告') : value.name; 
                                    
                                    // Use the stable text-only path for GIF icons with lowercase code
                                    const iconUrl = `https://www.hko.gov.hk/textonly/v2/forecast/images_e/warning_icon_${code.toLowerCase()}.gif`;
                                    
                                    // With fallback text
                                    div.innerHTML = `
                                        <img src="${iconUrl}" class="warning-icon" onerror="this.style.display='none'">
                                        <span>${name}</span>
                                    `;
                                    wContainer.appendChild(div);
                                }
                            }
                        }
                    }

                    const fRes = await fetch(`https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=fnd&lang=${langParam}`); const fData = await fRes.json();
                    const fGrid = document.getElementById('forecast-grid'); fGrid.innerHTML = '';
                    if(fData.weatherForecast) fData.weatherForecast.slice(0, 3).forEach(day => {
                        const dateStr = `${day.forecastDate.substring(4,6)}/${day.forecastDate.substring(6,8)}`;
                        // Improved HTML generation for Forecast Grid using text clamping and smaller font
                        fGrid.innerHTML += `
                            <div class="flex flex-col items-center">
                                <div class="text-slate-400 text-xs">${dateStr}</div>
                                <div class="text-white font-bold text-lg my-0.5">${day.forecastMintemp.value}-${day.forecastMaxtemp.value}°</div>
                                <div class="text-[10px] text-slate-500 leading-tight h-8 overflow-hidden flex items-center justify-center w-full px-1">
                                    <span class="line-clamp-2">${day.forecastWeather}</span>
                                </div>
                            </div>`;
                    });
                } catch (e) { console.error(e); }
            }
            generateAction(temp, rain, hum, lang) {
                if(temp === undefined) return; const dict = I18N[lang]; let tips = [];
                if (rain > 30) tips.push(dict.dtip_heavy_rain); else if (rain > 0) tips.push(dict.dtip_rain);
                if (temp < 12) tips.push(dict.dtip_very_cold); else if (temp < 18) tips.push(dict.dtip_cold); else if (temp < 24) tips.push(dict.dtip_cool); else if (temp < 28) tips.push(dict.dtip_warm); else if (temp < 33) tips.push(dict.dtip_hot); else tips.push(dict.dtip_very_hot);
                if (hum > 85) tips.push(dict.dtip_humid); if (hum < 45) tips.push(dict.dtip_dry);
                if (tips.length === 0 || (tips.length === 1 && tips[0] === dict.dtip_warm)) if(tips.length === 0) tips.push(dict.dtip_default);
                document.getElementById('action-tip').innerHTML = tips.map(t => `<div class="mb-2 border-l-2 border-accent pl-2">${t}</div>`).join('');
            }
        }

        async function initNews() {
            try {
                const res = await fetch('https://api.rss2json.com/v1/api.json?rss_url=https://rthk.hk/rthk/news/rss/c_expressnews_clocal.xml'); const data = await res.json();
                if (data.items && data.items.length > 0) document.getElementById('news-ticker').innerHTML = data.items.map(i => `<span class="mr-12 inline-block">📰 ${i.title}</span>`).join('');
            } catch (e) {}
        }

        class DataManager {
            constructor() { if (auth) this.init(); }
            async init() { if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) await signInWithCustomToken(auth, __initial_auth_token); else await signInAnonymously(auth); onAuthStateChanged(auth, u => { if(u && db) this.listen(); }); }
            listen() {
                onSnapshot(query(getCollection('schedules'), orderBy('timestamp', 'asc'), limit(5)), snap => {
                    const schedPanel = document.getElementById('schedule-panel'); const todoPanel = document.getElementById('todo-panel'); const list = document.getElementById('schedule-list'); list.innerHTML = '';
                    if (snap.empty) { schedPanel.classList.add('hidden'); schedPanel.classList.remove('flex'); todoPanel.classList.remove('h-1/2'); todoPanel.classList.add('h-full', 'flex-1'); } 
                    else { schedPanel.classList.remove('hidden'); schedPanel.classList.add('flex'); todoPanel.classList.remove('h-full', 'flex-1'); todoPanel.classList.add('h-1/2'); snap.forEach(d => { const dat = d.data(); list.innerHTML += `<div class="flex gap-3 items-center"><div class="w-12 h-12 bg-slate-700 rounded-lg flex flex-col items-center justify-center text-xs font-bold text-accent"><span>${dat.time.split(':')[0]}</span><span>${dat.time.split(':')[1]}</span></div><div class="flex-1 bg-slate-700/30 p-2 rounded-lg border border-slate-700 truncate"><p class="text-sm font-bold text-white truncate">${dat.title}</p></div></div>`; }); }
                });
                onSnapshot(query(getCollection('todos'), orderBy('timestamp', 'desc')), snap => {
                    const l = document.getElementById('todo-list'); l.innerHTML = '';
                    snap.forEach(d => { const dat = d.data(); l.innerHTML += `<div class="flex items-center gap-3 p-3 rounded bg-slate-800/50 border border-slate-700 ${dat.completed?'opacity-50':''}"><div class="w-5 h-5 rounded border ${dat.completed?'bg-green-500':'border-slate-500'}"></div><span class="${dat.completed?'line-through':''} text-sm flex-1 truncate">${dat.task}</span></div>`; });
                });
                onSnapshot(query(getCollection('alerts'), where('active', '==', true)), snap => {
                    if(!snap.empty) {
                        const d = snap.docs[0]; document.getElementById('alert-overlay').style.display = 'flex'; document.getElementById('alert-message').textContent = d.data().message;
                        if (window.audioMgr) window.audioMgr.startAlert(d.id, d.data().message);
                        document.getElementById('ack-btn').onclick = () => { updateDoc(doc(getCollection('alerts'), d.id), {active:false}); document.getElementById('alert-overlay').style.display = 'none'; if (window.audioMgr) window.audioMgr.stop(); };
                    } else { if (window.audioMgr && window.audioMgr.isAlerting) { window.audioMgr.stop(); document.getElementById('alert-overlay').style.display = 'none'; } }
                });
            }
        }

        document.getElementById('start-btn').onclick = () => {
            document.getElementById('setup-overlay').style.display = 'none';
            window.audioMgr = new AudioManager(); window.audioMgr.init(); window.holidayMgr = new HolidayManager();
            new SystemManager(); new BusManager(); new WeatherManager(); new DataManager(); initNews(); lucide.createIcons();
            document.documentElement.requestFullscreen().catch(()=>{});
        };
        lucide.createIcons();
    </script>
</body>
</html>
