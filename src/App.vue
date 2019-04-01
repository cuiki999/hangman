<template>
  <div id="app">
    <h1>HANGMAN</h1>
    <span class="streak">Streak: {{ streak }}&nbsp;&nbsp;|&nbsp;&nbsp;Best: {{ best }}</span>
    <img class="cog" title="Settings" src="./assets/img/cog.png" />
    <div id="paper">
      <img class="paper" src="./assets/img/grey-tape.png" />
      <div id="hangman-pics">
        <!-- :src is short for v-bind:src -->
        <img :src="require(`./assets/img/${imgNumber}.png`)" />
      </div>
      <div v-for="compLetter in compLetters" class="comp-letters">
        <span v-html="compLetter"></span>
      </div>
    </div>

    <div id="user-console">
      <div class="user-letters" v-for="userLetter in userLetters.slice(0,9)" @click="userLetter.hex === 1 ? guessLetter(userLetter.letter) : ''">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`./assets/img/hex${userLetter.hex}.png`)" />
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(9,17)" @click="userLetter.hex === 1 ? guessLetter(userLetter.letter) : ''">
        <span>{{ userLetter.letter }}</span>
        <img class="hexagon" :src="require(`./assets/img/hex${userLetter.hex}.png`)" />
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(17,26)" @click="userLetter.hex === 1 ? guessLetter(userLetter.letter) : ''">
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
      imgNumber: 0,
      word: '',
      userLetters: [],
      compLetters: [],
      allLetters: [],
      difficult: false,
      startingLettersOn: false,
      streak: 0,
      best: 0,
      resetting: false
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
          this.reset('You lose');
          this.userLetters.hex = 2;            
        }
        else if (this.imgNumber < 10) {
          this.imgNumber++;
        } else {
          this.imgNumber = 11;
          this.reset('You lose');
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
        this.reset('You won!');          
      }
    },
    reset(message) {
      this.resetting = true;
      for (let i = 0; i < this.word.length; i++) {
        if (this.compLetters[i] === '&nbsp;') {
          this.compLetters[i] = this.word[i].toUpperCase();
        }        
      }
      if (message === 'You won!') {
        this.streak += 1;
        if (this.streak > this.best) {
          this.best = this.streak;
          localStorage.setItem('best', this.streak);
        }
      } else if (message === 'You lose') {
        this.streak = 0;
      }
      setTimeout(() => {
        this.resetting = false;
        this.imgNumber = 0;
        this.userLetters = [];
        this.compLetters = [];
        this.generateUserArray();
        this.comeUpWithWord();
        alert(message);
      }, 1000)
    },
    onkey(event) {
      if (event.keyCode > 64 && event.keyCode < 91 && this.resetting === false) {
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

.cog {
  position: absolute;
  max-width: 50px;
  cursor: pointer;
  top: 60px;
  right: 15px;
}

#paper {
  text-align: center;
  margin-top: -20px;
  //border: 1px solid red;
  margin-bottom: -120px;

  .paper {
    width: 90%;
  }
}

#hangman-pics img {
  width: 120px;
  position: relative;
  top: -260px;
}

.comp-letters {
  font-family: 'Neucha', cursive;
  // font-weight: 700;
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

#user-console {
  display: block;
  text-align: center;
  margin-top: 30px;

  .user-letters {
  font-family: 'Crimson Text', serif;
  color: #fff;
  font-size: 2rem;
  width: 4.5rem;
  cursor: pointer;
  font-weight: bold;
  display: inline-block;
  margin-top: -26px;

    span {
      z-index: 100;
      position: relative;
      min-width: 3rem;
      text-align: center;
      top: 18px;
    }

    .hexagon {
      position: relative;
      margin-top: -80px;
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

</style>
