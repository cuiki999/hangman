<template>
  <div id="app">
    <h1>HANGMAN</h1>
    <span class="streak">Streak: {{ streak }}&nbsp;&nbsp;|&nbsp;&nbsp;Best: {{ best }}</span>
    <div class= "cog-container" title="Settings" @click="$refs.settings.style.display = 'block'"></div>
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

    <div id="settings" ref="settings">
      <div id="settings-all">
        <div class="close" @click="updateHexes('close')">
          <span>X</span>
        </div>
        <div class="options">
          <p>Number of failed attempts:</p>
          <div v-on:click="changeDiffHex(0)">
            <span>10</span>
            <img class="option-hex" :src="require(`./assets/img/hex${settings.tdHex[0]}.png`)" />  
          </div>
          <div v-on:click="changeDiffHex(1)">
            <span>5</span>
            <img class="option-hex" :src="require(`./assets/img/hex${settings.tdHex[1]}.png`)" />
          </div>
        </div>
        <div class="options">
          <p>Start with one letter displayed:</p>
          <div @click="changeStartHex(0)">
            <span>Y</span>
            <img class="option-hex" :src="require(`./assets/img/hex${settings.tsHex[0]}.png`)" />
          </div>
          <div @click="changeStartHex(1)">  
            <span>N</span>
            <img class="option-hex" :src="require(`./assets/img/hex${settings.tsHex[1]}.png`)" />
          </div>
        </div>
        <br><br>
        <span class="warning">Applying the changes will end the current game.</span>
        <br><br>
        <button v-show="!settings.changedDiff && !settings.changedStart">APPLY CHANGES</button>
        <button class="activebutton" v-show="settings.changedDiff || settings.changedStart" @click="applyChanges(); updateHexes('apply')">APPLY CHANGES</button>
      </div>
    </div>

    <!--
    <p>Number of failed attempts: 
        <span class="setting" v-bind:class="{ selected: !difficult }" @click="difficult = false; reset('Resetting...')">10</span> / 
        <span class="setting" v-bind:class="{ selected: difficult }" @click="difficult = true; reset('Resetting...')">5</span>
      </p>
      <p>Start with one letter displayed: 
        <span class="setting" v-bind:class="{ selected: startingLettersOn }" @click="startingLettersOn = true; reset('Resetting...')">yes</span> / 
        <span class="setting" v-bind:class="{ selected: !startingLettersOn }" @click="startingLettersOn = false; reset('Resetting...')">no</span>
      </p>
    -->

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
      word: '',
      compLetters: [],
      userLetters: [],
      allLetters: [],
      imgNumber: 0,
      streak: 0,
      best: 0,
      resetting: false,
      displayMessage: false,
      gameEndMessage: '',
      settings: {
        // (temporary) difficulty hex & (temporary) starting-letter-on hex
        // temporary is what is visible in the settings menu, non-temporary is what actually gets saved
        // if you press 'apply', temporary becomes non-temporary. If you press X, it's the other way around
        // 2 means selected (orange), 3 means not selected (gray)
        dHex: [2, 3],
        tdHex: [2, 3],
        sHex: [3, 2],
        tsHex: [3, 2],
        changedDiff: false,
        changedStart: false,
        difficult: false,
        startingLettersOn: false,
        madeChanges: false,
      }
    }
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
        if (this.settings.startingLettersOn) {
          let randomLetter = Math.floor(Math.random() * this.word.length);
          let letter = this.word.charAt(randomLetter).toUpperCase();
          this.guessLetter(letter);
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
        if (this.imgNumber === 5 && this.settings.difficult === true) {
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
    },
    changeDiffHex(hex) {
      hex === 0 ? this.settings.tdHex = [2, 3] : this.settings.tdHex = [3, 2];
      this.settings.tdHex[0] == this.settings.dHex[0] ? this.settings.changedDiff = false : this.settings.changedDiff = true;
    },
    changeStartHex(hex) {
      hex === 0 ? this.settings.tsHex = [2, 3] : this.settings.tsHex = [3, 2];
      this.settings.tsHex[0] == this.settings.sHex[0] ? this.settings.changedStart = false : this.settings.changedStart = true;
    },
    updateHexes(option) {
      this.$refs.settings.style.display = 'none'; 
      this.settings.changedDiff = false;
      this.settings.changedStart = false;
      if (option === 'close') {
        this.settings.tdHex = this.settings.dHex;
        this.settings.tsHex = this.settings.sHex; 
      } else if (option === 'apply') {
        this.settings.dHex = this.settings.tdHex;
        this.settings.sHex = this.settings.tsHex;
      }      
    },
    applyChanges() {
      let hexArray = [this.settings.tdHex, this.settings.tsHex];
      // Number of failed attempts -> 10 is selected
      if (hexArray[0][0] === 2) {
        this.settings.difficult = false;
      } else if (hexArray[0][0] === 3) {
        this.settings.difficult = true;
      }
      // Start with one letter displayed -> Y is selected
      if (hexArray[1][0] === 2) {
        this.settings.startingLettersOn = true;
      } else if (hexArray[1][0] === 3) {
        this.settings.startingLettersOn = false;
      }
      this.reset('RESETTING...');
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
  font-family: 'Crimson Text', serif;
}

h1 {
  text-align: center;
  font-size: 50px;
}

.streak {
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
    font-size: 2rem;
    font-weight: bold;
    color: #fff;
    margin-top: -24px;
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
      font-size: 1.5rem;
    }

    button {
      margin-top: 30px;
      background-color: #686968;
      color: #fff;
      border: 3px solid #DBC147;
      width: 70px;
      height: 40px;
    }
  }
}

#settings {
  display: none;
  position: absolute;
  width: 400px;
  height: 430px;
  top: 60px;
  right: 15px;
  border: 4px solid #fff;
  box-shadow: 0.31rem 0.31rem 0.31rem 0 rgba(170, 170, 170, 0.8);
  color: #fff;
  background-color: #686968;
  z-index: 500;
  text-align: center;
  -webkit-animation: grow 0.3s ease-in;
  -moz-animation: grow 0.3s ease-in;
  -o-animation: grow 0.3s ease-in;
  animation: grow 0.3s ease-in;

  #settings-all {
    -webkit-animation: appear 0.5s linear;
    -moz-animation: appear 0.5s linear;
    -o-animation: appear 0.5s linear;
    animation: appear 0.4s linear;  
  }

  .close {
    font-family: 'Neucha', cursive;
    position: relative;
    left: 360px;
    top: 13px;
    width: 30px;
    font-weight: bold;
    font-size: 2rem;
    cursor: pointer;
    margin-bottom: 40px;
  }

  .options {
    text-align: center;
    margin-top: 30px;
    display: block;
    margin-bottom: -50px;
    font-size: 1.3rem;

    div {
      display: inline-block;
      text-align: center;
      //border: 1px solid red;
      width: 40px;
      cursor: pointer;
      height: 35px;
      margin: 0 5px 0 5px;

      .option-hex {
        width: 40px;
        //display: inline-block;
        position: relative;
        top: -37px;


        //margin-right: 20px;
      }

      span {
        //display: inline-block;
        //text-align: center;
        position: relative;
        //margin-right: -30px;
        z-index: 200;

      }

    }

    
  }

  .warning {
    font-family: 'Neucha', cursive;
    //font-size: 1rem;
  }

  button {
    background-color: #686968;
    margin-top: -10px;
    border: 4px solid #fff;
    pointer-events: none;
    padding: 10px;
    font-weight: bold;
    font-family: 'Crimson Text', serif;
    font-size: 1.2rem;
    color: #fff;
  }

  .activebutton {
    background-color: #DBC147;
    cursor: pointer;
    pointer-events: auto;
  }
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
  49% {
    background-image: url("./assets/img/cog.png");
  }
  50% {
    background-image: url("./assets/img/cog-turn.png");
  }
  99% {
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

@keyframes grow {
  0% {
    width: 0%;
    height: 0%;
  }
  100% {
    width: 400px;
    height: 430px;
  }
}

@keyframes appear {
  0% {
    opacity: 0;
  }
  75% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

</style>
