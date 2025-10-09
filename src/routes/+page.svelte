<script lang="ts">
  type Player = 'X' | 'O';
  type Cell = { player: Player; timestamp: number } | null;

  const size = 3;
  let board: Cell[][] = Array.from({ length: size }, () => Array(size).fill(null));
  let currentPlayer: Player = 'X';
  let turn = 0;
  let message = 'Vez do jogador X';

  // Para armazenar a ordem dos símbolos de cada jogador
  let moves: Record<Player, { row: number; col: number; timestamp: number }[]> = {
    X: [],
    O: []
  };

  function handleClick(row: number, col: number) {
    if (board[row][col]) return;
    if (message.includes('venceu') || message === 'Empate') return;

    const timestamp = Date.now();
    board[row][col] = { player: currentPlayer, timestamp };
    moves[currentPlayer].push({ row, col, timestamp });

    // Limitar a 3 movimentos por jogador
    if (moves[currentPlayer].length > 3) {
      const oldest = moves[currentPlayer].shift();
      if (oldest) {
        board[oldest.row][oldest.col] = null;
      }
    }

    if (checkWin(currentPlayer)) {
      message = `Jogador ${currentPlayer} venceu!`;
      return;
    }

    if (board.flat().every(cell => cell)) {
      message = 'Empate';
      return;
    }

    currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
    message = `Vez do jogador ${currentPlayer}`;
  }

  function checkWin(player: Player): boolean {
    const lines = [
      // linhas
      ...board,
      // colunas
      ...[0, 1, 2].map(i => board.map(row => row[i])),
      // diagonais
      [0, 1, 2].map(i => board[i][i]),
      [0, 1, 2].map(i => board[i][2 - i])
    ];
    return lines.some(line => line.filter(cell => cell?.player === player).length === 3);
  }

  function reset() {
    board = Array.from({ length: size }, () => Array(size).fill(null));
    moves = { X: [], O: [] };
    currentPlayer = 'X';
    message = 'Vez do jogador X';
  }
</script>

<main class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
  <h1 class="text-2xl md:text-4xl font-bold mb-4 md:mb-8">Jogo sem Velha</h1>
  <div class="grid grid-cols-3 gap-2 md:gap-4 mb-4 md:mb-8">
    {#each board as row, rowIndex}
      {#each row as cell, colIndex}
        <button
          onclick={() => handleClick(rowIndex, colIndex)}
          class="size-20 md:size-40 flex items-center justify-center text-2xl md:text-4xl font-bold rounded bg-white shadow hover:bg-gray-200 transition hover:cursor-pointer {cell?.player === 'X' ? "text-pink-400" : "text-blue-400"}"
        >
          {cell?.player || ''}
        </button>
      {/each}
    {/each}
  </div>
  <p class="mb-4 text-lg md:text-2xl">{message}</p>
  <button onclick={reset} class="px-4 py-2 md:px-6 md:py-3 md:text-lg bg-pink-200 text-gray-500 rounded hover:bg-pink-300 hover:text-white transition hover:cursor-pointer">
    Reiniciar
  </button>
</main>