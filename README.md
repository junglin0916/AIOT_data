<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AIoT智慧家庭：數據保衛戰</title>
    <style>
        :root {
            --primary: #2563eb;
            --success: #16a34a;
            --danger: #dc2626;
            --dark: #1e293b;
            --light: #f8fafc;
        }

        body {
            font-family: 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif, "Microsoft JhengHei";
            background-color: var(--dark);
            color: var(--light);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        #game-container {
            width: 100%;
            max-width: 800px;
            background: #0f172a;
            border: 3px solid var(--primary);
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
            box-sizing: border-box;
        }

        h1 {
            text-align: center;
            color: #38bdf8;
            margin-top: 0;
            font-size: 24px;
        }

        #dashboard {
            display: flex;
            justify-content: space-between;
            background: #1e293b;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
            font-size: 18px;
            font-weight: bold;
        }

        .stat-box {
            padding: 5px 10px;
            border-radius: 4px;
        }

        #score-box { color: #4ade80; }
        #hp-box { color: #f87171; }
        #timer-box { color: #fbbf24; }

        #game-canvas {
            width: 100%;
            height: 400px;
            background: #020617;
            border: 2px dashed #334155;
            border-radius: 8px;
            position: relative;
            overflow: hidden;
            cursor: crosshair;
        }

        .data-node {
            position: absolute;
            padding: 10px 15px;
            border-radius: 20px;
            font-weight: bold;
            font-size: 14px;
            cursor: pointer;
            user-select: none;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
            transition: transform 0.1s;
            animation: floatDown 4s linear forwards;
        }

        .data-node:hover {
            transform: scale(1.1);
        }

        /* 正常數據樣式 */
        .node-normal {
            background: linear-gradient(135deg, #10b981, #059669);
            border: 2px solid #34d399;
        }

        /* 異常數據樣式 */
        .node-dirty {
            background: linear-gradient(135deg, #ef4444, #dc2626);
            border: 2px solid #f87171;
        }

        #intro-screen, #end-screen {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            text-align: center;
            padding: 40px 20px;
        }

        .btn {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            margin-top: 20px;
            transition: background 0.2s;
        }

        .btn:hover {
            background-color: #1d4ed8;
        }

        .hidden {
            display: none !important;
        }

        ol {
            text-align: left;
            max-width: 500px;
            line-height: 1.6;
            margin-bottom: 20px;
        }

        @keyframes floatDown {
            0% { top: -60px; }
            100% { top: 410px; }
        }

        #knowledge-tip {
            margin-top: 15px;
            background: #1e293b;
            padding: 12px;
            border-left: 4px solid var(--primary);
            border-radius: 4px;
            font-size: 14px;
        }
    </style>
</head>
<body>

<div id="game-container">
    <h1>智能家庭 AIoT 數據防衛戰</h1>

    <div id="dashboard" class="hidden">
        <div class="stat-box" id="score-box">得分: <span id="score">0</span></div>
        <div class="stat-box" id="timer-box">倒數: <span id="timer">30</span>s</div>
        <div class="stat-box" id="hp-box">主機防禦力: <span id="hp">100</span>%</div>
    </div>

    <div id="game-canvas">
        <div id="intro-screen">
            <h3>【教學情境】</h3>
            <p>智慧家庭的 IoT 感測器正在回傳環境數據到 Scratch 主機。但是，網路中夾雜了許多<b>「髒數據（Dirty Data，如異常值、空白、文字錯誤）」</b>！</p>
            <h3>【核心任務】</h3>
            <ol>
                <li>當看到<b>【綠色正常數據】</b>（例如：25°C、濕度60%）：讓它順利通過，主機將成功儲存資料並得分！</li>
                <li>當看到<b>【紅色異常數據】</b>（例如：-999°C、室溫9999度、空白或文字錯誤）：<b>請立刻點擊它進行「資料清理」將其消滅！</b>如果讓異常數據混入主機，防禦力就會下降！</li>
            </ol>
            <button class="btn" onclick="startGame()">開始執行程式</button>
        </div>

        <div id="end-screen" class="hidden">
            <h2 id="end-title">遊戲結束</h2>
            <p id="end-desc"></p>
            <button class="btn" onclick="restartGame()">重新偵錯與分析</button>
        </div>
    </div>

    <div id="knowledge-tip">
        <b>💡 資訊科技小知識：</b>資料分析前必須進行「資料清理（Data Cleaning）」，清除極端值與格式錯誤。在 Scratch 中，我們常使用「如果 則」積木進行資料驗證（Data Validation），確保 AIoT 系統不會因為髒數據而誤開冷氣或引發警報！
    </div>
</div>

<script>
    const canvas = document.getElementById('game-canvas');
    const introScreen = document.getElementById('intro-screen');
    const endScreen = document.getElementById('end-screen');
    const dashboard = document.getElementById('dashboard');
    
    const scoreEl = document.getElementById('score');
    const timerEl = document.getElementById('timer');
    const hpEl = document.getElementById('hp');
    const endTitle = document.getElementById('end-title');
    const endDesc = document.getElementById('end-desc');

    let score = 0;
    let hp = 100;
    let timeLeft = 30;
    let gameInterval;
    let spawnInterval;
    let isGameRunning = false;

    // 數據庫池：模擬 AIoT 回傳的原始資料
    const normalData = ["溫度: 26°C", "濕度: 55%", "光線: 350lm", "溫度: 22°C", "濕度: 62%", "冷氣: ON", "除濕機: OFF"];
    const dirtyData = ["溫度: -999°C", "溫度: 9999°C", "濕度: 空白", "光線: ERROR", "濕度: --%", "冷氣: 壞掉了", "溫度: NaN"];

    function startGame() {
        introScreen.classList.add('hidden');
        endScreen.classList.add('hidden');
        dashboard.classList.remove('hidden');
        
        score = 0;
        hp = 100;
        timeLeft = 30;
        isGameRunning = true;
        
        scoreEl.innerText = score;
        hpEl.innerText = hp;
        timerEl.innerText = timeLeft;

        // 清除舞台上舊有的節點
        document.querySelectorAll('.data-node').forEach(node => node.remove());

        // 啟動計時器
        gameInterval = setInterval(() => {
            timeLeft--;
            timerEl.innerText = timeLeft;
            if (timeLeft <= 0) {
                endGame(true);
            }
        }, 1000);

        // 隨機生成數據節點
        spawnInterval = setInterval(spawnDataNode, 800);
    }

    function spawnDataNode() {
        if (!isGameRunning) return;

        const node = document.createElement('div');
        node.classList.add('data-node');

        // 隨機決定是正常數據(60%)還是異常髒數據(40%)
        const isDirty = Math.random() < 0.4;
        
        if (isDirty) {
            node.classList.add('node-dirty');
            node.innerText = dirtyData[Math.floor(Math.random() * dirtyData.length)];
            node.dataset.type = 'dirty';
        } else {
            node.classList.add('node-normal');
            node.innerText = normalData[Math.floor(Math.random() * normalData.length)];
            node.dataset.type = 'normal';
        }

        // 隨機X軸位置
        const randomX = Math.random() * (canvas.clientWidth - 120);
        node.style.left = `${randomX}px`;
        node.style.top = `-50px`;

        // 點擊事件處理
        node.addEventListener('mousedown', (e) => {
            e.stopPropagation();
            if (node.dataset.type === 'dirty') {
                score += 10;
                scoreEl.innerText = score;
                node.remove(); // 成功清理
            } else {
                // 誤殺正常數據，扣分並扣主機防禦力
                score = Math.max(0, score - 5);
                hp = Math.max(0, hp - 10);
                scoreEl.innerText = score;
                hpEl.innerText = hp;
                node.remove();
                checkHp();
            }
        });

        // 動態監聽動畫結束（當資料掉落到最底部，未被點擊時）
        node.addEventListener('animationend', () => {
            if (node.dataset.type === 'dirty') {
                // 髒數據漏過去了，危害到主機，扣防禦力
                hp = Math.max(0, hp - 20);
                hpEl.innerText = hp;
                checkHp();
            } else {
                // 正常數據順利通過，代表系統成功收集
                score += 5;
                scoreEl.innerText = score;
            }
            node.remove();
        });

        canvas.appendChild(node);
    }

    function checkHp() {
        if (hp <= 0) {
            endGame(false);
        }
    }

    function endGame(isWin) {
        isGameRunning = false;
        clearInterval(gameInterval);
        clearInterval(spawnInterval);
        
        document.querySelectorAll('.data-node').forEach(node => node.remove());
        endScreen.classList.remove('hidden');

        if (isWin) {
            endTitle.innerText = "🎉 數據防衛成功！";
            endTitle.style.color = "#4ade80";
            endDesc.innerHTML = `太棒了！你的 Scratch 資料驗證程式發揮了作用。<br>最終得分：<b>${score}</b> 分<br>主機防禦狀態：良好 (${hp}%)`;
        } else {
            endTitle.innerText = "❌ 系統當機崩潰！";
            endTitle.style.color = "#f87171";
            endDesc.innerHTML = `太多「髒數據」流入智慧家庭伺服器，導致冷氣與門鎖系統產生邏輯混亂！<br>請重新檢查並進行資料前處理。`;
        }
    }

    function restartGame() {
        startGame();
    }
</script>

</body>
</html>
