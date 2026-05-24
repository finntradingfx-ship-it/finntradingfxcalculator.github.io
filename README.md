# finntradingfxcalculator.github.io
fx entry calculator
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>外匯進場計算機</title>
    <style>
        :root {
            --bg-color: #121212;
            --card-bg: #1e1e1e;
            --text-color: #e0e0e0;
            --primary: #4CAF50;
            --danger: #f44336;
            --info: #2196F3;
            --border: #333;
        }
        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            margin: 0;
            padding: 15px;
            box-sizing: border-box;
        }
        h2 { text-align: center; margin-top: 5px; color: #fff; }
        
        .section-title {
            font-size: 16px;
            font-weight: bold;
            color: var(--info);
            margin: 15px 0 8px 5px;
            border-left: 4px solid var(--info);
            padding-left: 8px;
        }

        .card {
            background-color: var(--card-bg);
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }

        .tab-container {
            display: flex;
            background-color: #2c2c2c;
            border-radius: 8px;
            overflow: hidden;
            margin-bottom: 15px;
            border: 1px solid var(--border);
        }
        .tab {
            flex: 1;
            text-align: center;
            padding: 12px;
            font-size: 15px;
            font-weight: bold;
            color: #888;
            cursor: pointer;
            transition: background-color 0.3s, color 0.3s;
        }
        .tab.active {
            background-color: var(--info);
            color: #fff;
        }

        .form-group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            border-bottom: 1px solid var(--border);
            padding-bottom: 8px;
        }
        .form-group:last-child { border-bottom: none; margin-bottom: 0; padding-bottom: 0; }
        label { font-size: 14px; font-weight: 500; flex: 1; }
        input, select {
            flex: 1;
            background-color: #2c2c2c;
            border: 1px solid var(--border);
            color: #fff;
            padding: 8px;
            border-radius: 5px;
            font-size: 16px;
            text-align: right;
            width: 100%;
        }
        input:focus, select:focus { outline: none; border-color: var(--info); }
        
        .rr-display {
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #2c2c2c;
            border: 1px solid var(--border);
            border-radius: 5px;
            padding: 0 10px;
            color: var(--info);
            font-weight: bold;
            font-size: 16px;
            width: 60px;
            margin-left: 10px;
        }

        #pnlDisplay {
            flex: 1;
            text-align: right;
            font-size: 16px;
            font-weight: bold;
            padding: 8px 0;
        }

        .btn-calc {
            width: 100%;
            padding: 15px;
            font-size: 18px;
            font-weight: bold;
            background-color: var(--info);
            color: white;
            border: none;
            border-radius: 8px;
            margin-top: 10px;
            margin-bottom: 15px;
        }
        .btn-calc:active { background-color: #1976D2; }
        
        .result-box {
            background-color: #2a2a2a;
            border-left: 5px solid var(--info);
            padding: 15px;
            border-radius: 5px;
            margin-bottom: 10px;
        }
        .result-item {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            font-size: 16px;
        }
        .result-item:last-child { margin-bottom: 0; }
        .highlight { font-size: 20px; font-weight: bold; color: var(--info); }
        .warning { color: #FF9800; font-size: 12px; margin-top: 5px; font-weight: bold; }
        .danger-text { color: #ff5252; }
        .dir-buy { color: var(--primary); font-weight: bold; }
        .dir-sell { color: var(--danger); font-weight: bold; }
        
        .hard-sl-container {
            margin-top: 12px; 
            border-top: 1px dashed #555; 
            padding-top: 12px;
        }
    </style>
</head>
<body>

    <h2>精確進場計算機</h2>

    <div class="section-title">一、帳戶設定區</div>
    <div class="card">
        <div class="form-group">
            <label>帳戶初始資金 ($)</label>
            <input type="number" id="initialBalance" value="10000" oninput="updatePnL(); saveSettings()">
        </div>
        <div class="form-group">
            <label>現階段資金 ($)</label>
            <input type="number" id="currentBalance" value="10000" oninput="updatePnL(); saveSettings()">
        </div>
        <div class="form-group">
            <label>現階段總盈虧</label>
            <div id="pnlDisplay">$0.00 (0.00%)</div>
        </div>
        <div class="form-group">
            <label>帳戶槓桿 (1:X)</label>
            <input type="number" id="leverage" value="100" onchange="saveSettings()">
        </div>
        <div class="form-group">
            <label>可虧損金額 ($)</label>
            <input type="number" id="riskAmount" value="200" onchange="saveSettings()">
        </div>
        <div class="form-group">
            <label>單手來回手續費 ($)</label>
            <input type="number" id="commission" value="6" step="0.5" onchange="saveSettings()">
        </div>
    </div>

    <div class="section-title">二、策略區</div>
    <div class="card">
        <div class="tab-container">
            <div id="tabPattern" class="tab active" onclick="switchTab('pattern')">型態策略</div>
            <div id="tabSwing" class="tab" onclick="switchTab('swing')">波段策略</div>
        </div>

        <div class="form-group">
            <label>交易商品</label>
            <select id="instrument" onchange="updateInstrumentSettings(); saveSettings()">
                <option value="EURUSD">EURUSD (等歐美系)</option>
                <option value="USDJPY">USDJPY (等日圓系)</option>
                <option value="XAUUSD">XAUUSD (黃金)</option>
            </select>
        </div>
        <div class="form-group">
            <label>盤面止損點 (SSL)</label>
            <input type="number" id="ssl" step="0.00001" placeholder="輸入止損報價">
        </div>
        <div class="form-group">
            <label>盤面止盈點 (STP)</label>
            <input type="number" id="stp" step="0.00001" placeholder="輸入止盈報價">
        </div>
        
        <div id="patternRRBox" class="form-group">
            <label>目標風報比 (1:X)</label>
            <input type="number" id="rr" value="4" step="0.1" onchange="saveSettings()">
        </div>

        <div id="swingFibBox" class="form-group" style="display: none;">
            <label>斐波那契進場點</label>
            <div style="display: flex; flex: 1;">
                <select id="fibLevel" onchange="updateFibRR(); saveSettings()">
                    <option value="0.382">0.382</option>
                    <option value="0.500">0.500</option>
                    <option value="0.618">0.618</option>
                    <option value="0.705">0.705 (OTE)</option>
                    <option value="0.786">0.786</option>
                    <option value="0.886">0.886</option>
                </select>
                <div class="rr-display">
                    1:<span id="fibRRDisplay">1.6</span>
                </div>
            </div>
        </div>
        
        <div class="form-group">
            <label>點差 Spread (點)</label>
            <input type="number" id="spread" value="20" onchange="saveSettings()">
        </div>
        <div class="form-group">
            <label>浪點 Buffer (點)</label>
            <input type="number" id="wave" value="0" onchange="saveSettings()">
        </div>
    </div>

    <button class="btn-calc" onclick="calculate()">計算進場與手數</button>

    <div id="resultArea" style="display: none;">
        <div class="section-title">三、計算結果區</div>
        
        <div class="result-box" id="dirBox">
            <div class="result-item">
                <span>自動判定方向:</span>
                <span id="resDir">-</span>
            </div>
        </div>

        <div class="result-box">
            <div class="result-item">
                <span>最佳進場點位 (RE):</span>
                <span id="resRE" class="highlight">-</span>
            </div>
            <div class="result-item" style="font-size: 14px; color: #aaa;">
                <span>掛單價位 (Limit):</span>
                <span id="resLimit">-</span>
            </div>
            
            <div class="hard-sl-container">
                <div class="result-item" style="color: #ff5252; font-weight: bold;">
                    <span>2倍OL止損點 (硬止損位):</span>
                    <span id="resHardSL">-</span>
                </div>
                <div class="warning danger-text" style="font-weight: normal;" id="resHardSLWarning"></div>
            </div>
        </div>
        
        <div class="result-box" style="border-left-color: #FF9800;">
            <div class="result-item">
                <span>應下手數 (Lot):</span>
                <span id="resLot" class="highlight" style="color: #FF9800;">-</span>
            </div>
            <div class="result-item">
                <span id="resRealRiskRewardText" style="font-size: 14px; color: #aaa;">實際淨盈虧預估:</span>
                <span id="resRealRiskReward" style="font-size: 14px; color: #aaa;">-</span>
            </div>
            <div class="result-item" style="margin-top: 5px;">
                <span style="font-weight: bold; color: #e0e0e0;">真實淨風報比:</span>
                <span id="resActualRR" style="font-weight: bold; color: #e0e0e0;">-</span>
            </div>
            <div class="result-item" style="font-size: 14px; color: #e0e0e0; margin-top: 8px; border-top: 1px solid #333; padding-top: 8px; font-weight: bold;">
                <span>2R極端預付款比例:</span>
                <span id="resMargin">-</span>
            </div>
            <div class="warning" id="resWarning"></div>
        </div>
    </div>

    <script>
        let currentStrategy = 'pattern';

        window.onload = function() {
            const fields = ['initialBalance', 'currentBalance', 'leverage', 'riskAmount', 'commission', 'instrument', 'rr', 'spread', 'wave', 'fibLevel'];
            fields.forEach(id => {
                if(localStorage.getItem(id)) document.getElementById(id).value = localStorage.getItem(id);
            });
            
            const savedTab = localStorage.getItem('activeTab');
            if(savedTab) switchTab(savedTab);
            
            updatePnL(); 
            updateFibRR(); 
            updateInstrumentSettings();
        };

        function saveSettings() {
            const fields = ['initialBalance', 'currentBalance', 'leverage', 'riskAmount', 'commission', 'instrument', 'rr', 'spread', 'wave', 'fibLevel'];
            fields.forEach(id => localStorage.setItem(id, document.getElementById(id).value));
        }

        function updatePnL() {
            const initial = parseFloat(document.getElementById('initialBalance').value) || 0;
            const current = parseFloat(document.getElementById('currentBalance').value) || 0;
            const pnlDisplay = document.getElementById('pnlDisplay');

            if (initial === 0) {
                pnlDisplay.innerText = "$0.00 (0.00%)";
                pnlDisplay.style.color = "#e0e0e0";
                return;
            }

            const pnlAmount = current - initial;
            const pnlPercent = (pnlAmount / initial) * 100;
            
            let sign = pnlAmount > 0 ? "+" : "";
            pnlDisplay.innerText = `${sign}$${pnlAmount.toFixed(2)} (${sign}${pnlPercent.toFixed(2)}%)`;

            if (pnlAmount > 0) {
                pnlDisplay.style.color = "#4CAF50"; 
            } else if (pnlAmount < 0) {
                pnlDisplay.style.color = "#f44336"; 
            } else {
                pnlDisplay.style.color = "#e0e0e0"; 
            }
        }

        function switchTab(tab) {
            currentStrategy = tab;
            localStorage.setItem('activeTab', tab);
            
            if (tab === 'pattern') {
                document.getElementById('tabPattern').classList.add('active');
                document.getElementById('tabSwing').classList.remove('active');
                document.getElementById('patternRRBox').style.display = 'flex';
                document.getElementById('swingFibBox').style.display = 'none';
            } else {
                document.getElementById('tabSwing').classList.add('active');
                document.getElementById('tabPattern').classList.remove('active');
                document.getElementById('swingFibBox').style.display = 'flex';
                document.getElementById('patternRRBox').style.display = 'none';
                updateFibRR();
            }
        }

        function updateFibRR() {
            const fib = parseFloat(document.getElementById('fibLevel').value);
            const rawRR = fib / (1 - fib);
            const flooredRR = Math.floor(rawRR * 10) / 10;
            document.getElementById('fibRRDisplay').innerText = flooredRR.toFixed(1);
            return flooredRR;
        }

        function updateInstrumentSettings() {
            const inst = document.getElementById('instrument').value;
            const sslInput = document.getElementById('ssl');
            const stpInput = document.getElementById('stp');
            
            if (inst === 'XAUUSD') {
                sslInput.step = "0.01";
                stpInput.step = "0.01";
            } else if (inst === 'USDJPY') {
                sslInput.step = "0.001";
                stpInput.step = "0.001";
            } else {
                sslInput.step = "0.00001";
                stpInput.step = "0.00001";
            }
        }

        function calculate() {
            const currentBalance = parseFloat(document.getElementById('currentBalance').value) || 0;
            const leverage = parseFloat(document.getElementById('leverage').value) || 1; 
            const riskAmount = parseFloat(document.getElementById('riskAmount').value) || 0;
            const commission = parseFloat(document.getElementById('commission').value) || 0; 
            
            const inst = document.getElementById('instrument').value;
            const ssl = parseFloat(document.getElementById('ssl').value);
            const stp = parseFloat(document.getElementById('stp').value);
            
            const spreadPoints = parseFloat(document.getElementById('spread').value) || 0;
            const wavePoints = parseFloat(document.getElementById('wave').value) || 0;

            if(isNaN(ssl) || isNaN(stp)) {
                alert("請完整輸入止損(SSL)與止盈(STP)點位！");
                return;
            }
            if(riskAmount <= 0) {
                alert("可虧損金額必須大於 0！");
                return;
            }

            let rr = 0;
            if (currentStrategy === 'pattern') {
                rr = parseFloat(document.getElementById('rr').value) || 0;
            } else {
                rr = updateFibRR(); 
            }

            if (rr <= 0) {
                alert("風報比設定異常，請檢查！");
                return;
            }

            let direction = '';
            const dirEl = document.getElementById('resDir');
            const dirBox = document.getElementById('dirBox');
            
            if (stp > ssl) {
                direction = 'buy';
                dirEl.innerText = "做多 (Buy)";
                dirEl.className = "dir-buy";
                dirBox.style.borderLeftColor = "#4CAF50";
            } else if (stp < ssl) {
                direction = 'sell';
                dirEl.innerText = "做空 (Sell)";
                dirEl.className = "dir-sell";
                dirBox.style.borderLeftColor = "#f44336";
            } else {
                alert("錯誤：止盈點不可等於止損點！");
                return;
            }

            let decimal = 0.00001;
            let contractSize = 100000;
            let toFixedPoint = 5;

            if(inst === 'USDJPY') {
                decimal = 0.001;
                toFixedPoint = 3;
            } else if(inst === 'XAUUSD') {
                decimal = 0.01;
                contractSize = 100;
                toFixedPoint = 2;
            }

            const S = spreadPoints * decimal;
            const W = wavePoints * decimal;

            let RE, RTP, RSL, limitPrice;

            if (direction === 'buy') {
                RTP = stp;
                RSL = ssl - W; 
                RE = (RTP + (rr * RSL)) / (1 + rr);
                limitPrice = RE + S; 
            } else {
                RTP = stp + S; 
                RSL = ssl + S + W; 
                RE = (RTP + (rr * RSL)) / (1 + rr);
                limitPrice = RE; 
            }

            const lossDist = Math.abs(RE - RSL);
            const profitDist = Math.abs(RTP - RE);

            let hardSL = 0;
            if (direction === 'buy') {
                hardSL = RSL - lossDist; 
            } else {
                hardSL = RSL + lossDist; 
            }

            let lotSize = 0;
            
            // 計算手數
            if (inst === 'USDJPY') {
                lotSize = riskAmount / (((lossDist * contractSize) / RE) + commission);
            } else {
                lotSize = riskAmount / ((lossDist * contractSize) + commission);
            }
            lotSize = Math.floor(lotSize * 100) / 100;

            const totalCommission = commission * lotSize;

            // 計算 1R 純浮動虧損，用以推算最壞情況的淨值
            let floatLoss1R = 0;
            if (inst === 'USDJPY') {
                floatLoss1R = (lossDist * contractSize * lotSize) / RE;
            } else {
                floatLoss1R = lossDist * contractSize * lotSize;
            }

            // === 核心更新：在 2R 硬止損時的極端保證金計算 ===
            // 淨值 = 現階段資金 - (2倍的純浮動虧損) - 雙向手續費
            const equityAtHardSL = currentBalance - (floatLoss1R * 2) - totalCommission;
            
            // 重新計算在硬止損點位時，券商所要求的保證金
            let marginReqAtHardSL = 0;
            if (inst === 'USDJPY') {
                marginReqAtHardSL = (lotSize * contractSize) / leverage; 
            } else {
                // EURUSD 或 XAUUSD 的保證金會隨當下硬止損的報價而變動
                marginReqAtHardSL = (lotSize * contractSize * hardSL) / leverage;
            }

            // 得出最極端情況下的預付款比例
            const marginLevelAtHardSL = marginReqAtHardSL > 0 ? (equityAtHardSL / marginReqAtHardSL) * 100 : 0;
            // ===============================================

            let actualRisk = floatLoss1R + totalCommission;
            let actualReward = 0;
            
            if (inst === 'USDJPY') {
                actualReward = ((profitDist * contractSize * lotSize) / RE) - totalCommission;
            } else {
                actualReward = (profitDist * contractSize * lotSize) - totalCommission;
            }

            const actualRR = (actualRisk > 0 && actualReward > 0) ? (actualReward / actualRisk) : 0;

            document.getElementById('resRE').innerText = RE.toFixed(toFixedPoint);
            document.getElementById('resLimit').innerText = limitPrice.toFixed(toFixedPoint);
            
            document.getElementById('resHardSL').innerText = hardSL.toFixed(toFixedPoint);
            document.getElementById('resHardSLWarning').innerText = "🛡️ 提醒：若價格打到此位，即虧損 2R (含手續費)。請確實將平台硬止損掛於此處。";

            document.getElementById('resLot').innerText = lotSize > 0 ? lotSize + " 手" : "風險過小無法下單";
            
            document.getElementById('resRealRiskReward').innerText = "-$" + actualRisk.toFixed(2) + " / +$" + actualReward.toFixed(2);
            document.getElementById('resActualRR').innerText = "1 : " + actualRR.toFixed(2);
            
            // 輸出極端預付款比例與警告邏輯
            document.getElementById('resMargin').innerText = marginLevelAtHardSL.toFixed(2) + " %";
            
            const warningEl = document.getElementById('resWarning');
            if (marginLevelAtHardSL < 100) {
                warningEl.innerText = "💀 致命警告：若打到硬止損，預付款將低於 100%，券商必定提前強制平倉 (爆倉)！請務必降低風控手數。";
                document.getElementById('resMargin').style.color = "#ff5252"; // 極度危險的紅色
            } else if (marginLevelAtHardSL < 200) {
                warningEl.innerText = "⚠️ 警告：2R 硬止損時的預付款比例低於 200%，保證金過載風險偏高。";
                document.getElementById('resMargin').style.color = "#FF9800"; // 橘色警告
            } else {
                warningEl.innerText = "✅ 安全：即使打到硬止損，保證金依然充足。";
                document.getElementById('resMargin').style.color = "#4CAF50"; // 綠色安全
            }

            document.getElementById('resultArea').style.display = 'block';
        }
    </script>
</body>
</html>
