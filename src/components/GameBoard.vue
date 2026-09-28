<template>
  <div id="gameboard">
    <ConfettiExplosion
      v-if="won"
      :particleCount="300"
    />
    <table>
      <tbody v-if="gameboardReady">
        <tr class="dashboard">
          <td :colspan="columns">
            <div class="flex-container">
              <div class="flex-child mine-counter">
                <div class="label">Mines Flagged:</div>
                <div class="count">{{ flaggedCount }}/{{ mineCount }}</div>
              </div>
              <div class="flex-child reset-button-container">
                <img
                  :src="buttonImage"
                  @click="reset"
                >
              </div>
              <div class="flex-child timer-container">
                <div class="label">Time Elapsed:</div>
                <div class="timer">{{ formattedTimer }}</div>
              </div>
            </div>
          </td>
        </tr>
        <tr
          v-for="row in rows"
          :key="`row-${row - 1}`"
          :id="`row-${row - 1}`"
        >
          <td
            :class="getCellClass(row, col)"
            v-for="col in columns"
            :key="getCellIndex(row, col)"
            :id="`cell-${getCellIndex(row, col)}`"
            @contextmenu="toggleFlag($event, getCellIndex(row, col))"
          >
            <input
              type="image"
              :src="gameMatrix[getCellIndex(row,col)].src"
              @click="handleCellClicked(getCellIndex(row, col))"
              :disabled="dead || won || !gameMatrix[getCellIndex(row,col)].covered"
            >
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import ConfettiExplosion from "vue-confetti-explosion"
export default {
  name: 'GameBoard',
  components: { ConfettiExplosion },
  props: {
    difficulty: {
      type: String,
      required: true,
    },
    customColumns: {
      type: Number,
      required: false,
    },
    customRows: {
      type: Number,
      required: false,
    },
    customMines: {
      type: Number,
      required: false,
    }
  },
  data() {
    return {
      gameboardReady: false,
      difficultyLevels: {
        beginner: {
          columns: 9,
          rows: 9,
          mines: 10
        },
        intermediate: {
          columns: 16,
          rows: 16,
          mines: 40
        },
        expert: {
          columns: 20,
          rows: 24,
          mines: 99
        }
      },
      dead: false,
      won: false,
      inGame: false,
      timeElapsed: 0,
      counter: null,
      columns: 9,
      rows: 9,
      mineCount: 10,
      totalCellCount: 81,
      gameMatrix: [],
      mineLocations: [],
      flaggedCells: [],
      images: {
        0: '../../src/assets/game-images/00.png',
        1: '../../src/assets/game-images/01.png',
        2: '../../src/assets/game-images/02.png',
        3: '../../src/assets/game-images/03.png',
        4: '../../src/assets/game-images/04.png',
        5: '../../src/assets/game-images/05.png',
        6: '../../src/assets/game-images/06.png',
        7: '../../src/assets/game-images/07.png',
        8: '../../src/assets/game-images/08.png',
      }
    }
  },
  mounted() {
    this.reset();
  },
  computed: {
    coveredCount() {
      if (this.gameboardReady) {
        return this.gameMatrix.filter((cell) => cell.covered).length;
      }

      return this.totalCellCount;
    },
    flaggedCount() {
      if (this.gameboardReady) {
        return this.gameMatrix.filter((cell) => cell.flagged).length;
      }

      return 0;
    },
    formattedTimer() {
      return new Date(this.timeElapsed * 1000).toISOString().slice(14, 19);
    },
    buttonImage() {
      if (this.dead) {
        return '../../src/assets/game-images/dead-face.png';
      } else if (this.won) {
        return '../../src/assets/game-images/party-face.png';
      } else {
        return '../../src/assets/game-images/smiley-face.png';
      }
    },
  },
  methods: {
    reset() {
      this.gameboardReady = false;
      clearInterval(this.counter);
      this.counter = null;

      if (this.difficulty === 'custom') {
        this.columns = this.customColumns;
        this.rows = this.customRows;
        this.mineCount = this.customMines;
      } else {
        this.columns = this.difficultyLevels[this.difficulty].columns;
        this.rows = this.difficultyLevels[this.difficulty].rows;
        this.mineCount = this.difficultyLevels[this.difficulty].mines;
      }

      this.totalCellCount = this.columns * this.rows;
      this.generateMineLocations();
      this.buildGameMatrix();

      this.timeElapsed = 0;
      this.counter = setInterval(this.timer, 1000);
      this.dead = false;
      this.won = false;
      this.inGame = false;

      this.gameboardReady = true;
    },
    buildGameMatrix() {
      const matrix = [];

      // Figure out which cells are on the left edge
      const leftEdge = [];
      for (let i = 0; i < this.totalCellCount; i += this.columns) {
        leftEdge.push(i);
      }

      // Figure out which cells are on the right edge
      const rightEdge = [];
      for (let i = this.columns -1; i < this.totalCellCount; i += this.columns) {
        rightEdge.push(i);
      }

      for (let i = 0; i < this.totalCellCount; i++) {
        const adjacentCells = [];
        const cellsToCheck = [
          i - (this.columns + 1), i - (this.columns), i - (this.columns - 1),
          i - 1, i + 1,
          i + (this.columns - 1), i + this.columns, i + this.columns + 1
        ];

        for(const cell of cellsToCheck) {
          if (cell > -1 && cell < this.totalCellCount) {
            // For side-edge cells, exclude cells that are on the opposite side-edge
            if (
              (leftEdge.includes(i) && rightEdge.includes(cell)) ||
              (rightEdge.includes(i) && leftEdge.includes(cell))
            ) {
              continue;
            }
            adjacentCells.push(cell);
          }
        }

        matrix[i] = {
          covered: true,
          flagged: false,
          src: '../../src/assets/game-images/covered.png',
          adjacentCells
        };
      }

      this.gameMatrix = matrix;
    },
    generateMineLocations() {
      let cells = [];

      // Generate an array of cell indices
      for (let i = 0; i < (this.rows * this.columns); i++) {
        cells[i] = i;
      }

      // Scramble the array
      for (let i = cells.length-1; i > 1; i--)
      {
        const r = Math.floor(Math.random() * i);
        const t = cells[i];
        cells[i] = cells[r];
        cells[r] = t;
      }

      // Return the first n indices from the scrambled array, where n is the number of mines in the game
      this.mineLocations = cells.slice(0, this.mineCount);
    },
    getCellIndex(row, col) {
      return this.columns * (row - 1) + col - 1;
    },
    getCellClass(row, col) {
      const cell = this.gameMatrix[this.getCellIndex(row, col)];

      return cell.covered
        ? 'covered'
        : 'uncovered';
    },
    toggleFlag(e, cellIndex) {
      e.preventDefault();

      if (!this.inGame && !this.dead && !this.won) {
        this.inGame = true;
      }

      if (this.inGame) {
        const cell = this.gameMatrix[cellIndex];

        if (cell.covered) {
          cell.flagged = !cell.flagged;
          cell.src = cell.flagged
            ? '../../src/assets/game-images/flag.png'
            : '../../src/assets/game-images/covered.png';
        }

        return false;
      }
    },
    handleCellClicked(cellIndex) {
      if (!this.gameMatrix[cellIndex].covered) {
        return;
      }

      if (this.mineLocations.includes(cellIndex)) {
        return this.handleMineClicked(cellIndex);
      }

      if (!this.inGame) {
        this.inGame = true;
      }

      this.cascade(cellIndex);

      if (this.coveredCount === this.mineCount) {
        this.gameOver();
        this.won = true;
      }
    },
    handleMineClicked(cellIndex) {
      const newMatrix = [];

      this.gameMatrix.forEach((cell, i) => {
        if (this.mineLocations.includes(i)) {
          newMatrix.push({
            covered: false,
            flagged: cell.flagged,
            adjacentCells: cell.adjacentCells,
            src: i === cellIndex
              ? '../../src/assets/game-images/triggerBomb.png'
              : '../../src/assets/game-images/bomb.png'
          });
        } else {
          newMatrix.push(cell);
        }
      });

      this.gameMatrix = newMatrix;
      this.dead = true;
      this.inGame = false;
    },
    cascade(cellIndex) {
      let cellsToCheck = [];
      cellsToCheck.unshift(cellIndex);

      while(cellsToCheck.length !== 0) {
        const currentCellIndex = cellsToCheck.pop();
        const cell = this.gameMatrix[currentCellIndex];

        // Uncover the cell
        cell.covered = false;

        const adjacentMines = this.countMines(cell.adjacentCells);

        // If no adjacent mines, perform cascade, checking the neighbors of the adjacent cells
        if (adjacentMines === 0) {
          cell.src = this.images[0];

          // Add the cell's neighbors to the queue
          cell.adjacentCells.forEach((adjacentCell) => {
            const cellObject = this.gameMatrix[adjacentCell];
            if (cellObject.covered && !cellsToCheck.includes(adjacentCell)) {
              cellsToCheck.unshift(adjacentCell);
            }
          });
        }
        else {
          // Put the image representing the number of adjacent mines in this cell
          cell.src = this.images[adjacentMines];
        }
      }
    },
    countMines(cells) {
      let count = 0;
      cells.forEach((cell) => {
        if (this.mineLocations.includes(cell)) {
          count++;
        }
      });

      return count;
    },
    timer() {
      if (this.inGame) {
        this.timeElapsed++;

        if (this.timeElapsed >= 3599) {
          clearInterval(this.counter);
        }
      }
    },
    gameOver() {
      this.inGame = false;
    },
  },
  watch: {
    difficulty() {
      this.reset();
    },
    customRows() {
      if (this.difficulty === 'custom') {
        this.reset();
      }
    },
    customColumns() {
      if (this.difficulty === 'custom') {
        this.reset();
      }
    },
    customMines() {
      if (this.difficulty === 'custom') {
        this.reset();
      }
    }
  }
}
</script>

<style lang="scss" scoped>
#gameboard {
  table {
    border: 8px ridge;
    background-color: #BDBDBD;
    table-layout: fixed;
    margin-left: auto;
    margin-right: auto;

    tr.dashboard {
      .flex-container {
        display: flex;
        width: 100%;

        div.flex-child {
          flex: 1;
          justify-content: center;

          &.mine-counter, &.timer-container {
            margin: 10px;
          }
          &.reset-button-container {
            margin: 10px 0;
            display: flex;
            align-items: center;

            img {
              width: 50px;
              height: auto;
            }
          }
          .label {
            font-size: 10px;
            color: #363636;
            font-weight: bold;
            padding-bottom: 1px;
          }
          .count, .timer {
            background-color: #181818;
            color: red;
            border: 3px inset;
            font-size: 1.6rem;
            font-family: "Kode Mono", monospace;
            font-optical-sizing: auto;
            font-style: normal;
            padding: 0 5px;
            min-width: 93px;
            text-align: center;
          }
        }
      }
    }

    tr {
      td {
        border-style: outset;
        text-align: center;
        padding: 0;
        width: 30px;
        height: 30px;
        min-width: 30px;

        &.covered {
          border-style: outset;
          text-align: center;
          overflow: hidden;
        }

        &.uncovered {
          border: 1px solid #BFBFBF;
          background-color: #CCCCCC;
          text-align: center;
        }

        input[type=image] {
          display: block;
          width: 100%;
          height: auto;
        }
      }
    }
  }
}
</style>