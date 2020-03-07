<template>
  <div class="container">
    <template v-for="x in board.length">
      <div
        v-for="y in board[x - 1].length"
        :key="`${y}-${x}`"
        class="cell"
        @click="putStone(x, y)"
      >
        <div
          v-if="isStone(x, y)"
          :class="['stone', isBlack(x, y) ? 'black' : 'white']"
        />
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
@Component
export default class extends Vue {
  currentColor = 1 // 1: black, 2:white

  board = [
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 1, 2, 0, 0, 0],
    [0, 0, 0, 2, 1, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0]
  ]

  // marginBoard = [
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 1, 2, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 2, 1, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
  //   [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  // ]

  connectedBoard = [
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  ]

  get isStone() {
    return (x: number, y: number) => this.board[x - 1][y - 1] !== 0
  }

  get isBlack() {
    return (x: number, y: number) => this.board[x - 1][y - 1] === 1
  }

  get setStone() {
    return (x: number, y: number) => {
      return (this.board[x - 1][y - 1] = this.currentColor)
    }
  }

  get isLonelyCell() {
    return (x: number, y: number) => {
      console.log(x, y)

      return !this.connectedBoard[x][y]
    }
  }

  get passTrun() {
    return () => {
      this.currentColor = 3 - this.currentColor
    }
  }
  // TODO: we should use mutate array method
  get reRenderBoard() {
    return () => {
      this.board = JSON.parse(JSON.stringify(this.board))
    }
  }

  // get updateMarginBoard() {
  //   return () => {
  //     this.board.forEach((row, x) => {
  //       row.forEach((cell, y) => {
  //         if (cell) {
  //           this.marginBoard[x + 1][y + 1] = cell
  //         }
  //       })
  //     })
  //   }
  // }

  get updateConnectedBoard() {
    return () => {
      this.board.forEach((row, x) => {
        row.forEach((cell, y) => {
          if (cell) {
            this.connectedBoard[x + 0][y + 0] = 1
            this.connectedBoard[x + 0][y + 1] = 1
            this.connectedBoard[x + 0][y + 2] = 1
            this.connectedBoard[x + 1][y + 0] = 1
            this.connectedBoard[x + 1][y + 2] = 1
            this.connectedBoard[x + 2][y + 0] = 1
            this.connectedBoard[x + 2][y + 1] = 1
            this.connectedBoard[x + 2][y + 2] = 1
          }
        })
      })
    }
  }

  get putStone() {
    return (x: number, y: number) => {
      if (this.isStone(x, y)) return false
      if (this.isLonelyCell(x, y)) return false
      this.setStone(x, y)
      this.passTrun()
      this.reRenderBoard()
      // this.updateMarginBoard()
      this.updateConnectedBoard()
    }
  }
}
</script>

<style scoped>
/* autoprefixer grid: no-autoplace */
.container {
  width: 640px;
  height: 640px;
  margin: 20px auto;
  background: #050;
}
.cell {
  float: left;
  width: 12.5%;
  height: 12.5%;
  border: #000 solid;
}

.stone {
  width: 70%;
  height: 70%;
  margin: 15%;
  border-radius: 50%;
}

.black {
  background: #000;
}
.white {
  background: #fff;
}
</style>
