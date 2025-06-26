# 01-boss
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Aviator Crash Game</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #111;
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 40px;
    }
    h1 {
      color: #ff3366;
    }
    .multiplier {
      font-size: 4em;
      margin: 20px 0;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
    }
    .start { background: #28a745; color: #fff; }
    .cashout { background: #ffc107; color: #000; }
    .status { margin-top: 20px; font-size: 18px; }
  </style>
</head>
<body>

  <h1>🚀 Aviator Game</h1>
  <div class="multiplier" id="multiplier">1.00x</div>
  <button class="start" id="startBtn">Start Game</button>
  <button class="cashout" id="cashoutBtn" disabled>Cash Out</button>
  <div class="status" id="status">Press Start to Play</div>

  <script>
    let multiplier = 1.00;
    let interval;
    let isPlaying = false;
    let crashed = false;
    let crashPoint;

    const multiplierDisplay = document.getElementById('multiplier');
    const statusDisplay = document.getElementById('status');
    const startBtn = document.getElementById('startBtn');
    const cashoutBtn = document.getElementById('cashoutBtn');

    function getCrashPoint() {
      const r = Math.random();
      return r < 0.01 ? 1.00 : parseFloat((Math.max(1.00, 2 + r * 10)).toFixed(2));
    }

    function startGame() {
      multiplier = 1.00;
      isPlaying = true;
      crashed = false;
      crashPoint = getCrashPoint();
      statusDisplay.textContent = `Game started! Crash at ${crashPoint}x`;
      multiplierDisplay.textContent = '1.00x';
      cashoutBtn.disabled = false;

      interval = setInterval(() => {
        multiplier = parseFloat((multiplier + 0.05).toFixed(2));
        multiplierDisplay.textContent = `${multiplier}x`;

        if (multiplier >= crashPoint) {
          clearInterval(interval);
          crashed = true;
          isPlaying = false;
          cashoutBtn.disabled = true;
          statusDisplay.textContent = `💥 Crashed at ${crashPoint}x! You lost.`;
        }
      }, 100);
    }

    function cashOut() {
      if (isPlaying && !crashed) {
        clearInterval(interval);
        isPlaying = false;
        cashoutBtn.disabled = true;
        statusDisplay.textContent = `✅ You cashed out at ${multiplier}x!`;
      }
    }

    startBtn.addEventListener('click', startGame);
    cashoutBtn.addEventListener('click', cashOut);
  </script>

</body>
</html>
