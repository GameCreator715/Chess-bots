# Chess-bots
Play a bot in chess
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Chess vs AI</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css" />
  <style>
    body {
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #f5f5f5;
      margin-top: 30px;
    }
    #board {
      width: 400px;
    }
    #status {
      margin-top: 10px;
    }
    #controls {
      margin: 10px 0;
    }
  </style>
</head>
<body>
  <h2>Chess vs AI</h2>
  <div id="controls">
    <label for="difficulty">AI Difficulty:</label>
    <select id="difficulty">
      <option value="1">Easy</option>
      <option value="5" selected>Medium</option>
      <option value="10">Hard</option>
    </select>
  </div>
  <div id="board"></div>
  <div id="status">Status: Ready</div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>
  <script src="https://unpkg.com/stockfish/stockfish.js"></script>
  <script src="script.js"></script>
</body>
</html>
const game = new Chess();
const statusElement = document.getElementById("status");
const difficultySelect = document.getElementById("difficulty");

const stockfish = new Worker("https://unpkg.com/stockfish/stockfish.js");

stockfish.onmessage = function (event) {
  if (event.data.startsWith("bestmove")) {
    const move = event.data.split(" ")[1];
    game.move({ from: move.slice(0, 2), to: move.slice(2, 4), promotion: 'q' });
    board.position(game.fen());
    updateStatus();
  }
};

function makeAIMove() {
  stockfish.postMessage("uci");
  stockfish.postMessage("ucinewgame");
  stockfish.postMessage(`setoption name Skill Level value ${difficultySelect.value}`);
  stockfish.postMessage(`position fen ${game.fen()}`);
  stockfish.postMessage("go depth 12");
}

function onDragStart(source, piece) {
  if (game.game_over() ||
      (game.turn() === 'w' && piece.search(/^b/) !== -1) ||
      (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
    return false;
  }
}

function onDrop(source, target) {
  const move = game.move({ from: source, to: target, promotion: 'q' });
  if (move === null) return "snapback";
  updateStatus();
  setTimeout(makeAIMove, 250);
}

function updateStatus() {
  let status = '';
  if (game.in_checkmate()) {
    status = 'Game over, ' + (game.turn() === 'w' ? 'Black' : 'White') + ' wins by checkmate.';
  } else if (game.in_draw()) {
    status = 'Game over, drawn position.';
  } else {
    status = (game.turn() === 'w' ? 'White' : 'Black') + ' to move.';
    if (game.in_check()) {
      status += ' Check!';
    }
  }
  statusElement.textContent = 'Status: ' + status;
}

const board = Chessboard('board', {
  draggable: true,
  position: 'start',
  onDragStart,
  onDrop,
  onSnapEnd: () => board.position(game.fen())
});

updateStatus();
