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
          <img class="slider-image" :src="photos[photoIndex].getUrl({maxWidth: 700, maxHeight: 700})">
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
import test from '@/mylib';
import gmapsInit from '@/gmaps';
import json from './json/state_capitals.json';

export default {
  name: 'CapitalImage',
  props: {
    locationCode: String,
    key: String,
  },
  data() {
    return {
      capitals: json,
      message: 'foo',
      ApiEndpoint: 'https://maps.googleapis.com/maps/api/streetview?',
      PlacesDetailsEndpoint: 'https://maps.googleapis.com/maps/api/place/details/json?',
      api_key: process.env.GOOGLE_API_KEY,
      photos: null,
      currentIndex: 0,
      loadedIndexes: [],
      googleService: null,
      google: null,
    };
  },
  computed: {
  },
  watch: {
    locationCode() {
      this.refresh();
    },
  },
  methods: {
    incrementUp() {
      if (!this.loadedIndexes.includes(this.currentIndex + 2)
      && (this.currentIndex < this.photos.length - 2)) {
        this.loadedIndexes.push(this.currentIndex + 2);
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
    refresh() {
      const request = {
        placeId: this.locationCode,
        fields: ['photos', 'geometry'],
      };
      this.googleService.getDetails(request, (place, status) => {
        if (status === this.google.maps.places.PlacesServiceStatus.OK) {
          console.log('status OK');
          console.log(place.photos);
          this.photos = place.photos;
          this.loadedIndexes = [0, 1];
          this.currentIndex = 0;
        }
      });
    },
  },
  async mounted() {
    console.log(test.foo());
    try {
      this.google = await gmapsInit();
      // const geocoder = new google.maps.Geocoder();
      const request = {
        placeId: this.locationCode,
        fields: ['photos', 'geometry'],
      };

      this.googleService = new this.google.maps.places.PlacesService(document.getElementById('imgs'));
      this.googleService.getDetails(request, (place, status) => {
        if (status === this.google.maps.places.PlacesServiceStatus.OK) {
          console.log('status OK');
          console.log(place.photos);
          this.photos = place.photos;
          this.loadedIndexes.push(0, 1);
        }
      });
    } catch (error) {
      console.error(error);
    }
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
