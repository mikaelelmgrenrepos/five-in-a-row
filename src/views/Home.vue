<template>
  <div class="home">
    <transition name="fade" enter-active-class="delay" leave-active-class="delay">
      <div  v-if="!gameStarted" class="welcome text-center">
        <div style="margin-bottom: 80px;">
          <h3>5-in-a-row</h3>
          <p class="mb-3"><i>by Mikael Elmgren</i></p>
          <b-button class="mt-3" size="sm" squared @click="startGame" v-if="!gameStarted">Start game</b-button>
        </div>
      </div>
    </transition>
    <div v-if="gameStarted" class="mt-3">
      <div class="score-board">
        <h3>Score board</h3>
        <p>Player 1 - Player 2</p>
        <h4><span class="one">{{playerOneScore}}</span> - <span class="two">{{playerTwoScore}}</span></h4>
        <b-button v-b-modal.modal-sm size="sm" class="mt-3" variant="outline-danger" @click="showModal">
          Delete results</b-button>
          <b-modal ref="delete-modal" hide-footer title="You sure you want to delete all results?">
            <div class="d-block text-center">
              <h3>You can´t regret this, you know?</h3>
            </div>
            <b-button class="mt-3" variant="info" block @click="hideModal">Nahh, keep´em</b-button>
            <b-button class="mt-2" variant="danger" block @click="clearAllUserData">Yup, delete it!</b-button>
          </b-modal>
      </div>
      <div class="game-container">
        <h5 v-if="this.player == 'p-one'" class="player one">Player 1´s turn</h5>
        <h5 v-else class="player two">Player 2´s turn</h5>   
        <div class="game-grid" :key="componentKey">
          <div v-for="(col,index) of columns" :id="index" :key="index" :data-col="index" class="game-col">
            <div 
              v-for="(row,i) of rows" 
              :key="i" 
              :data-row="i" 
              :data-col="index" 
              class="game-row empty"
              @mouseover="$event.target.classList.add('mouse-over')"
              @mouseleave="$event.target.classList.remove('mouse-over')"
              @click="makeAMove($event, col, row)"
              >
            </div>
          </div>
        </div>
        <b-button class="btn btn-primary mt-3" size="sm" squared @click="restartGame">Restart</b-button> 
        <div class="mt-3">
          <b-alert show variant="danger" v-if="chooseAnother">Choose another spot, this one´s full</b-alert>
        </div>
      </div>
    </div>
    <transition name="fade" enter-active-class="delay" leave-active-class="delay">
      <div v-if="isGameOver" class="game-over-container">
        <div class="game-over">
          <h1>GAME OVER</h1>
          <h3 v-if="isGameOver == 'p-one'">Player 1 won!!</h3>
          <h3 v-else>Player 2 won!!</h3>
          <b-button size="sm" squared @click="restartGame">Play again?</b-button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "home",
  data() {
    return {
      gameStarted: false,
      player: 'p-one',
      playerData: 'empty',
      columns: 7,
      rows: 6,
      currentCol: null,
      currentRow: null,
      chooseAnother: false,
      isGameOver: false,
      playerOneScore: 0,
      playerTwoScore: 0,
      componentKey: 0
    }
  },
  mounted(){
    this.getUserData(); //getting ev. prev. scores
  },
  methods: {
    showModal() {
      this.$refs['delete-modal'].show();
    },
    hideModal() {
        this.$refs['delete-modal'].hide()
      },
    getUserData(){ // getting scores from localStorage (if there is any)
      let p1 = localStorage.getItem("p1");
      let p2 = localStorage.getItem("p2");
      (p1 != null) ? this.playerOneScore = p1 : this.playerOneScore = 0;
      (p2 != null) ? this.playerTwoScore = p2 : this.playerTwoScore = 0;
    },
    clearAllUserData(){ // removes all items in localStorage and ressetting the scores
      localStorage.removeItem("p1");
      localStorage.removeItem("p2");
      this.playerOneScore = 0;
      this.playerTwoScore = 0;
      this.hideModal();
    },
    startGame(){
      this.gameStarted = !this.gameStarted;
      document.title = 'Game is on!!'
    },
    restartGame(){ // resets everything on restart
      this.player = (this.player == 'p-one') ? this.player = 'p-two' : this.player = 'p-one';
      this.playerData = 'empty';
      this.currentCol = null;
      this.currentRow = null;
      this.chooseAnother = false;
      this.isGameOver = false;
      document.title = 'Game restarted';
      this.componentKey += 1; // is used to "reset" the grid (removing classes, etc)
    },
    findLastBlock(column){ // gets the last Block in column
      for (let j = column.length; j >= 0, j--;) {
        let el = column[j];
        this.currentCol = el.dataset.col;
        this.currentRow = el.dataset.row;
        if(el.classList.contains('empty')){
          return j; // returns the last cell with the class of empty
        } else {
          // maybe we want to do something here in the future?
        }
      }
    },
    makeAMove($event, col, row){ // when user clickes a column
      this.chooseAnother = false; 
      // let column = $event.path[1].children; // NOTE: this does NOT work in Firefox & Safari
      let column = event.composedPath()[1].children || event.path[1].children;
      let last = this.findLastBlock(column);
      if(last == undefined){ // in case the column is full an alert will come out
        this.chooseAnother = true;
      } else {
        if(this.player == 'p-one'){
          column[last].classList.add('p-one');
        } else {
          column[last].classList.add('p-two');
        }
        column[last].classList.remove('empty'); // after adding the player-class, removing the empty-class
        // checking if game over:
        this.isGameOver = this.checkIfGameOver(this.currentRow, this.currentCol);
        // switching player
        if(!this.isGameOver){ // only toggles when game is not over
          this.togglePlayer();  
        }        
    } 
    },
    togglePlayer(){
      // changing the player after placing brick
      this.player = (this.player == 'p-one') ? this.player = 'p-two' : this.player = 'p-one';
      document.title = (this.player == 'p-one') ? 'Player 1´s turn' : 'Player 2´s turn';
    },
    getBlock(col,row){
      // getting blocks lying next to guess
        let i = parseInt(col);
        let j = parseInt(row);
        if(document.getElementById(i)){
          let div = document.getElementById(i);
          if(j <= 5 && j >= 0){
            let colRowNode = document.getElementById(i).querySelectorAll('[data-row="'+j+'"]');
            let nodeClasses = colRowNode[0].getAttribute('class');
            if(nodeClasses.includes('p-one')){
              this.playerData = 'p-one';
              return this.playerData;
            } else if(nodeClasses.includes('p-two')){
              this.playerData = 'p-two';          
              return this.playerData;
            }
          }
        }
      },
     checkIfGameOver(newRow,newCol){ // the main function
      let that = this;
      function checkDirection(direction){ // is run multiple times with all the different directions
        let total = 0;
        // parsing into numbers or else the equation crashes
        let col = parseInt(newCol);
        let row = parseInt(newRow);
        // creates new "coordinates" to search in
        let i = col + direction.i;
        let j = row + direction.j;
        // puts the calulated data into surroundingBlock
        let surroundingBlock = that.getBlock(i,j);
        while (
          i >= 0 &&
          i <= that.columns &&
          j >= 0 &&
          j <= that.rows &&
          surroundingBlock === that.player ) {
            total++;
            i += direction.i;
            j += direction.j;
            surroundingBlock = that.getBlock(i,j);
        }
        return total;
      }
      function checkIfWon(a, b){
        // adding up the total of matched blocks
        let total = 1 + checkDirection(a) + checkDirection(b);
        if(total >= 5){
          // GAME OVER (saving scores into localStorage)
          (that.player === 'p-one') ? that.playerOneScore++ : that.playerTwoScore++;
          let pl1Score = localStorage.setItem("p1", that.playerOneScore);
          let pl2Score = localStorage.setItem("p2", that.playerTwoScore);
          return that.player;
        } else {
          return null;
        }
        return total;
      }
      function checkHorizontal(){
        return checkIfWon({i: -1, j: 0}, {i: 1, j: 0})
      }
      function checkVertical(){
        return checkIfWon({i: 0, j: -1}, {i: 0, j: 1})
      }
      function checkDiagonalOne(){
        return checkIfWon({i: 1, j: -1}, {i: 1, j: 1})
      }
      function checkDiagonalTwo(){
        return checkIfWon({i: -1, j: 1}, {i: -1, j: -1})
      }
      return checkVertical() || checkHorizontal() || checkDiagonalOne() || checkDiagonalTwo();
    }
  }
};
</script>
<style lang="scss">
#app {
  font-family: 'Alata', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: #ccc6c6d9;
}
//============ Transitions
.fade-enter-active, .fade-leave-active {
  transition: opacity .8s;
}
.fade-leave-active {
  transition: opacity .8s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transition: opacity .8s;
  opacity: 0;
}
.delay {
  animation-delay: 0.7s;
}
//============ Generell styling
.welcome {
  height: calc(100vh - 15px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  button {
    transition: 0.4s ease-in-out;
    color: #000000;
    padding: 10px 20px;
  }
  .btn-secondary {
  background-color: #6c757d5c;
  border: none;
  }
}
.home {
  height: 100vh;
  padding: 30px;
}
//============ Game container
.game-container {
  h5.player {
    font-family: 'Bebas Neue', cursive;
    padding: 10px 5px;
    color: whitesmoke;
    width: fit-content;
    padding: 10px 40px;
    margin: 10px auto 10px auto;
    border-radius: 25px;
  }
  h5.player.one {
    background-color: #3e6f3e;
  }
  h5.player.two {
    background-color: #e04f4f;
  }
  .game-grid {
    margin-top: 40px;
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
  .game-col {
    width: 64px;
    flex-direction: column;
  }
  .game-row {
    cursor: pointer;
    display: inline-block;
    transition: 0.5s;
    height: 60px;
    width: 60px;
    border-radius: 25%;
    margin: 2px;
    background-color: rgba($color: #000000, $alpha: 0.1)
  }
  .mouse-over {
    background-color: rgba($color: #000000, $alpha: 0.3);
  }
  .p-one {
    background-color: #3e6f3e;
  }
  .p-two {
    background-color: #e04f4f;
  }
} // End Game container
//============ Scoreboard
.score-board {
  width: auto;
  position: absolute;
  top: 10px;
  right: 10px;
   h4 {
     margin-top: 30px;
     font-family: 'Bebas Neue', cursive;
   }
  .one {
    background-color: #3e6f3e;
    color: whitesmoke;
    padding: 10px;
  }
  .two {
    background-color: #e04f4f;
    color: whitesmoke;
    padding: 10px;
  }
  .btn-sm, .btn-group-sm > .btn {
    padding: 0.3rem 0.5rem;
    font-size: 0.875rem;
    line-height: 1.2;
    border-radius: 0rem;
  }
}
//============ Modal
.modal-content {
  font-family: 'Bebas Neue', cursive;
  background-color: #dadada!important;
  button {
    font-family: 'Alata', sans-serif;
  }
}
//============ Game Over
.game-over-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vW;
  height: 100vh;
  background-color: rgba($color: #000000, $alpha: 0.8);
  color: whitesmoke;
  z-index: 99;
  .game-over {
    width: 40%;
    margin: auto;
    height: 50%;
    margin-top: calc(50vh - 25%);
    display: flex;
    flex-direction: column;
    justify-content: center;
    button {
      padding: 20px;
      margin-top: 50px;
      width: 50%;
      margin: 50px auto;
    }
  }
}
</style>