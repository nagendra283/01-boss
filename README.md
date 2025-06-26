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
    input, button {
      padding: 10px;
      margin: 8px;
      font-size: 16px;
      border-radius: 8px;
      border: none;
    }
    input {
      width: 120px;
      text-align: center;
    }
    .start { background: #28a745; color: #fff; }
    .cashout { background: #ffc107; color: #000; }
    .logout { background: #dc3545; color: #fff; }
    .status { margin-top: 20px; font-size: 18px; }
    .balance { margin: 10px; font-size: 20px; color: #00ffcc; }
    .inputs { display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; }
    #gameArea { display: none; flex-direction: column; align-items: center; }
    #loginArea { display: flex; flex-direction: column; align-items: center; }
  </style>
</head>
<body>

  <h1>🚀 Aviator Game</h1>

  <!-- LOGIN AREA -->
  <div id="loginArea">
    <p>Enter your username to start:</p>
    <input type="text" id="usernameInput" placeholder="Username">
    <button onclick="login()">Login</button>
  </div>

  <!-- GAME AREA -->
  <div id="gameArea">
    <div class="balance">User: <span id="currentUser"></span> | Balance: $<span id="balance">0.00</span></div>
    <div class="inputs">
      <div>Bet: <input type="number" id="betAmount" value="10" min="1" max="100"></div>
      <div>Auto Cash-Out: <input type="number" id="autoCashOut" value="2.00" min="1.01" step="0.01"></div>
    </div>
    <div class="multiplier" id="multiplier">1.00x</div>
    <button class="start" id="startBtn">Start Game</button>
    <button class="cashout" id="cashoutBtn" disabled>Cash Out</button>
    <button class="logout" onclick="logout()">Logout</button>
    <div class="status" id="status">Place your bet and press Start</div>
  </div>

  <script>
    let multiplier = 1.00;
    let interval;
    let isPlaying = false;
    let crashed = false;
    let crashPoint;
    let currentUser = null;
    let currentBet = 0;
    let autoCashOutValue = 2.00;
    let userBalances = JSON.parse(localStorage.getItem("aviator_users")) || {};

    const loginArea = document.getElementById("loginArea");
    const gameArea = document.getElementById("gameArea");
    const balanceDisplay = document.getElementById('balance');
    const usernameInput = document.getElementById("usernameInput");
    const currentUserSpan = document.getElementById("currentUser");
    const multiplierDisplay = document.getElementById('multiplier');
    const statusDisplay = document.getElementById('status');
    const startBtn = document.getElementById('startBtn');
    const cashoutBtn = document.getElementById('cashoutBtn');
    const betAmountInput = document.getElementById('betAmount');
    const autoCashOutInput = document.getElementById('autoCashOut');

    function login() {
      const name = usernameInput.value.trim();
      if (!name) {
        alert("Enter a username.");
        return;
      }
      currentUser = name;
      if (!userBalances[name]) {
        userBalances[name] = 100.00;
        localStorage.setItem("aviator_users", JSON.stringify(userBalances));
      }
      loginArea.style.display = "none";
      gameArea.style.display = "flex";
      updateUI();
    }

    function logout() {
      saveBalance();
      currentUser = null;
      loginArea.style.display = "flex";
      gameArea.style.display = "none";
      usernameInput.value = "";
    }

    function getBalance() {
      return userBalances[currentUser];
    }

    function setBalance(val) {
      userBalances[currentUser] = val;
      localStorage.setItem("aviator_users", JSON.stringify(userBalances));
    }

    function updateUI() {
      currentUserSpan.textContent = currentUser;
      balanceDisplay.textContent = getBalance().toFixed(2);
    }

    function saveBalance() {
      localStorage.setItem("aviator_users", JSON.stringify(userBalances));
    }

    function getCrashPoint() {
      const r = Math.random();
      return r < 0.01 ? 1.00 : parseFloat((Math.max(1.00, 2 + r * 10)).toFixed(2));
    }

    function startGame() {
      currentBet = parseFloat(betAmountInput.value);
      autoCashOutValue = parseFloat(autoCashOutInput.value);
      let balance = getBalance();

      if (currentBet > balance || currentBet <= 0) {
        statusDisplay.textContent = "❌ Invalid bet amount.";
        return;
      }

      if (autoCashOutValue <= 1.00) {
        statusDisplay.textContent = "❌ Auto cash-out must be > 1.00x";
        return;
      }

      multiplier = 1.00;
      isPlaying = true;
      crashed = false;
      crashPoint = getCrashPoint();
      setBalance(balance - currentBet);
      updateUI();
      statusDisplay.textContent = `🛫 Game started! Crash at ${crashPoint}x`;
      multiplierDisplay.textContent = '1.00x';
      cashoutBtn.disabled = false;
      startBtn.disabled = true;
      betAmountInput.disabled = true;
      autoCashOutInput.disabled = true;

      interval = setInterval(() => {
        multiplier = parseFloat((multiplier + 0.05).toFixed(2));
        multiplierDisplay.textContent = `${multiplier}x`;

        if (multiplier >= autoCashOutValue && isPlaying && !crashed) {
          cashOut(true);
          return;
        }

        if (multiplier >= crashPoint) {
          clearInterval(interval);
          crashed = true;
          isPlaying = false;
          cashoutBtn.disabled = true;
          startBtn.disabled = false;
          betAmountInput.disabled = false;
          autoCashOutInput.disabled = false;
          statusDisplay.textContent = `💥 Crashed at ${crashPoint}x! You lost $${currentBet.toFixed(2)}.`;
          updateUI();
        }
      }, 100);
    }

    function cashOut(auto = false) {
      if (isPlaying && !crashed) {
        clearInterval(interval);
        isPlaying = false;
        let winAmount = parseFloat((currentBet * multiplier).toFixed(2));
        setBalance(getBalance() + winAmount);
        updateUI();
        statusDisplay.textContent = `${auto ? '🤖 Auto' : '✅'} Cashed out at ${multiplier}x and won $${winAmount.toFixed(2)}!`;
        cashoutBtn.disabled = true;
        startBtn.disabled = false;
        betAmountInput.disabled = false;
        autoCashOutInput.disabled = false;
      }
    }

    startBtn.addEventListener('click', startGame);
    cashoutBtn.addEventListener('click', () => cashOut(false));
  </script>

</body>
</html>
