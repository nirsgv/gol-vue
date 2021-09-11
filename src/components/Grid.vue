<template>
  <div>
    <div>
      <input type="number" v-model.number="rows" @change="setTable" />
      <input type="number" v-model.number="columns" @change="setTable" />
    </div>
    <div>
      <ul  class="grid" :style="{ 
        gridTemplateColumns: `repeat(${this.columns}, 10px)`, 
        gridTemplateRows: `repeat(${this.rows}, 10px)` }"
      >
        <Cell
          v-for="(item, index) in this.table"
          v-bind:on="Boolean(item)"
          class="cell"
          :key="`${index} ${item}`"
          >{{ item }}</Cell
        >
      </ul>
    </div>
  </div>
</template>

<script>
import Cell from "./Cell";

export default {
  name: "Grid",
  components: {
    Cell,
  },
  data: function() {
    return {
      rows: 30,
      columns: 30,
      intervalId: null,
      table: this.generateGrid(this.rows || 30, this.columns || 30),
    };
  },
  computed: {},
  methods: {
    log(message) {
      console.log(message);
    },
    setTable() {
      this.table = this.generateGrid(this.rows, this.columns);
    },
    retreiveIndexes(mapKey) {
      return mapKey.split('|rc|')
    },
    generateGrid(
      rows,
      columns,
      cellInitFunc = () => Boolean(Math.floor(Math.random() * 2))
    ) {
      const tableMap = {};
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < columns; c++) {
          tableMap[`${r}|rc|${c}`] = cellInitFunc();
        }
      }
      return tableMap;
    },

    countActiveNeighbours(rowIdx, colIdx, tableMap) {
      let count = 0;
      for (let x = -1; x <= 1; x++) {
        for (let y = -1; y <= 1; y++) {
          if (!x && !y) continue;
          tableMap[`${Number(rowIdx) + x}|rc|${Number(colIdx) + y}`] && count++;
        }
      }
      return count;
    },
    animate(cb) {
      return (this.intervalId = setInterval(cb, 60));
    },
    stopAnimate() {
      clearInterval(this.intervalId);
      return (this.intervalId = null);
    },
    proceed() {
      const newTable = { };
      for (let [index, val] of Object.entries(this.table)) {
          const indexes = this.retreiveIndexes(index);
          const activeNeighbours = this.countActiveNeighbours(...indexes,this.table)
          newTable[index] = val
                    ? activeNeighbours === 2 || activeNeighbours === 3 // initially alive cell
                    : activeNeighbours === 3; // initially dead cell
      }
      this.table = newTable;
    },
  },
  created: function() {
    var x = this.animate(this.proceed);
  },
};
</script>

<style>
.grid {
  font-size: 0;
  display: grid;
  grid-template-rows: repeat(30, 10px);
  grid-template-columns: repeat(30, 10px);

}
ul {
  list-style: none;
  display: block;
}
li {
  display: inline-block;
}
</style>
