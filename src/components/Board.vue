/*
Game of Life - Board
data.cells -> cell -> name, dead/alive
notes:
  data.cells - first sevens false and next pair false are for correct cell rendering and neigbours check
*/

<template>
  <section id="board">
    <p>Generation: {{generationNumber}}</p>
    <div>
      <button class="w3-button" @click="startGen" :disabled="!allowed">Start gen</button>
      <button class="w3-button" @click="stopGen">Stop gen</button>
      <button class="w3-button" @click="resetCells">Reset cells</button>
      <button class="w3-button" @click="genRandomCells">Random cells</button>
    </div>
    <div>
      <input type="number" min="3" v-model.number="gridSize" @keyup.enter="generateGridCells()">
    </div>
    <div class="grid-container">
      <Cell
        v-for="cell in cells"
        :key="cell[0]"
        :state="cell"
        @changeState="updateCellsOnClick($event)"
      />
    </div>
  </section>
</template>

<script>
import Cell from "./Cell";

export default {
  name: "Board",
  components: {
    Cell
  },
  data: function() {
    return {
      allowed: true,
      cells: [],
      generationNumber: 0,
      gridSize: 7,
      boardInterval: false,
      genBoard: false
    };
  },
  methods: {
    checkNeighbours() {
      /* check cells neighbours */
      const cellsNeigbours = []; // return value
      const cells = this.cells.map(cell => cell); // copy cells

      // check neigbours
      for (let index = 0; index < cells.length; index++) {
        const tl = cells[index - 8] ? cells[index - 8][1] : false;
        const tm = cells[index - 7] ? cells[index - 7][1] : false;
        const tr = cells[index - 6] ? cells[index - 6][1] : false;
        const ml = cells[index - 1] ? cells[index - 1][1] : false;
        const mr = cells[index + 1] ? cells[index + 1][1] : false;
        const bl = cells[index + 6] ? cells[index + 6][1] : false;
        const bm = cells[index + 7] ? cells[index + 7][1] : false;
        const br = cells[index + 8] ? cells[index + 8][1] : false;

        // collect neigbours; replace undefined with false
        const cellNeigbours = [tl, tm, tr, ml, mr, bl, bm, br].map(state =>
          state === undefined ? false : state
        );

        cellsNeigbours.push([index, cellNeigbours]); // collect neigbours data
      }

      return cellsNeigbours;
    },
    generateGridCells() {
      const gridSize = this.gridSize; // set grid size
      const newGrid = []; // return value
      const firstRow = gridSize + 2; // first row correction
      const loopLenght = gridSize ** 2 + gridSize; // calc loop length

      let cellAddCounter = 1; // how many cell added to row
      let namingIndex = 0; // cell naming index
      let rowStart = true; // row start
      let addCell = false; // add cell
      let rowEnd = false; // row end

      this.updateGridContainer(); // update grid size

      // add n false values to newGrid
      for (let i = 0; i < firstRow; i++) {
        newGrid.push(false); // add
      }

      // create rows
      for (let row = 0; row < loopLenght; row++) {
        // row start, add false
        if (rowStart) {
          newGrid.push(false);
          rowStart = false;
          addCell = true;
        }

        // addCell and not at rows end, add cell else stop adding cells
        if (addCell && cellAddCounter <= gridSize) {
          const cellName = "cell_" + namingIndex;
          newGrid.push([cellName, false]);
          cellAddCounter++;
          namingIndex++;
        } else {
          addCell = false;
          cellAddCounter = 1;
          rowEnd = true;
        }

        // row end, add false
        if (rowEnd) {
          newGrid.push(false);
          rowStart = true;
          rowEnd = false;
        }
      }

      this.cells = newGrid; // update board state
      this.genRandomCells(); // random cell states
    },
    genRandomCells() {
      /* generate random cells */
      this.cells = this.cells.map(cell => this.genRandomCell(cell)); // map genRandomCell over board cells state
    },
    genRandomCell(cell) {
      /* generate random cell */
      return cell ? [cell[0], this.randomCellState()] : false;
    },
    randomCellState() {
      /* generate random cell state */
      return Math.random() > 0.5 ? true : false;
    },
    resetCells() {
      /* reset board state */
      clearInterval(this.boardInterval); // clear interval
      this.allowed = true; // turn on start gen
      this.generationNumber = 0; // zero generation
      this.cells = this.cells.map(cell => (cell ? [cell[0], false] : false)); // cells to false
    },
    startGen() {
      /* start board gen */
      this.allowed = false; // turn off start gen
      // start generator
      this.boardInterval = setInterval(
        () => this.updateBoardState(this.checkNeighbours()),
        1000
      );
    },
    stopGen() {
      /* stop board gen */
      clearInterval(this.boardInterval); // stop generator
      this.allowed = true; // turn on start gen
    },
    updateBoardState(neigboursData) {
      /* state updater */
      let cellNewState = []; // live or die

      for (let cell of neigboursData) {
        const cellIndex = cell[0]; // cells index
        // count alive neigbours
        const aliveNeigbours = cell[1].reduce(
          (accumulator, currentValue) => accumulator + (currentValue ? 1 : 0)
        );
        const dataCellName = this.cells[cellIndex][0]; // get cell name

        let checkIf = true; // stop unnecessary ifs

        // dataCellName = undefined
        if (checkIf && dataCellName === undefined) {
          cellNewState.push(false);
          checkIf = false;
        }

        // dead cell has three alive neigbours, cell lives
        if (checkIf && !this.cells[cellIndex][1] && aliveNeigbours === 3) {
          cellNewState.push([dataCellName, true]);
          checkIf = false;
        }
        // less that two alive neigbours, cell dies
        if (checkIf && aliveNeigbours < 2) {
          cellNewState.push([dataCellName, false]);
          checkIf = false;
        }

        // two or three alive neigbours, cell lives
        if (checkIf && (aliveNeigbours === 2 || aliveNeigbours === 3)) {
          cellNewState.push(this.cells[cellIndex]);
          checkIf = false;
        }

        // more than three alive neigbours, cell dies
        if (checkIf && aliveNeigbours > 3) {
          cellNewState.push([dataCellName, false]);
          checkIf = false;
        }
      }

      this.cells = cellNewState; // update cells
      this.generationNumber = ++this.generationNumber; // update generation number
    },
    updateCellsOnClick(input) {
      /* changes clicked cell state */
      // look for clicked cells name in cells, update its state
      this.cells = this.cells.map(cell =>
        cell[0] === input[0] ? [input[0], input[1]] : cell
      );
    },
    updateGridContainer(input) {
      /* change grid-container class */
      document.querySelector(
        ".grid-container"
      ).style.gridTemplateColumns = `repeat(${this.gridSize}, 3.5em)`;
    }
  },
  mounted: function() {
    this.generateGridCells();
  }
};
</script>

<style>
.grid-container {
  display: inline-grid;
  grid-template-columns: repeat(5, 3.5em);
  grid-gap: 2%;
}
</style>