<template>
  <div id="app">
    <h1>HANGMAN</h1>
    <span class="streak">Streak: {{ streak }}&nbsp;&nbsp;|&nbsp;&nbsp;Best: {{ best }}</span>
    <div class= "cog-container" title="Settings"></div>
    <div id="paper">      
      <img class="paper-img" src="./assets/img/grey-tape.png" />
      <div id="hangman-pics">
        <!-- :src is short for v-bind:src -->
        <img :src="require(`./assets/img/${imgNumber}.png`)" />
      </div>
      <div v-for="compLetter in compLetters" class="comp-letters">
        <span v-html="compLetter"></span>
      </div>
      <div id="endGame" v-show="displayMessage">
        <span>{{ gameEndMessage }}</span>
      </div>
    </div>

    <div id="user-console">
      <div class="user-letters" v-for="userLetter in userLetters.slice(0,9)" @click="userLetter.hex === 1 && !resetting ? guessLetter(userLetter.letter) : ''">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`./assets/img/hex${userLetter.hex}.png`)" />
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(9,17)" @click="userLetter.hex === 1 && !resetting ? guessLetter(userLetter.letter) : ''">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`./assets/img/hex${userLetter.hex}.png`)" />
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(17,26)" @click="userLetter.hex === 1 && !resetting ? guessLetter(userLetter.letter) : ''">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`./assets/img/hex${userLetter.hex}.png`)" />
      </div> 
    </div>

    <br><br><br><br><br><br><br><br><br><br>
    <div>
      <p>Number of failed attempts: 
        <span class="setting" v-bind:class="{ selected: !difficult }" @click="difficult = false; reset('Resetting...')">10</span> / 
        <span class="setting" v-bind:class="{ selected: difficult }" @click="difficult = true; reset('Resetting...')">5</span>
      </p>
      <p>Start with one letter displayed: 
        <span class="setting" v-bind:class="{ selected: startingLettersOn }" @click="startingLettersOn = true; reset('Resetting...')">yes</span> / 
        <span class="setting" v-bind:class="{ selected: !startingLettersOn }" @click="startingLettersOn = false; reset('Resetting...')">no</span>
      </p>
    </div>
  </div>
</template>

<script>
import Wordlist from './static/wordlist.json';

export default {
  name: 'app',
  components: {

  },
  data () {
    return {
      streak: 0,
      best: 0,
      difficult: false,
      startingLettersOn: false,
      imgNumber: 0,
      word: '',
      userLetters: [],
      compLetters: [],
      allLetters: [],
      resetting: false,
      gameEnd: true,
      displayMessage: false,
      gameEndMessage: '',
    }
  },
  components: {

  },
  methods: {
    comeUpWithWord() {
      let random = Math.floor(Math.random() * Wordlist.nouns.length);
      this.word = Wordlist.nouns[random];
      if (this.word.length < 15) {
        for (let i = 0; i < this.word.length; i++) {
          this.compLetters.push("&nbsp;");     
          if (!/[a-zA-Z]/.test(this.word[i])) {
            this.compLetters[i] = this.word[i];
          }
        }
        if (this.startingLettersOn) {
          let randomLetter = Math.floor(Math.random() * this.word.length);
          let letter = this.word.charAt(randomLetter).toUpperCase();
          this.guessLetter(letter);
          this.userLetters[this.allLetters.indexOf(letter)].grey = true;
        }
      } else {
        this.comeUpWithWord();
      }      
    },
    guessLetter(letter) {
      // see which index a letter is at so its hexagon could be turned orange or grey
      let userIndex = this.allLetters.indexOf(letter);
      if (this.word.toUpperCase().indexOf(letter) !== -1) {
        // an array of all the indexes of a certain letter, because the letter can appear more than once within a word
        let indexes = [];
        this.userLetters[userIndex].hex = 2; 
        for (let i = 0; i < this.word.length; i++) {
          if (this.word.toUpperCase()[i] === letter) {
            indexes.push(i);
            for (let j = 0; j < indexes.length; j++) {
              let number = indexes[j];
              this.compLetters[number] = letter;  
              this.checkIfVictory();
            }
          }
        }
      } else {
        this.userLetters[userIndex].hex = 3;
        if (this.imgNumber === 5 && this.difficult === true) {
          this.imgNumber = 11;
          this.reset('YOU LOST');
          this.userLetters.hex = 2;            
        }
        else if (this.imgNumber < 10) {
          this.imgNumber++;
        } else {
          this.imgNumber = 11;
          this.reset('YOU LOST');
        }
      }
    },
    generateUserArray() {
      // generate the userLetters array by looping through ASCII codes: 65 = A etc.
      for (let i = 65; i < 91; i++) {
        let letter = String.fromCharCode(i);
        this.userLetters.push({'letter': letter, 'hex': 1});
        this.allLetters.push(letter);
      }
    },
    checkIfVictory() {
      if (this.compLetters.indexOf('&nbsp;') === -1) {
        this.reset('VICTORY!');        
      }
    },
    reset(message) {
      this.resetting = true;
      for (let i = 0; i < this.word.length; i++) {
        if (this.compLetters[i] === '&nbsp;') {
          this.compLetters[i] = this.word[i].toUpperCase();
        }        
      }
      if (message === 'VICTORY!') {
        this.streak += 1;
        if (this.streak > this.best) {
          this.best = this.streak;
          localStorage.setItem('best', this.streak);
        }
      } else if (message === 'YOU LOST') {
        this.streak = 0;
      }
      this.displayMessage = true;
      this.gameEndMessage = message;
      setTimeout(() => {
        this.displayMessage = false;
        this.resetting = false;
        this.imgNumber = 0;
        this.userLetters = [];
        this.compLetters = [];
        this.generateUserArray();
        this.comeUpWithWord();
      }, 2000);
    },
    onkey(event) {
      if (event.keyCode > 64 && event.keyCode < 91 && !event.ctrlKey && this.resetting === false) {
        let letter = String.fromCharCode(event.keyCode);
        let userIndex = this.allLetters.indexOf(letter);
        if (this.userLetters[userIndex].hex === 1) {  
          this.guessLetter(letter);
        }
      }
    }
  },
  created() {
    this.generateUserArray();
    this.comeUpWithWord();
    // retrieve best score from previous sessions
    localStorage.getItem('best') ? this.best = localStorage.getItem('best') : '';
  },
  mounted() {
    window.addEventListener('keyup', this.onkey);
  },
  beforeDestroy: function () {
    window.removeEventListener('keyup', this.onkey)
  }
}
</script>

<style lang="scss">

#app {
  background-color: #CDDEEB;
  width: 700px;
  height: 1000px;
  margin: 0 auto;
}

h1 {
  font-family: 'Crimson Text', serif;
  text-align: center;
  font-size: 50px;
}

.streak {
  font-family: 'Crimson Text', serif;
  font-size: 1.5rem;
  position: absolute;
  top: 10px;
  right: 15px;
}

.cog-container {
  position: absolute;
  width: 50px;
  height: 50px;
  cursor: pointer;
  top: 60px;
  right: 15px;
  background-image: url("./assets/img/cog.png");
  background-size: 50px;
    
  &:hover {
  -webkit-animation: cog-loop 0.5s infinite;
  -moz-animation: cog-loop 0.5s infinite;
  -o-animation: cog-loop 0.5s infinite;
  animation: cog-loop 0.5s infinite;    
  }
}

#paper {
  text-align: center;
  margin-top: -20px;
  margin-bottom: -120px;
  height: 350px;

  .paper-img {
    width: 90%;
  }

  #hangman-pics img {
    width: 120px;
    position: relative;
    top: -260px;
  }

  .comp-letters {
    font-family: 'Neucha', cursive;
    position: relative;
    top: -220px;
    font-size: 1.5rem;
    border-bottom: 2px solid black;
    margin-right: 0.3rem;
    margin-left: 0.3rem;
    min-width: 1.3rem;
    min-height: 1rem;
    padding: 0 0.3rem;
    display: inline-block;
    text-align: center;
  }

  #endGame {
    font-family: 'Neucha', cursive;
    font-size: 3rem;
    display: block;
    position: relative;
    top: -350px;
    left: 190px;
    transform: rotate(-20deg);
    max-width: 300px;
    -webkit-animation: fade-in 0.5s ease-in;
    -moz-animation: fade-in 0.5s ease-in;
    -o-animation: fade-in 0.5s ease-in;
    animation: fade-in 0.5s ease-in;   
    

    span {
      border: 4px solid #871205;
      color: #871205;
      line-height: 3rem;
      padding: 10px;
      padding-bottom: 0;
      background-color: #fff;
    }
  }
}

#user-console {
  display: block;
  text-align: center;
  margin-top: 180px;

  .user-letters {
    display: inline-block;
    font-family: 'Crimson Text', serif;
    font-size: 2rem;
    font-weight: bold;
    color: #fff;
    margin-top: -26px;
    width: 4.5rem;
    cursor: pointer;

    span {
      position: relative;
      text-align: center;
      min-width: 3rem;
      top: 18px;
      z-index: 100;
    }

    .hexagon {
      position: relative;
      margin-top: -80px;
    }
  }
}

#game-end {
  opacity: 0;
  width: 300px;
  height: 150px;
  margin-top: -75px;
  margin-left: -150px;
  background: #7DAACB;
  border: 1px solid black;
  position: fixed;
  z-index: 100;
  top: 35%;
  left: 50%;
  border-radius: 5px;
  box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);

  .inner-frame {
    width: 300px;
    height: 150px;
    top: 5px;
    left: 0px;
    border: 5px solid #fff;
    border-radius: 5px;
    text-align: center;
    padding: 30px;
    box-sizing: border-box;
  
    span {
      font-family: 'Crimson Text', serif;
      font-size: 1.5rem;
    }

    button {
      font-family: 'Crimson Text', serif;
      margin-top: 30px;
      background-color: #686968;
      color: #fff;
      border: 3px solid #DBC147;
      width: 70px;
      height: 40px;
    }
  }
}



.inactive {
  color: grey;
}

.setting {
  cursor: pointer;
}

.selected {
  font-weight: bold;
  color: orange;
}

@keyframes cog-loop {
  0% {
    background-image: url("./assets/img/cog.png");
  }
  50% {
    background-image: url("./assets/img/cog-turn.png");
  }
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

</style>
