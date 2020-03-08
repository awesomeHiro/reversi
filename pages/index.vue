<template>
  <div class="container">
    <template v-for="y in board.length - 2">
      <div
        v-for="x in board[y].length - 2"
        :key="`${y}-${x}`"
        class="cell"
        @click="putStone(y, x)"
      >
        <div
          v-if="isStone(y, x)"
          :class="['stone', isBlack(y, x) ? 'black' : 'white']"
        />
        {{ y }}
        {{ x }}
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
@Component
export default class extends Vue {
  black = 1
  white = 2
  color = 1
  currentTurn = 0
  // Directions by bit
  bitUp = 1
  bitUpRight = 2
  bitRight = 4
  bitDownRiht = 8
  bitDown = 16
  bitDownLeft = 32
  bitLeft = 64
  bitUpLeft = 128
  dirs = [
    // directions
    [-1, 0, this.bitUp],
    [-1, +1, this.bitUpRight],
    [-1, -1, this.bitUpLeft],
    [1, 0, this.bitDown],
    [1, +1, this.bitDownRiht],
    [1, -1, this.bitDownLeft],
    [0, +1, this.bitRight],
    [0, -1, this.bitLeft]
  ]

  // // 1: black, 2:white, -1: margin
  board = [
    [-1, -1, -1, -1, -1, -1, -1, -1, -1, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, 0, 0, 0, 1, 2, 0, 0, 0, -1],
    [-1, 0, 0, 0, 2, 1, 0, 0, 0, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, 0, 0, 0, 0, 0, 0, 0, 0, -1],
    [-1, -1, -1, -1, -1, -1, -1, -1, -1, -1]
  ]

  connects = [
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 1, 0, 0, 1, 0, 0, 0],
    [0, 0, 0, 1, 0, 0, 1, 0, 0, 0],
    [0, 0, 0, 1, 1, 1, 1, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  ]

  // please refer https://www.figma.com/file/Yn2muTd2riUxDZY53R1AaN/Untitled?node-id=0%3A1
  availables = [
    [], // dummy for correspond array index and color flag
    [
      // for Black
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 16, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 64, 0, 0, 0],
      [0, 0, 0, 4, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ],
    [
      // for White
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 16, 0, 0, 0, 0, 0],
      [0, 0, 0, 4, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 64, 0, 0, 0],
      [0, 0, 0, 0, 0, 1, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    ]
  ]

  get isStone() {
    return (y: number, x: number) => this.board[y][x] !== 0
  }

  get isBlack() {
    return (y: number, x: number) => this.board[y][x] === 1
  }

  get setStone() {
    return (y: number, x: number) => {
      this.flipStones(y, x)
      this.board[y][x] = this.color
      return true
    }
  }

  get flipEachDirection() {
    return (row: number, clm: number, dir: Array<number>) => {
      const [dy, dx, bitFlag] = dir
      const cellFlag = this.availables[this.color][row][clm]

      if (cellFlag | bitFlag) {
        let y = row + dy
        let x = clm + dx
        while (this.board[y][x] === 3 - this.color) {
          this.board[y][x] = this.color
          y = y + dy
          x = x + dx
        }
      }
      return false
    }
  }

  get flipStones() {
    return (y: number, x: number) => {
      this.board[y][x] = this.color
      this.dirs.forEach((dir) => {
        this.flipEachDirection(y, x, dir)
      })
      return true
    }
  }

  get passTrun() {
    return () => {
      this.currentTurn = this.currentTurn + 1
    }
  }
  get reRenderBoard() {
    return () => {
      // TODO: we should use mutate array method
      this.board = JSON.parse(JSON.stringify(this.board))
    }
  }

  get updateColor() {
    return () => (this.color = (this.currentTurn % 2) + 1)
  }

  get checkDirection() {
    return (row: number, clm: number, dir: Array<number>): number => {
      let availability = 0
      const color = this.board[row][clm]
      const [dy, dx, bitFlag] = dir
      if (this.board[row + dy][clm + dx] === 3 - color) {
        // if upper direction is opposite color
        let y = row + dy
        let x = clm + dx
        while (this.board[y][x] === 3 - color) {
          // follow the opposite stone sequence
          // this loop continue until oppsite stone streak is done
          y = y + dy
          x = x + dx
        }
        if (this.board[y][x] === color) {
          // if find a same color on the end of the direction
          availability = bitFlag
        }
      }
      return availability
    }
  }

  get calcAvail() {
    return (row: number, clm: number): number => {
      let availability = 0 // if no direction available, cell value is 0
      this.dirs.forEach((dir) => {
        availability = availability + this.checkDirection(row, clm, dir)
      })
      return availability
    }
  }

  get updateAvailables() {
    // FIXME
    return () => {
      for (let row = 1; row < this.board.length - 1; row++) {
        for (let clm = 1; clm < this.board[1].length - 1; clm++) {
          if (this.connects[row][clm] === 0) {
            this.availables[this.black][row][clm] = 0 // FIXME: this.color must be changed, we need calc both color
            this.availables[this.white][row][clm] = 0 // FIXME: this.color must be changed, we need calc both color
          } else {
            this.availables[this.black][row][clm] = this.calcAvail(row, clm)
            this.availables[this.white][row][clm] = this.calcAvail(row, clm)
          }
        }
      }
    }
  }

  get updateConnects() {
    return (y: number, x: number) => {
      this.dirs.forEach((dir) => {
        this.connects[y + dir[0]][x + dir[1]] = 1
      })
      for (let row = 1; row < this.board.length - 1; row++) {
        for (let clm = 1; clm < this.board[1].length - 1; clm++) {
          if (this.board[row][clm] !== 0) {
            this.connects[row][clm] = 0
          }
        }
      }
    }
  }

  get putStone() {
    return (y: number, x: number) => {
      if (!this.availables[this.color][y][x]) return false
      this.setStone(y, x) // Black
      this.reRenderBoard() // Black
      this.updateConnects(y, x) // Black
      this.updateAvailables() // Black
      this.passTrun() // Black
      this.updateColor() // White
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
