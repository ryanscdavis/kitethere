<template>
  <div class="search__map">
    <div id="gmap"></div>
    <div class="search__popup" v-if="popupVisible">popup</div>
  </div>
</template>

<script>
import googleMapsAPI from '@/assets/google-maps-api.js'

export default {
  data () {
    return {
      map: null,
      markers: {},
      popupVisible: false
    }
  },

  computed: {
    collection () { return this.$store.getters['spots/collection'] }
  },

  mounted () {
    googleMapsAPI().then(() => {
      this.initMap()
      this.addMarkers()
    })
  },

  methods: {
    initMap () {
      const options = {
        zoom: 2,
        center: { 'lat': 10, 'lng': -50 },
        mapTypeId: 'hybrid',
        gestureHandling: 'greedy'
      }
      var element = document.getElementById('gmap')
      this.map = new window.google.maps.Map(element, options)
      this.map.addListener('bounds_changed', this.boundsChanged)
    },

    boundsChanged () {
      var mapBounds = this.map.getBounds()
      var mapCenter = this.map.getCenter()

      var center = {
        'lat': mapCenter.lat(),
        'lng': mapCenter.lng()
      }

      var bounds = {
        'northEast': {
          'lat': mapBounds.getNorthEast().lat(),
          'lng': mapBounds.getNorthEast().lng()
        },
        'southWest': {
          'lat': mapBounds.getSouthWest().lat(),
          'lng': mapBounds.getSouthWest().lng()
        }
      }

      this.$store.commit('map/setCenter', center)
      this.$store.commit('map/setBounds', bounds)
    },

    addMarker (map, id, lat, lng, name) {
      var args = {
        'position': { 'lat': lat, 'lng': lng },
        'map': map
      }

      var str = name

      var infoWindow = new google.maps.InfoWindow({
        content: str,
        disableAutoPan: true
      })

      var marker = new window.google.maps.Marker(args)
      var clickEvent = () => { this.$store.dispatch('spots/select', id) }
      var mouseoverEvent = () => { infoWindow.open(this.map, marker) }
      var mouseoutEvent = () => { infoWindow.close() }

      marker.addListener('click', clickEvent)
      marker.addListener('mouseover', mouseoverEvent)
      marker.addListener('mouseout', mouseoutEvent)

      this.markers[id] = marker
    },

    addMarkers () {
      this.collection.forEach(spot => {
        this.addMarker(this.map, spot.spotID, spot.position.lat, spot.position.lng, spot.name)
      })
    }
  }

}
</script>

<style>
#gmap {
  width: 100%;
  height: calc(100vh - 80px);
}

.search__map {
  position: sticky;
  top: 80px;
}

.search__popup {
  background-color: white;
  z-index: 3;
  position: fixed;
  left: 20px;
  top: 200px;
}
.gm-style-iw + div {display: none;}
.gm-style-iw {text-align:center;}
</style>