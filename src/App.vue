<template>
  <div id="app">
    <h1>HANGMAN</h1>
    <div id="paper">
      <img class="paper" src="./assets/img/big-paper.png" />
      <div id="hangman-pics">
        <!-- :src is short for v-bind:src -->
        <img :src="require(`./assets/img/${imgNumber}.png`)" />
      </div>
      <div v-for="compLetter in compLetters" class="comp-letters">
        <span v-html="compLetter"></span>
      </div>
    </div>
    <br><br>

    <div id="user-console">
      <div class="user-letters" v-for="userLetter in userLetters.slice(0,9)" @click="!userLetter.grey ? guessLetter(userLetter.letter) : ''; userLetter.grey = true">
        <span v-bind:class="{ inactive: userLetter.grey }">{{ userLetter.letter }}</span>
        <div class="hexagon"></div>
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(9,17)" @click="!userLetter.grey ? guessLetter(userLetter.letter) : ''; userLetter.grey = true">
        <span v-bind:class="{ inactive: userLetter.grey }">{{ userLetter.letter }}</span>
      </div>
      <br>
      <div class="user-letters" v-for="userLetter in userLetters.slice(17,26)" @click="!userLetter.grey ? guessLetter(userLetter.letter) : ''; userLetter.grey = true">
        <span v-bind:class="{ inactive: userLetter.grey }">{{ userLetter.letter }}</span>
      </div>
    </div>

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
      allLeters: [],
      difficult: false,
      startingLettersOn: false
    }
  },
  components: {

  },
  methods: {
    comeUpWithWord() {
      let random = Math.floor(Math.random() * Wordlist.nouns.length);
      this.word = Wordlist.nouns[random];
      if (this.word.length < 13) {
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
          this.userLetters[this.allLeters.indexOf(letter)].grey = true;
        }
      } else {
        this.comeUpWithWord();
      }
      
    },
    guessLetter(letter) {
      if (this.word.toUpperCase().indexOf(letter) !== -1) {
        // an array of all the indexes of a certain letter, because the letter can appear more than once within a word
        let indexes = [];
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
        if (this.imgNumber === 5 && this.difficult === true) {
          this.imgNumber = 11;
          this.reset('You lose');            
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
        this.userLetters.push({'letter': letter, 'grey': false});
        // make this array of letters so you can refer to it when making a letter turn gray in comeUpWithWord() for when startingLettersOn is set to true
        this.allLeters.push(letter);
      }
    },
    checkIfVictory() {
      if (this.compLetters.indexOf('&nbsp;') === -1) {
        this.reset('You won!');          
      }
    },
    reset(message) {
      for (let i = 0; i < this.word.length; i++) {
        if (this.compLetters[i] === '&nbsp;') {
          this.compLetters[i] = this.word[i].toUpperCase();
        }        
      }
      setTimeout(() => {
        alert(message);
        this.imgNumber = 0;
        this.userLetters = [];
        this.compLetters = [];
        this.generateUserArray();
        this.comeUpWithWord();
      }, 1000)
    }
  },
  created() {
  this.generateUserArray();
  this.comeUpWithWord();    
  }
}
</script>

<style lang="scss">

#app {
  background-color: #CDDEEB;
  width: 1000px;
  height: 1000px;
  margin: 0 auto;
}

h1 {
  font-family: 'Crimson Text', serif;
  text-align: center;
  font-size: 50px;
}

#paper {
  text-align: center;
  margin-top: -20px;
  //border: 1px solid red;
  margin-bottom: -200px;

  .paper {
    width: 60%;
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
}

.user-letters {
  font-family: 'Crimson Text', serif;
  font-size: 2rem;
  min-width: 3rem;
  cursor: pointer;
  font-weight: bold;
  display: inline-block;
  margin-right: 1rem;
  margin-bottom: 1rem;
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

.hexagon {
  position: relative;
  width: 40px; 
  height: 23.09px;
  background-color: #7eaacb;
  margin: 11.55px 0;
  border-left: 4px solid #ffffff;
  border-right: 4px solid #ffffff;
}

.hexagon:before,
.hexagon:after {
  content: "";
  position: absolute;
  z-index: 1;
  width: 28.28px;
  height: 28.28px;
  -webkit-transform: scaleY(0.5774) rotate(-45deg);
  -ms-transform: scaleY(0.5774) rotate(-45deg);
  transform: scaleY(0.5774) rotate(-45deg);
  background-color: inherit;
  left: 2.8579px;
}

.hexagon:before {
  top: -14.1421px;
  border-top: 4px solid #ffffff;
  border-right: 4px solid #ffffff;
}

.hexagon:after {
  bottom: -14.1421px;
  border-bottom: 4px solid #ffffff;
  border-left: 4px solid #ffffff;
}


</style>
