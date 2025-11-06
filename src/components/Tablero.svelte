<script>
    let winner = true;

    const ROWS = 6;
    const COLS = 7;
    const OBJETIVO = 4;
    let board = Array.from(
            { length: ROWS }, 
            () => Array(COLS).fill(null));
    let currentPlayer = 'H'; // Jugador humano
    
    let isHumanTurn = true;
    let animatingCells = [];

    function dropDisc(col) {
      if (winner || !isHumanTurn) return;
        console.log(board)
      for (let row = ROWS - 1; row >= 0; row--) {
        if (!board[row][col]) {
          board[row][col] = currentPlayer;
          animatingCells[row][col] = true;
          setTimeout(() => animatingCells[row][col] = false, 500);
  
       /*    if (checkWin(row, col)) {
            winner = currentPlayer;
          } else {
            switchTurns();
          } */
          return;
        }
      }
    }


    function resetGame(){
        winner = false;
        //emptyBoard();
        //game = true
    }
</script>

<p>Conecta 4</p>

<div class="board">


</div>

<button on:click={resetGame}>Reiniciar juego</button>

<div class="board">
    {#each board as row, rowIndex}
        {#each row as cell, colIndex}
            {console.log(cell, colIndex)}
        {/each}
    {/each}
  </div>

{#if winner}
    <p>mensaje has ganado</p>
{/if}


<style>
    p{
        font-size: 2rem;
        font-weight: bold;
    }

    .board{
        width: 50vh;
        height: 50vh;
        background-color: blue;
    }

</style>