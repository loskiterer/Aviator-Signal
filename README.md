<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LoskiDeriv Aviator | Live Game</title>
    <style>
        :root {
            --aviator-red: #d61f27;
            --bg-dark: #141516;
            --card-bg: #1f2024;
            --success: #28a745;
            --text-gray: #9ea0a3;
        }

        body { font-family: 'Inter', sans-serif; background: var(--bg-dark); color: white; margin: 0; overflow: hidden; }
        
        /* Game Screen */
        .game-canvas {
            width: 100%;
            height: 350px;
            background: #000;
            position: relative;
            border-bottom: 2px solid #333;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
        }

        .multiplier {
            font-size: 5rem;
            font-weight: 800;
            z-index: 10;
            transition: color 0.2s;
        }

        .plane {
            position: absolute;
            bottom: 20%;
            left: 10%;
            width: 80px;
            z-index: 5;
            transition: transform 0.1s linear;
        }

        /* Betting Area */
        .betting-container {
            padding: 15px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            background: var(--bg-dark);
        }

        .bet-box {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 12px;
            text-align: center;
            border: 1px solid #333;
        }

        .input-group { display: flex; justify-content: center; margin: 10px 0; }
        input {
            width: 80px;
            background: #000;
            border: 1px solid #444;
            color: white;
            padding: 8px;
            text-align: center;
            border-radius: 4px;
        }

        .btn-bet {
            width: 100%;
            padding: 12px;
            border-radius: 8px;
            border: none;
            font-weight: bold;
            font-size: 1rem;
            text-transform: uppercase;
            cursor: pointer;
        }

        .btn-ready { background: var(--success); color: white; }
        .btn-cashout { background: #ff9800; color: white; }
        .disabled { background: #555 !important; cursor: not-allowed; }

        .flew-away { color: var(--aviator-red); position: absolute; top: 20%; font-weight: bold; font-size: 1.5rem; display: none; }

        /* History */
        .history-bar {
            display: flex;
            gap: 8px;
            padding: 10px;
            background: #1b1c20;
            overflow-x: auto;
        }
        .hist-item { padding: 2px 8px; border-radius: 10px; background: #333; font-size: 0.8rem; }
    </style>
</head>
<body>

<div class="history-bar" id="history">
    <span class="hist-item">1.22x</span>
    <span class="hist-item" style="color: #9c27b0;">15.4x</span>
    <span class="hist-item">2.05x</span>
</div>

<div class="game-canvas">
    <div id="multiplierText" class="multiplier">1.00x</div>
    <div id="flewAway" class="flew-away">FLEW AWAY!</div>
    <img src="https://i.ibb.co/L8N7hYy/plane.png" id="plane" class="plane" alt="plane">
    </div>

<div class="betting-container">
    <div class="bet-box">
        <div class="input-group">
            <input type="number" id="betAmount1" value="100">
        </div>
        <button id="btn1" class="btn-bet btn-ready" onclick="handleBet(1)">BET</button>
    </div>

    <div class="bet-box">
        <div class="input-group">
            <input type="number" id="betAmount2" value="100">
        </div>
        <button id="btn2" class="btn-bet btn-ready" onclick="handleBet(2)">BET</button>
    </div>
</div>

<script>
    let currentMultiplier = 1.00;
    let crashPoint = 0;
    let isFlying = false;
    let betActive1 = false;
    let betActive2 = false;
    let gameInterval;

    const plane = document.getElementById('plane');
    const multText = document.getElementById('multiplierText');
    const flewAwayMsg = document.getElementById('flewAway');

    function startNewRound() {
        // Reset UI
        isFlying = true;
        currentMultiplier = 1.00;
        flewAwayMsg.style.display = 'none';
        multText.style.color = 'white';
        
        // Decide crash point (Mimicking RNG)
        // 10% chance of instant crash at 1.00
        crashPoint = Math.random() < 0.1 ? 1.00 : (Math.random() * 5 + 1).toFixed(2);

        gameInterval = setInterval(() => {
            if (currentMultiplier >= crashPoint) {
                endRound();
            } else {
                currentMultiplier += 0.01;
                multText.innerText = currentMultiplier.toFixed(2) + "x";
                
                // Animate plane position
                let move = (currentMultiplier - 1) * 20;
                plane.style.transform = `translate(${move * 2}px, -${move}px)`;
            }
        }, 50);
    }

    function endRound() {
        clearInterval(gameInterval);
        isFlying = false;
        flewAwayMsg.style.display = 'block';
        multText.style.color = 'var(--aviator-red)';
        
        // Reset Buttons
        resetButton(1);
        resetButton(2);

        // Record history
        const hist = document.getElementById('history');
        const span = document.createElement('span');
        span.className = 'hist-item';
        span.innerText = currentMultiplier.toFixed(2) + "x";
        hist.prepend(span);

        // Auto-start next round after 5 seconds
        setTimeout(startNewRound, 5000);
    }

    function handleBet(num) {
        const btn = document.getElementById('btn' + num);
        
        if (!isFlying) {
            // Place bet for next round
            btn.innerText = "CANCEL";
            btn.classList.replace('btn-ready', 'btn-cashout');
            if(num === 1) betActive1 = true; else betActive2 = true;
        } else {
            // Cash out logic
            if ((num === 1 && betActive1) || (num === 2 && betActive2)) {
                alert(`Cashed out at ${currentMultiplier.toFixed(2)}x!`);
                resetButton(num);
            }
        }
    }

    function resetButton(num) {
        const btn = document.getElementById('btn' + num);
        btn.innerText = "BET";
        btn.classList.replace('btn-cashout', 'btn-ready');
        if(num === 1) betActive1 = false; else betActive2 = false;
    }

    // Start first round
    startNewRound();
</script>

</body>
</html>
