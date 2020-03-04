<template>
  <v-map :zoom="options.zoom" :center="options.center" style="height:1000px">
    <v-icondefault></v-icondefault>
    <v-tilelayer :url="url"></v-tilelayer>
    <v-marker-cluster
      ref="clusterRef"
      :options="clusterOptions"
      @clusterclick="click"
      @ready="ready"
    >
      <v-marker
        v-for="address in addressAry"
        :key="address.lat"
        :lat-lng="address.latlng"
        :icon="customicon"
      >
        <v-popup :content="address.title"></v-popup>
      </v-marker>
    </v-marker-cluster>
  </v-map>
</template>

<script>
import * as Vue2Leaflet from "vue2-leaflet";
import { latLng, Icon, icon, divIcon, point } from "leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";
import iconUrl from "leaflet/dist/images/marker-icon.png";
import shadowUrl from "leaflet/dist/images/marker-shadow.png";
import addressPoints from "../../maker";

export default {
  name: "Example",
  components: {
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "v-icondefault": Vue2Leaflet.LIconDefault,
    "v-marker": Vue2Leaflet.LMarker,
    "v-popup": Vue2Leaflet.LPopup,
    "v-marker-cluster": Vue2LeafletMarkerCluster
  },
  data() {
    return {
      addressAry: addressPoints.map(item => {
        return {
          lat: item[0],
          lng: item[1],
          title: item[2],
          isClick: false,
          latlng: latLng(item[0], item[1])
        };
      }),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      options: {
        zoom: 15,
        center: latLng(-37.8219782333, 175.2265028333),
        maxZoom: 18,
        tileSize: 512,
        zoomOffset: -1,
        showParagraph: false,
        mapOptions: {
          zoomSnap: 0.5
        },
        showMap: true
      },
      countMarker: 0,
      clusterOptions: {
        zoomToBoundsOnClick: false,
        showCoverageOnHover: false
      },
      customicon: icon(
        Object.assign({}, Icon.Default.prototype.options, {
          iconUrl,
          shadowUrl
        })
      )
    };
  },
  methods: {
    click(cluster) {
      console.log(cluster.layer.getAllChildMarkers());
    },
    ready: e => console.log("ready", e)
  },
  mounted() {
    this.$nextTick(() => {
      console.log(this.$refs.clusterRef.mapObject._defaultIconCreateFunction)
      this.$refs.clusterRef.mapObject._defaultIconCreateFunction = function(cluster) {
        return divIcon({
          html: cluster.getChildCount(),
          className: "myCustomMarker",
          iconSize: point(40, 40)
        });
      }
    })
  }
};
</script>

<style lang="scss">
@import "~leaflet/dist/leaflet.css";
@import "~leaflet.markercluster/dist/MarkerCluster.css";
@import "~leaflet.markercluster/dist/MarkerCluster.Default.css";
.myCustomMarker {
  border-radius: 50%;
  background-color: rgb(255, 145, 0);
  text-align: center;
  line-height: 40px;
  color: white;
  font-size: 18px;
}

.root {
  background: white;
  border: 4px solid rgb(110, 204, 57);
  color: rgb(110, 204, 57);
}

.anchor {
  background: white;
  border: 2px solid rgb(255, 145, 0);
  color: rgb(255, 145, 0);
  position: relative;
  &:hover {
    background: #ffc16f;
    color: white;
  }
}
</style>
