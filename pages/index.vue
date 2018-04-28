<template>
  <main>
    <div class="places" ref="places">
      <div v-for="place in places" class="location">
        <img :src="place.img" :alt="place.name" />
        <h2>{{ place.name }}</h2>
        <p><strong>Rating: {{ place.rating }}</strong></p>
        <p>{{ place.description }}</p>
        <hr />
      </div>
    </div>
    <div class="mapcontain" ref="mapcontain">
      <p>
        <icon-base icon-name="mappin"><icon-map-pin /></icon-base> 
        祥龙物流办公地点
      </p>
    </div>
  </main>
</template>

<script>
import { mapState } from 'vuex'
import IconBase from '~/components/IconBase.vue'
import IconMapPin from '~/components/IconMapPin.vue'

export default {
  components: {
    IconBase,
    IconMapPin
  },
  computed: mapState(['page', 'users', 'places']),
  mounted() {
    var mapboxgl = require('mapbox-gl/dist/mapbox-gl.js')
    mapboxgl.accessToken =
      'pk.eyJ1Ijoid3VoYWlibyIsImEiOiJjamdpenNqc3owM3huMnhxangyMTlzajZ3In0.Zo0po7Jr04l0N8FJu0MtYA'
    var map = new mapboxgl.Map({
      container: this.$refs.mapcontain,
      style: 'mapbox://styles/mapbox/streets-v10', //'mapbox://styles/sdrasner/cjfg0watl6rkv2sllf6hepdd5'
      center: [116.516416, 39.889805],
      zoom: 11.15
    })

    map.on('load', function () {
    // Add a layer showing the places.
      map.addLayer({
          "id": "places",
          "type": "symbol",
          "source": {
              "type": "geojson",
              "data": {
                  "type": "FeatureCollection",
                  "features": [{
                      "type": "Feature",
                      "properties": {
                          "description": "<strong>祥龙物流</strong><p>朝阳区广渠东路2号</p>",
                          "icon": "star"
                      },
                      "geometry": {
                          "type": "Point",
                          "coordinates": [116.516416, 39.889805]
                      }
                  }, ]
              }
          },
          "layout": {
              "icon-image": "{icon}-15",
              "icon-allow-overlap": true
          }
      });

      map.on('click', 'places', function (e) {
          var coordinates = e.features[0].geometry.coordinates.slice();
          var description = e.features[0].properties.description;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
              coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          new mapboxgl.Popup()
              .setLngLat(coordinates)
              .setHTML(description)
              .addTo(map);
      });

      // Change the cursor to a pointer when the mouse is over the places layer.
      map.on('mouseenter', 'places', function () {
          map.getCanvas().style.cursor = 'pointer';
      });

      // Change it back to a pointer when it leaves.
      map.on('mouseleave', 'places', function () {
          map.getCanvas().style.cursor = '';
      });
      });
  }
}
</script>

<style>
.mapboxgl-missing-css {
  display: none;
}
</style>

<style lang="scss" scoped>
main {
  padding-top: 20px;
  border-top: 1px solid #ddd;
  margin-top: 120px;
}

.places {
  img {
    float: left;
    margin: 0 15px 15px 0;
  }
  p {
    margin-top: 10px;
  }
  .location {
    padding: 10px 0;
  }
}

hr {
  border-top: 1px solid #ddd;
  border-bottom: none;
  margin-top: 15px;
}

.mapcontain {
  width: 35%;
  float: right;
  height: 400px;
  p {
    margin: 10px 0;
  }
}

@media screen and (max-width: 600px) {
  .mapcontain {
    width: 100%;
    margin: 10px 0;
  }
}
</style>
