<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Snake Neo-Pulse Pro Ultimate +</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');
        :root { --main-cyan: #00f2ff; --bg-dark: #050505; --neon-red: #ff0055; }

        html, body {
            background: var(--bg-dark); color: #fff; font-family: 'Orbitron', sans-serif;
            margin: 0; padding: 0; display: flex; flex-direction: column; 
            align-items: center; justify-content: center; min-height: 100vh; 
            overflow: hidden; touch-action: none;
        }

        .header { display: flex; width: 85vw; max-width: 400px; justify-content: space-between; align-items: flex-start; margin-bottom: 10px; }
        .ui-panel { color: var(--main-cyan); text-shadow: 0 0 10px var(--main-cyan); font-size: 0.8em; line-height: 1.5; }
        .settings-btn { background: none; border: none; color: var(--main-cyan); font-size: 1.8em; cursor: pointer; padding: 0; }

        canvas {
            border: 3px solid var(--main-cyan); background: #000; 
            box-shadow: 0 0 20px rgba(0, 242, 255, 0.2);
            width: 90vw; max-width: 380px; border-radius: 10px;
        }

        .controls-wrapper { margin-top: 20px; display: flex; justify-content: center; align-items: center; min-height: 220px; width: 100%; }

        /* D-PAD */
        .dpad-container { display: grid; grid-template-areas: ". up ." "left . right" ". down ."; gap: 15px; }
        .control-btn {
            width: 80px; height: 80px; background: rgba(0, 242, 255, 0.1);
            border: 3px solid var(--main-cyan); border-radius: 15px;
            display: flex; align-items: center; justify-content: center;
            color: var(--main-cyan); font-size: 2em; touch-action: none;
        }

        /* JOYSTICK */
        .joystick-base {
            width: 160px; height: 160px; background: rgba(0, 242, 255, 0.05);
            border: 4px solid rgba(0, 242, 255, 0.3); border-radius: 50%;
            position: relative; display: flex; align-items: center; justify-content: center;
        }
        .joystick-stick {
            width: 60px; height: 60px; background: #111;
            border: 3px solid var(--main-cyan); border-radius: 50%;
            transition: transform 0.1s ease, box-shadow 0.1s ease;
            box-shadow: 0 0 10px var(--main-cyan);
        }
        .stick-right { transform: translateX(35px); box-shadow: 15px 0 25px var(--main-cyan); border-color: #fff; }
        .stick-left { transform: translateX(-35px); box-shadow: -15px 0 25px var(--main-cyan); border-color: #fff; }
        .stick-up { transform: translateY(-35px); box-shadow: 0 -15px 25px var(--main-cyan); border-color: #fff; }
        .stick-down { transform: translateY(35px); box-shadow: 0 15px 25px var(--main-cyan); border-color: #fff; }

        .modal {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.95); display: none;
            flex-direction: column; align-items: center; justify-content: center; z-index: 1000;
        }
        .modal-content { background: #111; padding: 25px; border: 2px solid var(--main-cyan); border-radius: 20px; text-align: center; width: 300px; }

        .diff-selector { display: flex; flex-direction: column; gap: 8px; margin: 10px 0 20px 0; }
        .diff-option { padding: 12px; border: 1px solid var(--main-cyan); border-radius: 8px; color: var(--main-cyan); cursor: pointer; font-family: 'Orbitron'; }
        .diff-option.active { background: var(--main-cyan); color: #000; font-weight: bold; }

        .btn-action { background: var(--main-cyan); color: #000; font-weight: bold; padding: 15px; border: none; border-radius: 8px; width: 100%; cursor: pointer; font-family: 'Orbitron'; margin-top: 10px; }
        select { width: 100%; padding: 10px; background: #000; color: #fff; border: 1px solid var(--main-cyan); font-family: 'Orbitron'; border-radius: 5px; }
    </style>
</head>
<body>

    <div class="header">
        <div class="ui-panel">
            SCORE: <span id="score">0</span><br>
            BEST: <span id="bestScore">0</span><br>
            TIME: <span id="timer">00:00</span>
        </div>
        <button class="settings-btn" onclick="openSettings()">⚙️</button>
    </div>

    <canvas id="gameCanvas" width="400" height="400"></canvas>
    <div class="controls-wrapper" id="controlsWrapper"></div>

    <div id="menuSettings" class="modal">
        <div class="modal-content">
            <h2 style="color:var(--main-cyan); margin-bottom: 20px;">CONFIGURACIÓN</h2>
            <label style="font-size: 0.7em; color: var(--main-cyan);">DIFICULTAD</label>
            <div class="diff-selector">
                <div class="diff-option" id="opt-easy" onclick="setDiff('easy')">Fácil</div>
                <div class="diff-option active" id="opt-normal" onclick="setDiff('normal')">Normal</div>
                <div class="diff-option" id="opt-hard" onclick="setDiff('hard')">Difícil</div>
            </div>
            <label style="font-size: 0.7em; color: var(--main-cyan);">ESTILO DE CONTROL</label>
            <select id="controlStyleSelect" onchange="changeControlStyle()">
                <option value="dpad">Cuadritos (D-Pad)</option>
                <option value="joystick">Círculo (Joystick)</option>
            </select>
            <label style="font-size: 0.7em; color: var(--main-cyan); display: block; margin-top: 15px;">TAMAÑO</label>
            <input type="range" id="sizeSlider" min="50" max="110" value="85" oninput="updateControlSize()" style="width:100%; accent-color:var(--main-cyan);">
            <button class="btn-action" onclick="closeSettings()">CONFIRMAR</button>
        </div>
    </div>

    <div id="menuGameOver" class="modal">
        <div class="modal-content">
            <h1 style="color:var(--neon-red); text-shadow: 0 0 15px var(--neon-red);">GAME OVER</h1>
            <p id="finalScore" style="font-size: 1.2em;">SCORE: 0</p>
            <p id="timeResult" style="font-size: 1em; color: var(--main-cyan);">TIEMPO: 00:00</p>
            <button class="btn-action" onclick="reiniciarJuego()">REINTENTAR</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        let snake, comida = [], dx, dy, nextDir, score, running = false, waitingToStart = true;
        let difficulty = "normal", speed = 95, lastTime = 0;
        let bestScore = localStorage.getItem("snakeBestScore") || 0;
        let seconds = 0, timerInterval = null;
        let audioCtx;

        document.getElementById("bestScore").innerText = bestScore;

        function initAudio() { if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)(); }

        // SONIDO POTENCIADO
        function playSound(freq, dur, vol, type = 'sine') {
            if (!audioCtx) return;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(vol, audioCtx.currentTime); // Volumen más alto
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + dur);
            osc.connect(gain); gain.connect(audioCtx.destination);
            osc.start(); osc.stop(audioCtx.currentTime + dur);
        }

        function updateTimer() {
            seconds++;
            let mins = Math.floor(seconds / 60).toString().padStart(2, '0');
            let secs = (seconds % 60).toString().padStart(2, '0');
            document.getElementById("timer").innerText = `${mins}:${secs}`;
        }

        function openSettings() {
            running = false;
            clearInterval(timerInterval);
            document.getElementById("menuSettings").style.display = "flex";
        }

        function closeSettings() {
            document.getElementById("menuSettings").style.display = "none";
            if (!waitingToStart) {
                running = true;
                timerInterval = setInterval(updateTimer, 1000);
                requestAnimationFrame(gameLoop);
            }
        }

        function setDiff(val) {
            difficulty = val;
            document.querySelectorAll('.diff-option').forEach(el => el.classList.remove('active'));
            document.getElementById('opt-' + val).classList.add('active');
            speed = (val === "easy") ? 135 : (val === "normal") ? 95 : 65;
            reiniciarJuego();
        }

        function changeControlStyle() {
            const style = document.getElementById("controlStyleSelect").value;
            const wrapper = document.getElementById("controlsWrapper");
            wrapper.innerHTML = (style === "dpad") ? 
                `<div class="dpad-container"><div class="control-btn" style="grid-area: up" ontouchstart="handleInput(0,-20)">▲</div><div class="control-btn" style="grid-area: left" ontouchstart="handleInput(-20,0)">◀</div><div class="control-btn" style="grid-area: right" ontouchstart="handleInput(20,0)">▶</div><div class="control-btn" style="grid-area: down" ontouchstart="handleInput(0,20)">▼</div></div>` :
                `<div class="joystick-base" id="jBase" ontouchmove="handleJoystick(event)" ontouchend="resetStick()"><div class="joystick-stick" id="jStick"></div></div>`;
            updateControlSize();
        }

        function handleJoystick(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const base = document.getElementById("jBase");
            const stick = document.getElementById("jStick");
            const rect = base.getBoundingClientRect();
            const x = touch.clientX - (rect.left + rect.width / 2);
            const y = touch.clientY - (rect.top + rect.height / 2);
            stick.className = "joystick-stick";
            if (Math.abs(x) > Math.abs(y)) {
                if (x > 20) { handleInput(20, 0); stick.classList.add('stick-right'); }
                else if (x < -20) { handleInput(-20, 0); stick.classList.add('stick-left'); }
            } else {
                if (y > 20) { handleInput(0, 20); stick.classList.add('stick-down'); }
                else if (y < -20) { handleInput(0, -20); stick.classList.add('stick-up'); }
            }
        }

        function resetStick() { const s = document.getElementById("jStick"); if(s) s.className = "joystick-stick"; }

        function updateControlSize() {
            const size = document.getElementById("sizeSlider").value;
            const style = document.getElementById("controlStyleSelect").value;
            if (style === "dpad") {
                document.querySelectorAll('.control-btn').forEach(btn => { btn.style.width = size + "px"; btn.style.height = size + "px"; });
            } else {
                const base = document.getElementById("jBase");
                if(base) { base.style.width = (size * 1.9) + "px"; base.style.height = (size * 1.9) + "px"; }
            }
        }

        function handleInput(nx, ny) {
            if (nx === -dx || ny === -dy) return;
            nextDir = {x: nx, y: ny};
            if (waitingToStart) { 
                waitingToStart = false; 
                running = true; 
                initAudio();
                timerInterval = setInterval(updateTimer, 1000);
                requestAnimationFrame(gameLoop); 
            }
        }

        function generarComida() {
            comida = [];
            const cant = (difficulty === "hard") ? 3 : 1;
            while(comida.length < cant) {
                let p = { x: Math.floor(Math.random()*20)*20, y: Math.floor(Math.random()*20)*20 };
                if (!snake.some(s => s.x === p.x && s.y === p.y)) comida.push(p);
            }
        }

        function reiniciarJuego() {
            snake = [{x: 200, y: 200}, {x: 180, y: 200}, {x: 160, y: 200}];
            dx = 20; dy = 0; nextDir = {x: 20, y: 0};
            score = 0; running = false; waitingToStart = true;
            seconds = 0; clearInterval(timerInterval);
            document.getElementById("score").innerText = "0";
            document.getElementById("timer").innerText = "00:00";
            document.getElementById("menuGameOver").style.display = "none";
            generarComida();
            dibujar();
        }

        function gameLoop(timestamp) {
            if (!running) return;
            if (timestamp - lastTime > speed) { mover(); lastTime = timestamp; }
            dibujar();
            requestAnimationFrame(gameLoop);
        }

        function mover() {
            dx = nextDir.x; dy = nextDir.y;
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            if (head.x < 0 || head.x >= 400 || head.y < 0 || head.y >= 400 || snake.some(p => p.x === head.x && p.y === head.y)) {
                running = false;
                clearInterval(timerInterval);
                playSound(150, 0.5, 0.4, 'sawtooth'); // Sonido Muerte
                if (score > bestScore) {
                    bestScore = score;
                    localStorage.setItem("snakeBestScore", bestScore);
                    document.getElementById("bestScore").innerText = bestScore;
                }
                document.getElementById("finalScore").innerText = "SCORE: " + score;
                document.getElementById("timeResult").innerText = "TIEMPO: " + document.getElementById("timer").innerText;
                document.getElementById("menuGameOver").style.display = "flex";
                return;
            }
            snake.unshift(head);
            let eaten = false;
            comida.forEach((f, i) => {
                if(head.x === f.x && head.y === f.y) {
                    score += 10; document.getElementById("score").innerText = score;
                    playSound(800, 0.1, 0.3); // Sonido Comer
                    comida.splice(i, 1); eaten = true;
                }
            });
            if (eaten) { 
                const max = (difficulty === "hard") ? 3 : 1;
                while(comida.length < max) {
                    let p = { x: Math.floor(Math.random()*20)*20, y: Math.floor(Math.random()*20)*20 };
                    if(!snake.some(s => s.x === p.x && s.y === p.y)) comida.push(p);
                }
            } else { snake.pop(); }
        }

        function dibujar() {
            ctx.fillStyle = "#000"; ctx.fillRect(0, 0, 400, 400);
            comida.forEach(f => {
                ctx.fillStyle = "#fff"; ctx.shadowBlur = 15; ctx.shadowColor = "#00f2ff";
                ctx.beginPath(); ctx.arc(f.x+10, f.y+10, 8, 0, 7); ctx.fill();
            });
            snake.forEach((p, i) => {
                ctx.shadowBlur = 10; ctx.shadowColor = "#00f2ff";
                if (difficulty === "easy") {
                    ctx.fillStyle = (i === 0) ? "#fff" : "#00f2ff";
                    ctx.fillRect(p.x + 1, p.y + 1, 18, 18);
                } else if (difficulty === "normal") {
                    ctx.strokeStyle = (i === 0) ? "#fff" : "#00f2ff"; ctx.lineWidth = 5;
                    ctx.strokeRect(p.x + 2.5, p.y + 2.5, 15, 15);
                } else {
                    ctx.strokeStyle = (i === 0) ? "#fff" : "#00f2ff"; ctx.lineWidth = 2.5;
                    ctx.strokeRect(p.x + 3, p.y + 3, 14, 14);
                }
            });
            ctx.shadowBlur = 0;
        }

        changeControlStyle();
        reiniciarJuego();
    </script>
</body>
</html>
#80c3e7
