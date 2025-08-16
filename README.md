<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tic Tac Toe</title>
<style>
    body {
        font-family: Arial, sans-serif;
        text-align: center;
        background-color: #f4f4f4;
    }
    h1 {
        color: #333;
    }
    .board {
        display: grid;
        grid-template-columns: repeat(3, 100px);
        grid-template-rows: repeat(3, 100px);
        gap: 5px;
        margin: 20px auto;
        width: max-content;
    }
    .cell {
        background-color: white;
        border: 2px solid #333;
        font-size: 2em;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
    }
    .cell:hover {
        background-color: #e1e1e1;
    }
    .winner {
        color: green;
        font-size: 1.5em;
        margin-top: 15px;
    }
    .reset-btn, .difficulty-select {
        margin-top: 10px;
        padding: 10px 20px;
        background: #333;
        color: white;
        border: none;
        cursor: pointer;
    }
    .difficulty-select {
        background: #555;
        color: white;
    }
    .reset-btn:hover {
        background: #555;
    }
</style>
</head>
<body>

<h1>Tic Tac Toe</h1>

<select id="difficulty" class="difficulty-select">
    <option value="easy">Easy</option>
    <option value="medium">Medium</option>
    <option value="impossible" selected>Impossible</option>
</select>

<div class="board" id="board"></div>
<div class="winner" id="winner"></div>
<button class="reset-btn" onclick="resetGame()">Reset Game</button>

<script>
    const board = document.getElementById("board");
    const winnerText = document.getElementById("winner");
    const difficultySelect = document.getElementById("difficulty");

    let currentPlayer = "X"; // Player always starts
    let gameActive = true;
    let gameState = ["", "", "", "", "", "", "", "", ""];

    const winPatterns = [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8],
        [0, 3, 6],
        [1, 4, 7],
        [2, 5, 8],
        [0, 4, 8],
        [2, 4, 6],
    ];

    // Create cells
    for (let i = 0; i < 9; i++) {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        cell.dataset.index = i;
        cell.addEventListener("click", playerMove);
        board.appendChild(cell);
    }

    function playerMove(e) {
        const index = e.target.dataset.index;
        if (gameState[index] !== "" || !gameActive || currentPlayer !== "X") return;

        gameState[index] = "X";
        e.target.textContent = "X";

        if (checkWinner("X")) {
            winnerText.textContent = "You Win! 🎉";
            gameActive = false;
            return;
        }
        if (!gameState.includes("")) {
            winnerText.textContent = "It's a Draw! 😐";
            gameActive = false;
            return;
        }

        currentPlayer = "O";
        setTimeout(aiMove, 300);
    }

    function aiMove() {
        if (!gameActive) return;

        let difficulty = difficultySelect.value;
        let move;

        if (difficulty === "easy") {
            move = getRandomMove();
        } else if (difficulty === "medium") {
            move = findBestMove("O") || findBestMove("X") || getRandomMove();
        } else if (difficulty === "impossible") {
            move = getMinimaxMove();
        }

        if (move === null) return;

        gameState[move] = "O";
        document.querySelectorAll(".cell")[move].textContent = "O";

        if (checkWinner("O")) {
            winnerText.textContent = "AI Wins! 🤖";
            gameActive = false;
            return;
        }
        if (!gameState.includes("")) {
            winnerText.textContent = "It's a Draw! 😐";
            gameActive = false;
            return;
        }

        currentPlayer = "X";
    }

    // EASY - Random Move
    function getRandomMove() {
        let emptyCells = gameState
            .map((val, idx) => (val === "" ? idx : null))
            .filter(v => v !== null);
        return emptyCells.length ? emptyCells[Math.floor(Math.random() * emptyCells.length)] : null;
    }

    // MEDIUM - Win or Block
    function findBestMove(player) {
        for (let pattern of winPatterns) {
            const [a, b, c] = pattern;
            let values = [gameState[a], gameState[b], gameState[c]];
            if (values.filter(v => v === player).length === 2 && values.includes("")) {
                return pattern[values.indexOf("")];
            }
        }
        return null;
    }

    // IMPOSSIBLE - Minimax Algorithm
    function getMinimaxMove() {
        let bestScore = -Infinity;
        let move;
        for (let i = 0; i < 9; i++) {
            if (gameState[i] === "") {
                gameState[i] = "O";
                let score = minimax(gameState, 0, false);
                gameState[i] = "";
                if (score > bestScore) {
                    bestScore = score;
                    move = i;
                }
            }
        }
        return move;
    }

    function minimax(state, depth, isMaximizing) {
        if (checkWinner("O")) return 10 - depth;
        if (checkWinner("X")) return depth - 10;
        if (!state.includes("")) return 0;

        if (isMaximizing) {
            let bestScore = -Infinity;
            for (let i = 0; i < 9; i++) {
                if (state[i] === "") {
                    state[i] = "O";
                    let score = minimax(state, depth + 1, false);
                    state[i] = "";
                    bestScore = Math.max(score, bestScore);
                }
            }
            return bestScore;
        } else {
            let bestScore = Infinity;
            for (let i = 0; i < 9; i++) {
                if (state[i] === "") {
                    state[i] = "X";
                    let score = minimax(state, depth + 1, true);
                    state[i] = "";
                    bestScore = Math.min(score, bestScore);
                }
            }
            return bestScore;
        }
    }

    function checkWinner(player) {
        return winPatterns.some(pattern => pattern.every(index => gameState[index] === player));
    }

    function resetGame() {
        gameState = ["", "", "", "", "", "", "", "", ""];
        currentPlayer = "X";
        gameActive = true;
        winnerText.textContent = "";
        document.querySelectorAll(".cell").forEach(cell => cell.textContent = "");
    }
</script>

</body>
</html>
