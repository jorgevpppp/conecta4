<script>
  // Constantes para definir el tamaño del tablero y condición de victoria
  const ROWS = 7;
  const COLS = 8;
  const OBJETIVO = 4; // Número de fichas necesarias para ganar

  // Inicialización del estado del juego
  let board = Array.from({ length: ROWS }, () => Array(COLS).fill(null)); // Tablero vacío
  let currentPlayer = 'H'; // H para humano, M para máquina
  let winner = null; // Almacena el ganador cuando hay uno
  let isHumanTurn = true; // Controla de quién es el turno
  let animatingCells = []; // Matriz para controlar las animaciones de las fichas

  function initializeAnimation() {
    animatingCells = Array.from({ length: ROWS }, () => Array(COLS).fill(false));
  }

  initializeAnimation();

  // --- Función para soltar ficha del jugador humano ---
  function dropDisc(col) {
    if (winner || !isHumanTurn) return;

    for (let row = ROWS - 1; row >= 0; row--) {
      if (!board[row][col]) {
        board[row][col] = currentPlayer;
        animatingCells[row][col] = true;
        setTimeout(() => (animatingCells[row][col] = false), 500);

        if (checkWin(row, col, currentPlayer)) {
          winner = currentPlayer;
        } else {
          switchTurns();
        }
        return;
      }
    }
  }

  // --- Cambiar turno ---
  function switchTurns() {
    currentPlayer = currentPlayer === 'H' ? 'M' : 'H';
    isHumanTurn = !isHumanTurn;

    if (!isHumanTurn && !winner) {
      setTimeout(aiMove, 700);
    }
  }

  // Sistema de IA avanzado que analiza el tablero completo
  // Implementa una estrategia en niveles de prioridad:
  // 1. Buscar victoria inmediata
  // 2. Bloquear victoria del oponente
  // 3. Formar líneas de 3
  // 4. Movimiento estratégico basado en puntuación
  // 5. Movimiento aleatorio como último recurso
  function aiMove() {
    const ai = 'M'; // Identificador de la máquina
    const human = 'H'; // Identificador del humano

    // --- Nivel 1: Buscar victoria inmediata ---
    // La IA primero verifica si puede ganar en el siguiente movimiento
    for (let col = 0; col < COLS; col++) {
      let row = getAvailableRow(col);
      if (row === -1) continue; // Columna llena
      board[row][col] = ai;
      if (checkWin(row, col, ai)) {
        makeAIMove(row, col);
        return;
      }
      board[row][col] = null;
    }

    // --- Nivel 2: Bloquear victoria del oponente ---
    // Si la IA no puede ganar, verifica si necesita bloquear al jugador
    // Simula el movimiento del jugador para ver si ganaría en su próximo turno
    for (let col = 0; col < COLS; col++) {
      let row = getAvailableRow(col);
      if (row === -1) continue;
      board[row][col] = human;
      if (checkWin(row, col, human)) {
        board[row][col] = ai; // Bloquea la victoria del jugador
        makeAIMove(row, col);
        return;
      }
      board[row][col] = null;
    }

    // --- Nivel 3: Crear oportunidades ofensivas ---
    // Si no hay amenazas inmediatas, busca crear una línea de 3 fichas
    // Esto puede crear oportunidades de victoria para el siguiente turno
    for (let col = 0; col < COLS; col++) {
      let row = getAvailableRow(col);
      if (row === -1) continue;
      board[row][col] = ai;
      if (hasThreeInARow(ai)) {
        makeAIMove(row, col);
        return;
      }
      board[row][col] = null;
    }

    // --- 4️ Si no hay jugadas peligrosas o ganadoras, moverse estratégicamente ---
    let bestMove = findBestStrategicMove(ai, human);
    if (bestMove !== null) {
      let row = getAvailableRow(bestMove);
      makeAIMove(row, bestMove);
      return;
    }

    // --- 5️ Movimiento aleatorio ---
    let availableCols = [];
    for (let col = 0; col < COLS; col++) {
      if (!board[0][col]) availableCols.push(col);
    }

    if (availableCols.length > 0) {
      let randomCol = availableCols[Math.floor(Math.random() * availableCols.length)];
      let row = getAvailableRow(randomCol);
      makeAIMove(row, randomCol);
    }
  }

  // --- Buscar primera fila libre en una columna ---
  function getAvailableRow(col) {
    for (let row = ROWS - 1; row >= 0; row--) {
      if (!board[row][col]) return row;
    }
    return -1;
  }

  // --- Colocar ficha de IA con animación ---
  function makeAIMove(row, col) {
    board[row][col] = currentPlayer;
    animatingCells[row][col] = true;
    setTimeout(() => (animatingCells[row][col] = false), 500);

    if (checkWin(row, col, currentPlayer)) {
      winner = currentPlayer;
    } else {
      switchTurns();
    }
  }

  // --- Verificar si hay 4 seguidas ---
  function checkWin(row, col, player) {
    return (
      checkDirection(row, col, 1, 0, player) ||
      checkDirection(row, col, 0, 1, player) ||
      checkDirection(row, col, 1, 1, player) ||
      checkDirection(row, col, 1, -1, player)
    );
  }

  function checkDirection(row, col, rowDir, colDir, player) {
    let total = 1;
    total += countDiscs(row, col, rowDir, colDir, player);
    total += countDiscs(row, col, -rowDir, -colDir, player);
    return total >= OBJETIVO;
  }

  function countDiscs(row, col, rowDir, colDir, player) {
    let r = row + rowDir;
    let c = col + colDir;
    let count = 0;
    while (
      r >= 0 &&
      r < ROWS &&
      c >= 0 &&
      c < COLS &&
      board[r][c] === player
    ) {
      count++;
      r += rowDir;
      c += colDir;
    }
    return count;
  }

  // --- Sistema de detección de amenazas (tres en línea) ---
  // Esta función analiza el tablero buscando secuencias de 3 fichas del mismo jugador
  // Verifica en todas las direcciones posibles:
  // - Vertical (1,0)
  // - Horizontal (0,1)
  // - Diagonal ascendente (1,1)
  // - Diagonal descendente (1,-1)
  function hasThreeInARow(player) {
    for (let r = 0; r < ROWS; r++) {
      for (let c = 0; c < COLS; c++) {
        if (board[r][c] === player) {
          // Comprueba si hay tres en línea en cualquier dirección
          if (
            // Vertical: arriba + abajo
            countDiscs(r, c, 1, 0, player) +
              countDiscs(r, c, -1, 0, player) >= 2 ||
            // Horizontal: izquierda + derecha
            countDiscs(r, c, 0, 1, player) +
              countDiscs(r, c, 0, -1, player) >= 2 ||
            // Diagonal \: arriba-derecha + abajo-izquierda
            countDiscs(r, c, 1, 1, player) +
              countDiscs(r, c, -1, -1, player) >= 2 ||
            // Diagonal /: arriba-izquierda + abajo-derecha
            countDiscs(r, c, 1, -1, player) +
              countDiscs(r, c, -1, 1, player) >= 2
          ) {
            return true;
          }
        }
      }
    }
    return false;
  }

  // --- Sistema de evaluación de posiciones estratégicas ---
  // Esta función asigna puntuaciones a cada columna basándose en varios factores:
  // 1. Proximidad al centro (las posiciones centrales son más valiosas)
  // 2. Potencial para formar líneas de 3
  // 3. Bloqueo de jugadas del oponente
  // 4. Creación de amenazas múltiples
  function findBestStrategicMove(ai, human) {
    let scores = Array(COLS).fill(0);

    for (let col = 0; col < COLS; col++) {
      let row = getAvailableRow(col);
      if (row === -1) continue;

      // Bonus por proximidad al centro - las posiciones centrales ofrecen más oportunidades
      let centerBonus = COLS / 2 - Math.abs(col - COLS / 2);
      scores[col] += centerBonus;

      // Sistema de puntuación basado en simulación de jugadas
      board[row][col] = ai;
      if (hasThreeInARow(ai)) scores[col] += 4;      // Bonus por formar tres en línea
      if (checkWin(row, col, ai)) scores[col] += 10; // Bonus mayor por victoria posible
      board[row][col] = human;
      if (hasThreeInARow(human)) scores[col] -= 4;   // Penalización por permitir tres en línea del oponente
      board[row][col] = null;
    }

    let maxScore = Math.max(...scores);
    let bestCols = scores
      .map((score, idx) => (score === maxScore ? idx : null))
      .filter((v) => v !== null);

    if (bestCols.length > 0)
      return bestCols[Math.floor(Math.random() * bestCols.length)];
    return null;
  }

  // --- Reiniciar juego ---
  function resetGame() {
    board = Array.from({ length: ROWS }, () => Array(COLS).fill(null));
    currentPlayer = 'H';
    winner = null;
    isHumanTurn = true;
    initializeAnimation();
  }
</script>

{#if winner}
  <p class="winner">{winner} ha ganado!</p>
{/if}

<div class="board">
  {#each board as row, rowIndex}
    {#each row as cell, colIndex}
      <div
        role="button"
        tabindex="0"
        class="cell"
        on:click={() => isHumanTurn && dropDisc(colIndex)}
        on:keydown={() => isHumanTurn && dropDisc(colIndex)}
      >
        {#if cell}
          <div class="disc" class:animating={animatingCells[rowIndex][colIndex]}>
            <div
              class="ficha"
              class:humano={cell === "H"}
              class:maquina={cell === "M"}
            ></div>
          </div>
        {/if}
      </div>
    {/each}
  {/each}
</div>

<button on:click={resetGame}>Reiniciar juego</button>

<style>
  .board {
    display: grid;
    grid-template-columns: repeat(8, 60px);
    gap: 5px;
    justify-content: center;
    margin: 20px auto;
    background-image: url("conecta4.jpg");
    background-repeat: no-repeat;
    padding: 15px;
  }

  .cell {
    width: 60px;
    height: 60px;
    background-color: #00bfff;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 40px;
    border-radius: 50%;
    position: relative;
  }

  .ficha {
    width: 60px;
    height: 60px;
    border-radius: 50%;
  }

  .ficha::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background: repeating-conic-gradient(
      rgb(184, 183, 183) 0deg 15deg,
      transparent 15deg 30deg
    );
    mask-image: radial-gradient(circle, #e33a72 60%, transparent 50%);
  }

  .humano {
    background-color: #e33a72;
  }

  .maquina {
    background-color: #f5fc94;
  }

  .disc {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    transition: transform 0.6s cubic-bezier(0.68, -0.55, 0.27, 1.55);
  }

  .animating {
    transform: translateY(-600%) rotate(-760deg);
  }

  .winner {
    font-size: 30px;
    margin-top: 20px;
    color: green;
  }

  button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    background-color: #ff6f61;
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 5px;
  }

  button:hover {
    background-color: #ff5a4a;
  }
</style>
