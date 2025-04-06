<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Goal Game</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #121212;
      color: white;
      text-align: center;
      padding: 20px;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 10px;
      max-width: 320px;
      margin: 20px auto;
    }

    .tile {
      width: 100%;
      padding-top: 100%; /* Square */
      position: relative;
      background: #4caf50;
      border: none;
      border-radius: 5px;
      transition: background 0.3s, transform 0.2s;
    }

    .tile.revealed.safe {
      background: #81c784;
      transform: scale(1.05);
    }

    .tile.revealed.bomb {
      background: #e53935;
      transform: scale(1.05);
    }

    .tile span {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 20px;
      user-select: none;
    }

    input, button {
      padding: 10px;
      font-size: 16px;
      margin: 5px;
      border: none;
      border-radius: 5px;
    }

    #cashout {
      background-color: gold;
    }

    .info {
      margin-top: 10px;
    }
  </style>
</head>
<body>

  <h1>💣 Goal Game</h1>

  <input type="number" id="bet" placeholder="Enter bet amount" min="1" />
  <button onclick="startGame()">Start Game</button>
  <button id="cashout" onclick="cashOut()" disabled>💰 Cash Out</button>

  <div class="info">
    <p>Multiplier: <span id="multiplier">1.00x</span></p>
    <p>Winnings: <span id="winnings">0</span></p>
    <p>High Score: <span id="highscore">0.00x</span></p>
  </div>

  <div class="grid" id="grid"></div>

  <script>
    const rows = 5, cols = 5, totalBombs = 5;
    let bombs = [], multiplier = 1.0, revealed = new Set(), gameActive = false;
    let betAmount = 0;

    const grid = document.getElementById("grid");
    const multiplierDisplay = document.getElementById("multiplier");
    const winningsDisplay = document.getElementById("winnings");
    const highscoreDisplay = document.getElementById("highscore");
    const cashoutButton = document.getElementById("cashout");

    function generateBombs() {
      bombs = [];
      while (bombs.length < totalBombs) {
        const index = Math.floor(Math.random() * (rows * cols));
        if (!bombs.includes(index)) bombs.push(index);
      }
    }

    function startGame() {
      const betInput = document.getElementById("bet");
      betAmount = parseFloat(betInput.value);
      if (isNaN(betAmount) || betAmount <= 0) {
        alert("Please enter a valid bet amount.");
        return;
      }

      gameActive = true;
      revealed.clear();
      multiplier = 1.0;
      updateUI();

      generateBombs();
      buildGrid();
      cashoutButton.disabled = false;
    }

    function buildGrid() {
      grid.innerHTML = "";
      for (let i = 0; i < rows * cols; i++) {
        const tile = document.createElement("button");
        tile.classList.add("tile");
        tile.dataset.index = i;
        tile.onclick = () => handleClick(i, tile);
        const span = document.createElement("span");
        tile.appendChild(span);
        grid.appendChild(tile);
      }
    }

    function handleClick(index, tile) {
      if (!gameActive || revealed.has(index)) return;

      revealed.add(index);
      tile.classList.add("revealed");

      const span = tile.querySelector("span");

      if (bombs.includes(index)) {
        tile.classList.add("bomb");
        span.textContent = "💣";
        gameActive = false;
        cashoutButton.disabled = true;
        revealAllBombs();
        winningsDisplay.textContent = "0";
        alert("Boom! You hit a bomb!");
      } else {
        tile.classList.add("safe");
        span.textContent = "✅";
        multiplier += 0.29;
        updateUI();
      }
    }

    function updateUI() {
      multiplierDisplay.textContent = multiplier.toFixed(2) + "x";
      winningsDisplay.textContent = (betAmount * multiplier).toFixed(2);

      const currentHigh = parseFloat(localStorage.getItem("highscore")) || 0;
      if (multiplier > currentHigh) {
        localStorage.setItem("highscore", multiplier.toFixed(2));
        highscoreDisplay.textContent = multiplier.toFixed(2) + "x";
      } else {
        highscoreDisplay.textContent = currentHigh.toFixed(2) + "x";
      }
    }

    function revealAllBombs() {
      document.querySelectorAll(".tile").forEach((tile) => {
        const idx = parseInt(tile.dataset.index);
        if (bombs.includes(idx)) {
          tile.classList.add("revealed", "bomb");
          tile.querySelector("span").textContent = "💣";
        }
        tile.disabled = true;
      });
    }

    function cashOut() {
      if (!gameActive) return;

      const winnings = (betAmount * multiplier).toFixed(2);
      alert("You cashed out: $" + winnings + "!");
      gameActive = false;
      cashoutButton.disabled = true;
      revealAllBombs();
    }

    document.addEventListener("DOMContentLoaded", () => {
      const storedHigh = localStorage.getItem("highscore");
      if (storedHigh) {
        highscoreDisplay.textContent = storedHigh + "x";
      }
    });
  </script>
</body><!-- 🔊 Sound Effects -->
<audio id="click-sound" src="sounds/click.mp3"></audio>
<audio id="boom-sound" src="sounds/boom.mp3"></audio>
<audio id="win-sound" src="sounds/win.mp3"></audio>
<audio id="success-sound" src="sounds/success.mp3"></audio>

</html>
