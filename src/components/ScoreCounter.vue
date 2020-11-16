<template>
  <div class="score-wrapper" style="position:relative">
    <h1 class="score-title">{{title}}</h1>
    <div class="score">
        {{displayScore}}
    </div>
  </div>
</template>

<script>

export default {
  name: 'ScoreCounter',
  props: {
    title: String,
    score: Number,
  },
  data() {
    return {
      displayScore: 0,
      interval: false,
    };
  },
  computed: {
  },
  watch: {
    score(newScore, oldScore) { // watch it
      console.log('watch fired');
      console.log(newScore);
      console.log(oldScore);
      this.animateScore(newScore, oldScore);
    },
  },
  methods: {
    animateScore(newScore, oldScore) {
      const that = this;
      const addedPoints = newScore - oldScore;
      const duration = Math.max(750, Math.min(addedPoints, 200) * 10);
      console.log('added points');
      console.log(addedPoints);
      // this.displayScore = newScore;
      let start;
      function step(timestamp) {
        if (start === undefined) {
          start = timestamp;
        }
        const elapsed = timestamp - start;
        const percentElapsed = Math.min(elapsed / duration, 1);
        that.displayScore = Math.floor(percentElapsed * addedPoints)
        + oldScore;
        if (elapsed < duration) { // Stop the animation
          window.requestAnimationFrame(step);
        }
      }
      window.requestAnimationFrame(step);
    },
  },
  created() {
  },
  mounted() {
    this.displayScore = this.score;
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.score-wrapper {
  display: inline-block;
  display: flex;
  justify-content: center;
  flex-direction: column;
  max-width: 20em;
  text-align: center;
  line-height: 1;
}

.score {
  font-family: 'Oswald';
  font-weight: 300;
  font-size: 4em;
}

.score-title {
  text-transform: uppercase;
  margin: 0;
  font-family: 'Oswald', sans-serif;
  font-weight: 300;
  font-size: 1.7em;
}
</style>
