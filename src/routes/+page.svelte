<script lang="ts">
  type Player = "X" | "O" | null;
  type Board = Player[];

  let board = $state<Board>(Array(9).fill(null));
  let isPlayerTurn = $state(true);
  let gameOver = $state(false);

  const winner = $derived(calculateWinner(board));
  const isDraw = $derived(!winner && board.every((cell) => cell !== null));
  const statusMessage = $derived(getStatusMessage());

  function calculateWinner(board: Board): Player {
    const lines = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6],
    ];

    for (const [a, b, c] of lines) {
      if (board[a] && board[a] === board[b] && board[a] === board[c]) {
        return board[a];
      }
    }
    return null;
  }

  function getStatusMessage(): string {
    if (winner === "X") return "🎉 You Win!";
    if (winner === "O") return "🤖 AI Wins!";
    if (isDraw) return "🤝 Draw!";
    return isPlayerTurn ? "👤 Your Turn" : "🤖 AI Thinking...";
  }

  function handleCellClick(index: number) {
    if (board[index] || !isPlayerTurn || winner || isDraw) return;

    board[index] = "X";
    isPlayerTurn = false;

    if (!calculateWinner(board) && !board.every((cell) => cell !== null)) {
      setTimeout(makeAIMove, 500);
    }
  }

  function makeAIMove() {
    const bestMove = findBestMove(board);
    if (bestMove !== -1) {
      board[bestMove] = "O";
      isPlayerTurn = true;
    }
  }

  function findBestMove(board: Board): number {
    let bestScore = -Infinity;
    let bestMove = -1;

    for (let i = 0; i < 9; i++) {
      if (board[i] === null) {
        board[i] = "O";
        const score = minimax(board, 0, false);
        board[i] = null;

        if (score > bestScore) {
          bestScore = score;
          bestMove = i;
        }
      }
    }

    return bestMove;
  }

  function minimax(board: Board, depth: number, isMaximizing: boolean): number {
    const winner = calculateWinner(board);

    if (winner === "O") return 10 - depth;
    if (winner === "X") return depth - 10;
    if (board.every((cell) => cell !== null)) return 0;

    if (isMaximizing) {
      let bestScore = -Infinity;
      for (let i = 0; i < 9; i++) {
        if (board[i] === null) {
          board[i] = "O";
          const score = minimax(board, depth + 1, false);
          board[i] = null;
          bestScore = Math.max(score, bestScore);
        }
      }
      return bestScore;
    } else {
      let bestScore = Infinity;
      for (let i = 0; i < 9; i++) {
        if (board[i] === null) {
          board[i] = "X";
          const score = minimax(board, depth + 1, true);
          board[i] = null;
          bestScore = Math.min(score, bestScore);
        }
      }
      return bestScore;
    }
  }

  function resetGame() {
    board = Array(9).fill(null);
    isPlayerTurn = true;
    gameOver = false;
  }
</script>

<div
  class="flex min-h-screen items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 p-4"
>
  <div class="w-full max-w-md rounded-2xl bg-white p-8 shadow-2xl">
    <h1 class="mb-2 text-center text-4xl font-bold text-gray-800">
      Tic-Tac-Toe
    </h1>
    <p class="mb-6 text-center text-gray-600">Challenge the AI</p>

    <div class="mb-6 text-center">
      <p class="h-8 text-2xl font-semibold text-gray-700">{statusMessage}</p>
    </div>

    <div class="mb-6 grid grid-cols-3 gap-3">
      {#each board as cell, i}
        <button
          onclick={() => handleCellClick(i)}
          disabled={!isPlayerTurn || cell !== null || winner !== null || isDraw}
          class="flex aspect-square items-center justify-center rounded-xl
                 border-2 border-gray-200 bg-gray-50 text-5xl
                 font-bold transition-all duration-200
                 hover:border-indigo-300 hover:bg-gray-100 disabled:cursor-not-allowed disabled:hover:border-gray-200 disabled:hover:bg-gray-50
                 {cell === 'X' ? 'text-blue-600' : 'text-red-600'}
                 {!cell && isPlayerTurn && !winner && !isDraw
            ? 'hover:shadow-md'
            : ''}"
        >
          {cell || ""}
        </button>
      {/each}
    </div>

    {#if winner || isDraw}
      <button
        onclick={resetGame}
        class="w-full rounded-xl bg-indigo-600 px-6 py-3 font-semibold text-white
               shadow-lg transition-colors duration-200 hover:bg-indigo-700 hover:shadow-xl"
      >
        Play Again
      </button>
    {/if}

    <div class="mt-6 border-t border-gray-200 pt-6">
      <div class="flex justify-around text-center">
        <div>
          <p class="text-3xl font-bold text-blue-600">X</p>
          <p class="text-sm text-gray-600">You</p>
        </div>
        <div>
          <p class="text-3xl font-bold text-red-600">O</p>
          <p class="text-sm text-gray-600">AI</p>
        </div>
      </div>
    </div>
  </div>
</div>
