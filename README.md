<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Aviator Pro - Precision Flight</title>
    
    <link rel="manifest" id="pwa-manifest">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="theme-color" content="#0b0b0b">
    
    <link rel="apple-touch-icon" href="my-photo.jpg">

    <style>
        :root {
            --bg-dark: #0b0b0b;
            --panel-bg: #1c1c1c;
            --accent-gold: #ffca28;
            --accent-red: #e91e63;
            --success-green: #28a745;
            --mpesa-green: #49aa27;
            --text-main: #ffffff;
            --text-dim: #888;
        }

        body {
            background-color: var(--bg-dark); color: var(--text-main);
            font-family: 'Segoe UI', Roboto, sans-serif;
            margin: 0; padding: 0; display: flex; justify-content: center;
            height: 100vh; overflow: hidden;
        }

        /* REGISTRATION OVERLAY */
        #auth-overlay {
            position: fixed; inset: 0; background: rgba(0,0,0,0.95);
            z-index: 2000; display: none; justify-content: center; align-items: center; padding: 20px;
        }
        .auth-card {
            background: #1c1c1c; width: 100%; max-width: 350px; border-radius: 20px;
            padding: 30px; text-align: center; border: 1px solid #333; box-shadow: 0 10px 40px rgba(0,0,0,0.8);
        }
        .auth-input {
            width: 100%; padding: 12px; margin-bottom: 15px; background: #000;
            border: 1px solid #444; color: white; border-radius: 8px; box-sizing: border-box; outline: none;
        }
        .auth-input:focus { border-color: var(--mpesa-green); }

        /* GAME LAYOUT */
        .game-container {
            width: 100vw; height: 100vh; max-width: 400px;
            background: var(--panel-bg); display: flex; flex-direction: column;
            position: relative;
        }

        #toast {
            position: absolute; top: -100px; left: 10px; right: 10px;
            background: #333; color: white; padding: 12px; border-radius: 8px;
            z-index: 1000; transition: 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            text-align: center; border-left: 5px solid var(--mpesa-green);
            box-shadow: 0 5px 15px rgba(0,0,0,0.5); font-size: 0.85rem;
        }
        #toast.show { top: 20px; }

        .history-bar {
            background: #000; height: 35px; display: flex; align-items: center;
            gap: 8px; padding: 0 10px; overflow-x: auto; border-bottom: 1px solid #333;
        }
        .hist-val { font-size: 0.65rem; font-weight: bold; padding: 3px 8px; border-radius: 10px; background: #222; white-space: nowrap; }

        .header {
            padding: 10px 12px; display: flex; justify-content: space-between;
            align-items: center; background: #1a1a1a; border-bottom: 1px solid #333;
        }

        .chart-area { flex: 1.5; background: #050505; position: relative; overflow: hidden; border-bottom: 2px solid #111; }

        #multiplier {
            position: absolute; top: 25%; width: 100%; text-align: center;
            font-size: 3.5rem; font-weight: 900; z-index: 10;
        }

        #next-round-timer {
            position: absolute; top: 40%; width: 100%; text-align: center; z-index: 30; display: none;
        }
        .progress-bar { width: 140px; height: 6px; background: #333; margin: 10px auto; border-radius: 5px; overflow: hidden; }
        #progress-fill { width: 100%; height: 100%; background: var(--mpesa-green); transition: width 0.05s linear; }

        #plane-wrapper {
            position: absolute; bottom: 0; left: 0; width: 60px; z-index: 25; display: none;
            margin-left: -5px; margin-bottom: -5px; 
        }
        .avatar-icon {
            width: 100%; height: 60px; border-radius: 50%; border: 2px solid var(--accent-red);
            box-shadow: 0 0 15px var(--accent-red); object-fit: cover;
        }

        #flight-curve { position: absolute; width: 100%; height: 100%; z-index: 20; }

        .bet-section { padding: 12px; background: #111; display: flex; flex-direction: column; gap: 12px; }
        .bet-row {
            display: grid; grid-template-columns: 1fr 1.2fr; gap: 10px;
            background: #1c1c1c; padding: 12px; border-radius: 12px; border: 1px solid #333;
        }
        .bet-input {
            background: #000; color: #fff; border: 1px solid #444; border-radius: 8px;
            text-align: center; font-weight: bold; font-size: 1.2rem; width: 100%; outline: none;
        }
        .bet-btn {
            border-radius: 8px; border: none; font-weight: 900; color: white;
            height: 50px; cursor: pointer; font-size: 1.1rem; text-transform: uppercase;
        }

        .players-box {
            flex: 1; background: #080808; margin: 0 12px 12px; border-radius: 12px;
            overflow-y: auto; padding: 12px; border: 1px solid #222;
        }
        .player-header { color: var(--text-dim); font-size: 0.6rem; font-weight: bold; margin-bottom: 8px; text-transform: uppercase; border-bottom: 1px solid #222; padding-bottom: 5px; }
        .player-row { display: flex; justify-content: space-between; font-size: 0.75rem; padding: 6px 0; border-bottom: 1px solid #151515; }

        .modal {
            display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9);
            z-index: 600; justify-content: center; align-items: center;
        }
    </style>
</head>
<body>

<div id="auth-overlay">
    <div class="auth-card">
        <img src="my-photo.jpg" style="width:90px; height:90px; border-radius:50%; border:3px solid var(--mpesa-green); margin-bottom:15px; object-fit: cover;">
        <h2 style="margin:0 0 10px 0; color:white;">Aviator Pro</h2>
        <p style="color:#888; font-size:0.85rem; margin-bottom:25px;">Enter details to sync with your M-Pesa Wallet</p>
        
        <input type="text" id="reg-name" class="auth-input" placeholder="Username / Nickname">
        <input type="number" id="reg-phone" class="auth-input" placeholder="M-Pesa Phone Number">
        
        <button onclick="registerUser()" style="width:100%; padding:16px; background:var(--mpesa-green); border:none; color:white; border-radius:10px; font-weight:bold; font-size:1.1rem; cursor:pointer; margin-top:10px;">JOIN & PLAY</button>
        <p style="font-size:0.65rem; color:#555; margin-top:15px;">By clicking, you confirm you are 18+ years old.</p>
    </div>
</div>

<div id="toast">Message</div>

<div class="game-container">
    <div class="history-bar" id="history-list"></div>

    <div class="header">
        <div>
            <div id="user-display" style="font-size: 0.55rem; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase;">WALLET BALANCE</div>
            <div id="balance-amount" style="color:var(--success-green); font-weight:800; font-size: 1.1rem;">KES 1,000.00</div>
        </div>
        <div style="display:flex; gap:8px;">
            <button onclick="triggerWithdraw()" style="background:#333; border:none; color:white; border-radius:6px; padding:6px 12px; font-weight:bold; font-size:0.7rem;">WITHDRAW</button>
            <button onclick="triggerMpesaPush()" style="background:var(--mpesa-green); border:none; color:white; border-radius:6px; padding:6px 15px; font-weight:bold; font-size:0.7rem;">DEPOSIT</button>
        </div>
    </div>

    <div class="chart-area" id="chart">
        <div id="next-round-timer">
            <div style="color: var(--accent-gold); font-weight: bold; font-size: 0.8rem;">WAITING FOR NEXT ROUND</div>
            <div class="progress-bar"><div id="progress-fill"></div></div>
        </div>
        
        <svg id="flight-curve" viewBox="0 0 400 300" preserveAspectRatio="none">
            <path id="curve-path" d="" fill="none" stroke="#e91e63" stroke-width="4" stroke-linecap="round"/>
        </svg>

        <div id="multiplier">1.00x</div>
        
        <div id="plane-wrapper">
            <img src="my-photo.jpg" class="avatar-icon">
        </div>
    </div>

    <div class="bet-section">
        <div class="bet-row">
            <input type="number" id="bet-1-amt" class="bet-input" value="100" min="10">
            <button id="btn-1" class="bet-btn" onclick="handleBetAction(1)" style="background:var(--success-green);">BET</button>
        </div>
        <div class="bet-row">
            <input type="number" id="bet-2-amt" class="bet-input" value="100" min="10">
            <button id="btn-2" class="bet-btn" onclick="handleBetAction(2)" style="background:var(--success-green);">BET</button>
        </div>
    </div>
    
    <div class="players-box" id="player-feed">
        <div class="player-header">LIVE BETS</div>
        <div id="bet-list"></div>
    </div>
</div>

<div id="mpesa-modal" class="modal">
    <div style="background:green; padding:25px; border-radius:15px; width:260px; text-align:center;">
        <h3 style="color:var(--mpesa-green); margin-top:0;">M-PESA DEPOSIT</h3>
        <input type="password" id="mpesa-pin" maxlength="4" placeholder="ENTER PIN" style="width:90%; padding:12px; margin-bottom:15px; text-align:center; font-size:1.5rem; letter-spacing:5px; border: 1px solid #ccc; border-radius: 8px;">
        <button onclick="confirmDeposit()" style="width:100%; padding:14px; background:var(--mpesa-green); border:none; color:white; border-radius:8px; font-weight:bold; font-size: 1rem;">CONFIRM</button>
    </div>
</div>

<script>
    let balance = 1000.00;
    let multiplier = 1.00;
    let isFlying = false;
    let history = ["1.25", "4.50", "1.10", "2.80", "1.02"];
    let bets = { 1: { state: 'none', amount: 0 }, 2: { state: 'none', amount: 0 } };
    let livePlayers = [];

    const namesPool = ["Kioko_KE", "Mama_Wahu", "Nairobi_Ace", "Otieno_Win", "Shiko_254", "Pilot_Mike", "Hustler_01", "Lucky_Bee", "Joe_Mpesa", "Captain_K"];

    // --- REGISTRATION & AUTH ---
    function checkAuth() {
        const savedUser = localStorage.getItem('aviator_pro_user');
        if (!savedUser) {
            document.getElementById('auth-overlay').style.display = 'flex';
        } else {
            const user = JSON.parse(savedUser);
            setupGameForUser(user.name);
        }
    }

    function registerUser() {
        const name = document.getElementById('reg-name').value;
        const phone = document.getElementById('reg-phone').value;
        if (name.length < 2 || phone.length < 10) {
            showToast("Please enter a valid Name & Phone", false);
            return;
        }
        localStorage.setItem('aviator_pro_user', JSON.stringify({name, phone}));
        document.getElementById('auth-overlay').style.display = 'none';
        setupGameForUser(name);
        showToast("Welcome to Aviator Pro!");
    }

    function setupGameForUser(name) {
        document.getElementById('user-display').innerText = name + "'S BALANCE";
        startCooldown();
    }

    // --- LIVE PLAYER LOGIC ---
    function generatePlayers() {
        livePlayers = [];
        const count = 4 + Math.floor(Math.random() * 6);
        for(let i=0; i<count; i++) {
            livePlayers.push({
                name: namesPool[Math.floor(Math.random() * namesPool.length)],
                bet: (Math.random() * 800 + 50).toFixed(0),
                cashOutAt: (Math.random() * 3.8 + 1.05).toFixed(2),
                cashed: false
            });
        }
        renderPlayers();
    }

    function renderPlayers() {
        const list = document.getElementById('bet-list');
        list.innerHTML = livePlayers.map(p => `
            <div class="player-row">
                <span>${p.name} <small style="color:#444">Ksh ${p.bet}</small></span>
                <span>${p.cashed ? '<b style="color:var(--success-green)">' + p.actual + 'x</b>' : '...'}</span>
            </div>
        `).join('');
    }

    function updatePlayers(m) {
        let changed = false;
        livePlayers.forEach(p => {
            if (!p.cashed && m >= p.cashOutAt) {
                p.cashed = true;
                p.actual = m.toFixed(2);
                changed = true;
            }
        });
        if (changed) renderPlayers();
    }

    // --- GAME ENGINE ---
    function showToast(msg, success = true) {
        const t = document.getElementById('toast');
        t.innerText = msg;
        t.style.borderLeftColor = success ? 'var(--mpesa-green)' : 'var(--accent-red)';
        t.classList.add('show');
        setTimeout(() => t.classList.remove('show'), 3000);
    }

    function updateBalanceUI() {
        document.getElementById('balance-amount').innerText = "KES " + balance.toFixed(2);
    }

    function handleBetAction(id) {
        const btn = document.getElementById(`btn-${id}`);
        const input = document.getElementById(`bet-${id}-amt`);
        const amt = parseFloat(input.value);

        if (bets[id].state === 'none') {
            if (amt > balance || amt < 10) {
                showToast("Invalid Amount", false);
                return;
            }
            balance -= amt;
            bets[id] = { state: 'waiting', amount: amt };
            btn.innerText = "CANCEL";
            btn.style.background = "var(--accent-red)";
            updateBalanceUI();
        } 
        else if (bets[id].state === 'waiting') {
            balance += bets[id].amount;
            bets[id] = { state: 'none', amount: 0 };
            btn.innerText = "BET";
            btn.style.background = "var(--success-green)";
            updateBalanceUI();
        } 
        else if (bets[id].state === 'active') {
            const win = bets[id].amount * multiplier;
            balance += win;
            bets[id].state = 'cashed';
            btn.innerText = "WON!";
            btn.style.background = "#333";
            btn.style.color = "var(--success-green)";
            showToast(`CASHOUT: KES ${win.toFixed(2)}`);
            updateBalanceUI();
        }
    }

    function startCooldown() {
        isFlying = false;
        generatePlayers();
        document.getElementById('multiplier').style.display = 'none';
        document.getElementById('plane-wrapper').style.display = 'none';
        document.getElementById('curve-path').setAttribute('d', '');
        document.getElementById('next-round-timer').style.display = 'block';
        
        let timeLeft = 3000;
        const interval = setInterval(() => {
            timeLeft -= 50;
            document.getElementById('progress-fill').style.width = (timeLeft/3000)*100 + "%";
            if (timeLeft <= 0) {
                clearInterval(interval);
                startFlight();
            }
        }, 50);
    }

    function startFlight() {
        isFlying = true;
        multiplier = 1.00;
        const crashPoint = (Math.random() * 5 + 1.01).toFixed(2);
        
        document.getElementById('next-round-timer').style.display = 'none';
        document.getElementById('multiplier').style.display = 'block';
        document.getElementById('multiplier').style.color = 'white';
        document.getElementById('plane-wrapper').style.display = 'block';
        
        [1, 2].forEach(id => {
            if (bets[id].state === 'waiting') {
                bets[id].state = 'active';
                const btn = document.getElementById(`btn-${id}`);
                btn.style.background = "var(--accent-gold)";
                btn.style.color = "black";
            }
        });

        const flightInterval = setInterval(() => {
            multiplier += 0.01;
            const mText = multiplier.toFixed(2);
            document.getElementById('multiplier').innerText = mText + "x";

            updatePlayers(multiplier);

            [1, 2].forEach(id => {
                if (bets[id].state === 'active') {
                    document.getElementById(`btn-${id}`).innerText = "CASH " + (bets[id].amount * multiplier).toFixed(1);
                }
            });

            const progress = multiplier - 1;
            const x = Math.min(progress * 85, 350);
            const y = Math.min(Math.pow(progress * 38, 1.25), 250);
            document.getElementById('plane-wrapper').style.left = x + "px";
            document.getElementById('plane-wrapper').style.bottom = y + "px";
            document.getElementById('curve-path').setAttribute('d', `M 0 300 Q ${x * 0.4} 300 ${x} ${300 - y}`);

            if (multiplier >= crashPoint) {
                clearInterval(flightInterval);
                document.getElementById('curve-path').setAttribute('d', ''); 
                document.getElementById('multiplier').innerText = "FLEW AWAY!";
                document.getElementById('multiplier').style.color = "var(--accent-red)";
                
                history.unshift(mText);
                if (history.length > 10) history.pop();
                renderHistory();
                
                [1,2].forEach(id => {
                    const btn = document.getElementById(`btn-${id}`);
                    btn.innerText = "BET";
                    btn.style.background = "var(--success-green)";
                    btn.style.color = "white";
                    bets[id] = { state: 'none', amount: 0 };
                });

                setTimeout(startCooldown, 2000);
            }
        }, 60);
    }

    function renderHistory() {
        const h = document.getElementById('history-list');
        h.innerHTML = history.map(v => `<span class="hist-val" style="color:${v > 2 ? 'var(--accent-gold)' : '#fff'}">${v}x</span>`).join('');
    }

    function triggerMpesaPush() { document.getElementById('mpesa-modal').style.display = 'flex'; }
    function confirmDeposit() {
        balance += 500;
        updateBalanceUI();
        document.getElementById('mpesa-modal').style.display = 'none';
        showToast("M-PESA: KES 500 received to your Aviator account.");
    }
    function triggerWithdraw() {
        if (balance < 50) return showToast("Minimum withdrawal is KES 50", false);
        showToast(`KES ${balance.toFixed(2)} sent to your M-Pesa account succesfully.`);
        balance = 0;
        updateBalanceUI();
    }

    renderHistory();
    checkAuth();
</script>
</body>
</html>
