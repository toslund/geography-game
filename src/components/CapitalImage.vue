<template>
<!-- eslint-disable max-len -->
  <div>
    <div class="backdrop">
      <img class="placeholder" src="../assets/700.png">
      <div class=""
        v-for="photoIndex in loadedIndexes"
        :key="photoIndex"
        v-show="photoIndex === currentIndex">
        <div class="slider-image-wrapper">
          <img class="slider-image" :src="photos[photoIndex].location">
            <div
              class="btn attribution-btn"
              v-html="photos[photoIndex].html_attributions[0]"
              v-if="photos">
            </div>
        </div>
    </div>
        <button class="btn slider-button-left" :disabled="currentIndex === 0" v-if="photos && currentIndex != 0" @click="incrementDown">
          <svg class="icon"><use xlink:href="defs.svg#arrow-left"></use></svg>
        </button>
        <button class="btn slider-button-right" v-if="photos && (currentIndex + 1) < photos.length" @click="incrementUp">
          <svg class="icon"><use xlink:href="defs.svg#arrow-right"></use></svg>
        </button>
    </div>
    <div id="imgs"></div>
  </div>
</template>

<script>
import axios from 'axios';
import json from './json/state_capitals.json';

export default {
  name: 'CapitalImage',
  props: {
    locationCode: String,
  },
  data() {
    return {
      capitals: json,
      message: 'foo',
      placesDetailsEndpoint: process.env.VUE_APP_PLACES_ENDPOINT,
      placesPhotosEndpoint: process.env.VUE_APP_PHOTOS_ENDPOINT,
      photos: null,
      currentIndex: 0,
      loadIndexes: [],
      loadedIndexes: [],
    };
  },
  computed: {
  },
  watch: {
    locationCode() {
      this.refresh();
    },
    loadIndexes: {
      deep: true,
      handler(newVal) {
        console.log('in watch loadedindex');
        console.log(this.photos);
        console.log(newVal);
        console.log('in watch loadedindex');
        newVal.forEach((index) => {
          if (!('location' in this.photos[index])) {
            this.fetchPlacePhoto(index);
          }
        });
      },
    },
  },
  methods: {
    incrementUp() {
      if (!this.loadIndexes.includes(this.currentIndex + 2)
      && (this.currentIndex < this.photos.length - 2)) {
        this.loadIndexes.push(this.currentIndex + 2);
      }
      if (this.currentIndex + 1 >= this.photos.length) { return; }
      this.currentIndex += 1;
    },
    canFlipForward() {
      if (!this.photos) { return false; }
      if (this.currentIndex + 1 < this.photos.length) { console.log('can flip'); return true; }
      return false;
    },
    incrementDown() {
      if (this.currentIndex <= 0) { return; }
      this.currentIndex -= 1;
    },
    fetchPlaceDetail() {
      console.log('fetching place data');
      axios
        .get(`${this.placesDetailsEndpoint}/${this.locationCode}`)
        .then((response) => {
          console.log(response.data);
          this.photos = response.data.photos;
          this.loadIndexes.push(0, 1);
          this.currentIndex = 0;
        });
    },
    fetchPlacePhoto(index) {
      console.log('fetching photo');
      axios
        .get(`${this.placesPhotosEndpoint}/${this.photos[index].photo_reference}`)
        .then((response) => {
          this.photos[index].location = response.data.location;
          this.loadedIndexes.push(index);
        });
    },
    refresh() {
      this.loadIndexes.length = 0;
      this.loadedIndexes.length = 0;
      this.fetchPlaceDetail();
    },
  },
  mounted() {
    this.refresh();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.backdrop {
  position: relative;
  display: inline-block;
  margin: auto;
}

.slider-buttons {
  position: absolute;
  top: 50%;
  left: 50%;
  /* transform: translate(-50%, 0%); */
}

.slider-button-left {
  position: absolute;
  top: 50%;
  left: 5%;
  transform: translate(-50%, 0%);
}

.slider-button-right {
  position: absolute;
  top: 50%;
  left: 95%;
  transform: translate(-50%, 0%);
}

.btn {
    display: inline-block;
    border: none;
    text-decoration: none;
    padding: .5em  1.25em;
    color: black;
    background: rgba(237, 237, 237, .8);
    font-size: .8rem;
    text-transform: uppercase;
    border-radius: .25em;
    transition:
        transform 250ms ease-in-out,
        opacity 250ms linear;
}

.btn:hover {
  background: rgba(0, 0, 0, .8);
  color: white;
}

.icon {
    background-color: transparent;
    width: 24px;
    height: 24px;
  }

  .attribution-btn {
    --clr-text: #black;
    background: none;
    text-align: right;
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translate(-50%, 0%);
    font-size: .5em;
}

</style>
