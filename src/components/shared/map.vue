<template>
  <div class="App"/>
</template>

<script>
import { mapState } from 'vuex'
import MarkerClusterer from '@google/markerclusterer';
import gmapsInit from '../../utils/gmaps';

export default {
  name: 'App',
  data() {
    return {
      location: []
    }
  },
  computed: {
    myCases () {
      return this.$store.getters['medical/getCases']
    }
  },
  async mounted() {
    var caseLocation = []
    this.myCases.forEach(function(thiscase){
      let position = {lat: thiscase.coords.lat, lng: thiscase.coords.lon}
      caseLocation.push({position: position});
    });
    this.locations = caseLocation;

    try {
      const google = await gmapsInit();
      const geocoder = new google.maps.Geocoder();
      const map = new google.maps.Map(this.$el);

      geocoder.geocode({ address: this.myCases[0].zip }, (results, status) => {
        if (status !== 'OK' || !results[0]) {
          throw new Error(status);
        }

        map.setCenter(results[0].geometry.location);
        map.fitBounds(results[0].geometry.viewport);
        map.setZoom(13);
        map.setClickableIcons(false);

      });

      const markerClickHandler = (marker) => {
        map.setZoom(8);
        map.setCenter(marker.getPosition());
      };

      const markers = this.locations
        .map((location) => {
          const marker = new google.maps.Marker({ ...location, map });
          marker.addListener('click', () => markerClickHandler(marker));

          return marker;
        });

      new MarkerClusterer(map, markers, {
        imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m',
      });
    } catch (error) {
      console.error(error);
    }
  },
  watch: {
    caseLoad(value) {
    }
}
};
</script>

<style>

.App {
  width: 100%;
  height: 60vh;
}
</style>