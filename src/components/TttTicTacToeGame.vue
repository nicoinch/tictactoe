<template>
  <div>
    <div id="ttt-container">
      <table id="ttt" 
             :class="{ 'shake-it': isBoardShaking, 'disabled': !gameState.isHumanPlaying && !gameState.gameEnded }">
        <div class="mask">
          <svg 
            viewBox="0 0 132 132" 
            width="132px"   
            height="132px"   
            xml:space="preserve" 
            class="svg" 
            xmlns="http://www.w3.org/2000/svg">
            <circle 
              cx="66" 
              cy="66" 
              r="70"
              fill="none" 
              stroke-linecap="round" />
          </svg>
        </div>
        <div v-if="gameState.gameEnded" 
             class="results">
          <div v-if="gameState.winner !== ''">
            <ttt-circle 
              v-if="gameState.winner === 'O'" 
              :displayed="gameState.winner === 'O'" 
              :size="64"
              :stroke-width="7" />
            <ttt-cross 
              v-if="gameState.winner === 'X'" 
              :displayed="gameState.winner === 'X'" 
              :size="64"
              :stroke-width="7" />
            Wins
          </div>
          <div v-else>
            Nobody wins...
          </div>
          <button @click="finish">Play again!</button>
        </div>
        <tr id="row-1">
          <td 
            id="cell-1-1" 
            :class="{ 'filled' : gameState.game[0][0] !== '', 'animated' : cellsToAnimate[0][0] === '✨' }"
            @click="humanPlay(0, 0)">
            <ttt-circle :displayed="gameState.game[0][0] === 'O'" />
            <ttt-cross :displayed="gameState.game[0][0] === 'X'" />
          </td>
          <td 
            id="cell-1-2" 
            :class="{ 'filled' : gameState.game[0][1] !== '', 'animated' : cellsToAnimate[0][1] === '✨' }"
            @click="humanPlay(0, 1)">
            <ttt-circle :displayed="gameState.game[0][1] === 'O'" />
            <ttt-cross :displayed="gameState.game[0][1] === 'X'" />
          </td>
          <td 
            id="cell-1-3" 
            :class="{ 'filled' : gameState.game[0][2] !== '', 'animated' : cellsToAnimate[0][2] === '✨' }"
            @click="humanPlay(0, 2)">
            <ttt-circle :displayed="gameState.game[0][2] === 'O'" />
            <ttt-cross :displayed="gameState.game[0][2] === 'X'" />
          </td>
        </tr>
        <tr id="row-2">
          <td 
            id="cell-2-1" 
            :class="{ 'filled' : gameState.game[1][0] !== '', 'animated' : cellsToAnimate[1][0] === '✨' }"
            @click="humanPlay(1, 0)">
            <ttt-circle :displayed="gameState.game[1][0] === 'O'" />
            <ttt-cross :displayed="gameState.game[1][0] === 'X'" />
          </td>
          <td 
            id="cell-2-2" 
            :class="{ 'filled' : gameState.game[1][1] !== '', 'animated' : cellsToAnimate[1][1] === '✨' }"
            @click="humanPlay(1, 1)">
            <ttt-circle :displayed="gameState.game[1][1] === 'O'" />
            <ttt-cross :displayed="gameState.game[1][1] === 'X'" />
          </td>
          <td 
            id="cell-2-3" 
            :class="{ 'filled' : gameState.game[1][2] !== '', 'animated' : cellsToAnimate[1][2] === '✨' }"
            @click="humanPlay(1, 2)">
            <ttt-circle :displayed="gameState.game[1][2] === 'O'" />
            <ttt-cross :displayed="gameState.game[1][2] === 'X'" />
          </td>
        </tr>
        <tr id="row-3">
          <td 
            id="cell-3-1" 
            :class="{ 'filled' : gameState.game[2][0] !== '', 'animated' : cellsToAnimate[2][0] === '✨' }"
            @click="humanPlay(2, 0)">
            <ttt-circle :displayed="gameState.game[2][0] === 'O'" />
            <ttt-cross :displayed="gameState.game[2][0] === 'X'" />
          </td>
          <td 
            id="cell-3-2" 
            :class="{ 'filled' : gameState.game[2][1] !== '', 'animated' : cellsToAnimate[2][1] === '✨' }"
            @click="humanPlay(2, 1)">
            <ttt-circle :displayed="gameState.game[2][1] === 'O'" />
            <ttt-cross :displayed="gameState.game[2][1] === 'X'" />
          </td>
          <td 
            id="cell-3-3" 
            :class="{ 'filled' : gameState.game[2][2] !== '', 'animated' : cellsToAnimate[2][2] === '✨' }"
            @click="humanPlay(2, 2)">
            <ttt-circle :displayed="gameState.game[2][2] === 'O'" />
            <ttt-cross :displayed="gameState.game[2][2] === 'X'" />
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import TttCircle from './TttCircle';
import TttCross from './TttCross';
import { playTtt, checkEndOfGame, computeOctalBoard } from './../lib/tictactoe';

export default {
  name: 'TttTicTacToeGame',

  components: {
    TttCircle,
    TttCross,
  },

  props: {
    strategies: {
      type: Object,
      required: true,
    },
  },

  data: () => ({
    displayed: false,
    gameState: {
      game: [['', '', ''], ['', '', ''], ['', '', '']],
      currentPlayer: 'X',
      winner: '',
      gameEnded: false,
      status: '',
      isHumanPlaying: false,
      lastMove: {},
      ttt: {
        board: 0,
        history: [],
        turn: 0,
        ai: {
          X: null,
          O: null,
        },
      },
    },
    cellsToAnimate: [['', '', ''], ['', '', ''], ['', '', '']],
    isBoardShaking: false,
  }),

  mounted() {
    this.play();
  },

  methods: {
    async play() {
      const toPlay = await playTtt(
        this.gameState,
        this.strategies,
        this.animateCells,
        this.shakeBoard
      );

      console.log('To play');
      console.log(toPlay);

      if (toPlay !== null) {
        const { rowIndex, colIndex } = toPlay;
        setTimeout(() => {
          this.applyPlay(rowIndex, colIndex);
        }, 1000);
      } else {
        // Allow human to play
        this.gameState.isHumanPlaying = true;
      }
    },

    humanPlay(row, col) {
      this.gameState.isHumanPlaying = false;
      this.applyPlay(row, col);
    },

    applyPlay(row, col) {
      this.updateTtt(row, col);
      this.gameState.lastMove = { rowIndex: row, colIndex: col };
      Vue.set(this.gameState.game[row], col, this.gameState.currentPlayer);
      this.gameState.currentPlayer = this.gameState.currentPlayer === 'X' ? this.gameState.currentPlayer = 'O' : this.gameState.currentPlayer = 'X';
      checkEndOfGame(
        this.gameState,
        this.strategies,
        this.animateCells,
        this.shakeBoard
      );
      if (!this.gameState.gameEnded) {
        this.play();
      }
    },

    updateTtt(row, col) {
      const square = 3 * row + col;
      const piece = this.gameState.currentPlayer === 'X' ? 1 : 3;

      this.gameState.ttt.history.push(this.gameState.ttt.board);
      this.gameState.ttt.board = computeOctalBoard(this.gameState.ttt.board, square, piece);
    },

    animateCells(cells) {
      this.cellsToAnimate = [['', '', ''], ['', '', ''], ['', '', '']];

      for (let i = 0; i < cells.length; i++) {
        setTimeout(() => {
          Vue.set(this.cellsToAnimate[cells[i][0]], cells[i][1], '✨');
        }, (i + 1) * 300);
      }
    },

    shakeBoard() {
      this.isBoardShaking = true;

      setTimeout(() => {
        this.isBoardShaking = false;
      }, 1000);
    },

    finish() {
      this.$emit('game-finished');
    },
  },
};
</script>

<style lang="scss" scoped>
@import './../assets/styles/ttt-styles.scss';

#ttt-container {
  align-items: center;
  background-color: $primary-color;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 16px;

  #ttt {
    border-collapse: collapse;
    position: relative;

    &.shake-it {
      animation: shake 500ms linear 300ms;
    }

    &.disabled {
      pointer-events: none;
    }

    @keyframes shake {
      0% {
        transform: rotate(0deg);
      }
      16% {
        transform: rotate(10deg);
      }
      33% {
        transform: rotate(-10deg);
      }
      50% {
        transform: rotate(10deg);
      }
      66% {
        transform: rotate(-10deg);
      }
      84% {
        transform: rotate(10deg);
      }
      100% {
        transform: rotate(0deg);
      }
    }

    .mask {
      bottom: 0;
      left: 0;
      position: absolute;
      right: 0;
      top: 0;
      transform-origin: center center;

      svg {
        animation: hide-when-finish $appear-duration ease-out;
        opacity: 0;

        circle {
          animation: exploding-circle $appear-duration ease-out;
          stroke: $primary-color;
        }

        @keyframes exploding-circle {
          0% {
            opacity: 1;
            stroke-width: 130px;
          }
          99% {
            opacity: 1;
            stroke-width: 1px;
          }
          100% {
            opacity: 0;
            stroke-width: 0;
          }
        }
      }
    }

    .results {
      animation: slide-down 1800ms ease-out;
      background-color: $primary-color;
      bottom: 0;
      color: $text-color;
      font-family: 'Lato', sans-serif;
      left: 0;
      position: absolute;
      right: 0;
      text-align: center;
      top: 0;
      z-index: 1;

      button {
        margin-top: 16px;
      }
    }

    @keyframes slide-down {
      0% {
        opacity: 0;
        transform: translateY(-200%);
      }
      90% {
        opacity: 0;
        transform: translateY(-100%);
      }
      100% {
        transform: translateY(0);
      }
    }

    td > * {
      animation: hide-during-loading $appear-duration;
    }

    @keyframes remove-when-finish {
      0% {
        z-index: 1;
      }
    99% {
        z-index: 1;
      }
    }
    @keyframes hide-when-finish {
      0% {
        opacity: 1;
      }
      99% {
        opacity: 1;
      }
      100% {
        opacity: 0;
      }
    }

    @keyframes hide-during-loading {
      0% {
        visibility: hidden;
      }
      99% {
        visibility: hidden;
      }
      100% {
        visibility: visible;
      }
    }

    &:hover {
      .line1,
      .line2 {
        animation: line 0.2s ease 0.45s forwards;
      }
    }

    td {
      border: 4px solid;
      border-color: $text-color;
      height: $cell-size;
      position: relative;
      width: $cell-size;

      &.filled {
        pointer-events: none;
      }

      &.animated {
        animation: youhou 300ms ease-out;
        opacity: 1;
        transform: scale(1.5);
      }

      @keyframes youhou {
        0% {
          transform: scale(1);
        }
        50% {
          transform: scale(2);
        }
        100% {
          transform: scale(1.5);
        }
      }

      & > * {
        left: 8px;
        position: absolute;
        top: 8px;
      }
    }

    tr {
      td:first-child {
        border-left: none;
      }

      td:last-child {
        border-right: none;
      }
    }

    #row-1 td {
      border-top: none;
    }

    #row-3 td {
      border-bottom: none;
    }
  }
}
</style>
