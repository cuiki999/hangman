<template>
  <div id="app">
    <h1>Hangman</h1>
    <div id="hangman-pics">
      <!-- :src is short for v-bind:src -->
      <img :src="require(`./assets/img/${imgNumber}.png`)" />
    </div>
    <div v-for="compLetter in compLetters" class="comp-letters">
      <span v-html="compLetter"></span>
    </div>
    <br><br>
    <div class="user-letters" v-for="userLetter in userLetters" @click="!userLetter.grey ? guessLetter(userLetter.letter) : ''; userLetter.grey = true">
      <span v-bind:class="{ inactive: userLetter.grey }">{{ userLetter.letter }}</span>
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
      <p>Streak: {{ streak }}</p>
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
      startingLettersOn: false,
      streak: 0
    }
  },
  components: {

  },
  methods: {
    comeUpWithWord() {
      let random = Math.floor(Math.random() * Wordlist.nouns.length);
      this.word = Wordlist.nouns[random];
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
          this.streak = 0;
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
        this.streak++;          
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

<style>
#hangman-pics img {
  width: 100px;
}

.user-letters {
  font-size: 2rem;
  padding: 1rem;
  cursor: pointer;
  font-weight: bold;
  display: inline-block;
  margin-right: 1rem;
}

.inactive {
  color: grey;
}

.comp-letters {
  font-size: 2rem;
  border-bottom: 3px solid black;
  margin-right: 1rem;
  margin-left: 1rem;
  min-width: 2rem;
  min-height: 2rem;
  background-color: orange;
  padding: 0 0.7rem;
  display: inline-block;
  text-align: center;
}

.setting {
  cursor: pointer;
}

.selected {
  font-weight: bold;
  color: orange;
}
</style>
