<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>賽狗會 Doggg Club - 模擬馬會投注系統</title>
    <!-- 引入 Tailwind CSS 提供現代化響應式 UI 設計 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        hkjc: {
                            blue: '#003A8C',      /* 馬會經典深藍 */
                            lightBlue: '#1890FF', /* 天藍輔助色 */
                            gold: '#D4B106',      /* 經典金黃色 */
                            lightGold: '#FADB14', /* 亮金色 */
                            dark: '#002140',      /* 暗藍底色 */
                            gray: '#F0F2F5'       /* 淺灰背景底色 */
                        }
                    }
                }
            }
        }
    </script>
    <!-- 引入 Google Fonts 提升中文字體質感 -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans TC', sans-serif;
            /* 預留底部導航欄的高度，防止內容被遮擋 */
            padding-bottom: 90px; 
        }
        /* 隱藏滾動條但保持滾動功能 */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="bg-hkjc-gray min-h-screen">

    <!-- 頂部馬會風格導航欄 -->
    <header class="bg-hkjc-blue text-white shadow-lg border-b-4 border-hkjc-gold sticky top-0 z-50">
        <div class="max-w-4xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <span class="text-3xl">🐕</span>
                <div>
                    <h1 class="text-xl font-black tracking-wider text-hkjc-lightGold">賽狗會</h1>
                    <p class="text-xs font-bold uppercase tracking-widest text-gray-300">Doggg Club</p>
                </div>
            </div>
            <!-- 右側顯示當前登入的用戶和餘額 -->
            <div class="flex flex-col items-end">
                <span class="text-[10px] text-gray-300">當前帳戶</span>
                <div class="flex items-center space-x-2 bg-hkjc-dark px-3 py-1 rounded-full border border-hkjc-gold/30">
                    <span id="header-user-name" class="font-bold text-sm text-white">---</span>
                    <span class="text-gray-400">|</span>
                    <span id="header-user-balance" class="font-black text-hkjc-lightGold text-sm">$0</span>
                </div>
            </div>
        </div>
    </header>

    <!-- 自定義 Toast 通知組件 (取代會干擾遊戲的 alert) -->
    <div id="toast" class="hidden fixed top-24 left-1/2 transform -translate-x-1/2 z-50 bg-gray-900/95 text-white px-6 py-3 rounded-xl shadow-2xl flex items-center gap-3 transition-all duration-300 border border-hkjc-gold w-[90%] max-w-sm">
        <span id="toast-icon" class="text-xl">📢</span>
        <span id="toast-message" class="font-bold text-sm"></span>
    </div>

    <!-- 主內容區 -->
    <main class="max-w-4xl mx-auto px-4 mt-5">
        
        <!-- 全域帳戶管理：切換帳戶 (永遠置頂，方便多人聚會時輪流操作) -->
        <section class="bg-white rounded-xl shadow-md p-4 mb-6 border-t-4 border-hkjc-blue">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                <h2 class="text-sm font-bold text-gray-700 flex items-center gap-1">
                    <span>👤</span> 點擊頭像切換玩家：
                </h2>
                <!-- 新增玩家表單 -->
                <div class="flex gap-2">
                    <input id="new-username-input" type="text" placeholder="輸入新朋友名字" class="border border-gray-300 rounded-lg px-3 py-1.5 text-sm focus:outline-none focus:ring-2 focus:ring-hkjc-blue w-full sm:w-auto">
                    <button onclick="addNewUser()" class="bg-hkjc-blue hover:bg-hkjc-dark text-white text-sm font-bold px-4 py-1.5 rounded-lg transition shadow shrink-0">
                        新增
                    </button>
                </div>
            </div>
            <!-- 橫向滑動的玩家清單 -->
            <div id="users-list-container" class="flex gap-2 mt-3 overflow-x-auto pb-2 no-scrollbar">
                <!-- 由 JavaScript 動態生成 -->
            </div>
        </section>

        <!-- ==================== 分頁 1: 投注面板 ==================== -->
        <section id="section-betting" class="space-y-6 block">
            <!-- 進行中賽事列表 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-blue">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2">
                        <span>🎟️</span> 進行中的賽事
                    </h2>
                    <span class="bg-red-100 text-red-700 text-xs font-bold px-2.5 py-1 rounded-full animate-pulse">
                        接受投注中
                    </span>
                </div>
                <div id="active-matches-container" class="space-y-4">
                    <!-- 由 JavaScript 動態生成 -->
                </div>
            </div>

            <!-- 簡單遊戲說明 -->
            <div class="bg-blue-50 rounded-xl p-4 border border-blue-200">
                <h3 class="font-bold text-hkjc-blue text-sm mb-1">💡 投注小提示</h3>
                <p class="text-xs text-gray-600">請先在上方點選你的名字，確認餘額後，選擇下方的賽事並輸入金額即可下注。要查看過往記錄與排名請點擊下方「記錄與排行榜」。</p>
            </div>
        </section>

        <!-- ==================== 分頁 2: 記錄與排行榜 ==================== -->
        <section id="section-records" class="space-y-6 hidden">
            <!-- 財富排行榜 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-gold relative overflow-hidden">
                <div class="absolute -right-6 -top-6 text-7xl text-yellow-100 opacity-40 select-none">🏆</div>
                <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2 mb-4 relative z-10">
                    <span>🏆</span> 財富排行榜
                </h2>
                <div id="leaderboard-container" class="space-y-3 relative z-10">
                    <!-- 由 JavaScript 動態生成 -->
                </div>
            </div>

            <!-- 投注紀錄 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-gray-400 overflow-hidden">
                <h2 class="text-lg font-bold text-gray-800 flex items-center gap-2 mb-4">
                    <span>📋</span> 所有投注歷史紀錄
                </h2>
                <div class="overflow-x-auto -mx-5 px-5">
                    <table class="min-w-full text-left text-sm whitespace-nowrap">
                        <thead class="bg-gray-100 text-gray-700 font-bold">
                            <tr>
                                <th class="p-3">玩家</th>
                                <th class="p-3">賽事 / 選項</th>
                                <th class="p-3 text-right">金額</th>
                                <th class="p-3 text-center">狀態</th>
                            </tr>
                        </thead>
                        <tbody id="bets-history-tbody">
                            <!-- 由 JavaScript 動態生成 -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

        <!-- ==================== 分頁 3: 賽事結算 (莊家專用) ==================== -->
        <section id="section-admin" class="space-y-6 hidden">
            
            <!-- 賽事結算中心 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-green-500">
                <h2 class="text-lg font-bold text-green-700 flex items-center gap-2 mb-3">
                    <span>🏁</span> 賽事結算中心
                </h2>
                <p class="text-xs text-gray-500 mb-4">比賽結束後，莊家於此選出獲勝選項並派彩。系統會自動按賠率發放模擬資金給贏家。</p>
                <div id="admin-settle-container" class="space-y-4">
                    <!-- 由 JavaScript 動態生成 -->
                </div>
            </div>

            <!-- 開設新賽事 -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-hkjc-blue">
                <h2 class="text-lg font-bold text-hkjc-blue flex items-center gap-2 mb-4">
                    <span>🎯</span> 開設新賽事
                </h2>
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-bold text-gray-700 mb-1">賽事名稱</label>
                        <input id="match-title-input" type="text" placeholder="例：今晚食飯邊個負責洗碗？" class="w-full border border-gray-300 rounded-lg px-4 py-2 text-sm focus:ring-2 focus:ring-hkjc-blue focus:outline-none">
                    </div>
                    <div>
                        <label class="block text-sm font-bold text-gray-700 mb-2">投注選項及賠率</label>
                        <div id="options-inputs-container" class="space-y-2">
                            <!-- 預設選項 A & B -->
                            <div class="flex items-center gap-2 option-row">
                                <span class="text-xs font-bold text-gray-400 w-4">A</span>
                                <input type="text" placeholder="選項名稱" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm option-name focus:ring-1 focus:ring-hkjc-blue focus:outline-none">
                                <input type="number" step="0.1" min="1.01" placeholder="賠率" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm option-odds focus:ring-1 focus:ring-hkjc-blue focus:outline-none">
                                <button onclick="removeOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                            </div>
                            <div class="flex items-center gap-2 option-row">
                                <span class="text-xs font-bold text-gray-400 w-4">B</span>
                                <input type="text" placeholder="選項名稱" class="flex-grow border border-gray-300 rounded-lg px-2 py-1.5 text-sm option-name focus:ring-1 focus:ring-hkjc-blue focus:outline-none">
                                <input type="number" step="0.1" min="1.01" placeholder="賠率" class="w-20 border border-gray-300 rounded-lg px-2 py-1.5 text-sm option-odds focus:ring-1 focus:ring-hkjc-blue focus:outline-none">
                                <button onclick="removeOptionRow(this)" class="text-red-400 hover:text-red-600 p-1">❌</button>
                            </div>
                        </div>
                        <button onclick="addOptionRow()" class="mt-2 text-xs font-bold text-hkjc-lightBlue hover:text-hkjc-blue flex items-center gap-1">
                            ➕ 增加選項
                        </button>
                    </div>
                    <button onclick="createMatch()" class="w-full bg-hkjc-blue hover:bg-hkjc-dark text-white font-bold py-2.5 rounded-lg shadow-md transition">
                        📢 發布賽事、開放投注！
                    </button>
                </div>
            </div>

            <!-- 用戶帳戶管理 (可刪除/重置成員) -->
            <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-red-400">
                <h2 class="text-sm font-bold text-red-600 flex items-center gap-2 mb-3">
                    <span>⚙️</span> 成員管理 (刪除帳戶)
                </h2>
                <div id="admin-users-list" class="grid grid-cols-2 sm:grid-cols-3 gap-2">
                    <!-- 由 JavaScript 動態生成 -->
                </div>
            </div>
        </section>

    </main>

    <!-- 手機 App 風格底部固定導航列 -->
    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 shadow-[0_-5px_15px_rgba(0,0,0,0.05)] z-50 pb-safe">
        <div class="max-w-4xl mx-auto flex">
            <!-- 導航按鈕 1: 投注面板 (預設激活) -->
            <button onclick="switchTab('betting')" id="nav-betting" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-hkjc-blue bg-blue-50 text-hkjc-blue transition-all">
                <span class="text-xl leading-none">🎟️</span>
                <span class="text-[11px] font-bold">投注面板</span>
            </button>
            <!-- 導航按鈕 2: 記錄與排行榜 -->
            <button onclick="switchTab('records')" id="nav-records" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all">
                <span class="text-xl leading-none">🏆</span>
                <span class="text-[11px] font-bold">記錄與排行榜</span>
            </button>
            <!-- 導航按鈕 3: 賽事結算 -->
            <button onclick="switchTab('admin')" id="nav-admin" class="flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all">
                <span class="text-xl leading-none">⚙️</span>
                <span class="text-[11px] font-bold">賽事結算</span>
            </button>
        </div>
    </nav>

    <!-- 前端交互邏輯代碼 -->
    <script>
        // 1. 初始化資料與預設值
        const DEFAULT_USERS = ['泰臣', '辣椒', '秋葵', '薯片', '蘇媚', '水晶', '牛屎', '西瓜', '騾仔'];
        
        let state = {
            users: [],
            currentUser: null,
            matches: [],
            bets: []
        };

        // 從 LocalStorage 載入資料，避免重新整理時進度遺失
        function loadGameData() {
            const savedState = localStorage.getItem('doggg_club_state');
            if (savedState) {
                state = JSON.parse(savedState);
            } else {
                // 初始化預設帳戶，每個人有 $1,000
                state.users = DEFAULT_USERS.map(name => ({ name: name, balance: 1000 }));
                // 預設兩場趣味賽事
                state.matches = [
                    {
                        id: 'm1', title: '🐕 第一場：賽狗總決賽 (黃金組)',
                        options: [
                            { id: 'o1', name: '閃電哈士奇', odds: 1.8 },
                            { id: 'o2', name: '旋風柴犬', odds: 2.5 },
                            { id: 'o3', name: '火箭吉娃娃', odds: 4.2 }
                        ],
                        settled: false, winner: null
                    },
                    {
                        id: 'm2', title: '🤔 聽日邊個係書童？',
                        options: [
                            { id: 'o4', name: '會', odds: 1.8 },
                            { id: 'o5', name: '不會', odds: 2.2 }
                        ],
                        settled: false, winner: null
                    }
                ];
                state.bets = [];
                saveGameData();
            }
            if (state.users.length > 0) {
                state.currentUser = state.users[0].name;
            }
        }

        // 儲存資料到 LocalStorage
        function saveGameData() {
            localStorage.setItem('doggg_club_state', JSON.stringify(state));
        }

        // 自定義 Toast 提示訊息
        function showToast(message, icon = '📢') {
            const toast = document.getElementById('toast');
            document.getElementById('toast-message').textContent = message;
            document.getElementById('toast-icon').textContent = icon;
            
            toast.classList.remove('hidden');
            toast.classList.add('flex');
            setTimeout(() => {
                toast.classList.add('hidden');
                toast.classList.remove('flex');
            }, 3000);
        }

        // 重新渲染整個網頁的 UI
        function renderAll() {
            renderHeader();
            renderUsersList();
            renderActiveMatches();
            renderBetsHistory();
            renderLeaderboard();
            renderAdminView();
        }

        // 渲染頂部當前選擇的帳戶及餘額
        function renderHeader() {
            const userObj = state.users.find(u => u.name === state.currentUser);
            if (userObj) {
                document.getElementById('header-user-name').textContent = userObj.name;
                document.getElementById('header-user-balance').textContent = `$${userObj.balance.toLocaleString()}`;
            }
        }

        // 渲染頂部的玩家切換頭像
        function renderUsersList() {
            const container = document.getElementById('users-list-container');
            container.innerHTML = '';
            state.users.forEach(user => {
                const isActive = user.name === state.currentUser;
                const activeClasses = isActive 
                    ? 'bg-hkjc-blue text-white ring-2 ring-hkjc-gold' 
                    : 'bg-white text-gray-700 hover:bg-gray-100 border border-gray-200';

                const btn = document.createElement('button');
                btn.className = `flex-shrink-0 px-4 py-2 rounded-xl font-bold text-sm transition shadow-sm ${activeClasses}`;
                btn.onclick = () => selectUser(user.name);
                
                // 計算並自動加上第一名「犬神」徽章
                const sortedUsers = [...state.users].sort((a, b) => b.balance - a.balance);
                const isLeader = sortedUsers.length > 0 && sortedUsers[0].name === user.name;
                
                btn.innerHTML = `${isActive ? '🐕 ' : ''}${user.name}${isLeader ? ' 👑' : ''}`;
                container.appendChild(btn);
            });
        }

        // 切換玩家帳戶
        function selectUser(name) {
            state.currentUser = name;
            saveGameData();
            renderAll();
            showToast(`已切換至帳戶：${name}`, '👤');
        }

        // 新增自訂玩家
        function addNewUser() {
            const input = document.getElementById('new-username-input');
            const name = input.value.trim();
            if (!name) return showToast('請輸入玩家名字！', '⚠️');
            if (state.users.some(u => u.name === name)) return showToast('這個名字已經存在！', '⚠️');

            state.users.push({ name: name, balance: 1000 });
            state.currentUser = name; // 自動切換到新玩家
            saveGameData();
            renderAll();
            input.value = '';
            showToast(`成功新增玩家「${name}」，獲得 $1,000 模擬資金！`, '🎉');
        }

        // 渲染進行中的賽事
        function renderActiveMatches() {
            const container = document.getElementById('active-matches-container');
            container.innerHTML = '';
            const activeMatches = state.matches.filter(m => !m.settled);

            if (activeMatches.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-8 text-gray-400 font-bold">
                        <p class="text-3xl mb-2">🏁</p>
                        <p>目前沒有進行中的賽事。</p>
                        <p class="text-xs font-normal text-gray-500 mt-1">請提醒莊家去「賽事結算」分頁開設新賽事！</p>
                    </div>`;
                return;
            }

            activeMatches.forEach(match => {
                const card = document.createElement('div');
                card.className = 'border border-gray-200 rounded-xl bg-gray-50 p-4';
                
                let optionsHtml = match.options.map(opt => `
                    <label class="flex items-center justify-between bg-white border border-gray-200 rounded-lg p-2.5 hover:bg-blue-50 cursor-pointer">
                        <div class="flex items-center gap-2">
                            <input type="radio" name="match-${match.id}" value="${opt.id}" class="w-4 h-4 text-hkjc-blue">
                            <span class="font-bold text-sm text-gray-800">${opt.name}</span>
                        </div>
                        <span class="bg-hkjc-gold/20 text-hkjc-blue text-xs font-black px-2 py-1 rounded border border-hkjc-gold/40">
                            賠率 @${opt.odds.toFixed(2)}
                        </span>
                    </label>
                `).join('');

                card.innerHTML = `
                    <div class="font-black text-gray-800 text-sm mb-3 border-b pb-2">${match.title}</div>
                    <div class="space-y-2 mb-3">${optionsHtml}</div>
                    <div class="flex gap-2">
                        <input id="bet-amount-${match.id}" type="number" min="1" placeholder="輸入下注金額" class="w-full border border-gray-300 rounded-lg px-3 text-sm focus:outline-none focus:ring-2 focus:ring-hkjc-blue">
                        <button onclick="placeBet('${match.id}')" class="bg-hkjc-blue hover:bg-hkjc-dark text-white font-bold px-4 py-2 rounded-lg text-sm shrink-0 shadow">
                            確認投注 🎟️
                        </button>
                    </div>
                `;
                container.appendChild(card);
            });
        }

        // 下注邏輯
        function placeBet(matchId) {
            const userObj = state.users.find(u => u.name === state.currentUser);
            if (!userObj) return showToast('請先選擇帳戶！', '⚠️');

            const match = state.matches.find(m => m.id === matchId);
            const selectedRadio = document.querySelector(`input[name="match-${matchId}"]:checked`);
            
            if (!selectedRadio) return showToast('請先選擇投注選項！', '⚠️');
            
            const option = match.options.find(o => o.id === selectedRadio.value);
            const amountInput = document.getElementById(`bet-amount-${matchId}`);
            const amount = parseInt(amountInput.value);

            if (isNaN(amount) || amount <= 0) return showToast('請輸入正確金額 (大於 $0)！', '⚠️');
            if (userObj.balance < amount) return showToast('你的餘額不足！', '❌');

            // 扣除模擬本金
            userObj.balance -= amount;
            
            // 寫入投注紀錄
            state.bets.unshift({
                id: 'b_' + Date.now() + '_' + Math.random().toString(36).substr(2, 4),
                username: userObj.name,
                matchId: match.id,
                matchTitle: match.title,
                optionId: option.id,
                optionName: option.name,
                odds: option.odds,
                amount: amount,
                status: 'pending',
                payout: 0
            });

            saveGameData();
            renderAll();
            amountInput.value = '';
            showToast(`下注成功！扣除 $${amount.toLocaleString()} 模擬資金。`, '✅');
        }

        // 渲染排行榜（資產最高者自動獲得「犬神」頭銜）
        function renderLeaderboard() {
            const container = document.getElementById('leaderboard-container');
            container.innerHTML = '';
            
            // 按餘額由多到少排序
            const sortedUsers = [...state.users].sort((a, b) => b.balance - a.balance);

            sortedUsers.forEach((user, index) => {
                const isWinner = index === 0;
                let rankStyle = index === 0 ? "bg-hkjc-gold text-hkjc-dark" : (index === 1 ? "bg-gray-300 text-gray-800" : (index === 2 ? "bg-amber-600 text-white" : "bg-gray-100 text-gray-500"));
                let icon = index === 0 ? "🥇 犬神" : `No.${index + 1}`;

                container.innerHTML += `
                    <div class="flex items-center justify-between p-3 rounded-lg border ${isWinner ? 'bg-amber-50 border-hkjc-gold shadow-sm' : 'bg-white border-gray-100'}">
                        <div class="flex items-center gap-3">
                            <span class="text-xs font-bold px-2 py-1 rounded ${rankStyle}">${icon}</span>
                            <span class="font-bold text-sm text-gray-800">${user.name}</span>
                        </div>
                        <span class="font-black text-hkjc-blue text-sm">$${user.balance.toLocaleString()}</span>
                    </div>
                `;
            });
        }

        // 渲染歷史投注紀錄
        function renderBetsHistory() {
            const tbody = document.getElementById('bets-history-tbody');
            tbody.innerHTML = '';
            if (state.bets.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-gray-400">暫無下注紀錄</td></tr>`;
                return;
            }

            state.bets.forEach(bet => {
                let statusHtml = '';
                if (bet.status === 'pending') {
                    statusHtml = `<span class="text-gray-500 text-xs font-bold bg-gray-100 px-2 py-1 rounded">待結算</span>`;
                } else if (bet.status === 'won') {
                    statusHtml = `<span class="text-green-600 text-xs font-bold bg-green-100 px-2 py-1 rounded">中獎 (+$${bet.payout.toLocaleString()})</span>`;
                } else {
                    statusHtml = `<span class="text-red-500 text-xs font-bold bg-red-50 px-2 py-1 rounded">未中獎</span>`;
                }

                tbody.innerHTML += `
                    <tr class="border-b border-gray-100">
                        <td class="p-3 font-bold text-gray-800">${bet.username}</td>
                        <td class="p-3 text-xs">
                            <div class="text-gray-500 truncate w-32 md:w-auto">${bet.matchTitle}</div>
                            <div class="text-hkjc-blue font-bold">${bet.optionName} <span class="text-gray-400">@${bet.odds}</span></div>
                        </td>
                        <td class="p-3 font-bold text-right text-gray-700">$${bet.amount.toLocaleString()}</td>
                        <td class="p-3 text-center">${statusHtml}</td>
                    </tr>
                `;
            });
        }

        // 渲染莊家控制台面板
        function renderAdminView() {
            // 1. 待結算列表
            const settleContainer = document.getElementById('admin-settle-container');
            settleContainer.innerHTML = '';
            const pendingMatches = state.matches.filter(m => !m.settled);

            if (pendingMatches.length === 0) {
                settleContainer.innerHTML = `<div class="text-center py-4 text-gray-400 text-sm">目前沒有需要結算的賽事。</div>`;
            } else {
                pendingMatches.forEach(match => {
                    let opts = match.options.map(opt => `
                        <label class="flex items-center gap-2 bg-white px-2 py-1.5 border rounded cursor-pointer">
                            <input type="radio" name="settle-${match.id}" value="${opt.id}">
                            <span class="text-sm font-bold">${opt.name} (${opt.odds.toFixed(2)})</span>
                        </label>
                    `).join('');

                    settleContainer.innerHTML += `
                        <div class="p-3 border rounded-lg bg-gray-50">
                            <div class="font-bold text-sm mb-2 text-gray-800">${match.title}</div>
                            <div class="space-y-2 mb-3">${opts}</div>
                            <button onclick="settleMatch('${match.id}')" class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-2 rounded text-sm transition">
                                🏆 結算這場賽事（派彩）
                            </button>
                        </div>
                    `;
                });
            }

            // 2. 成員刪除大名單
            const usersList = document.getElementById('admin-users-list');
            usersList.innerHTML = '';
            state.users.forEach(user => {
                usersList.innerHTML += `
                    <div class="flex justify-between items-center bg-gray-50 border p-2 rounded">
                        <span class="text-xs font-bold text-gray-700">${user.name}</span>
                        <button onclick="deleteUser('${user.name}')" class="text-red-500 hover:text-red-700 text-xs">刪除</button>
                    </div>
                `;
            });
        }

        // 新增賽事選項框
        function addOptionRow() {
            const container = document.getElementById('options-inputs-container');
            const rowCount = container.children.length;
            const label = String.fromCharCode(65 + rowCount);
            
            const div = document.createElement('div');
            div.className = "flex items-center gap-2 option-row";
            div.innerHTML = `
                <span class="text-xs font-bold text-gray-400 w-4">${label}</span>
                <input type="text" placeholder="選項名稱" class="flex-grow border rounded-lg px-2 py-1.5 text-sm option-name focus:outline-none">
                <input type="number" step="0.1" min="1.01" placeholder="賠率" class="w-20 border rounded-lg px-2 py-1.5 text-sm option-odds focus:outline-none">
                <button onclick="removeOptionRow(this)" class="text-red-400 p-1">❌</button>
            `;
            container.appendChild(div);
        }

        // 刪除賽事選項框
        function removeOptionRow(btn) {
            const container = document.getElementById('options-inputs-container');
            if (container.children.length <= 2) return showToast('最少需要保留兩個選項！', '⚠️');
            btn.parentElement.remove();
            
            // 重新理順 A, B, C, D... 序號
            Array.from(container.children).forEach((row, i) => {
                row.querySelector('span').textContent = String.fromCharCode(65 + i);
            });
        }

        // 開設發布新賽事
        function createMatch() {
            const title = document.getElementById('match-title-input').value.trim();
            if (!title) return showToast('請填寫賽事名稱！', '⚠️');

            const rows = document.getElementsByClassName('option-row');
            const options = [];
            for (let row of rows) {
                const name = row.querySelector('.option-name').value.trim();
                const odds = parseFloat(row.querySelector('.option-odds').value);
                if (!name || isNaN(odds) || odds <= 1) return showToast('所有選項與賠率（大於1）必須填妥！', '⚠️');
                options.push({ id: 'o_' + Date.now() + Math.random().toString(36).substr(2, 4), name, odds });
            }

            state.matches.push({ 
                id: 'm_' + Date.now(), 
                title: '🐕 ' + title, 
                options, 
                settled: false, 
                winner: null 
            });
            
            saveGameData();
            renderAll();
            
            // 清理表格
            document.getElementById('match-title-input').value = '';
            showToast('賽事開設成功！開放下注。', '🎉');
            switchTab('betting'); // 跳轉到投注大廳
        }

        // 結算賽事並自動派彩
        function settleMatch(matchId) {
            const selected = document.querySelector(`input[name="settle-${matchId}"]:checked`);
            if (!selected) return showToast('請先選出一個勝出的選項！', '⚠️');

            const winId = selected.value;
            const match = state.matches.find(m => m.id === matchId);
            
            match.settled = true;
            match.winner = winId;

            let winnerCount = 0;
            let totalPayout = 0;

            // 對所有該賽事的投注進行清算與派彩
            state.bets.filter(b => b.matchId === matchId && b.status === 'pending').forEach(bet => {
                if (bet.optionId === winId) {
                    bet.status = 'won';
                    bet.payout = Math.floor(bet.amount * bet.odds);
                    const user = state.users.find(u => u.name === bet.username);
                    if (user) {
                        user.balance += bet.payout;
                    }
                    winnerCount++;
                    totalPayout += bet.payout;
                } else {
                    bet.status = 'lost';
                }
            });

            saveGameData();
            renderAll();
            showToast(`結算完成！共有 ${winnerCount} 位贏家，共派彩 $${totalPayout.toLocaleString()}！`, '💰');
        }

        // 刪除玩家帳戶
        function deleteUser(name) {
            if (state.users.length <= 1) return showToast('必須保留至少一名玩家！', '⚠️');
            if (confirm(`確定要刪除玩家「${name}」的帳戶和資料嗎？`)) {
                state.users = state.users.filter(u => u.name !== name);
                if (state.currentUser === name) {
                    state.currentUser = state.users[0].name;
                }
                // 同步刪除其未結算的投注
                state.bets = state.bets.filter(b => b.username !== name);
                saveGameData();
                renderAll();
                showToast(`已刪除 ${name}。`, '🗑️');
            }
        }

        // 底部導航欄分頁切換邏輯
        function switchTab(tabId) {
            ['betting', 'records', 'admin'].forEach(id => {
                // 隱藏非當前分頁
                document.getElementById(`section-${id}`).classList.add('hidden');
                document.getElementById(`section-${id}`).classList.remove('block');
                
                // 重設導航列按鈕樣式為未選中狀態
                const navBtn = document.getElementById(`nav-${id}`);
                navBtn.className = "flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-transparent text-gray-400 hover:bg-gray-50 transition-all";
            });

            // 顯示當前選中的分頁
            document.getElementById(`section-${tabId}`).classList.remove('hidden');
            document.getElementById(`section-${tabId}`).classList.add('block');
            
            // 亮起選中的導航按鈕樣式 (HKJC 經典深藍與藍底)
            const activeBtn = document.getElementById(`nav-${tabId}`);
            activeBtn.className = "flex-1 py-3 flex flex-col items-center justify-center gap-1 border-t-4 border-hkjc-blue bg-blue-50 text-hkjc-blue transition-all";
            
            // 頁面自動平滑捲動到最頂部
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        // 網頁加載完畢後自動執行
        window.onload = () => {
            loadGameData();
            renderAll();
        };
    </script>
</body>
</html>
