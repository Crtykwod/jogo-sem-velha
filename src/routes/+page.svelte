<script lang="ts">
  const sleep = (ms: number) => new Promise(r => setTimeout(r, ms));

  type Theme = 'light' | 'dark';

  let theme: Theme = $state("light");
  let isAnimating = $state(false);

  type Player = 'X' | 'O';
  type Cell = { player: Player; timestamp: number } | null;

  const size = 3;
  let board: Cell[][] = $state(Array.from({ length: size }, () => Array(size).fill(null)));
  let currentPlayer: Player = $state('X');
  let message = $state('Vez do jogador X');
  let isGameOver = $derived(message.includes('venceu') || message === 'Empate');

  // Para armazenar a ordem dos símbolos de cada jogador
  let moves: Record<Player, { row: number; col: number; timestamp: number }[]> = $state({
    X: [],
    O: []
  });

  function handleClick(row: number, col: number) {
    if (isGameOver) return;
    if (board[row][col]) return;

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

  $effect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    
    const handleChange = (e: MediaQueryListEvent) => {
      theme = e.matches ? 'dark' : 'light';
    };
    
    mediaQuery.addEventListener('change', handleChange);
    
    theme = mediaQuery.matches ? 'dark' : 'light';
    
    return () => mediaQuery.removeEventListener('change', handleChange);
  });
</script>

<main class="flex min-h-screen flex-col items-center justify-center {theme == 'dark' ? 'bg-slate-900 text-slate-100' : 'bg-gray-100 text-gray-800'} p-4">
  <button 
    class="theme-button {theme == 'light' ? "bg-[url(/icons/sun.svg)]" : "bg-[url(/icons/moon.svg)]"} {isAnimating ? 'theme-transition' : ''}" 
    aria-label="Change Theme" 
    onclick={() => {
      theme = theme === 'dark' ? 'light' : 'dark';
      isAnimating = true;
      setTimeout(() => {
        isAnimating = false;
      }, 1100);
    }}
  >
  </button>
	<h1 class="text-2xl font-bold md:text-4xl mb-4 md:mb-8">
		Jogo sem Velha
	</h1>
	<div class="mb-4 grid grid-cols-3 gap-2 md:mb-8 md:gap-4">
		{#each board as row, rowIndex}
			{#each row as cell, colIndex}
				<button
					onclick={() => handleClick(rowIndex, colIndex)}
					class="flex size-20 items-center justify-center rounded {theme == 'dark' ? 'bg-slate-300' : 'bg-white'} text-2xl font-bold shadow transition hover:cursor-pointer hover:bg-slate-200 md:size-40 md:text-4xl"
          class:text-pink-400={cell?.player == 'X'}
          class:text-blue-400={cell?.player == 'O'}
          class:text-transparent={!cell?.player}>
					{cell?.player || ''}
				</button>
			{/each}
		{/each}
	</div>
	<p class="mb-4 text-lg md:text-2xl">{message}</p>
	<button
		onclick={reset}
		class="rounded {theme == 'dark' ? 'bg-pink-300 hover:bg-pink-400 text-gray-700': 'bg-pink-200 hover:bg-pink-300'} px-4 py-2 transition hover:cursor-pointer hover:text-white md:px-6 md:py-3 md:text-lg">
		Reiniciar
	</button>
</main>
