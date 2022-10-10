<template>
  <div :class="$style.container">
    <dir :class="$style.info">
      <dir :class="$style['info-players']">
        <span :class="[config.names.plr1 === curPlayer ? $style['active-player'] : '', 'has-text-weight-bold', $style[config.players.plr1]]">
          {{ config.names.plr1 }}: <span class="has-text-grey">{{ plr1.length }}</span>
        </span>
        <span :class="[config.names.plr2 === curPlayer ? $style['active-player'] : '', 'has-text-weight-bold', $style[config.players.plr2]]">
          {{ config.names.plr2 }}: <span class="has-text-grey">{{ plr2.length }}</span>
        </span>
      </dir>
    </dir>
    <div :class="$style.grid">
      <Transition>
        <div v-show="msgScreen" :class="$style['front-screen']">
            <div :class="$style.message">{{ msgScreen }}</div>
          </div>
      </Transition>
      <div :class="$style.row">
        <div :class="[$style.tile, $style['line'], canDraw ? $style[drawWinLine[1]] : '']" @click="setCoord(1)">
          <div
            :class="[$style['game-element'], $style[getClass(1)]]"
            v-text="getSymbol(1)"
          ></div>
        </div>
        <div :class="$style.tile" @click="setCoord(2)">
          <div
            :class="[$style['game-element'], $style[getClass(2)]]"
            v-text="getSymbol(2)"
          ></div>
        </div>
        <div :class="$style.tile" @click="setCoord(3)">
          <div
            :class="[$style['game-element'], $style[getClass(3)]]"
            v-text="getSymbol(3)"
          ></div>
        </div>
      </div>
      <div :class="$style.row">
        <div :class="$style.tile" @click="setCoord(4)">
          <div
            :class="[$style['game-element'], $style[getClass(4)]]"
            v-text="getSymbol(4)"
          ></div>
        </div>
        <div :class="[$style.tile, $style['line'], canDraw ? $style[drawWinLine[5]] : '']" @click="setCoord(5)">
          <div
            :class="[$style['game-element'], $style[getClass(5)]]"
            v-text="getSymbol(5)"
          ></div>
        </div>
        <div :class="$style.tile" @click="setCoord(6)">
          <div
            :class="[$style['game-element'], $style[getClass(6)]]"
            v-text="getSymbol(6)"
          ></div>
        </div>
      </div>
      <div :class="$style.row">
        <div :class="$style.tile" @click="setCoord(7)">
          <div
            :class="[$style['game-element'], $style[getClass(7)]]"
            v-text="getSymbol(7)"
          ></div>
        </div>
        <div :class="$style.tile" @click="setCoord(8)">
          <div
            :class="[$style['game-element'], $style[getClass(8)]]"
            v-text="getSymbol(8)"
          ></div>
        </div>
        <div :class="[$style.tile, $style['line'], canDraw ? $style[drawWinLine[9]] : '']" @click="setCoord(9)">
          <div
            :class="[$style['game-element'], $style[getClass(9)]]"
            v-text="getSymbol(9)"
          ></div>
        </div>
      </div>
    </div>
    <div :class="$style['buttons']">
      <button :class="$style['btn--start-game']" @click="startGame">
        <template v-if="isStarted">Restart</template>
        <template v-else>Start game</template>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    msg: String,
  },
  data() {
    this.winCombinations = [
      // 0..2: 0deg
      [1, 2, 3],
      [4, 5, 6],
      [7, 8, 9],
      // 3..5: 90deg
      [1, 4, 7],
      [2, 5, 8],
      [3, 6, 9],
      // 6: 45deg
      [1, 5, 9],
      // 7: 135deg
      [3, 5, 7],
    ];
    return {
      isStarted: false,
      plr1: [],
      plr2: [],
      grid: {
        /*1: {
          class: null,
          value: null
        }*/
      },
      drawWinLine: {
        /*2: 'line-0'
        5: 'line-0'
        8: 'line-0' */
      },
      current: "plr1",
      config: {
        names: {
          plr1: "Player 1",
          plr2: "Player 2",
        },
        symbols: {
          cross: "X",
          zero: "0",
        },
        players: {
          plr1: "cross",
          plr2: "zero",
        },
      },
    };
  },
  computed: {
    curPlayer() {
      return this.config.names[this.current];
    },
    canDraw () {
      return Object.keys(this.drawWinLine).length === 3;
    },
    isDrawn() {
      return Object.keys(this.grid).length === 9;
    },
    msgScreen() {
      if (!this.isStarted) {
        return 'Click start game!';
      }

      if (this.canDraw) {
        return `${this.config.names[this.current]} won.`;
      }

      if (this.isDrawn) {
        return 'Drawn game.';
      }

      return null;
    }
  },
  methods: {
    startGame() {
      this.grid = {};
      this.drawWinLine = {};

      if (this.isStarted) {
        Object.keys(this.config.players).forEach((plr) => {
          this[plr] = [];
        });
      }

      this.isStarted = !this.isStarted;
    },
    switchPlayer() {
      this.current = this.current === "plr1" ? "plr2" : "plr1";
    },
    addElemPlr(pos) {
      this[this.current].push(pos);
    },
    fillGrid(pos) {
      const curClass = this.config.players[this.current];
      this.grid[pos] = {
        class: curClass,
        symbol: this.config.symbols[curClass],
      };
    },
    setCoord(pos) {
      if (!this.isStarted || this.grid[pos]) {
        return;
      }

      this.addElemPlr(pos);
      this.fillGrid(pos);

      this.findWin().then((sucess) => {
        if (!sucess) {
          this.switchPlayer();
        }
      });
    },
    defineRotateByCombs(index) {
      let val = 135;

      if (index >= 0 && index <= 2) {
        val = 0;
      }
      else if (index >= 3 && index <= 5) {
        val = 90;
      }
      else if (index === 6) {
        val = 45;
      }

      return val;
    },
    async findWin() {
      const plr = this[this.current];
      if (plr.length < 3) {
        return false;
      }

      for (let i = 0; i < this.winCombinations.length; i++) {
        let cnt = 0;
        const _drawWinLine = {};
        for (let j = 0; j < 3; j++) {
          const num = this.winCombinations[i][j];
          if (plr.includes(num)) {
            _drawWinLine[num] = `line-${this.defineRotateByCombs(i)}`;
            cnt += 1;
            if (3 === cnt) {
              this.drawWinLine = _drawWinLine;
              return true;
            }
          }
        }
      }

      return false;
    },
    getClass(pos) {
      if (!this.grid[pos]) {
        return "";
      }

      return this.grid[pos].class;
    },
    getSymbol(pos) {
      if (!this.grid[pos]) {
        return "";
      }

      return this.grid[pos].symbol;
    },
  },
};
</script>

<style module lang="scss">
$color_cross: lightgreen;
$color_zero: lightpink;
$color_border: #333;
$primary_color: #736d65;

.container {
  width: 34%;
  margin: 0 auto;

  .info {
    padding: 0;
    margin-bottom: 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    &-players {
      font-size: 2rem;
      padding: 0 0 0 5px;
      margin: 0;
      min-width: 370px;
      display: flex;
      justify-content: space-between;
      align-items: center;

      .active-player::after {
        position: absolute;
        content: '';
        display: block;
        width: 152px;
        height: 5px;
        border-radius: 3px;
        background: $primary_color;
      }
    }

    @media only screen and (max-width: 935px) {
      &-players {
        font-size: 20px;
        min-width: 230px;

        .active-player::after {
          width: 75px;
        }
      }
    }
    @media only screen and (max-width: 672px) {
      &-players {
        font-size: 12px;
        min-width: 150px;

        .active-player::after {
          width: 60px;
        }
      }
    }

  }
  .grid {
    margin: 0 auto;
    position: relative;
    overflow: hidden;

    .front-screen {
      z-index: 10;
      position: absolute;
      width: 100%;
      height: 100%;
      background: rgba(253, 255, 254, 0.82);
      display: flex;
      justify-content: center;
      align-items: center;

      .message {
        font-size: 3rem;
        color: $primary_color;
      }

      @media only screen and (max-width: 820px) {
        .message {
          font-size: 2rem;
        }
      }
    }
    .row {
      margin: 0 auto;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 0;
      border-right: 1px solid $color_border;

      .tile {
        cursor: pointer;
        border: 1px solid $color_border;
        border-right: none;
        border-bottom: none;

        aspect-ratio: 1/ 1;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 1%;
        user-select: none;

        .game-element {
          font-size: 12rem;
          line-height: 0;
        }
        @media only screen and (max-width: 1600px) {
          .game-element {
            font-size: 11rem;
          }
        }
        @media only screen and (max-width: 1400px) {
          .game-element {
            font-size: 10rem;
          }
        }
        @media only screen and (max-width: 1257px) {
          .game-element {
            font-size: 8rem;
          }
        }
        @media only screen and (max-width: 1065px) {
          .game-element {
            font-size: 6rem;
          }
        }
        @media only screen and (max-width: 720px) {
          .game-element {
            font-size: 5rem;
          }
        }
        @media only screen and (max-width: 250px) {
          .game-element {
            font-size: 3rem;
          }
        }
        @media only screen and (max-width: 180px) {
          .game-element {
            font-size: 2rem;
          }
        }

        &.line::after {
          content: '';
          position: absolute;
          background: $primary_color;
          width: 200%;
          height: 0;
        }
        &.line {
          &-0::after {
            height: 1px;
            transform: rotate(0deg);
          }
          &-90::after {
            height: 1px;
            transform: rotate(90deg);
          }
          &-45::after {
            height: 1px;
            transform: rotate(45deg);
          }
          &-135::after {
            height: 1px;
            transform: rotate(135deg);
          }
        }
      }

      &:last-of-type {
        border-bottom: 1px solid $color_border;
      }
    }
  }
  .buttons {
    margin-top: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    .btn--start-game {
      width: 180px;
      cursor: pointer;
      line-height: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 2rem;
      background: #fff;
      padding: 10px;
      border: 1px solid $color_border;
      color: $color_border;
      border-radius: 0;

      &:active {
        background: $color_border;
        color: #fff;
      }
    }
    @media only screen and (max-width: 820px) {
      .btn--start-game {
        font-size: 1rem;
      }
    }
  }
  .cross {
    color: $color_cross !important;
  }
  .zero {
    color: $color_zero !important;
  }
}

@media only screen and (max-width: 935px) {
  .container {
    width: 45%;
  }
}
@media only screen and (max-width: 635px) {
  .container {
    width: 55%;
  }
}
@media only screen and (max-width: 535px) {
  .container {
    width: 65%;
  }
}
@media only screen and (max-width: 435px) {
  .container {
    width: 75%;
  }
}
@media only screen and (max-width: 335px) {
  .container {
    width: 85%;
  }
}
</style>

<style>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
