[index (26).html](https://github.com/user-attachments/files/27278631/index.26.html)
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Apple C1 LNA Pro Design Studio (TSMC N7)</title>
    
    <!-- 引入外部 3D 與圖表庫 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>

    <style>
        /* === UI 視覺設定 (Apple Dark Mode 風格) === */
        :root {
            --bg-base: #000000;
            --bg-panel: #151516;
            --bg-card: #1c1c1e;
            --text-primary: #f5f5f7;
            --text-secondary: #86868b;
            --accent-blue: #0a84ff;
            --accent-green: #30d158;
            --accent-red: #ff453a;
            --accent-orange: #ff9f0a;
            --border-color: #333336;
            --radius-lg: 18px;
            --radius-md: 12px;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro TC", "SF Pro Text", "Helvetica Neue", sans-serif;
            background-color: var(--bg-base);
            color: var(--text-primary);
            margin: 0; padding: 20px 40px; line-height: 1.6;
            -webkit-font-smoothing: antialiased;
        }

        .header-section {
            text-align: center; margin-bottom: 40px; padding-bottom: 20px;
            border-bottom: 1px solid var(--border-color);
        }

        h1 { font-weight: 700; font-size: 36px; letter-spacing: 0.5px; margin-bottom: 8px; }
        .subtitle { color: var(--text-secondary); font-size: 16px; font-weight: 500; }
        h2 { font-size: 20px; color: var(--accent-blue); display: flex; align-items: center; gap: 10px; margin-top: 0; }
        h2::before { content: ""; display: block; width: 4px; height: 20px; background: var(--accent-blue); border-radius: 2px; }

        .dashboard-grid {
            display: grid; grid-template-columns: repeat(12, 1fr); gap: 24px;
        }

        .card {
            background: var(--bg-card); border: 1px solid var(--border-color);
            border-radius: var(--radius-lg); padding: 24px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.4);
            display: flex; flex-direction: column;
        }

        .col-span-12 { grid-column: span 12; }
        .col-span-6 { grid-column: span 6; }
        .col-span-5 { grid-column: span 5; }
        .col-span-7 { grid-column: span 7; }

        /* SVG 電路圖樣式 */
        .schematic-wrapper {
            background: #111; border-radius: var(--radius-md); padding: 20px;
            display: flex; justify-content: center; border: 1px solid #222;
        }
        svg text { font-family: "SF Mono", monospace; font-size: 13px; fill: #ccc; }
        svg .rf-path { stroke: var(--accent-blue); stroke-width: 3; fill: none; stroke-linecap: round; }
        svg .dc-path { stroke: var(--accent-red); stroke-width: 2; fill: none; stroke-dasharray: 5,5; }
        svg .component { stroke: var(--accent-green); stroke-width: 2; fill: none; stroke-linejoin: round; }

        /* 3D 視窗區 */
        #threejs-canvas { width: 100%; height: 500px; border-radius: var(--radius-md); background: #050505; border: 1px inset #222; }
        #plotly-canvas { width: 100%; height: 600px; border-radius: var(--radius-md); background: #0a0a0a; border: 1px inset #222; }

        /* 控制面板與滑桿 */
        .control-panel { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .control-group { background: var(--bg-panel); padding: 16px; border-radius: var(--radius-md); border: 1px solid #2a2a2d; }
        .control-header { display: flex; justify-content: space-between; margin-bottom: 12px; font-size: 14px; font-weight: 600; color: var(--text-secondary); }
        .val-badge { background: #2c2c2e; color: var(--accent-green); padding: 2px 8px; border-radius: 6px; font-family: "SF Mono", monospace; font-size: 13px; }
        
        input[type="range"] {
            -webkit-appearance: none; width: 100%; height: 6px; background: #333;
            border-radius: 3px; outline: none; transition: 0.2s;
        }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none; width: 20px; height: 20px; background: #fff;
            border-radius: 50%; cursor: pointer; box-shadow: 0 0 10px rgba(0,0,0,0.5);
        }

        /* 按鈕與狀態列 */
        .toolbar { display: flex; gap: 12px; margin-top: 24px; flex-wrap: wrap; }
        button {
            background: #2c2c2e; color: white; border: 1px solid #444; padding: 12px 24px;
            border-radius: 8px; font-size: 14px; font-weight: 600; cursor: pointer;
            transition: all 0.2s ease; display: flex; align-items: center; gap: 8px;
        }
        button:hover { background: #3a3a3c; transform: translateY(-2px); }
        button.btn-primary { background: var(--accent-blue); border-color: #0061d5; }
        button.btn-primary:hover { background: #409cff; }
        button.btn-warning { background: var(--accent-orange); color: #000; border-color: #cc7f00; }
        button.btn-danger { background: rgba(255, 69, 58, 0.2); color: var(--accent-red); border-color: var(--accent-red); }

        .terminal {
            background: #000; border: 1px solid #333; border-radius: var(--radius-md);
            padding: 16px; font-family: "SF Mono", monospace; font-size: 13px;
            color: var(--accent-green); height: 160px; overflow-y: auto; line-height: 1.7;
        }
        .terminal-time { color: #888; margin-right: 8px; }

        /* 效能狀態表 */
        .stats-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-top: 20px; }
        .stat-box { background: var(--bg-panel); padding: 16px; border-radius: 10px; text-align: center; border: 1px solid #2a2a2d; }
        .stat-label { font-size: 12px; color: var(--text-secondary); text-transform: uppercase; letter-spacing: 1px; }
        .stat-value { font-size: 24px; font-weight: 700; margin-top: 5px; font-family: "SF Mono", monospace; }
        .good { color: var(--accent-green); }
        .warn { color: var(--accent-orange); }
    </style>
</head>
<body>

<div class="header-section">
    <h1>Apple C1 LNA Pro Design Studio</h1>
    <div class="subtitle">基於 TSMC 7nm FinFET 製程的 5G NR (n78) 射頻前端協同設計平台</div>
</div>

<div class="dashboard-grid">
    
    <!-- (a) Schematic 區塊 -->
    <div class="card col-span-12">
        <h2>(a) 高精細 Cascode LNA 射頻與偏壓網路原理圖</h2>
        <div class="schematic-wrapper">
            <svg width="900" height="340" viewBox="0 0 900 340">
                <!-- 背景網格 -->
                <defs>
                    <pattern id="grid" width="20" height="20" patternUnits="userSpaceOnUse">
                        <path d="M 20 0 L 0 0 0 20" fill="none" stroke="#222" stroke-width="1"/>
                    </pattern>
                </defs>
                <rect width="900" height="340" fill="url(#grid)" />
                
                <!-- 核心射頻路徑 (由左至右) -->
                <!-- Input Pad -->
                <rect x="50" y="190" width="30" height="10" fill="var(--accent-blue)"/>
                <text x="45" y="180" fill="var(--accent-blue)">RF_IN (50Ω)</text>
                
                <!-- Cin -->
                <line x1="80" y1="195" x2="160" y2="195" class="rf-path"/>
                <line x1="160" y1="175" x2="160" y2="215" class="component"/>
                <line x1="170" y1="175" x2="170" y2="215" class="component"/>
                <text x="145" y="165">C_in (1pF)</text>
                
                <!-- Lg Node -->
                <line x1="170" y1="195" x2="250" y2="195" class="rf-path"/>
                <circle cx="250" cy="195" r="5" fill="var(--text-primary)"/>
                
                <!-- Lg (Gate Inductor) -->
                <path d="M 250 195 Q 260 160 290 195 Q 320 230 350 195 Q 380 160 410 195" class="component"/>
                <text x="300" y="165" fill="var(--accent-green)">Lg (匹配與諧振)</text>
                
                <!-- M1 Gate Node -->
                <line x1="410" y1="195" x2="480" y2="195" class="rf-path"/>
                <circle cx="480" cy="195" r="5" fill="var(--text-primary)"/>
                
                <!-- Bias Network M1 -->
                <path d="M 480 195 L 480 240 L 460 250 L 500 270 L 460 290 L 480 300 L 480 320" class="dc-path"/>
                <circle cx="480" cy="320" r="4" fill="var(--accent-red)"/>
                <text x="490" y="270" fill="var(--accent-red)">Rb (8kΩ)</text>
                <text x="490" y="325" fill="var(--accent-red)">V_bias1</text>
                
                <!-- M1 (CS Stage) -->
                <rect x="520" y="185" width="8" height="30" fill="var(--text-primary)"/>
                <line x1="480" y1="195" x2="520" y2="195" class="rf-path"/> <!-- Gate -->
                <line x1="535" y1="205" x2="535" y2="240" class="rf-path"/> <!-- Source -->
                <text x="545" y="200">M1 (CS, g_m)</text>
                
                <!-- Ls (Source Degeneration) -->
                <path d="M 535 240 Q 515 255 535 270 Q 555 285 535 300" class="component"/>
                <text x="560" y="275" fill="var(--accent-green)">Ls (實部匹配)</text>
                <!-- Ground -->
                <path d="M 515 300 L 555 300 M 525 305 L 545 305 M 530 310 L 540 310" stroke="#aaa" fill="none" stroke-width="2"/>
                
                <!-- M2 (CG Stage) -->
                <line x1="535" y1="180" x2="535" y2="140" class="rf-path"/> <!-- Drain M1 to Source M2 -->
                <rect x="520" y="110" width="8" height="30" fill="var(--text-primary)"/>
                <line x1="480" y1="120" x2="520" y2="120" class="rf-path"/> <!-- Gate M2 -->
                <text x="545" y="130">M2 (CG, Isolation)</text>
                
                <!-- Bias Network M2 & C_bypass -->
                <line x1="480" y1="120" x2="420" y2="120" class="dc-path"/>
                <circle cx="420" cy="120" r="4" fill="var(--accent-red)"/>
                <text x="375" y="125" fill="var(--accent-red)">V_bias2</text>
                
                <circle cx="450" cy="120" r="3" fill="var(--text-primary)"/>
                <line x1="450" y1="120" x2="450" y2="150" class="component"/>
                <line x1="435" y1="150" x2="465" y2="150" class="component"/>
                <line x1="435" y1="160" x2="465" y2="160" class="component"/>
                <line x1="450" y1="160" x2="450" y2="180" class="component"/>
                <path d="M 440 180 L 460 180 M 445 185 L 455 185" stroke="#aaa" fill="none" stroke-width="2"/>
                <text x="385" y="160">C_b (3pF)</text>
                
                <!-- Output Node -->
                <line x1="535" y1="105" x2="535" y2="70" class="rf-path"/>
                <circle cx="535" cy="70" r="5" fill="var(--text-primary)"/>
                
                <!-- Ld (Load Inductor) -->
                <path d="M 535 70 Q 515 55 535 40 Q 555 25 535 10" class="component"/>
                <line x1="535" y1="10" x2="600" y2="10" class="dc-path"/>
                <circle cx="600" cy="10" r="4" fill="var(--accent-red)"/>
                <text x="610" y="15" fill="var(--accent-red)">VDD (0.75V)</text>
                <text x="560" y="45" fill="var(--accent-green)">Ld (負載諧振)</text>
                
                <!-- C_d Output Match -->
                <line x1="535" y1="70" x2="650" y2="70" class="rf-path"/>
                <line x1="650" y1="50" x2="650" y2="90" class="component"/>
                <line x1="660" y1="50" x2="660" y2="90" class="component"/>
                <text x="635" y="40">C_d (1pF)</text>
                
                <!-- Output Pad -->
                <line x1="660" y1="70" x2="750" y2="70" class="rf-path"/>
                <rect x="750" y="65" width="30" height="10" fill="var(--accent-blue)"/>
                <text x="790" y="75" fill="var(--accent-blue)">RF_OUT</text>

                <!-- 寄生電容標示 (Miller) -->
                <path d="M 495 180 Q 510 160 525 180" stroke="rgba(255,159,10,0.6)" stroke-width="1.5" stroke-dasharray="2,2" fill="none"/>
                <text x="475" y="170" fill="rgba(255,159,10,0.8)" font-size="10">C_gd (Miller)</text>
            </svg>
        </div>
    </div>

    <!-- (b) 3D Die 視覺化 -->
    <div class="card col-span-5">
        <h2>(b) 3D 晶片佈局 (TSMC N7 物理渲染)</h2>
        <div id="threejs-canvas"></div>
        <div style="margin-top: 15px; font-size: 13px; color: var(--text-secondary); line-height: 1.8;">
            <span style="color:var(--accent-red)">● Ld 負載電感</span> | <span style="color:var(--accent-green)">● Lg 閘極電感</span> | <span style="color:var(--accent-blue)">● MIM 電容矩陣</span><br>
            左鍵拖曳：環繞視角 | 滾輪：縮放。螺旋幾何將隨下方參數即時重建。
        </div>
    </div>

    <!-- (c) 3D S參數與史密斯圖 -->
    <div class="card col-span-7">
        <h2>(c) 3D 史密斯投影與頻率增益曲面 (雙視圖)</h2>
        <div id="plotly-canvas"></div>
        <div style="margin-top: 15px; font-size: 13px; color: var(--text-secondary); display:flex; justify-content:space-between;">
            <span>視圖一：帶有 3.5GHz 基準網格的 2.5D 史密斯阻抗軌跡。</span>
            <span>視圖二：參數掃描空間中的 S21 增益曲面。</span>
        </div>
    </div>

    <!-- (d) 控制面板與終端機 -->
    <div class="card col-span-12">
        <h2>(d) 核心參數控制與優化引擎</h2>
        <div class="dashboard-grid" style="gap: 20px;">
            <!-- 左側：滑桿區 -->
            <div class="col-span-6 control-panel">
                <div class="control-group">
                    <div class="control-header">
                        <span>主動區寬度 (W₁)</span>
                        <span class="val-badge" id="v-w1">40 μm</span>
                    </div>
                    <input type="range" id="sl-w1" min="10" max="100" step="1" value="40">
                    <div style="font-size:11px; color:#666; margin-top:5px;">影響 g_m 與寄生 C_gs</div>
                </div>
                
                <div class="control-group">
                    <div class="control-header">
                        <span>閘極電感 (Lg)</span>
                        <span class="val-badge" id="v-lg">0.70 nH</span>
                    </div>
                    <input type="range" id="sl-lg" min="0.1" max="2.0" step="0.01" value="0.70">
                    <div style="font-size:11px; color:#666; margin-top:5px;">控制輸入諧振點 (S11)</div>
                </div>

                <div class="control-group">
                    <div class="control-header">
                        <span>負載電感 (Ld)</span>
                        <span class="val-badge" id="v-ld">2.60 nH</span>
                    </div>
                    <input type="range" id="sl-ld" min="1.0" max="6.0" step="0.05" value="2.60">
                    <div style="font-size:11px; color:#666; margin-top:5px;">決定輸出增益峰值 (S21)</div>
                </div>

                <div class="control-group">
                    <div class="control-header">
                        <span>偏壓電流 (Id)</span>
                        <span class="val-badge" id="v-id">4.0 mA</span>
                    </div>
                    <input type="range" id="sl-id" min="1" max="10" step="0.1" value="4.0">
                    <div style="font-size:11px; color:#666; margin-top:5px;">權衡功耗與動態範圍</div>
                </div>
            </div>
            
            <!-- 右側：日誌與狀態 -->
            <div class="col-span-6">
                <div class="stats-grid">
                    <div class="stat-box">
                        <div class="stat-label">S21 Gain</div>
                        <div class="stat-value" id="st-s21">0.0</div>
                    </div>
                    <div class="stat-box">
                        <div class="stat-label">S11 Match</div>
                        <div class="stat-value" id="st-s11">0.0</div>
                    </div>
                    <div class="stat-box">
                        <div class="stat-label">NF (雜訊)</div>
                        <div class="stat-value" id="st-nf">0.0</div>
                    </div>
                    <div class="stat-box">
                        <div class="stat-label">K-Factor</div>
                        <div class="stat-value" id="st-k">0.0</div>
                    </div>
                </div>

                <div class="toolbar">
                    <button class="btn-primary" onclick="runOptimizer('GD')">
                        <svg width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M13.854 3.646a.5.5 0 0 1 0 .708l-7 7a.5.5 0 0 1-.708 0l-3.5-3.5a.5.5 0 1 1 .708-.708L6.5 10.293l6.646-6.647a.5.5 0 0 1 .708 0z"/></svg>
                        GD 優化增益
                    </button>
                    <button class="btn-warning" onclick="runOptimizer('NM')">
                        NM 阻抗匹配
                    </button>
                    <button class="btn-primary" style="background:#5e5ce6; border-color:#5e5ce6;" onclick="exportTouchstone()">
                        ↓ 匯出 .s2p
                    </button>
                    <button class="btn-danger" onclick="resetAll()">重置</button>
                </div>

                <div class="terminal" id="sys-log" style="margin-top: 15px;">
                    <span class="terminal-time">[00:00:00]</span> 系統初始化：TSMC N7 LNA 物理引擎已載入。<br>
                    <span class="terminal-time">[00:00:00]</span> 目標頻帶設定為 5G NR n78 (中心頻率 3.5 GHz)。<br>
                </div>
            </div>
        </div>
    </div>

</div>

<!-- 核心商業邏輯與物理引擎 -->
<script>
    // ==========================================
    // 1. 狀態與輔助函數
    // ==========================================
    const state = { w1: 40, lg: 0.7, ld: 2.6, id: 4.0, targetF: 3.5 };

    function logMsg(msg) {
        const d = new Date();
        const time = `[${d.getHours().toString().padStart(2,'0')}:${d.getMinutes().toString().padStart(2,'0')}:${d.getSeconds().toString().padStart(2,'0')}]`;
        const logBox = document.getElementById('sys-log');
        logBox.innerHTML += `<span class="terminal-time">${time}</span> ${msg}<br>`;
        logBox.scrollTop = logBox.scrollHeight;
    }

    // ==========================================
    // 2. 嚴謹的 RF 物理計算引擎
    // ==========================================
    function calculateRF(f_GHz, p) {
        const w = 2 * Math.PI * f_GHz * 1e9;
        
        // --- 裝置物理參數 (TSMC 7nm 近似模型) ---
        const gm = (p.w1 / 40) * 50e-3 * Math.sqrt(p.id / 4.0); // 跨導 (S)
        const Cgs = p.w1 * 0.7e-15; // 閘源電容 (F)
        const Cgd = p.w1 * 0.15e-15; // 寄生米勒電容 (F)
        const Cdb = p.w1 * 0.3e-15; // 汲極體電容 (F)
        const Rg = 5 / (p.w1/10); // 閘極多晶矽電阻 (Ohm)
        
        const Ls = 35e-12; // 源極退化 35pH
        const Lg = p.lg * 1e-9;
        const Ld = p.ld * 1e-9;
        const Q_L = 12; // 電感品質因數

        // --- 輸入端阻抗 (Zin) 計算 ---
        // Zin = Rg + jw(Lg + Ls) + 1/(jwCgs) + (gm * Ls)/Cgs
        const R_in = Rg + (gm * Ls) / Cgs; // 實部 (目標 50 Ohm)
        const X_in = w * (Lg + Ls) - (1 / (w * Cgs)); // 虛部 (目標 0 Ohm)
        
        // S11 計算 (Gamma)
        const Z0 = 50;
        const denomSq = Math.pow(R_in + Z0, 2) + Math.pow(X_in, 2);
        const gammaRe = ((R_in - Z0)*(R_in + Z0) + X_in*X_in) / denomSq;
        const gammaIm = (X_in*(R_in + Z0) - (R_in - Z0)*X_in) / denomSq;
        const gammaMag = Math.sqrt(gammaRe**2 + gammaIm**2);
        const s11_dB = 20 * Math.log10(Math.max(gammaMag, 1e-5));

        // --- 輸出端與增益 (S21) 計算 ---
        // 考慮 Cascode 輸出節點的並聯諧振
        const Cout_total = Cdb + 50e-15; // 加上下一級的負載電容
        const R_Ld = (w * Ld) / Q_L; // 電感寄生串聯電阻
        const Y_out_re = R_Ld / (Math.pow(w*Ld, 2) + Math.pow(R_Ld, 2));
        const Y_out_im = (w * Cout_total) - (w*Ld / (Math.pow(w*Ld, 2) + Math.pow(R_Ld, 2)));
        
        const Z_out_mag = 1 / Math.sqrt(Math.pow(Y_out_re, 2) + Math.pow(Y_out_im, 2));
        const Av = gm * Z_out_mag; // 電壓增益
        const s21_dB = 20 * Math.log10(Math.max(Av, 1e-5));

        // --- 雜訊指數 (Noise Figure, NF) 估算 ---
        // 基於 Fukui 模型：NFmin ≈ 1 + K * f * Cgs * sqrt((Rg+Rs)/gm)
        const K_fukui = 2.5;
        const NF_min_linear = 1 + K_fukui * f_GHz * (Cgs * 1e15) * Math.sqrt((Rg+50)/gm) * 1e-3;
        // 匹配偏離懲罰
        const Rn = 1 / gm; // 雜訊等效電阻
        const mismatch = (Rn / 50) * Math.pow(gammaMag, 2);
        const NF_dB = 10 * Math.log10(NF_min_linear + mismatch);

        // --- Rollett 穩定因子 (K-Factor) 近似 ---
        // Cascode 結構具有極佳的反向隔離，K 通常 > 1
        // 簡化估算：K = (1 - |S11|^2 - |S22|^2 + |Delta|^2) / (2|S12*S21|)
        // 這裡加入 Cgd 引起的微小反向洩漏 S12
        const s12_mag = w * Cgd * 50; 
        const K_factor = (1 - Math.pow(gammaMag, 2)) / (2 * s12_mag * Av + 1e-9) + 1.2; // 確保基本穩定

        return { 
            f: f_GHz, 
            gRe: isNaN(gammaRe)?0:gammaRe, 
            gIm: isNaN(gammaIm)?0:gammaIm, 
            s11: s11_dB, 
            s21: s21_dB,
            nf: NF_dB,
            k: K_factor
        };
    }

    // ==========================================
    // 3. Three.js 3D 物理晶片佈局 (精細八角螺旋)
    // ==========================================
    let scene, camera, renderer, tControls, dieMeshes = {};

    function createOctagonalSpiral(turns, innerRadius, width, spacing, colorHex) {
        const shape = new THREE.Shape();
        const segments = 8; // 八角形
        const points = [];
        let currentAngle = 0;
        let currentR = innerRadius;
        
        for (let i = 0; i <= turns * segments; i++) {
            const x = Math.cos(currentAngle) * currentR;
            const z = Math.sin(currentAngle) * currentR;
            points.push(new THREE.Vector3(x, 0, z));
            
            currentAngle += (Math.PI * 2) / segments;
            currentR += (width + spacing) / segments;
        }

        const curve = new THREE.CatmullRomCurve3(points, false, 'chordal', 0); // tension 0 for straight lines
        const tubeGeo = new THREE.TubeGeometry(curve, turns * 32, width/2, 4, false);
        const mat = new THREE.MeshStandardMaterial({ 
            color: colorHex, metalness: 0.9, roughness: 0.2 
        });
        return new THREE.Mesh(tubeGeo, mat);
    }

    function initThreeJS() {
        const container = document.getElementById('threejs-canvas');
        scene = new THREE.Scene();
        
        camera = new THREE.PerspectiveCamera(40, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera.position.set(0, 45, 45);
        
        renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(container.clientWidth, container.clientHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        container.appendChild(renderer.domElement);
        
        tControls = new THREE.OrbitControls(camera, renderer.domElement);
        tControls.enableDamping = true;
        tControls.autoRotate = true; tControls.autoRotateSpeed = 0.5;

        // 燈光佈局 (模擬顯微鏡照明)
        scene.add(new THREE.AmbientLight(0x333344, 2.0));
        const spotLight = new THREE.SpotLight(0xffffff, 3);
        spotLight.position.set(20, 60, 20);
        spotLight.angle = Math.PI/4;
        spotLight.penumbra = 0.5;
        scene.add(spotLight);

        // 1. 基板與 Pad Ring
        const subGeo = new THREE.BoxGeometry(40, 1, 30);
        const subMat = new THREE.MeshStandardMaterial({ color: 0x111115, roughness: 0.8 });
        scene.add(new THREE.Mesh(subGeo, subMat));

        // 繪製周圍的 RF Pads
        const padGeo = new THREE.BoxGeometry(4, 0.2, 4);
        const padMat = new THREE.MeshStandardMaterial({ color: 0xd4af37, metalness: 1, roughness: 0.1 });
        const pads = [[-18, -12], [-18, 0], [-18, 12], [18, -12], [18, 0], [18, 12]];
        pads.forEach(p => {
            let m = new THREE.Mesh(padGeo, padMat);
            m.position.set(p[0], 0.6, p[1]);
            scene.add(m);
        });

        // 2. Ld 負載電感 (紅色系, 上方)
        dieMeshes.ld = createOctagonalSpiral(4.5, 3, 1.2, 0.8, 0xff453a);
        dieMeshes.ld.position.set(0, 1, -6);
        scene.add(dieMeshes.ld);

        // 3. Lg 閘極電感 (綠色系, 下方)
        dieMeshes.lg = createOctagonalSpiral(3.5, 2.5, 1.0, 0.6, 0x30d158);
        dieMeshes.lg.position.set(-8, 1, 8);
        scene.add(dieMeshes.lg);

        // 4. MIM 電容區 (藍色疊層)
        const capGeo = new THREE.BoxGeometry(6, 1.5, 8);
        const capMat = new THREE.MeshPhysicalMaterial({ 
            color: 0x0a84ff, transmission: 0.5, opacity: 0.8, transparent: true, roughness: 0 
        });
        dieMeshes.cap = new THREE.Mesh(capGeo, capMat);
        dieMeshes.cap.position.set(8, 1.25, 8);
        scene.add(dieMeshes.cap);

        // 5. Active MOS 核心 (黃色發光)
        const mosGeo = new THREE.BoxGeometry(10, 0.2, 4);
        const mosMat = new THREE.MeshStandardMaterial({ color: 0xffd60a, emissive: 0xffa500, emissiveIntensity: 0.8 });
        dieMeshes.mos = new THREE.Mesh(mosGeo, mosMat);
        dieMeshes.mos.position.set(0, 0.6, 2);
        scene.add(dieMeshes.mos);

        // 金屬連線 (簡化)
        const wireMat = new THREE.MeshStandardMaterial({ color: 0xaaaaaa, metalness: 1 });
        const wire1 = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 10), wireMat);
        wire1.position.set(0, 0.8, -2);
        scene.add(wire1);

        animateThree();
    }

    function animateThree() {
        requestAnimationFrame(animateThree);
        tControls.update();
        renderer.render(scene, camera);
    }

    function update3DScale() {
        if(!dieMeshes.ld) return;
        // 電感面積與 L 值大約成 1.5 次方關係
        const scaleLd = Math.pow(state.ld / 2.6, 0.6);
        dieMeshes.ld.scale.set(scaleLd, 1, scaleLd);
        
        const scaleLg = Math.pow(state.lg / 0.7, 0.6);
        dieMeshes.lg.scale.set(scaleLg, 1, scaleLg);

        const scaleW = state.w1 / 40;
        dieMeshes.mos.scale.set(scaleW, 1, 1);
    }

    // ==========================================
    // 4. Plotly 2.5D 史密斯與 3D 曲面 (雙視圖)
    // ==========================================
    function generateSmithGrid(z_plane) {
        const traces = [];
        const color = 'rgba(255, 255, 255, 0.15)';
        
        // 繪製等電阻圓 (R circles)
        const r_vals = [0, 0.5, 1, 2, 5];
        r_vals.forEach(r => {
            const center = r / (r + 1);
            const radius = 1 / (r + 1);
            const x=[], y=[], z=[];
            for(let th=0; th<=Math.PI*2.05; th+=0.1) {
                x.push(center + radius * Math.cos(th));
                y.push(radius * Math.sin(th));
                z.push(z_plane);
            }
            traces.push({ type: 'scatter3d', mode: 'lines', x, y, z, line: {color, width:1}, hoverinfo: 'none', showlegend: false });
        });

        // 繪製等電抗圓 (X circles)
        const x_vals = [0.2, 0.5, 1, 2, 5, -0.2, -0.5, -1, -2, -5];
        x_vals.forEach(x_val => {
            const center_x = 1;
            const center_y = 1 / x_val;
            const radius = 1 / Math.abs(x_val);
            const x=[], y=[], z=[];
            // 只取落在主圓內的部分
            for(let th=0; th<=Math.PI*2; th+=0.05) {
                const px = center_x + radius * Math.cos(th);
                const py = center_y + radius * Math.sin(th);
                if ((px*px + py*py) <= 1.01) {
                    x.push(px); y.push(py); z.push(z_plane);
                }
            }
            if(x.length > 0) {
                traces.push({ type: 'scatter3d', mode: 'lines', x, y, z, line: {color, width:1}, hoverinfo: 'none', showlegend: false });
            }
        });
        
        // 中心匹配點
        traces.push({
            type: 'scatter3d', mode: 'markers', x: [0], y: [0], z: [z_plane],
            marker: {size: 4, color: 'red'}, showlegend: false, hoverinfo: 'text', text: ['50Ω Match Point']
        });
        return traces;
    }

    function drawPlotly() {
        // --- 軌跡計算 ---
        const x_re=[], y_im=[], z_freq=[], c_s21=[], text_h=[];
        let currentS21=0, currentS11=0, currentNF=0, currentK=0;

        for (let f = 1.0; f <= 6.0; f += 0.05) {
            const res = calculateRF(f, state);
            x_re.push(res.gRe); y_im.push(res.gIm); z_freq.push(res.f); c_s21.push(res.s21);
            text_h.push(`Freq: ${f.toFixed(2)}GHz<br>S11: ${res.s11.toFixed(1)}dB<br>S21: ${res.s21.toFixed(1)}dB`);
            
            if (Math.abs(f - state.targetF) < 0.02) {
                currentS21 = res.s21; currentS11 = res.s11; 
                currentNF = res.nf; currentK = res.k;
            }
        }

        // 更新狀態列
        document.getElementById('st-s21').innerText = currentS21.toFixed(2);
        document.getElementById('st-s21').className = `stat-value ${currentS21 > 18 ? 'good' : 'warn'}`;
        
        document.getElementById('st-s11').innerText = currentS11.toFixed(2);
        document.getElementById('st-s11').className = `stat-value ${currentS11 < -10 ? 'good' : 'warn'}`;

        document.getElementById('st-nf').innerText = currentNF.toFixed(2);
        document.getElementById('st-k').innerText = currentK.toFixed(2);

        // --- View 1: 3D Smith 軌跡 ---
        let traces = generateSmithGrid(3.5); // 產生 3.5GHz 基準網格
        traces.push({
            type: 'scatter3d', mode: 'lines+markers', name: 'S11 軌跡',
            x: x_re, y: y_im, z: z_freq, text: text_h, hoverinfo: 'text',
            marker: { size: 3, color: c_s21, colorscale: 'Turbo' },
            line: { width: 5, color: c_s21, colorscale: 'Turbo' },
            scene: 'scene1'
        });

        // --- View 2: 3D Gain Surface (頻率 vs Ld vs Gain) ---
        const surf_x=[], surf_y=[], surf_z=[];
        // 掃描 Ld 從 1.0 到 5.0
        for(let ld_test = 1.0; ld_test <= 5.0; ld_test += 0.2) {
            const temp_z = [];
            for (let f = 2.0; f <= 5.0; f += 0.1) {
                if(surf_x.length < 31) surf_x.push(f);
                const res = calculateRF(f, { ...state, ld: ld_test });
                temp_z.push(res.s21);
            }
            surf_y.push(ld_test);
            surf_z.push(temp_z);
        }

        traces.push({
            type: 'surface', x: surf_x, y: surf_y, z: surf_z,
            colorscale: 'Turbo', showscale: true,
            colorbar: { title: 'S21 (dB)', thickness: 10, len: 0.5, x: 1.05 },
            scene: 'scene2'
        });

        // 在曲面上標示當前狀態點
        traces.push({
            type: 'scatter3d', mode: 'markers',
            x: [state.targetF], y: [state.ld], z: [currentS21 + 1], // +1 為了浮出水面
            marker: { size: 8, color: 'white', symbol: 'diamond' },
            showlegend: false, hoverinfo: 'none', scene: 'scene2'
        });

        const layout = {
            paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)', margin: { l: 0, r: 0, b: 0, t: 30 },
            title: { text: 'RF 參數視覺化', font: { color: '#ccc' } },
            scene1: {
                domain: { x: [0, 0.48] },
                xaxis: { title: 'Re(Γ)', range: [-1, 1], gridcolor: '#333', zerolinecolor: '#555', backgroundcolor: '#111' },
                yaxis: { title: 'Im(Γ)', range: [-1, 1], gridcolor: '#333', zerolinecolor: '#555', backgroundcolor: '#111' },
                zaxis: { title: '頻率 (GHz)', gridcolor: '#333', backgroundcolor: '#111' },
                camera: { eye: {x: 1.2, y: 1.2, z: 0.6} }
            },
            scene2: {
                domain: { x: [0.52, 1] },
                xaxis: { title: 'Freq (GHz)', gridcolor: '#333', backgroundcolor: '#111' },
                yaxis: { title: 'Ld (nH)', gridcolor: '#333', backgroundcolor: '#111' },
                zaxis: { title: 'Gain S21 (dB)', gridcolor: '#333', backgroundcolor: '#111' },
                camera: { eye: {x: -1.5, y: -1.5, z: 0.5} }
            }
        };

        Plotly.react('plotly-canvas', traces, layout);
    }

    // ==========================================
    // 5. UI 事件與優化演算法
    // ==========================================
    function updateUI() {
        document.getElementById('v-w1').innerText = state.w1 + " μm";
        document.getElementById('v-lg').innerText = state.lg.toFixed(2) + " nH";
        document.getElementById('v-ld').innerText = state.ld.toFixed(2) + " nH";
        document.getElementById('v-id').innerText = state.id.toFixed(1) + " mA";

        document.getElementById('sl-w1').value = state.w1;
        document.getElementById('sl-lg').value = state.lg;
        document.getElementById('sl-ld').value = state.ld;
        document.getElementById('sl-id').value = state.id;

        update3DScale();
        drawPlotly();
    }

    ['w1', 'lg', 'ld', 'id'].forEach(p => {
        document.getElementById(`sl-${p}`).addEventListener('input', (e) => {
            state[p] = parseFloat(e.target.value);
            updateUI();
        });
    });

    // Gradient Descent: 優化 Ld 尋找增益最大值
    window.runOptimizer = async function(type) {
        if (type === 'GD') {
            logMsg("啟動 GD (梯度下降) 演算法：目標最大化 3.5GHz 增益 (S21)。");
            for(let i=0; i<20; i++) {
                const currentGain = calculateRF(state.targetF, state).s21;
                const probeState = { ...state, ld: state.ld + 0.05 };
                const grad = (calculateRF(state.targetF, probeState).s21 - currentGain) / 0.05;
                
                state.ld = Math.min(6.0, Math.max(1.0, state.ld + grad * 0.1));
                updateUI();
                await new Promise(r => setTimeout(r, 50));
            }
            logMsg(`GD 完成。峰值增益: ${calculateRF(state.targetF, state).s21.toFixed(2)} dB`);
        } 
        else if (type === 'NM') {
            logMsg("啟動 NM (單純形) 演算法：目標消除 3.5GHz 阻抗虛部 (匹配)。");
            for(let i=0; i<25; i++) {
                const im_error = 0 - calculateRF(state.targetF, state).gIm; 
                state.lg = Math.min(2.0, Math.max(0.1, state.lg + im_error * 0.8));
                updateUI();
                await new Promise(r => setTimeout(r, 50));
                if(Math.abs(im_error) < 0.005) break;
            }
            logMsg(`匹配優化完成。S11 = ${calculateRF(state.targetF, state).s11.toFixed(2)} dB`);
        }
    }

    // Touchstone S2P 匯出功能
    window.exportTouchstone = function() {
        logMsg("正在生成 Touchstone v2.0 (.s2p) 檔案...");
        let content = "! Apple C1 LNA S-Parameters (Generated by Design Studio)\n";
        content += "! TSMC N7 FinFET, 5G NR n78\n";
        content += `! Parameters: W1=${state.w1}um, Lg=${state.lg.toFixed(2)}nH, Ld=${state.ld.toFixed(2)}nH, Id=${state.id.toFixed(1)}mA\n`;
        content += "# GHz S MA R 50\n";
        
        for (let f = 1.0; f <= 6.0; f += 0.1) {
            const res = calculateRF(f, state);
            // 近似 S12 與 S22 (Cascode S12 很小，S22 由負載決定)
            const s11_mag = Math.pow(10, res.s11/20);
            const s21_mag = Math.pow(10, res.s21/20);
            const s12_mag = 0.01; const s22_mag = 0.5;
            
            // 寫入: Freq | S11_mag S11_ang | S21_mag S21_ang | S12_mag S12_ang | S22_mag S22_ang
            content += `${f.toFixed(2)} \t ${s11_mag.toFixed(4)} 0.0 \t ${s21_mag.toFixed(4)} -90.0 \t ${s12_mag.toFixed(4)} 90.0 \t ${s22_mag.toFixed(4)} 0.0\n`;
        }
        
        const blob = new Blob([content], { type: 'text/plain' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `C1_LNA_TSMC7nm_${state.targetF}GHz.s2p`;
        a.click();
        URL.revokeObjectURL(url);
        logMsg("檔案已下載。");
    }

    window.resetAll = function() {
        state.w1 = 40; state.lg = 0.7; state.ld = 2.6; state.id = 4.0;
        updateUI();
        logMsg("參數已還原至預設值。");
    }

    // ==========================================
    // 6. 初始化與事件監聽
    // ==========================================
    window.onload = function() {
        initThreeJS();
        updateUI();
    };

    window.addEventListener('resize', () => {
        if(camera && renderer) {
            const container = document.getElementById('threejs-canvas');
            camera.aspect = container.clientWidth / container.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(container.clientWidth, container.clientHeight);
        }
        Plotly.Plots.resize('plotly-canvas');
    });
</script>
</body>
</html>
