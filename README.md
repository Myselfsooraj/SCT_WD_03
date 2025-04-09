<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Tic Tac Toe</title>
  <style>
    :root {
      --primary: #4a6fa5;
      --secondary: #ff6b6b;
      --dark: #333;
      --light: #f8f9fa;
      --success: #51cf66;
      --draw: #fcc419;
    }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      margin: 0;
      padding: 20px;
      color: var(--dark);
    }

    h1 {
      margin-bottom: 20px;
      color: var(--primary);
      text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
      font-size: 2.5rem;
    }

    .scoreboard {
      display: flex;
      justify-content: space-around;
      width: 300px;
      margin-bottom: 20px;
      background: white;
      padding: 10px;
      border-radius: 10px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    .score {
      text-align: center;
      padding: 5px 15px;
      border-radius: 5px;
      font-weight: bold;
    }

    .score-x {
      color: var(--primary);
    }

    .score-o {
      color: var(--secondary);
    }

    .board {
      display: grid;
      grid-template-columns: repeat(3, 100px);
      gap: 10px;
      background: var(--dark);
      padding: 10px;
      border-radius: 10px;
      box-shadow: 0 10px 20px rgba(0,0,0,0.2);
    }

    .cell {
      width: 100px;
      height: 100px;
      background-color: white;
      border-radius: 5px;
      font-size: 3rem;
      text-align: center;
      line-height: 100px;
      cursor: pointer;
      transition: all 0.3s ease;
      user-select: none;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }

    .cell:hover {
      transform: translateY(-3px);
      box-shadow: 0 6px 12px rgba(0,0,0,0.15);
    }

    .cell.x {
      color: var(--primary);
    }

    .cell.o {
      color: var(--secondary);
    }

    .cell.winner {
      background-color: var(--success);
      color: white;
      animation: pulse 1s infinite;
    }

    .cell.draw {
      background-color: var(--draw);
    }

    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.05); }
      100% { transform: scale(1); }
    }

    #status {
      margin: 20px 0;
      font-size: 1.3rem;
      font-weight: bold;
      min-height: 30px;
      padding: 10px 20px;
      background: white;
      border-radius: 50px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    .buttons {
      display: flex;
      gap: 10px;
    }

    button {
      margin-top: 10px;
      padding: 12px 25px;
      font-size: 1rem;
      font-weight: bold;
      background-color: var(--primary);
      color: white;
      border: none;
      cursor: pointer;
      border-radius: 50px;
      transition: all 0.3s;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    }

    button:hover {
      background-color: #3a5a8f;
      transform: translateY(-2px);
      box-shadow: 0 6px 10px rgba(0,0,0,0.15);
    }

    button:active {
      transform: translateY(0);
    }

    .status-x {
      color: var(--primary);
    }

    .status-o {
      color: var(--secondary);
    }

    .status-draw {
      color: var(--draw);
    }

    .status-win {
      animation: pulse 1s infinite;
    }

    @media (max-width: 400px) {
      .board {
        grid-template-columns: repeat(3, 80px);
      }
      
      .cell {
        width: 80px;
        height: 80px;
        font-size: 2.5rem;
        line-height: 80px;
      }
      
      h1 {
        font-size: 2rem;
      }
    }
  </style>
</head>
<body>

  <h1>Tic Tac Toe</h1>
  
  <div class="scoreboard">
    <div class="score score-x">Player X: <span id="scoreX">0</span></div>
    <div class="score score-o">Player O: <span id="scoreO">0</span></div>
  </div>
  
  <div class="board" id="board"></div>
  
  <div id="status">Player X's turn</div>
  
  <div class="buttons">
    <button onclick="resetGame()">Reset Game</button>
    <button onclick="resetScores()">Reset Scores</button>
  </div>

  <script>
    const board = document.getElementById("board");
    const statusText = document.getElementById("status");
    const scoreXElement = document.getElementById("scoreX");
    const scoreOElement = document.getElementById("scoreO");
    
    let currentPlayer = "X";
    let gameActive = true;
    let cells = Array(9).fill("");
    let scoreX = 0;
    let scoreO = 0;
    let winningCells = [];

    const winPatterns = [
      [0, 1, 2], [3, 4, 5], [6, 7, 8], // rows
      [0, 3, 6], [1, 4, 7], [2, 5, 8], // columns
      [0, 4, 8], [2, 4, 6]             // diagonals
    ];

    // Uncomment to add sound effects (requires sound files)
    /*
    const sounds = {
      x: new Audio('x-sound.mp3'),
      o: new Audio('o-sound.mp3'),
      win: new Audio('win-sound.mp3'),
      draw: new Audio('draw-sound.mp3')
    };
    */

    function drawBoard() {
      board.innerHTML = "";
      cells.forEach((val, idx) => {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        if (val) cell.classList.add(val.toLowerCase());
        if (winningCells.includes(idx)) cell.classList.add("winner");
        cell.textContent = val;
        cell.addEventListener("click", () => handleCellClick(idx));
        board.appendChild(cell);
      });
    }

    function handleCellClick(index) {
      if (cells[index] || !gameActive) return;

      cells[index] = currentPlayer;
      // if (sounds.x && sounds.o) sounds[currentPlayer.toLowerCase()].play();
      
      drawBoard();
      checkResult();
      
      if (gameActive) {
        currentPlayer = currentPlayer === "X" ? "O" : "X";
        updateStatus(`Player ${currentPlayer}'s turn`, currentPlayer === "X" ? "status-x" : "status-o");
      }
    }

    function checkResult() {
      let won = false;

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (cells[a] && cells[a] === cells[b] && cells[a] === cells[c]) {
          won = true;
          winningCells = pattern;
          break;
        }
      }

      if (won) {
        updateStatus(`Player ${currentPlayer} wins!`, "status-win");
        // if (sounds.win) sounds.win.play();
        
        if (currentPlayer === "X") {
          scoreX++;
          scoreXElement.textContent = scoreX;
        } else {
          scoreO++;
          scoreOElement.textContent = scoreO;
        }
        
        gameActive = false;
        highlightWinningCells();
        return;
      }

      if (!cells.includes("")) {
        updateStatus("It's a draw!", "status-draw");
        // if (sounds.draw) sounds.draw.play();
        
        // Mark all cells as draw
        document.querySelectorAll('.cell').forEach(cell => {
          cell.classList.add("draw");
        });
        
        gameActive = false;
      }
    }

    function highlightWinningCells() {
      winningCells.forEach(index => {
        const cells = document.querySelectorAll('.cell');
        cells[index].classList.add("winner");
      });
    }

    function updateStatus(message, className = "") {
      statusText.textContent = message;
      statusText.className = "";
      if (className) statusText.classList.add(className);
    }

    function resetGame() {
      cells = Array(9).fill("");
      currentPlayer = "X";
      gameActive = true;
      winningCells = [];
      drawBoard();
      updateStatus("Player X's turn", "status-x");
    }

    function resetScores() {
      scoreX = 0;
      scoreO = 0;
      scoreXElement.textContent = "0";
      scoreOElement.textContent = "0";
      resetGame();
    }

    // Start the game
    resetGame();
  </script>
</body>
</html>
