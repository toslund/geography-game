<!-- eslint-disable max-len -->
<template>
  <div class="main-container" v-if="capitals && capitalIndex">
      <div class="place-images">
      <CapitalImage class="" v-bind:locationCode="locationCode"/>
      </div>
      <div class="user-controls">
      <BGuesser
        v-if="guessingStep === 'GuessCountry'"
        v-bind:possibleAnswers="capitals"
        v-bind:correctAnswerIndex="capitalIndex"
        target-key="country"
        v-on:chose="onChoseCountry"
        btnClass="btn-accent-A">
        Can you guess the <span class="text-accent-A">country?</span>
      </BGuesser>
        <BGuesser
        v-if="guessingStep === 'GuessCapital'"
        v-bind:possibleAnswers="capitals"
        v-bind:correctAnswerIndex="capitalIndex"
        target-key="capital"
        v-on:chose="onChoseCapital"
        btnClass="btn-accent-B">
        What is the <span class="text-accent-B">Capital</span> of {{ capitals[capitalIndex]["country"] }}?
        </BGuesser>
        <div class="split">
          <ScoreCounter v-bind:score="score" title="Score:"/>
        </div>
        </div>
        <!-- <button @click="score+=100">+100 points</button>
        <button @click="score-=100">-100 points</button> -->
  </div>
</template>

<script>
import json from './components/json/country_capitals.json';
import CapitalImage from './components/CapitalImage.vue';
import BGuesser from './components/BGuesser.vue';
import ScoreCounter from './components/ScoreCounter.vue';

export default {
  name: 'App',
  components: {
    BGuesser,
    CapitalImage,
    ScoreCounter,
  },
  data() {
    return {
      capitals: json, // eslint-disable-line
      capitalIndex: null,
      baseUrl: process.env.VUE_APP_BASE_URL,
      guessingStep: null,
      score: 0,
      roundScore: 100,
      roundMaxScore: 100,
    };
  },
  computed: {
    locationCode() {
      if (this.capitals) {
        return this.capitals[this.capitalIndex].place_id;
      }
      return null;
    },
    prompt() {
      const prompt = `What is the capital of ${this.capitals[this.capitalIndex].country}`;
      return prompt;
    },
  },
  watch: {
    guessingStep(newVal) {
      if (newVal === 'finished') {
        this.resetRound();
      }
    },
  },
  methods: {
    onScoreChange(result) {
      this.score += result;
    },
    onChoseCountry(result) {
      console.log('chose country');
      console.log(result);
      if (result) {
        this.score += this.roundMaxScore;
        this.guessingStep = 'GuessCapital';
        return;
      }
      // User guessed the wrong region
      this.score -= 10;
      console.log(result);
    },
    onChoseCapital(result) {
      if (result) {
        this.score += this.roundMaxScore;
        this.roundScore = this.roundMaxScore;
        this.guessingStep = 'finished';
      }
      // User guessed the wrong city
      console.log(result);
      this.score -= 10;
    },
    resetRound() {
      console.log(this.capitals.length);
      this.capitalIndex = Math.floor(Math.random() * this.capitals.length);
      this.guessingStep = 'GuessCountry';
      console.log(this.capitalIndex);
    },
  },
  mounted() {
    console.log(this.capitals.length);
    this.capitalIndex = Math.floor(Math.random() * this.capitals.length);
    this.guessingStep = 'GuessCountry';
    console.log(this.capitalIndex);
  },
};
</script>

<style>
.main-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin: auto;
  width: 90%;
}

.place-images{
  margin: 0 auto;
}
.user-controls{
  margin: 0 auto;
}

@media screen and (min-width: 700px) {
    .main-container {
    flex-direction: row;
  }

  .place-images{
    margin: auto;
    width: 50%;
  }
  .user-controls{
    margin: auto;
    width: 50%;
}
}
</style>
