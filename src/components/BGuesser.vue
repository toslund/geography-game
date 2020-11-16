<template>
  <div class="button-container">
        <h4 class="button-prompt"><slot></slot></h4>
        <div class="split">
        <button class="btn" v-bind:class="btnClass" v-for="(choice, idx) in formattedAnswers"
        :key=idx
        v-on:click="guessHandler(choice, idx)"
        :disabled="wrongGuesses.includes(idx)">
          {{choice[targetKey]}}
        </button>
        </div>
      </div>
</template>

<script>
/* eslint-disable max-len */
export default {
  name: 'BGuesser',
  props: {
    possibleAnswers: Array,
    correctAnswerIndex: Number,
    targetKey: String,
    btnClass: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      randomWrongIndexes: [],
      wrongAnswersObjs: Array,
      formattedAnswers: Array,
      wrongGuesses: [],
    };
  },
  computed: {
    correctAnswer() {
      return this.possibleAnswers[this.correctAnswerIndex];
    },
  },
  methods: {
    resetRound() {
      this.randomWrongIndexes = this.getRandomIndexes(this.possibleAnswers.length, this.correctAnswerIndex, 5);
      const wrongAnswers = this.possibleAnswers.filter((n, idx) => {
        if (this.randomWrongIndexes.includes(idx)) { return true; }
        return false;
      });
      const allPossibleAnswers = wrongAnswers.concat([this.correctAnswer]);
      this.formattedAnswers = this.shuffle(allPossibleAnswers);
    },
    getRandomIndexes(totalIndexes, excludeIndex, n) {
      const tempArray = Array.from(Array(totalIndexes).keys());
      tempArray.splice(excludeIndex, 1);
      return this.shuffle(tempArray).slice(0, n);
    },
    guessHandler(guess, guessIndex) {
      if (guess === this.correctAnswer) {
        this.$emit('chose', true);
      } else {
        this.wrongGuesses.push(guessIndex);
        this.$emit('chose', false);
      }
    },
    shuffle(array) {
      const inputArray = array;
      let currentIndex = inputArray.length;
      let temporaryValue;
      let randomIndex;
      // While there remain elements to shuffle...
      while (currentIndex !== 0) {
        // Pick a remaining element...
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;

        // And swap it with the current element.
        temporaryValue = inputArray[currentIndex];
        inputArray[currentIndex] = inputArray[randomIndex];
        inputArray[randomIndex] = temporaryValue;
      }
      return inputArray;
    },
  },
  mounted() {
    this.resetRound();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.button-container{
  text-align: center;
  width: 95%;
  margin: auto;
}

.button-prompt{
  margin: 10px 0px 0px 0px;
}
</style>
