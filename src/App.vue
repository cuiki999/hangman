<template>
  <div id="app">
    <h1>Hangman</h1>
    <div id="hangman-pics">
      <img :src="require(`./assets/img/${imgNumber}.png`)" />
    </div>
    <p v-on:click="getWord">I choose: {{ word }}</p>
    <!--<input v-model="animalInput" v-on:keydown.enter="enterAnimal" type="text" />-->
  </div>
</template>

<script>


export default {
  name: 'app',
  data () {
    return {
      imgNumber: 5,
      word: '',
      wordArray: ["dog", "donkey", "house", "hangman"]
    }
  },
  components: {

  },
  methods: {
    getWord: function() {
      let random = Math.floor(Math.random() * 4)
      this.word = this.wordArray[random]
    }
  }
}
</script>

<style>
#hangman-pics img {
  width: 100px;
}
</style>
