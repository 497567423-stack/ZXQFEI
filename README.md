<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>张晓琪的美食冒险（动作版-复活机制完整版）</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: "微软雅黑", sans-serif;
        }

        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background-color: #f0f8ff;
            padding: 10px;
            touch-action: none;
            overflow: hidden;
        }

        .game-container {
            position: relative;
            border: 5px solid #8b4513;
            border-radius: 10px;
            background-color: #e8f4f8;
            max-width: 100%;
            overflow: hidden;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
        }

        #gameCanvas {
            display: block;
            width: 100%;
            height: auto;
        }

        /* 动作游戏UI优化 */
        .game-hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 10px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
            z-index: 7;
        }

        .hud-item {
            background-color: rgba(0,0,0,0.7);
            color: white;
            padding: 8px 12px;
            border-radius: 8px;
            font-size: 14px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .cooldown-bar {
            width: 80px;
            height: 8px;
            background-color: #333;
            border-radius: 4px;
            overflow: hidden;
        }

        .cooldown-fill {
            height: 100%;
            background-color: #4CAF50;
            width: 0%;
            transition: width 0.05s linear;
        }

        .game-info {
            margin-top: 15px;
            font-size: 18px;
            color: #333;
            text-align: center;
            max-width: 800px;
        }

        .controls {
            margin-top: 10px;
            font-size: 14px;
            color: #666;
            text-align: center;
        }

        /* 动作特效样式 */
        .hit-effect {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(255,69,0,0.3);
            display: none;
            z-index: 5;
            animation: hitFlash 0.2s ease-out;
        }

        @keyframes hitFlash {
            0% { opacity: 1; }
            100% { opacity: 0; }
        }

        .dash-effect {
            position: absolute;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background-color: rgba(0,191,255,0.5);
            display: none;
            z-index: 6;
            animation: dashFade 0.3s ease-out forwards;
        }

        @keyframes dashFade {
            0% { opacity: 1; transform: scale(1); }
            100% { opacity: 0; transform: scale(2); }
        }

        /* 提示文字样式增强 */
        .damage-hint {
            position: absolute;
            color: red;
            font-size: 22px;
            font-weight: bold;
            pointer-events: none;
            z-index: 9;
            animation: damagePop 0.8s ease-out forwards;
            text-shadow: 0 0 5px black;
        }

        .gain-hint {
            position: absolute;
            color: #4CAF50;
            font-size: 22px;
            font-weight: bold;
            pointer-events: none;
            z-index: 9;
            animation: gainPop 0.8s ease-out forwards;
            text-shadow: 0 0 5px black;
        }

        .combo-hint {
            position: absolute;
            color: #FFD700;
            font-size: 24px;
            font-weight: bold;
            pointer-events: none;
            z-index: 9;
            animation: comboPop 1s ease-out forwards;
            text-shadow: 0 0 8px black;
        }

        @keyframes damagePop {
            0% { opacity: 1; transform: translateY(0) scale(1); }
            50% { transform: translateY(-15px) scale(1.2); }
            100% { opacity: 0; transform: translateY(-30px) scale(1); }
        }

        @keyframes gainPop {
            0% { opacity: 1; transform: translateY(0) scale(1); }
            50% { transform: translateY(-15px) scale(1.2); }
            100% { opacity: 0; transform: translateY(-30px) scale(1); }
        }

        @keyframes comboPop {
            0% { opacity: 1; transform: translateY(0) scale(1); }
            50% { transform: translateY(-20px) scale(1.5); }
            100% { opacity: 0; transform: translateY(-40px) scale(1); }
        }

        /* 复活倒计时样式 */
        .respawn-timer {
            position: absolute;
            color: #FFD700;
            font-size: 20px;
            font-weight: bold;
            text-shadow: 0 0 5px black;
            pointer-events: none;
            z-index: 10;
            animation: timerPulse 1s infinite;
        }

        @keyframes timerPulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        /* 游戏结束弹窗优化 */
        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.95);
            color: white;
            padding: 40px 30px;
            border-radius: 20px;
            font-size: 28px;
            text-align: center;
            display: none;
            z-index: 20;
            min-width: 85%;
            box-shadow: 0 0 40px rgba(255, 215, 0, 0.8);
        }

        .game-win {
            background-color: rgba(24, 134, 24, 0.95);
            box-shadow: 0 0 40px rgba(144, 238, 144, 0.9);
        }

        .stats-container {
            margin-top: 20px;
            text-align: left;
            font-size: 16px;
            background-color: rgba(0,0,0,0.3);
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
        }

        .restart-btn, .rank-btn {
            margin-top: 10px;
            padding: 12px 24px;
            font-size: 20px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            touch-action: manipulation;
            margin-right: 10px;
            transition: all 0.2s ease;
        }

        .restart-btn {
            background-color: #4CAF50;
            color: white;
        }

        .restart-btn:hover {
            background-color: #45a049;
            transform: scale(1.05);
        }

        .rank-btn {
            background-color: #2196F3;
            color: white;
        }

        .rank-btn:hover {
            background-color: #0b7dda;
            transform: scale(1.05);
        }

        /* 移动端虚拟摇杆和动作按钮 */
        .mobile-controls {
            position: fixed;
            bottom: 20px;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            pointer-events: none;
            z-index: 8;
        }

        .joystick-container {
            width: 120px;
            height: 120px;
            position: relative;
            pointer-events: auto;
        }

        .joystick-base {
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.3);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .joystick-knob {
            width: 60px;
            height: 60px;
            background-color: rgba(255,255,255,0.8);
            border-radius: 50%;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            touch-action: none;
        }

        .action-buttons {
            display: flex;
            gap: 15px;
            pointer-events: auto;
        }

        .attack-btn, .dash-btn {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            color: white;
            font-size: 16px;
            border: none;
            display: flex;
            align-items: center;
            justify-content: center;
            touch-action: manipulation;
            transition: all 0.1s ease;
        }

        .attack-btn {
            background-color: rgba(255,0,0,0.7);
        }

        .attack-btn:active {
            background-color: rgba(255,0,0,0.9);
            transform: scale(0.95);
        }

        .dash-btn {
            background-color: rgba(0,191,255,0.7);
        }

        .dash-btn:active {
            background-color: rgba(0,191,255,0.9);
            transform: scale(0.95);
        }

        /* 通关庆祝特效增强 */
        .celebration {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 8;
            display: none;
        }

        .confetti {
            position: absolute;
            width: 10px;
            height: 10px;
            background-color: #ff0;
            opacity: 0.8;
            animation: confettiFall linear infinite;
        }

        @keyframes confettiFall {
            0% { transform: translateY(-10px) rotate(0deg); opacity: 1; }
            100% { transform: translateY(600px) rotate(720deg); opacity: 0; }
        }

        /* 进度条 */
        .progress-container {
            width: 80%;
            height: 20px;
            background-color: #ddd;
            border-radius: 10px;
            margin: 10px auto 0;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background-color: #4CAF50;
            width: 10%;
            border-radius: 10px;
            transition: width 0.3s ease;
        }

        /* 响应式适配 */
        @media (max-width: 768px) {
            .game-info {
                font-size: 16px;
            }
            .controls {
                font-size: 12px;
            }
            .game-over {
                font-size: 22px;
                padding: 30px 20px;
            }
            .restart-btn, .rank-btn {
                font-size: 18px;
                padding: 10px 20px;
            }
            .damage-hint, .gain-hint {
                font-size: 18px;
            }
            .combo-hint {
                font-size: 20px;
            }
        }

        @media (min-width: 769px) {
            .mobile-controls {
                display: none;
            }
        }
    </style>
</head>
<body>
    <div class="game-container">
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        
        <!-- 动作游戏HUD -->
        <div class="game-hud">
            <div class="hud-item">
                体重: <span id="weight">100</span>斤
            </div>
            <div class="hud-item">
                连击: <span id="combo">0</span>
            </div>
            <div class="hud-item">
                闪避
                <div class="cooldown-bar">
                    <div class="cooldown-fill" id="dashCooldown"></div>
                </div>
            </div>
        </div>
        
        <div class="hit-effect" id="hitEffect"></div>
        <div class="dash-effect" id="dashEffect"></div>
        
        <!-- 游戏结束弹窗 -->
        <div class="game-over" id="gameOver">
            <p id="gameOverText"></p>
            <div class="progress-container">
                <div class="progress-bar" id="progressBar"></div>
            </div>
            <div class="stats-container" id="gameStats">
                游戏时长: <span id="playTime">0</span>秒<br>
                吃掉鸡腿: <span id="eatenFood">0</span>个<br>
                击败冯川: <span id="defeatedBoss">0</span>个<br>
                最高连击: <span id="maxCombo">0</span>连<br>
                闪避次数: <span id="dashCount">0</span>次<br>
                总攻击次数: <span id="attackCount">0</span>次<br>
                冯川复活: <span id="respawnCount">0</span>次
            </div>
            <div>
                <button class="restart-btn" onclick="restartGame()">重新挑战</button>
                <button class="rank-btn" onclick="showRank()">查看排行榜</button>
            </div>
        </div>
        
        <!-- 通关庆祝特效 -->
        <div class="celebration" id="celebration"></div>
    </div>
    
    <div class="game-info">
        🎮 动作版玩法：方向键/摇杆移动 | 空格键攻击 | Shift键闪避 | 攻击有硬直 | 闪避无敌 | 连击涨体重加成
    </div>
    
    <div class="controls">
        🖥️ 电脑端：↑↓←→移动 | 空格攻击 | Shift闪避 | 📱 手机端：左摇杆移动 | 红键攻击 | 蓝键闪避 | 
        🍗 鸡腿+5斤(连击>5时+8斤) | 👹 冯川-2斤(未击败)/-5斤(击败) | ⚡ 闪避冷却3秒 | 🎯 连击越高奖励越多 |
        ⏳ 冯川击败后5秒复活 | 🔥 复活后恢复3点生命值 | 🚫 最多复活3次
    </div>

    <!-- 移动端动作控制按钮 -->
    <div class="mobile-controls">
        <div class="joystick-container" id="joystickContainer">
            <div class="joystick-base"></div>
            <div class="joystick-knob" id="joystickKnob"></div>
        </div>
        <div class="action-buttons">
            <button class="attack-btn" id="attackBtn" ontouchstart="handleMobileAttackStart()" ontouchend="handleMobileAttackEnd()">攻击</button>
            <button class="dash-btn" id="dashBtn" ontouchstart="handleMobileDashStart()" ontouchend="handleMobileDashEnd()">闪避</button>
        </div>
    </div>

    <script>
        // 获取DOM元素
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const weightDisplay = document.getElementById('weight');
        const comboDisplay = document.getElementById('combo');
        const dashCooldownBar = document.getElementById('dashCooldown');
        const hitEffect = document.getElementById('hitEffect');
        const dashEffect = document.getElementById('dashEffect');
        const gameOverScreen = document.getElementById('gameOver');
        const gameOverText = document.getElementById('gameOverText');
        const progressBar = document.getElementById('progressBar');
        const celebration = document.getElementById('celebration');
        const gameContainer = document.querySelector('.game-container');
        
        // 统计数据DOM
        const playTimeDisplay = document.getElementById('playTime');
        const eatenFoodDisplay = document.getElementById('eatenFood');
        const defeatedBossDisplay = document.getElementById('defeatedBoss');
        const maxComboDisplay = document.getElementById('maxCombo');
        const dashCountDisplay = document.getElementById('dashCount');
        const attackCountDisplay = document.getElementById('attackCount');
        const respawnCountDisplay = document.getElementById('respawnCount');

        // 动作游戏核心配置（新增复活机制）
        const GAME_CONFIG = {
            // 基础属性
            basePigSize: 60,
            maxPigSize: 180,
            foodSize: 40,
            bossSize: 60,
            moveSpeed: 6,          
            dashSpeed: 15,         
            bossMoveSpeed: 2.2,    
            bossAcceleration: 0.1, 
            // 战斗属性
            initialWeight: 100,
            winWeight: 1000,
            foodWeightGain: 5,
            foodWeightGainCombo: 8,
            bossWeightLoss: 5,
            bossDamageLoss: 2,
            // 动作系统
            attackRange: 90,
            attackCd: 600,         
            attackStun: 150,       
            dashCd: 3000,          
            dashDuration: 150,     
            comboTime: 2000,       
            comboBonusThreshold: 5,
            // 生成配置
            foodCount: 25,
            bossCount: 4,          
            bossHp: 3,             
            attackDamage: 1,
            foodRefreshDelay: 800, 
            damageHintCd: 400,
            // 复活机制（核心新增）
            respawnTime: 5000,     // 复活倒计时5秒
            maxRespawnTimes: 3,    // 单个冯川最多复活3次
            respawnAnimation: true // 显示复活倒计时文字
        };

        // 游戏状态
        let gameState = {
            pig: { 
                x: canvas.width/2, 
                y: canvas.height/2, 
                dx: 0, 
                dy: 0,
                isAttacking: false,
                isDashing: false,
                isStunned: false,
                dashDirection: {x: 0, y: 0}
            },
            foods: [],
            bosses: [],
            weight: GAME_CONFIG.initialWeight,
            gameOver: false,
            startTime: 0,
            playTime: 0,
            // 动作统计
            eatenFoodCount: 0,
            defeatedBossCount: 0,
            comboCount: 0,
            maxComboCount: 0,
            dashCount: 0,
            attackCount: 0,
            respawnCount: 0,       // 总复活次数统计
            lastAttackTime: 0,
            lastDashTime: 0,
            lastComboTime: 0,
            // 控制状态
            keys: {
                ArrowUp: false,
                ArrowDown: false,
                ArrowLeft: false,
                ArrowRight: false,
                Space: false,
                Shift: false
            },
            joystick: {
                isActive: false,
                startX: 0,
                startY: 0,
                currentX: 0,
                currentY: 0,
                maxDistance: 60
            },
            lastDamageTime: 0
        };

        // 排行榜数据
        let rankData = JSON.parse(localStorage.getItem('zhangxiaoqi_action_rank')) || [];

        // 计算当前角色尺寸
        function getCurrentPigSize() {
            const weightRatio = (gameState.weight - GAME_CONFIG.initialWeight) / (GAME_CONFIG.winWeight - GAME_CONFIG.initialWeight);
            const clampedRatio = Math.max(0, Math.min(1, weightRatio));
            return GAME_CONFIG.basePigSize + clampedRatio * (GAME_CONFIG.maxPigSize - GAME_CONFIG.basePigSize);
        }

        // 计算攻击范围
        function getCurrentAttackRange() {
            const pigSizeRatio = getCurrentPigSize() / GAME_CONFIG.basePigSize;
            return GAME_CONFIG.attackRange * pigSizeRatio;
        }

        // 更新UI（冷却条、进度条、连击数）
        function updateUI() {
            const progress = (gameState.weight / GAME_CONFIG.winWeight) * 100;
            progressBar.style.width = `${progress}%`;
            
            const now = Date.now();
            const dashCdRemaining = Math.max(0, GAME_CONFIG.dashCd - (now - gameState.lastDashTime));
            const dashCdPercent = (dashCdRemaining / GAME_CONFIG.dashCd) * 100;
            dashCooldownBar.style.width = `${100 - dashCdPercent}%`;
            
            weightDisplay.textContent = gameState.weight;
            comboDisplay.textContent = gameState.comboCount;
            
            // 连击超时重置
            if (now - gameState.lastComboTime > GAME_CONFIG.comboTime && gameState.comboCount > 0) {
                gameState.comboCount = 0;
            }
        }

        // 初始化食物
        function initFoods() {
            gameState.foods = [];
            for (let i = 0; i < GAME_CONFIG.foodCount; i++) {
                addNewFood();
            }
        }

        function addNewFood() {
            const food = {
                x: Math.random() * (canvas.width - GAME_CONFIG.foodSize),
                y: Math.random() * (canvas.height - GAME_CONFIG.foodSize),
                eaten: false,
                spawnTime: Date.now()
            };
            gameState.foods.push(food);
        }

        function refreshFood(foodIndex) {
            setTimeout(() => {
                if (!gameState.gameOver) {
                    gameState.foods[foodIndex] = {
                        x: Math.random() * (canvas.width - GAME_CONFIG.foodSize),
                        y: Math.random() * (canvas.height - GAME_CONFIG.foodSize),
                        eaten: false,
                        spawnTime: Date.now()
                    };
                }
            }, GAME_CONFIG.foodRefreshDelay);
        }

        // 初始化BOSS（增加复活相关属性）
        function initBosses() {
            gameState.bosses = [];
            for (let i = 0; i < GAME_CONFIG.bossCount; i++) {
                gameState.bosses.push({
                    x: Math.random() * (canvas.width - GAME_CONFIG.bossSize),
                    y: Math.random() * (canvas.height - GAME_CONFIG.bossSize),
                    eaten: false,
                    defeated: false,
                    hp: GAME_CONFIG.bossHp,
                    hitFrame: 0,
                    moveSpeed: GAME_CONFIG.bossMoveSpeed,
                    aggro: false,
                    aggroRange: 200,
                    respawnTimer: null,  // 复活计时器
                    respawnTimes: 0,     // 已复活次数
                    respawnTimeLeft: 0   // 剩余复活时间
                });
            }
        }

        // 显示提示文字
        function showHint(x, y, text, className) {
            const hint = document.createElement('div');
            hint.className = className;
            hint.textContent = text;
            hint.style.left = `${x}px`;
            hint.style.top = `${y}px`;
            gameContainer.appendChild(hint);
            
            setTimeout(() => {
                if (hint.parentNode) {
                    gameContainer.removeChild(hint);
                }
            }, 800);
        }

        // 显示复活倒计时文字
        function showRespawnTimer(boss, timeLeft) {
            if (!GAME_CONFIG.respawnAnimation || boss.respawnTimes >= GAME_CONFIG.maxRespawnTimes) return;
            
            const timer = document.createElement('div');
            timer.className = 'respawn-timer';
            timer.textContent = `复活 ${Math.ceil(timeLeft / 1000)}s`;
            timer.id = `respawnTimer_${Date.now()}`;
            timer.style.left = `${boss.x + GAME_CONFIG.bossSize/2}px`;
            timer.style.top = `${boss.y - 20}px`;
            gameContainer.appendChild(timer);
            
            setTimeout(() => {
                const timerEl = document.getElementById(timer.id);
                if (timerEl && timerEl.parentNode) {
                    timerEl.parentNode.removeChild(timerEl);
                }
            }, 1000);
        }

        // BOSS复活核心函数
        function respawnBoss(boss, index) {
            if (boss.respawnTimes >= GAME_CONFIG.maxRespawnTimes) return;
            
            // 清除旧计时器
            if (boss.respawnTimer) {
                clearTimeout(boss.respawnTimer);
                boss.respawnTimer = null;
            }

            // 倒计时更新函数
            let timeLeft = GAME_CONFIG.respawnTime;
            const updateTimer = () => {
                if (boss.defeated && !gameState.gameOver) {
                    showRespawnTimer(boss, timeLeft);
                    timeLeft -= 1000;
                    if (timeLeft > 0) {
                        setTimeout(updateTimer, 1000);
                    }
                }
            };
            updateTimer();
            
            // 设置复活定时器
            boss.respawnTimer = setTimeout(() => {
                if (gameState.gameOver) return;

                // 随机生成复活位置，远离玩家
                let newX, newY;
                const currentSize = getCurrentPigSize();
                const pigCenterX = gameState.pig.x + currentSize/2;
                const pigCenterY = gameState.pig.y + currentSize/2;
                
                do {
                    newX = Math.random() * (canvas.width - GAME_CONFIG.bossSize);
                    newY = Math.random() * (canvas.height - GAME_CONFIG.bossSize);
                } while (Math.hypot(newX + GAME_CONFIG.bossSize/2 - pigCenterX, 
                                   newY + GAME_CONFIG.bossSize/2 - pigCenterY) < 150);
                
                // 重置BOSS状态
                boss.x = newX;
                boss.y = newY;
                boss.defeated = false;
                boss.hp = GAME_CONFIG.bossHp;
                boss.aggro = false;
                boss.respawnTimes++;
                gameState.respawnCount++;
                respawnCountDisplay.textContent = gameState.respawnCount;
                
                // 复活提示
                showHint(newX + GAME_CONFIG.bossSize/2, newY, '冯川复活!', 'combo-hint');
            }, GAME_CONFIG.respawnTime);
        }

        // 初始化游戏
        function initGame() {
            gameState = {
                pig: { 
                    x: canvas.width/2, 
                    y: canvas.height/2, 
                    dx: 0, 
                    dy: 0,
                    isAttacking: false,
                    isDashing: false,
                    isStunned: false,
                    dashDirection: {x: 0, y: 0}
                },
                foods: [],
                bosses: [],
                weight: GAME_CONFIG.initialWeight,
                gameOver: false,
                startTime: Date.now(),
                playTime: 0,
                eatenFoodCount: 0,
                defeatedBossCount: 0,
                comboCount: 0,
                maxComboCount: 0,
                dashCount: 0,
                attackCount: 0,
                respawnCount: 0,
                lastAttackTime: 0,
                lastDashTime: 0,
                lastComboTime: 0,
                keys: {
                    ArrowUp: false,
                    ArrowDown: false,
                    ArrowLeft: false,
                    ArrowRight: false,
                    Space: false,
                    Shift: false
                },
                joystick: {
                    isActive: false,
                    startX: 0,
                    startY: 0,
                    currentX: 0,
                    currentY: 0,
                    maxDistance: 60
                },
                lastDamageTime: 0
            };
            
            initFoods();
            initBosses();
            gameOverScreen.style.display = 'none';
            gameOverScreen.classList.remove('game-win');
            celebration.style.display = 'none';
            resetJoystick();
            updateUI();
            gameLoop();
        }

        // 绘制角色
        function drawPig() {
            const currentSize = getCurrentPigSize();
            const { x, y, isAttacking, isDashing, isStunned } = gameState.pig;
            const pigHalf = currentSize / 2;
            const headX = x + pigHalf;
            const headY = y + pigHalf;
            const attackRange = getCurrentAttackRange();
            
            if (isAttacking) {
                ctx.fillStyle = 'rgba(255,0,0,0.3)';
                ctx.beginPath();
                ctx.arc(headX, headY, attackRange/2, 0, Math.PI * 2);
                ctx.fill();
            }
            
            if (isDashing) {
                ctx.shadowColor = 'rgba(0,191,255,0.8)';
                ctx.shadowBlur = 20;
            } else if (isStunned) {
                ctx.shadowColor = 'rgba(255,69,0,0.8)';
                ctx.shadowBlur = 10;
            }
            
            ctx.fillStyle = isDashing ? '#ffd700' : '#f8c471';
            const bodyScale = isAttacking ? 1.1 : (isDashing ? 0.9 : 1);
            ctx.beginPath();
            ctx.ellipse(headX, headY + 10 * (currentSize / GAME_CONFIG.basePigSize), 
                       pigHalf * bodyScale, (pigHalf - 5) * bodyScale, 0, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = '#f8c471';
            ctx.beginPath();
            ctx.arc(headX, headY - 15 * (currentSize / GAME_CONFIG.basePigSize), 
                   (pigHalf - 10) * bodyScale, 0, Math.PI * 2);
            ctx.fill();
            
            const earAngle = isAttacking ? 0 : (isDashing ? Math.PI/6 : Math.PI/4);
            const earSize = isAttacking ? 18 : 12;
            const scaledEarSize = earSize * (currentSize / GAME_CONFIG.basePigSize);
            
            ctx.fillStyle = '#f5b041';
            ctx.beginPath();
            ctx.ellipse(headX - 18 * (currentSize / GAME_CONFIG.basePigSize), 
                       headY - 25 * (currentSize / GAME_CONFIG.basePigSize), 
                       scaledEarSize, scaledEarSize + 5 * (currentSize / GAME_CONFIG.basePigSize), 
                       earAngle, 0, Math.PI * 2);
            ctx.fill();
            ctx.beginPath();
            ctx.ellipse(headX + 18 * (currentSize / GAME_CONFIG.basePigSize), 
                       headY - 25 * (currentSize / GAME_CONFIG.basePigSize), 
                       scaledEarSize, scaledEarSize + 5 * (currentSize / GAME_CONFIG.basePigSize), 
                       -earAngle, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = 'black';
            const eyeScale = isAttacking ? 1.5 : (isDashing ? 0.5 : 1);
            const eyeSize = 3 * eyeScale * (currentSize / GAME_CONFIG.basePigSize);
            
            ctx.beginPath();
            ctx.arc(headX - 10 * (currentSize / GAME_CONFIG.basePigSize), 
                   headY - 18 * (currentSize / GAME_CONFIG.basePigSize), eyeSize, 0, Math.PI * 2);
            ctx.fill();
            ctx.beginPath();
            ctx.arc(headX + 10 * (currentSize / GAME_CONFIG.basePigSize), 
                   headY - 18 * (currentSize / GAME_CONFIG.basePigSize), eyeSize, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.shadowColor = 'transparent';
            ctx.shadowBlur = 0;
            
            if (gameState.comboCount > 0) {
                ctx.fillStyle = '#FFD700';
                ctx.font = `${20 + gameState.comboCount}px Arial`;
                ctx.fontWeight = 'bold';
                ctx.textAlign = 'center';
                ctx.fillText(`${gameState.comboCount}连`, headX, headY - pigHalf - 10);
            }
        }

        // 绘制食物
        function drawFoods() {
            gameState.foods.forEach(food => {
                if (!food.eaten) {
                    const centerX = food.x + GAME_CONFIG.foodSize/2;
                    const centerY = food.y + GAME_CONFIG.foodSize/2;
                    
                    if (gameState.comboCount >= GAME_CONFIG.comboBonusThreshold) {
                        ctx.fillStyle = 'rgba(255,215,0,0.2)';
                        ctx.beginPath();
                        ctx.arc(centerX, centerY, GAME_CONFIG.foodSize, 0, Math.PI * 2);
                        ctx.fill();
                    }

                    ctx.fillStyle = '#d2b48c';
                    ctx.beginPath();
                    ctx.moveTo(centerX + 10, centerY - 5);
                    ctx.lineTo(centerX + 20, centerY - 10);
                    ctx.lineTo(centerX + 18, centerY - 5);
                    ctx.lineTo(centerX + 10, centerY);
                    ctx.fill();

                    ctx.fillStyle = '#ffcc66';
                    ctx.beginPath();
                    ctx.ellipse(centerX, centerY, GAME_CONFIG.foodSize/2, GAME_CONFIG.foodSize/3, 0, 0, Math.PI * 2);
                    ctx.fill();

                    ctx.fillStyle = '#ff9933';
                    ctx.beginPath();
                    ctx.ellipse(centerX + 5, centerY + 3, GAME_CONFIG.foodSize/2 - 3, GAME_CONFIG.foodSize/3 - 3, 0, 0, Math.PI * 2);
                    ctx.fill();

                    ctx.fillStyle = 'white';
                    ctx.font = '12px Arial';
                    ctx.textAlign = 'center';
                    const gainText = gameState.comboCount >= GAME_CONFIG.comboBonusThreshold ? 
                                    `+${GAME_CONFIG.foodWeightGainCombo}斤` : `+${GAME_CONFIG.foodWeightGain}斤`;
                    ctx.fillText(gainText, centerX, centerY + GAME_CONFIG.foodSize/2 + 5);
                }
            });
        }

        // 绘制BOSS
        function drawBosses() {
            gameState.bosses.forEach(boss => {
                if (!boss.eaten) {
                    const centerX = boss.x + GAME_CONFIG.bossSize/2;
                    const centerY = boss.y + GAME_CONFIG.bossSize/2;
                    
                    if (boss.aggro) {
                        ctx.fillStyle = 'rgba(255,0,0,0.2)';
                        ctx.beginPath();
                        ctx.arc(centerX, centerY, GAME_CONFIG.bossSize, 0, Math.PI * 2);
                        ctx.fill();
                    }
                    
                    if (boss.hitFrame > 0) {
                        ctx.globalAlpha = boss.hitFrame % 2 === 0 ? 0.5 : 1;
                        boss.hitFrame--;
                    }
                    
                    ctx.fillStyle = boss.defeated ? '#ec7063' : '#e74c3c';
                    ctx.beginPath();
                    ctx.arc(centerX, centerY, GAME_CONFIG.bossSize/2, 0, Math.PI * 2);
                    ctx.fill();
                    
                    ctx.fillStyle = '#333';
                    ctx.fillRect(boss.x + 10, boss.y - 15, GAME_CONFIG.bossSize - 20, 8);
                    ctx.fillStyle = '#e74c3c';
                    const hpWidth = (boss.hp / GAME_CONFIG.bossHp) * (GAME_CONFIG.bossSize - 20);
                    ctx.fillRect(boss.x + 10, boss.y - 15, hpWidth, 8);
                    
                    ctx.fillStyle = 'white';
                    ctx.font = '14px Arial';
                    ctx.textAlign = 'center';
                    ctx.fillText(boss.defeated ? '可食用' : '冯川', centerX, centerY + 20);
                    
                    if (boss.respawnTimes > 0 && boss.respawnTimes < GAME_CONFIG.maxRespawnTimes) {
                        ctx.fillStyle = '#FFD700';
                        ctx.font = '12px Arial';
                        ctx.fillText(`复活x${boss.respawnTimes}`, centerX, centerY + 35);
                    } else if (boss.respawnTimes >= GAME_CONFIG.maxRespawnTimes) {
                        ctx.fillStyle = '#888';
                        ctx.font = '12px Arial';
                        ctx.fillText('不可复活', centerX, centerY + 35);
                    }
                    
                    ctx.globalAlpha = 1;
                }
            });
        }

        // BOSS移动AI
        function moveBosses() {
            const currentSize = getCurrentPigSize();
            const pigCenterX = gameState.pig.x + currentSize/2;
            const pigCenterY = gameState.pig.y + currentSize/2;
            
            gameState.bosses.forEach(boss => {
                if (!boss.eaten && !boss.defeated) {
                    const bossCenterX = boss.x + GAME_CONFIG.bossSize/2;
                    const bossCenterY = boss.y + GAME_CONFIG.bossSize/2;
                    
                    const distance = Math.hypot(pigCenterX - bossCenterX, pigCenterY - bossCenterY);
                    boss.aggro = distance < boss.aggroRange;
                    
                    const moveSpeed = boss.aggro ? 
                                     boss.moveSpeed + GAME_CONFIG.bossAcceleration * gameState.comboCount : 
                                     boss.moveSpeed;
                    
                    const dxToPig = pigCenterX - bossCenterX;
                    const dyToPig = pigCenterY - bossCenterY;
                    const normalizedDx = dxToPig / distance;
                    const normalizedDy = dyToPig / distance;
                    
                    const chaseFactor = boss.aggro ? 0.9 : 0.6;
                    const randomFactor = boss.aggro ? 0.1 : 0.4;
                    
                    const chaseDx = normalizedDx * chaseFactor;
                    const chaseDy = normalizedDy * chaseFactor;
                    const randomDx = (Math.random() - 0.5) * 2 * randomFactor;
                    const randomDy = (Math.random() - 0.5) * 2 * randomFactor;
                    
                    const finalDx = (chaseDx + randomDx) * moveSpeed;
                    const finalDy = (chaseDy + randomDy) * moveSpeed;
                    
                    boss.x += finalDx;
                    boss.y += finalDy;
                    
                    if (boss.x < 0) boss.x = 0;
                    if (boss.x > canvas.width - GAME_CONFIG.bossSize) boss.x = canvas.width - GAME_CONFIG.bossSize;
                    if (boss.y < 0) boss.y = 0;
                    if (boss.y > canvas.height - GAME_CONFIG.bossSize) boss.y = canvas.height - GAME_CONFIG.bossSize;
                }
            });
        }

        // 攻击系统
        function handleAttack() {
            const now = Date.now();
            const currentCd = Math.max(0, GAME_CONFIG.attackCd - (now - gameState.lastAttackTime));
            
            if (gameState.keys.Space && currentCd === 0 && 
                !gameState.pig.isStunned && !gameState.pig.isDashing) {
                
                gameState.attackCount++;
                gameState.lastAttackTime = now;
                gameState.pig.isAttacking = true;
                gameState.pig.isStunned = true;
                
                hitEffect.style.display = 'block';
                setTimeout(() => {
                    hitEffect.style.display = 'none';
                }, 200);
                
                const currentSize = getCurrentPigSize();
                const pigCenterX = gameState.pig.x + currentSize/2;
                const pigCenterY = gameState.pig.y + currentSize/2;
                const attackRange = getCurrentAttackRange();
                
                let hitBoss = false;
                gameState.bosses.forEach((boss, index) => {
                    if (!boss.eaten && !boss.defeated) {
                        const bossCenterX = boss.x + GAME_CONFIG.bossSize/2;
                        const bossCenterY = boss.y + GAME_CONFIG.bossSize/2;
                        const distance = Math.hypot(pigCenterX - bossCenterX, pigCenterY - bossCenterY);
                        
                        if (distance < attackRange) {
                            boss.hp -= GAME_CONFIG.attackDamage;
                            boss.hitFrame = 15;
                            hitBoss = true;
                            
                            gameState.comboCount++;
                            gameState.maxComboCount = Math.max(gameState.maxComboCount, gameState.comboCount);
                            gameState.lastComboTime = now;
                            
                            if (gameState.comboCount % 5 === 0) {
                                showHint(bossCenterX, bossCenterY, `${gameState.comboCount}连！`, 'combo-hint');
                            }
                            
                            // BOSS血量清零，触发击败和复活
                            if (boss.hp <= 0) {
                                boss.defeated = true;
                                gameState.defeatedBossCount++;
                                gameState.comboCount += 2;
                                gameState.maxComboCount = Math.max(gameState.maxComboCount, gameState.comboCount);
                                // 调用复活函数
                                respawnBoss(boss, index);
                            }
                        }
                    }
                });
                
                setTimeout(() => {
                    gameState.pig.isAttacking = false;
                    setTimeout(() => {
                        gameState.pig.isStunned = false;
                    }, GAME_CONFIG.attackStun);
                }, GAME_CONFIG.attackCd / 2);
                
                gameState.keys.Space = false;
            }
        }

        // 闪避系统
        function handleDash() {
            const now = Date.now();
            const dashCdRemaining = Math.max(0, GAME_CONFIG.dashCd - (now - gameState.lastDashTime));
            
            if (gameState.keys.Shift && dashCdRemaining === 0 && !gameState.pig.isStunned) {
                gameState.dashCount++;
                gameState.lastDashTime = now;
                gameState.pig.isDashing = true;
                gameState.pig.isStunned = true;
                
                let dashX = 0, dashY = 0;
                if (gameState.keys.ArrowUp) dashY = -1;
                if (gameState.keys.ArrowDown) dashY = 1;
                if (gameState.keys.ArrowLeft) dashX = -1;
                if (gameState.keys.ArrowRight) dashX = 1;
                
                if (gameState.joystick.isActive) {
                    const dx = gameState.joystick.currentX - gameState.joystick.startX;
                    const dy = gameState.joystick.currentY - gameState.joystick.startY;
                    const distance = Math.hypot(dx, dy);
                    
                    if (distance > 0) {
                        dashX = dx / distance;
                        dashY = dy / distance;
                    }
                }
                
                if (dashX === 0 && dashY === 0) dashX = 1;
                gameState.pig.dashDirection = {x: dashX, y: dashY};
                
                dashEffect.style.left = `${gameState.pig.x}px`;
                dashEffect.style.top = `${gameState.pig.y}px`;
                dashEffect.style.display = 'block';
                
                setTimeout(() => {
                    gameState.pig.isDashing = false;
                    setTimeout(() => {
                        gameState.pig.isStunned = false;
                        dashEffect.style.display = 'none';
                    }, 50);
                }, GAME_CONFIG.dashDuration);
                
                gameState.keys.Shift = false;
            }
        }

        // 碰撞检测
        function checkCollisions() {
            const currentSize = getCurrentPigSize();
            const pigRect = {
                x: gameState.pig.x,
                y: gameState.pig.y,
                width: currentSize,
                height: currentSize
            };
            const now = Date.now();
            
            if (gameState.pig.isDashing) return;
            
            // 食物碰撞
            gameState.foods.forEach((food, index) => {
                if (!food.eaten) {
                    const foodRect = {
                        x: food.x,
                        y: food.y,
                        width: GAME_CONFIG.foodSize,
                        height: GAME_CONFIG.foodSize
                    };
                    
                    if (checkRectCollision(pigRect, foodRect)) {
                        food.eaten = true;
                        gameState.eatenFoodCount++;
                        
                        const weightGain = gameState.comboCount >= GAME_CONFIG.comboBonusThreshold ? 
                                          GAME_CONFIG.foodWeightGainCombo : GAME_CONFIG.foodWeightGain;
                        
                        gameState.weight += weightGain;
                        showHint(food.x + GAME_CONFIG.foodSize/2, food.y, `+${weightGain}斤`, 'gain-hint');
                        gameState.lastComboTime = now;
                        
                        refreshFood(index);
                        updateUI();
                    }
                }
            });
            
            // BOSS碰撞
            gameState.bosses.forEach(boss => {
                if (!boss.eaten) {
                    const bossRect = {
                        x: boss.x,
                        y: boss.y,
                        width: GAME_CONFIG.bossSize,
                        height: GAME_CONFIG.bossSize
                    };
                    
                    if (checkRectCollision(pigRect, bossRect)) {
                        if (boss.defeated) {
                            boss.eaten = true;
                            gameState.weight -= GAME_CONFIG.bossWeightLoss;
                            if (gameState.weight < 0) gameState.weight = 0;
                            showHint(boss.x + GAME_CONFIG.bossSize/2, boss.y, `- ${GAME_CONFIG.bossWeightLoss}斤`, 'damage-hint');
                            gameState.comboCount = 0;
                        } else {
                            if (now - gameState.lastDamageTime > GAME_CONFIG.damageHintCd) {
                                gameState.weight -= GAME_CONFIG.bossDamageLoss;
                                if (gameState.weight < 0) gameState.weight = 0;
                                gameState.lastDamageTime = now;
                                showHint(boss.x + GAME_CONFIG.bossSize/2, boss.y, `- ${GAME_CONFIG.bossDamageLoss}斤`, 'damage-hint');
                                gameState.comboCount = 0;
                                
                                if (gameState.weight <= 0) {
                                    gameOver(false);
                                }
                            }
                        }
                        updateUI();
                    }
                }
            });
            
            if (gameState.weight >= GAME_CONFIG.winWeight) {
                gameOver(true);
            }
        }

        // 矩形碰撞检测
        function checkRectCollision(rect1, rect2) {
            return rect1.x < rect2.x + rect2.width &&
                   rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.height &&
                   rect1.y + rect1.height > rect2.y;
        }

        // 更新角色位置
        function updatePigPosition() {
            let { x, y } = gameState.pig;
            const joystick = gameState.joystick;
            const currentSize = getCurrentPigSize();
            
            if (gameState.pig.isDashing) {
                x += gameState.pig.dashDirection.x * GAME_CONFIG.dashSpeed;
                y += gameState.pig.dashDirection.y * GAME_CONFIG.dashSpeed;
            } else if (!gameState.pig.isStunned) {
                if (gameState.keys.ArrowUp) y -= GAME_CONFIG.moveSpeed;
                if (gameState.keys.ArrowDown) y += GAME_CONFIG.moveSpeed;
                if (gameState.keys.ArrowLeft) x -= GAME_CONFIG.moveSpeed;
                if (gameState.keys.ArrowRight) x += GAME_CONFIG.moveSpeed;
                
                if (joystick.isActive) {
                    const dx = joystick.currentX - joystick.startX;
                    const dy = joystick.currentY - joystick.startY;
                    const distance = Math.hypot(dx, dy);
                    
                    if (distance > 0) {
                        const normalizedDx = dx / distance;
                        const normalizedDy = dy / distance;
                        
                        x += normalizedDx * GAME_CONFIG.moveSpeed;
                        y += normalizedDy * GAME_CONFIG.moveSpeed;
                    }
                }
            }
            
            if (x < 0) x = 0;
            if (x > canvas.width - currentSize) x = canvas.width - currentSize;
            if (y < 0) y = 0;
            if (y > canvas.height - currentSize) y = canvas.height - currentSize;
            
            gameState.pig.x = x;
            gameState.pig.y = y;
        }

        // 庆祝特效
        function createCelebration() {
            celebration.style.display = 'block';
            celebration.innerHTML = '';
            
            for (let i = 0; i < 100; i++) {
                const confetti = document.createElement('div');
                confetti.className = 'confetti';
                confetti.style.left = `${Math.random() * 100}%`;
                confetti.style.top = `${Math.random() * 10}px`;
                confetti.style.backgroundColor = `hsl(${Math.random() * 360}, 100%, 50%)`;
                confetti.style.animationDuration = `${2 + Math.random() * 3}s`;
                celebration.appendChild(confetti);
            }
        }

        // 游戏结束处理
        function gameOver(isWin) {
            gameState.gameOver = true;
            gameState.playTime = Math.floor((Date.now() - gameState.startTime) / 1000);
            
            // 更新统计数据显示
            playTimeDisplay.textContent = gameState.playTime;
            eatenFoodDisplay.textContent = gameState.eatenFoodCount;
            defeatedBossDisplay.textContent = gameState.defeatedBossCount;
            maxComboDisplay.textContent = gameState.maxComboCount;
            dashCountDisplay.textContent = gameState.dashCount;
            attackCountDisplay.textContent = gameState.attackCount;
            respawnCountDisplay.textContent = gameState.respawnCount;
            
            // 显示游戏结束弹窗
            gameOverScreen.style.display = 'block';
            if (isWin) {
                gameOverText.textContent = '恭喜通关！张晓琪成功变胖啦🎉';
                gameOverScreen.classList.add('game-win');
                createCelebration();
                
                // 保存排行榜数据
                const newRecord = {
                    time: new Date().toLocaleString(),
                    playTime: gameState.playTime,
                    weight: gameState.weight,
                    maxCombo: gameState.maxComboCount,
                    defeatedBoss: gameState.defeatedBossCount
                };
                rankData.push(newRecord);
                rankData = rankData.sort((a, b) => a.playTime - b.playTime).slice(0, 10);
                localStorage.setItem('zhangxiaoqi_action_rank', JSON.stringify(rankData));
            } else {
                gameOverText.textContent = '游戏结束！张晓琪体重清零了😢';
            }
        }

        // 重置摇杆
        function resetJoystick() {
            gameState.joystick.isActive = false;
            gameState.joystick.currentX = gameState.joystick.startX;
            gameState.joystick.currentY = gameState.joystick.startY;
            const knob = document.getElementById('joystickKnob');
            knob.style.transform = 'translate(-50%, -50%)';
        }

        // 游戏主循环
        function gameLoop() {
            if (gameState.gameOver) return;
            
            // 清空画布
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 更新游戏状态
            updatePigPosition();
            moveBosses();
            handleAttack();
            handleDash();
            checkCollisions();
            updateUI();
            
            // 绘制游戏元素
            drawPig();
            drawFoods();
            drawBosses();
            
            // 更新游戏时长
            gameState.playTime = Math.floor((Date.now() - gameState.startTime) / 1000);
            
            // 循环调用
            requestAnimationFrame(gameLoop);
        }

        // 重新开始游戏
        function restartGame() {
            initGame();
        }

        // 显示排行榜
        function showRank() {
            if (rankData.length === 0) {
                alert('暂无排行榜数据！');
                return;
            }
            
            let rankText = '🏆 通关排行榜 🏆\n\n';
            rankData.forEach((item, index) => {
                rankText += `${index + 1}. 时间: ${item.time}\n`;
                rankText += `   耗时: ${item.playTime}秒 | 体重: ${item.weight}斤 | 最高连击: ${item.maxCombo}连 | 击败冯川: ${item.defeatedBoss}个\n\n`;
            });
            
            alert(rankText);
        }

        // 移动端攻击控制
        function handleMobileAttackStart() {
            gameState.keys.Space = true;
        }

        function handleMobileAttackEnd() {
            gameState.keys.Space = false;
        }

        // 移动端闪避控制
        function handleMobileDashStart() {
            gameState.keys.Shift = true;
        }

        function handleMobileDashEnd() {
            gameState.keys.Shift = false;
        }

        // 摇杆控制
        function initJoystick() {
            const joystickContainer = document.getElementById('joystickContainer');
            const joystickKnob = document.getElementById('joystickKnob');
            let touchId = -1;
            
            joystickContainer.addEventListener('touchstart', (e) => {
                e.preventDefault();
                touchId = e.touches[0].identifier;
                const rect = joystickContainer.getBoundingClientRect();
                gameState.joystick.isActive = true;
                gameState.joystick.startX = e.touches[0].clientX - rect.left;
                gameState.joystick.startY = e.touches[0].clientY - rect.top;
                gameState.joystick.currentX = gameState.joystick.startX;
                gameState.joystick.currentY = gameState.joystick.startY;
            });
            
            document.addEventListener('touchmove', (e) => {
                if (!gameState.joystick.isActive) return;
                
                for (let i = 0; i < e.touches.length; i++) {
                    if (e.touches[i].identifier === touchId) {
                        const rect = joystickContainer.getBoundingClientRect();
                        const x = e.touches[i].clientX - rect.left;
                        const y = e.touches[i].clientY - rect.top;
                        const dx = x - gameState.joystick.startX;
                        const dy = y - gameState.joystick.startY;
                        const distance = Math.hypot(dx, dy);
                        
                        if (distance > gameState.joystick.maxDistance) {
                            const ratio = gameState.joystick.maxDistance / distance;
                            gameState.joystick.currentX = gameState.joystick.startX + dx * ratio;
                            gameState.joystick.currentY = gameState.joystick.startY + dy * ratio;
                        } else {
                            gameState.joystick.currentX = x;
                            gameState.joystick.currentY = y;
                        }
                        
                        const knobX = gameState.joystick.currentX - gameState.joystick.startX;
                        const knobY = gameState.joystick.currentY - gameState.joystick.startY;
                        joystickKnob.style.transform = `translate(${knobX}px, ${knobY}px)`;
                        break;
                    }
                }
            });
            
            document.addEventListener('touchend', (e) => {
                for (let i = 0; i < e.changedTouches.length; i++) {
                    if (e.changedTouches[i].identifier === touchId) {
                        resetJoystick();
                        touchId = -1;
                        break;
                    }
                }
            });
            
            // 鼠标模拟摇杆（用于PC测试）
            let isMouseDown = false;
            joystickContainer.addEventListener('mousedown', (e) => {
                e.preventDefault();
                isMouseDown = true;
                const rect = joystickContainer.getBoundingClientRect();
                gameState.joystick.isActive = true;
                gameState.joystick.startX = e.clientX - rect.left;
                gameState.joystick.startY = e.clientY - rect.top;
                gameState.joystick.currentX = gameState.joystick.startX;
                gameState.joystick.currentY = gameState.joystick.startY;
            });
            
            document.addEventListener('mousemove', (e) => {
                if (!isMouseDown || !gameState.joystick.isActive) return;
                
                const rect = joystickContainer.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                const dx = x - gameState.joystick.startX;
                const dy = y - gameState.joystick.startY;
                const distance = Math.hypot(dx, dy);
                
                if (distance > gameState.joystick.maxDistance) {
                    const ratio = gameState.joystick.maxDistance / distance;
                    gameState.joystick.currentX = gameState.joystick.startX + dx * ratio;
                    gameState.joystick.currentY = gameState.joystick.startY + dy * ratio;
                } else {
                    gameState.joystick.currentX = x;
                    gameState.joystick.currentY = y;
                }
                
                const knobX = gameState.joystick.currentX - gameState.joystick.startX;
                const knobY = gameState.joystick.currentY - gameState.joystick.startY;
                joystickKnob.style.transform = `translate(${knobX}px, ${knobY}px)`;
            });
            
            document.addEventListener('mouseup', () => {
                if (isMouseDown) {
                    isMouseDown = false;
                    resetJoystick();
                }
            });
        }

        // 键盘控制
        function initKeyboardControls() {
            document.addEventListener('keydown', (e) => {
                if (e.key in gameState.keys) {
                    gameState.keys[e.key] = true;
                }
            });
            
            document.addEventListener('keyup', (e) => {
                if (e.key in gameState.keys) {
                    gameState.keys[e.key] = false;
                }
            });
        }

        // 初始化游戏
        window.addEventListener('load', () => {
            initKeyboardControls();
            initJoystick();
            initGame();
        });
    </script>
</body>
</html>
