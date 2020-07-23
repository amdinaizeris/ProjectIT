<template>
  <div>
    <div class="game-board">

      <div class="game-board__register" v-show="marks !== 'full'">
        <input type="text" placeholder="Vardas" v-model="playerData.name" >
        <button v-for="(mark, index) in marks" :key="index" @click="markSelected(mark)"
        :class="{'game-board__register-btn--active' : playerData.mark === mark}"
        class="game-board__register-btn"
        >
          {{mark}}
        </button>
        <button class="game-board__register-createBtn" @click="createUser">Pradėti</button>
      </div>



      <div class="game-board__info">
        <div class="game-board__info-msgContent" v-if="gameStat === 'OFF'">
          {{msg}}
        </div>
        <div v-if="gameStat === 'ON'" class="game-board__info-board">


          <div class="game-board__info-board__descriptions">
            <div class="subtitle">Žaidėjas</div>
            <div class="name">{{players[0].name}}</div>
            <div class="name">{{players[1].name}}</div>
          </div>



          <div class="game-board__info-board__stats">
            <div class="">
              <span class="subtitle" style="margin-right: 10px;">W</span>
              <span class="subtitle" >L</span>
            </div>
            <div class="">
              <span style="font-size: 24px; font-weight: 600; margin-right: 10px;">{{this.players[0].ranking.win}}</span>
              <span style="font-size: 24px; font-weight: 600;">{{this.players[0].ranking.losses}}</span>
            </div>
            <div class="">
              <span style="font-size: 24px; font-weight: 600; margin-right: 10px;">{{this.players[1].ranking.win}}</span>
              <span style="font-size: 24px; font-weight: 600;">{{this.players[1].ranking.losses}}</span>
            </div>
          </div>

          <div class="game-board__info-board__turn">
            <div class="subtitle">Eilė</div>
            <div class="mark">{{this.players[turn].mark}}</div>
          </div>

        </div>
      </div>

    </div>


    <div class="tictactoe-grid" >
      <div class="A0 square" @click.self="toMark('A', '0', turn)">
        <span >{{tictacBoard.A[0]}}</span>
      </div>
      <div class="A1 square" @click.self="toMark('A', '1', turn)">
        <span >{{tictacBoard.A[1]}}</span>
      </div>
      <div class="A2 square" @click.self="toMark('A', '2', turn)">
        <span>{{tictacBoard.A[2]}}</span>
      </div>
      <div class="B0 square" @click.self="toMark('B', '0', turn)">
        <span>{{tictacBoard.B[0]}}</span>
      </div>
      <div class="B1 square" @click.self="toMark('B', '1', turn)">
        <span>{{tictacBoard.B[1]}}</span>
      </div>
      <div class="B2 square" @click.self="toMark('B', '2', turn)">
        <span>{{tictacBoard.B[2]}}</span>
      </div>
      <div class="C0 square" @click.self="toMark('C', '0', turn)">
        <span>{{tictacBoard.C[0]}}</span>
      </div>
      <div class="C1 square" @click.self="toMark('C', '1', turn)">
        <span>{{tictacBoard.C[1]}}</span>
      </div>
      <div class="C2 square" @click.self="toMark('C', '2', turn)">
        <span>{{tictacBoard.C[2]}}</span>
      </div>
    </div>

    <transition name="modalAnimation">
      <Modal v-if="winner || draw === 9" :name="winner">
        <button class="btnModal" @click="resetGame(turn)">Pradėti iš naujo</button>
      </Modal>
    </transition>


  </div>
</template>

<script>
import { setTimeout } from 'timers';
import {checkingResult} from '@/functions/checkResults.js';
import Modal from './Modal';

class Player {
  constructor(name, mark, ranking) {
    this.name = name;
    this.mark = mark;
    this.ranking = {
      win: 0,
      losses: 0
    },
    this.historyMoves = []
  }
}

export default {
  name: 'TicTacToe',
  components: {
    Modal
  },
  data() {
    return {
      players: [],
      playerData: {
        name: '',
        mark: null
      },
      marks: ['X', 'O'],
      turn: 0,
      tictacBoard: {
        A: ['', '', ''],
        B: ['', '', ''],
        C: ['', '', '']
      },
      msg: null,
      gameStat: 'OFF',
      winner: '',
      draw: 0,
    }
  },
  created() {
    this.markSelected('X')
  },
  methods: {
    createUser() {
      let regexSpaces = /^\S+(?: \S+)*$/;

      if(this.players.length < 2 && this.playerData.name.match(regexSpaces) && this.playerData.mark) {

        let { name, mark } = this.playerData
        let player = new Player(name, mark, {win: 0, losses: 0})

        this.players.push(player);
        this.playerData.name = '';

        this.marks.splice(this.marks.indexOf(this.playerData.mark), 1)
        this.playerData.mark = null;
      }

      else {
        this.msg = 'Pasirinkite X arba 0'
      }

    },
    markSelected(mark) {
      this.playerData.mark = mark
    },
    toMark(letter, number, turnOfPlayer) {

      if(this.gameStat === 'ON') {

        this.draw += 1

        let currentPlayer = this.players[turnOfPlayer]
        let rowArray = this.tictacBoard[letter];
        let gridSquare = this.tictacBoard[letter][number]


        if(!gridSquare) {

          currentPlayer.historyMoves.push(`${letter+number}`)

          this.$set(rowArray, number, currentPlayer.mark)


          document.querySelector(`.${letter+number}`).firstChild.classList.add('animate');


          let result = checkingResult(currentPlayer)

          if(result) {
            currentPlayer.ranking.win += 1
            this.winner = currentPlayer.name

            this.turn === 0 ? this.turn = 1 : this.turn = 0
            this.players[this.turn].ranking.losses += 1
          }
          else if(this.draw === 9 && !this.winner) {
            console.log('draw!')
          }
          else {
            this.changeTurn()
          }
        }
      }
    },
    changeTurn() {
      switch(this.turn) {
        case 0:
          this.turn = 1;
          break;
        case 1:
          this.turn = 0;
      }
    },
    resetGame(turn) {
      let squares = document.querySelectorAll('.square');
      squares.forEach(square => {
        square.firstChild.classList.remove('animate')
      })
      let newTictacBoard = {
        A: ['', '', ''],
        B: ['', '', ''],
        C: ['', '', '']
      }
      this.tictacBoard = newTictacBoard;
      this.draw = 0;
      this.winner = '';
      this.players.forEach(player => {
        player.historyMoves = []
      })
    }
  },
  watch: {
    marks() {
      if(this.marks.length === 0) {
        this.marks = 'full';
        this.gameStat = 'ON'
      }
    },
    msg() {
      if(this.msg) {
        setTimeout(() => {

          this.msg = ''
        }, 1500)
      }
    }
  }
}
</script>


<style scoped lang="scss">
.tictactoe-grid {
  background: rgba(205, 43, 35, 0.89);
  width: 100vw;
  height: 50vh;
  padding: 0px 1%;
  @media only screen and (min-width: 400px) {
    width: 350px;
    height: 350px;
  }
  display: grid;
  grid-template-columns: 33.33% 33.33% 33.33%;
  grid-template: 33.33% 33.33% 33.33%;
  .square {
    width: 100%;
    height: 100%;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    span {
      position: absolute;
      font-size: 34px;
      font-weight: 900;
      transition: .2s;
      transform: scale(0.1);
    }
    .animate {
      transform: scale(1);
    }
  }
  .square:nth-child(1) {
    border-bottom: 4px solid black;
    border-right: 4px solid black;
  }
  .square:nth-child(2) {
    border-bottom: 4px solid black;
  }
  .square:nth-child(3) {
    border-bottom: 4px solid black;
    border-left: 4px solid black;
  }
  .square:nth-child(4) {
    border-bottom: 4px solid black;
    border-right: 4px solid black;
  }
  .square:nth-child(5) {
    border-bottom: 4px solid black;
  }
  .square:nth-child(6) {
    border-bottom: 4px solid black;
    border-left: 4px solid black;
  }
  .square:nth-child(7) {
    border-right: 4px solid black;
  }
  .square:nth-child(9) {
    border-left: 4px solid black;
  }
}
.game-board {
  background: rgba(205,205,205,0.89);
  width: 100vw;
  height: 150px;
  @media only screen and (min-width: 400px) {
    width: 350px;
    height: 150px;
  }
  display: flex;
  flex-direction: column;
  padding: 0px 1%;
  margin-bottom: 40px;
  &__register {
    width: 100%;
    height: 60px;
    display: flex;
    justify-content: flex-start;
    input[type=text], select {
      width: 50%;
      height: 100%;
      font-size: 14px;
      padding: 5px;
      outline: none;
      border: 0.5px solid #000000;
      box-sizing: border-box;
    }
    &-btn {
      outline: none;
      width: 60px;
      font-size: 16px;
      border: none;
      background: transparent;
      &--active {
        background: #69bd21;
      }
    }
    &-createBtn {
      outline: none;
      width: 60px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 16px;
      background: #ff4d4d;
      border: none;
    }
  }
  &__info {
    width: 100%;
    height: 100%;
    &-msgContent {
      font-size: 14px;
      margin-top: 2rem;
    }
    &-board {
      width: 100%;
      height: 100%;
      display: grid;
      grid-template-columns: 2fr 1fr 1fr;
      &__descriptions {
        width: 100%;
        height: 100%;
        display: grid;
        grid-template-columns: 1fr;
        grid-template-rows: 30px 1fr 1fr;
        align-items: center;
        .name {
          font-size: 24px;
          font-weight: 600;
          overflow:hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
      &__stats {
        display: grid;
        grid-template-columns: 1fr;
        grid-template-rows: 30px 1fr 1fr;
        align-items: center;
        .subtitle {
          font-size: 16px;
          font-weight: 400;
          letter-spacing: 1px;
        }
      }
      &__turn {
        height: 100%;
        display: grid;
        grid-template-columns: 1fr;
        grid-template-rows: 30px 1fr;
        align-items: center;
        align-items: center;
        .mark {
          font-size: 34px;
          font-weight: 900;
        }
      }
    }
  }
}
.center {
  display: grid;
  justify-content: center;
}



.btnModal {
  border: none;
  width: 50%;
  height: 50px;
  background: #00E676;
  font-size: 16px;
  margin: auto;
  outline: none;
}



.modalAnimation-enter-active {
  animation: modalAnimation-in .5s;
}
.modalAnimation-leave-active {
  animation: modalAnimation-in .5s reverse;
}
@keyframes modalAnimation-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}
</style>
