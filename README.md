<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        :root {
            --bg-color: transparent;
            --text-color: #37352f;
            --accent-color: #eb5757;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: color 0.3s ease;
        }

        /* Modos */
        .modes {
            display: flex;
            gap: 10px;
            margin-bottom: 5px;
        }

        .mode-btn {
            font-size: 0.75rem;
            text-transform: uppercase;
            letter-spacing: 1px;
            cursor: pointer;
            border: none;
            background: none;
            color: #a0a0a0;
            padding: 5px 10px;
            transition: 0.3s;
        }

        .mode-btn.active {
            color: var(--accent-color);
            font-weight: bold;
            border-bottom: 2px solid var(--accent-color);
        }

        /* Timer */
        .timer-display {
            font-size: 4.5rem;
            font-weight: 700;
            margin: 5px 0;
            letter-spacing: -3px;
            color: var(--text-color);
        }

        /* Botones de Control */
        .controls {
            display: flex;
            gap: 15px;
        }

        .controls button {
            background: none;
            border: 1px solid #e0e0e0;
            padding: 8px 20px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.85rem;
            color: var(--text-color);
            transition: all 0.2s ease;
        }

        .controls button:hover {
            background: #f4f4f4;
            border-color: var(--accent-color);
        }

        /* Configuración */
        .settings-btn {
            position: fixed;
            top: 15px;
            right: 15px;
            cursor: pointer;
            opacity: 0.2;
            font-size: 1.2rem;
            transition: opacity 0.3s;
        }

        .settings-btn:hover { opacity: 1; }

        #settings-panel {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            border: 1px solid #eee;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            z-index: 100;
            width: 200px;
        }

        .settings-field {
            margin-bottom: 12px;
        }

        .settings-field label {
            display: block;
            font-size: 0.8rem;
            margin-bottom: 4px;
            color: #666;
        }

        input {
            width: 100%;
            padding: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
            box-sizing: border-box;
        }

        .save-btn {
            width: 100%;
            background: var(--text-color);
            color: white;
            border: none;
            padding: 8px;
            border-radius: 6px;
            cursor: pointer;
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <div class="settings-btn" onclick="toggleSettings()">⚙️</div>

    <div id="settings-panel">
        <div class="settings-field">
            <label>Minutos Trabajo:</label>
            <input type="number" id="workInput" value="25">
        </div>
        <div class="settings-field">
            <label>Minutos Descanso:</label>
            <input type="number" id="breakInput" value="5">
        </div>
        <div class="settings-field">
            <label>Color de Acento:</label>
            <input type="color" id="colorInput" value="#eb5757">
        </div>
        <button class="save-btn" onclick="saveSettings()">Guardar Configuración</button>
    </div>

    <div class="modes">
        <button class="mode-btn active" id="workModeBtn" onclick="switchMode('work')">Work</button>
        <button class="mode-btn" id="breakModeBtn" onclick="switchMode('break')">Break</button>
    </div>

    <div class="timer-display" id="display">25:00</div>
    
    <div class="controls">
        <button onclick="startTimer()" id="startBtn">Start</button>
        <button onclick="pauseTimer()">Pause</button>
        <button onclick="resetTimer()">Reset</button>
    </div>

    <audio id="alarmSound" src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg"></audio>

    <script>
        // Cargar configuración guardada o usar valores por defecto
        let workMins = localStorage.getItem('workMins') || 25;
        let breakMins = localStorage.getItem('breakMins') || 5;
        let accentColor = localStorage.getItem('accentColor') || '#eb5757';
        
        let timeLeft = workMins * 60;
        let timerId = null;
        let currentMode = 'work'; // 'work' o 'break'

        const display = document.getElementById('display');
        const alarm = document.getElementById('alarmSound');

        // Inicializar visualmente
        function init() {
            document.documentElement.style.setProperty('--accent-color', accentColor);
            document.getElementById('workInput').value = workMins;
            document.getElementById('breakInput').value = breakMins;
            document.getElementById('colorInput').value = accentColor;
            updateDisplay();
        }

        function updateDisplay() {
            const minutes = Math.floor(timeLeft / 60);
            const seconds = timeLeft % 60;
            display.textContent = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
            document.title = `(${display.textContent}) Notion Timer`;
        }

        function startTimer() {
            if (timerId) return;
            timerId = setInterval(() => {
                timeLeft--;
                updateDisplay();
                if (timeLeft <= 0) {
                    clearInterval(timerId);
                    timerId = null;
                    alarm.play();
                    alert("¡Tiempo completado!");
                }
            }, 1000);
        }

        function pauseTimer() {
            clearInterval(timerId);
            timerId = null;
        }

        function resetTimer() {
            pauseTimer();
            timeLeft = (currentMode === 'work' ? workMins : breakMins) * 60;
            updateDisplay();
        }

        function switchMode(mode) {
            currentMode = mode;
            document.getElementById('workModeBtn').classList.toggle('active', mode === 'work');
            document.getElementById('breakModeBtn').classList.toggle('active', mode === 'break');
            resetTimer();
        }

        function toggleSettings() {
            const panel = document.getElementById('settings-panel');
            panel.style.display = panel.style.display === 'block' ? 'none' : 'block';
        }

        function saveSettings() {
            workMins = document.getElementById('workInput').value;
            breakMins = document.getElementById('breakInput').value;
            accentColor = document.getElementById('colorInput').value;

            // Guardar en el navegador
            localStorage.setItem('workMins', workMins);
            localStorage.setItem('breakMins', breakMins);
            localStorage.setItem('accentColor', accentColor);

            document.documentElement.style.setProperty('--accent-color', accentColor);
            resetTimer();
            toggleSettings();
        }

        init();
    </script>
</body>
</html>
