<template>
  <v-map v-bind="options" style="height:1000px">
    <v-icondefault></v-icondefault>
    <v-tilelayer :url="url"></v-tilelayer>
    <v-marker-cluster ref="clusterRef" :options="clusterOptions" @clusterclick="clusterclick">
      <v-marker
        v-for="address in addressAry"
        :key="address.lat"
        :lat-lng="address.latlng"
        ref="marker"
        @click="markerClick"
      />
    </v-marker-cluster>
  </v-map>
</template>

<script>
import * as Vue2Leaflet from "vue2-leaflet";
import { latLng, divIcon, point } from "leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";
import addressPoints from "../../maker";
const CLUSTER_SETTING = {
  spiderfyOnMaxZoom: false,
  zoomToBoundsOnClick: false,
  showCoverageOnHover: false,
  singleMarkerMode: true
};
// const jobMarker = L.Marker.extend({
//   title: "",
//   isClick: "",
//   lat: "",
//   lng: ""
// });

export default {
  name: "Example",
  components: {
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "v-icondefault": Vue2Leaflet.LIconDefault,
    "v-marker": Vue2Leaflet.LMarker,
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
      clusterOptions: {},
      tempMarkerAry: [],
      tempLayer: null
    };
  },
  methods: {
    clusterclick(cluster) {
      const markerGroup = cluster.layer.getAllChildMarkers();
      if (markerGroup.every(item => item.options.isClick)) return;
      this.countMarker = markerGroup.length;
      this.clearLastMarker();
      markerGroup.forEach(marker => (marker.options.isClick = true));
      this.$refs.clusterRef.mapObject.refreshClusters(cluster.layer);
      this.tempMarkerAry = markerGroup;
      this.tempLayer = cluster.layer;
    },
    clearLastMarker() {
      if (this.tempMarkerAry.length > 0)
        this.tempMarkerAry.forEach(marker => (marker.options.isClick = false));
      if (this.tempLayer)
        this.$refs.clusterRef.mapObject.refreshClusters(this.tempLayer);
    },
    markerClick(marker) {
      if (marker.target.options.isClick) return;
      this.clearLastMarker();
      this.countMarker = 1;
      marker.target.options.isClick = true;
      this.$refs.clusterRef.mapObject.refreshClusters(marker.target);
      this.tempMarkerAry = [marker.target];
      this.tempLayer = marker.target;
    }
  },
  created() {
    this.clusterOptions = {
      iconCreateFunction: cluster => {
        const markerGroup = cluster.getAllChildMarkers();
        const isClick = markerGroup.every(item => item.options.isClick);
        let className =
          this.countMarker == markerGroup.length && isClick
            ? "myCustomMarker root"
            : isClick
            ? "myCustomMarker anchor"
            : "myCustomMarker";
        return divIcon({
          html: markerGroup.length,
          className,
          iconSize: point(40, 40)
        });
      },
      ...CLUSTER_SETTING
    };
  },
  mounted() {}
};
</script>

<style lang="scss">
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
  border: 2px solid rgb(255, 145, 0);
  color: rgb(255, 145, 0);
  &::before {
    content: "";
    display: inline-block;
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 20px 8px 0 8px;
    border-color: white transparent transparent transparent;
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    bottom: -16px;
    z-index: 1;
  }
  &::after {
    content: "";
    display: inline-block;
    width: 0;
    height: 0;
    border-style: solid;
    border-width: 22px 10px 0 10px;
    border-color: rgb(255, 145, 0) transparent transparent transparent;
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    bottom: -20px;
    z-index: -1;
  }
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