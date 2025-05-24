<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>高级贪吃蛇游戏</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 20px;
        }
        
        #game-container {
            display: flex;
            gap: 20px;
            margin-top: 20px;
        }
        
        #game-board {
            border: 2px solid #333;
            background-color: #222;
            position: relative;
        }
        
        #ai-board {
            border: 2px solid #333;
            background-color: #222;
            position: relative;
        }
        
        .board-container {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .board-title {
            margin-bottom: 10px;
            font-weight: bold;
        }
        
        #menu {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            width: 300px;
            margin-bottom: 20px;
        }
        
        #shop {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            width: 300px;
            display: none;
        }
        
        button {
            padding: 8px 15px;
            margin: 5px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        .skin-item {
            border: 1px solid #ddd;
            padding: 10px;
            margin: 5px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .stats {
            display: flex;
            justify-content: space-between;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <h1>高级贪吃蛇游戏</h1>
    
    <div id="menu">
        <h2>主菜单</h2>
        <button id="single-player">单人模式</button>
        <button id="ai-battle">AI对战</button>
        <button id="open-shop">皮肤商城</button>
        <div class="stats">
            <div>金币: <span id="gold-amount">0</span></div>
            <div>当前皮肤: <span id="current-skin">默认</span></div>
        </div>
    </div>
    
    <div id="shop">
        <h2>皮肤商城</h2>
        <div id="skin-list">
            <!-- 皮肤商品将通过JS动态加载 -->
        </div>
        <button id="close-shop">返回</button>
    </div>
    
    <div id="game-container" style="display: none;">
        <div class="board-container">
            <div class="board-title">玩家</div>
            <canvas id="game-board" width="400" height="400"></canvas>
            <div>分数: <span id="player-score">0</span></div>
        </div>
        
        <div class="board-container" id="ai-container" style="display: none;">
            <div class="board-title">AI</div>
            <canvas id="ai-board" width="400" height="400"></canvas>
            <div>分数: <span id="ai-score">0</span></div>
        </div>
    </div>
    
    <script>
        // 游戏状态
        const gameState = {
            singlePlayer: true,
            gameRunning: false,
            gold: localStorage.getItem('snakeGold') ? parseInt(localStorage.getItem('snakeGold')) : 0,
            currentSkin: localStorage.getItem('snakeSkin') || 'default',
            skins: [
                { id: 'default', name: '默认', price: 0, color: '#4CAF50', headColor: '#2E7D32', owned: true },
                { id: 'fire', name: '火焰', price: 100, color: '#FF5722', headColor: '#E64A19' },
                { id: 'ice', name: '冰霜', price: 100, color: '#00BCD4', headColor: '#00838F' },
                { id: 'golden', name: '黄金', price: 200, color: '#FFD700', headColor: '#FFC000' },
                { id: 'rainbow', name: '彩虹', price: 300, color: 'linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet)', headColor: '#FFFFFF' }
            ],
            highScore: localStorage.getItem('snakeHighScore') ? parseInt(localStorage.getItem('snakeHighScore')) : 0
        };

        // 初始化UI
        document.getElementById('gold-amount').textContent = gameState.gold;
        document.getElementById('current-skin').textContent = gameState.skins.find(skin => skin.id === gameState.currentSkin).name;
        
        // 加载皮肤商店
        function loadShop() {
            const shopList = document.getElementById('skin-list');
            shopList.innerHTML = '';
            
            gameState.skins.forEach(skin => {
                const skinItem = document.createElement('div');
                skinItem.className = 'skin-item';
                
                const skinInfo = document.createElement('div');
                skinInfo.innerHTML = `<strong>${skin.name}</strong> - ${skin.price}金币`;
                
                const buyButton = document.createElement('button');
                if (skin.owned || skin.id === 'default') {
                    buyButton.textContent = skin.id === gameState.currentSkin ? '使用中' : '使用';
                    buyButton.style.backgroundColor = skin.id === gameState.currentSkin ? '#9E9E9E' : '#2196F3';
                    buyButton.onclick = () => {
                        if (skin.id !== gameState.currentSkin) {
                            gameState.currentSkin = skin.id;
                            localStorage.setItem('snakeSkin', skin.id);
                            document.getElementById('current-skin').textContent = skin.name;
                            loadShop();
                        }
                    };
                } else {
                    buyButton.textContent = '购买';
                    buyButton.style.backgroundColor = '#FF9800';
                    buyButton.onclick = () => {
                        if (gameState.gold >= skin.price) {
                            gameState.gold -= skin.price;
                            skin.owned = true;
                            localStorage.setItem('snakeGold', gameState.gold);
                            document.getElementById('gold-amount').textContent = gameState.gold;
                            loadShop();
                        } else {
                            alert('金币不足！');
                        }
                    };
                }
                
                skinItem.appendChild(skinInfo);
                skinItem.appendChild(buyButton);
                shopList.appendChild(skinItem);
            });
        }

        // 菜单按钮事件
        document.getElementById('single-player').addEventListener('click', () => {
            gameState.singlePlayer = true;
            startGame();
        });
        
        document.getElementById('ai-battle').addEventListener('click', () => {
            gameState.singlePlayer = false;
            startGame();
        });
        
        document.getElementById('open-shop').addEventListener('click', () => {
            document.getElementById('menu').style.display = 'none';
            document.getElementById('shop').style.display = 'block';
            loadShop();
        });
        
        document.getElementById('close-shop').addEventListener('click', () => {
            document.getElementById('shop').style.display = 'none';
            document.getElementById('menu').style.display = 'block';
        });

        // 游戏逻辑
        const canvas = document.getElementById('game-board');
        const ctx = canvas.getContext('2d');
        const aiCanvas = document.getElementById('ai-board');
        const aiCtx = aiCanvas.getContext('2d');
        
        const gridSize = 20;
        const tileCount = canvas.width / gridSize;
        
        let snake = [];
        let aiSnake = [];
        let food = {};
        let aiFood = {};
        let xVelocity = 0;
        let yVelocity = 0;
        let aiXVelocity = 0;
        let aiYVelocity = 0;
        let score = 0;
        let aiScore = 0;
        let gameSpeed = 100;
        let gameLoop;
        let aiGameLoop;
        
        function startGame() {
            document.getElementById('menu').style.display = 'none';
            document.getElementById('shop').style.display = 'none';
            document.getElementById('game-container').style.display = 'flex';
            
            if (!gameState.singlePlayer) {
                document.getElementById('ai-container').style.display = 'block';
            } else {
                document.getElementById('ai-container').style.display = 'none';
            }
            
            // 初始化蛇
            snake = [
                {x: 10 * gridSize, y: 10 * gridSize},
                {x: 9 * gridSize, y: 10 * gridSize},
                {x: 8 * gridSize, y: 10 * gridSize}
            ];
            
            if (!gameState.singlePlayer) {
                aiSnake = [
                    {x: 10 * gridSize, y: 5 * gridSize},
                    {x: 9 * gridSize, y: 5 * gridSize},
                    {x: 8 * gridSize, y: 5 *