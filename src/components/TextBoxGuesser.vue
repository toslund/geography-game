<template>
  <div class="holder" style="position:relative">
        <label for="guess">{{promptMessage}}?</label>
        <input
          id="guess"
          type="text"
          v-model="guess"
          v-on:keyup="evaluate"
          v-bind:class="{ error: isError }"
          autocomplete="off">
        <div
          class="suggestions-holder"
          style="position:absolute"
          v-if="suggesting">
          <div class="suggestions">
            <div
              class="suggestion"
              v-for="(suggestion, idx) in suggestions"
              v-bind:key="idx"
              @click="guess = suggestion">
              <span class="inputed">
                {{suggestion.slice(0, guess.length)}}
              </span>
              <span class="suggested">
                {{suggestion.slice(guess.length, suggestion.length)}}
              </span>
            </div>
          </div>
        </div>
        <div class="">
          <button class="btn" @click="hint">Hint</button>
          <div class="hint" v-if="hinting"> {{ hintMessage }} </div>
        </div>
  </div>
</template>

<script>

export default {
  name: 'TextBoxGuesser',
  props: {
    choices: Array,
    correctChoiceIdx: Number,
    targetKey: String,
    promptMessage: String,
  },
  data() {
    return {
      guess: '',
      awaitingSearch: false,
      isError: false,
      hinting: false,
      hintArray: [],
      hiddenHintIdx: [],
      visibleHintIdxs: [],
      hintPenalty: -10,
    };
  },
  computed: {
    hintMessage() {
      console.log('fired hintMessage');
      const visibleHintArray = Array(this.hintArray.length);
      for (let index = 0; index < this.hintArray.length; index += 1) {
        if (this.visibleHintIdxs.includes(index)) {
          visibleHintArray[index] = this.hintArray[index];
        } else { visibleHintArray[index] = '_'; }
      }
      return visibleHintArray.join('');
    },
    suggesting() {
      if (!this.guess) { return false; }
      if (this.suggestions.length > 0 && !(this.lowerCaseGuess === this.firstSuggestion)) {
        return true;
      }
      return false;
    },
    lowerCaseGuess() {
      if (this.guess) {
        return this.guess.toLowerCase();
      }
      return '';
    },
    correctAnswer() {
      return this.choices[this.correctChoiceIdx][this.targetKey].toLowerCase();
    },
    guessCorrect() {
      if (this.guess.toLowerCase() === this.correctAnswer) {
        return true;
      }
      return false;
    },
    suggestions() {
      const allSuggestions = this.choices.map((n) => n[this.targetKey]);
      return allSuggestions.filter((n) => n.toLowerCase().startsWith(this.lowerCaseGuess));
    },
    firstSuggestion() {
      return this.suggestions[0].toLowerCase();
    },
  },
  methods: {
    hint() {
      this.hinting = true;
      const revealedIndex = this.hiddenHintIdx.pop();
      console.log('revealed index');
      console.log(revealedIndex);
      if (revealedIndex !== undefined) { // There is still a hint to be given
        this.$emit('scorechange', this.hintPenalty);
        this.visibleHintIdxs.push(revealedIndex);
      }
      if (this.hiddenHintIdx.length === 0) { this.$emit('chose', false); } // No more hints to give and user forefeits
    },
    prompt() {
      return this.promptMessage;
    },
    mouseMove() {
      console.log('mousemove');
    },
    debounce(fn, delay) {
      let timer;
      return function () {
        console.log(fn);
        clearTimeout(timer);
        timer = setTimeout(() => {
          fn();
        }, delay);
      };
    },
    evaluate() {
      if (this.guess === '') { return; }
      if (this.guessCorrect) {
        this.$emit('chose', true);
        console.log(this.guessCorrect);
        return;
      }
      console.log('setting');
      this.isError = true;
      this.guess = '';
      setTimeout(() => {
        console.log('resetting');
        this.isError = false;
      }, 1000);
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
  created() {
    this.evaluate = this.debounce(this.evaluate, 2000);
    window.addEventListener('mousedown', this.mouseMove);
  },
  mounted() {
    const answer = this.choices[this.correctChoiceIdx][this.targetKey];
    this.hintArray = answer.split('');
    console.log('hint array:');
    console.log(this.hintArray);
    this.hiddenHintIdx = [...Array(this.hintArray.length).keys()];
    for (let index = 0; index < this.hintArray.length; index += 1) {
      console.log('inside for loop');
      const element = this.hintArray[index];
      if (element === ' ') {
        this.visibleHintIdxs.push(index);
        this.hiddenHintIdx.splice(index, 1);
      }
    }
    this.hiddenHintIdx = this.shuffle(this.hiddenHintIdx);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.hint {
  letter-spacing: 0.1em;
}

.holder {
  margin: 2em;
  max-width: 30em;
}

input {
  font-size:1.2em;
  padding-left: 20px;
  padding-right: 20px;
  border-radius: .3rem;
  width: 100%;
  transition: box-shadow 700ms;
}

input:focus, textarea:focus {
  box-shadow: 0 0 5px rgba(81, 203, 238, 1);
  outline: none;
}

input.error
    {
      animation: shake 0.2s ease-in-out 0s 2;
      box-shadow: 0 0 0.5em red;
      transition: box-shadow 0ms;
    }

.suggestions-holder {
  width: 100%;
  background-color: #fff;
  border: 1px solid #d9d9d9;
  border-top: none;
  border-radius: .3rem;
}

.suggestions {
  list-style-type: none;
}

.inputed {
  font-weight: bold;
}

.suggested {
  font-weight: normal;
}

.suggestion:hover {
  background-color: #e0e0e0;
}

@keyframes shake {
  0% { margin-left: 0rem; }
  25% { margin-left: 0.5rem; }
  75% { margin-left: -0.5rem; }
  100% { margin-left: 0rem; }
}

</style>
