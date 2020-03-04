<template>
  <div ref="map" id="map" style="height:1000px"></div>
</template>

<script>
import addressPoints from "../../maker";
import L from "leaflet";
import "leaflet.markercluster";
let markers;
let map;
const CLUSTER_SETTING = {
  spiderfyOnMaxZoom: false,
  zoomToBoundsOnClick: false,
  showCoverageOnHover: false,
  singleMarkerMode: true
};
const jobMarker = L.Marker.extend({
  title: "",
  isClick: "",
  lat: "",
  lng: ""
});
export default {
  name: "Example",
  components: {},
  data() {
    return {
      tempMarkerAry: [],
      tempLayer: null,
      addressAry: addressPoints.map(item => {
        return {
          lat: item[0],
          lng: item[1],
          title: item[2],
          isClick: false
        };
      }),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      options: {
        zoom: 15,
        center: L.latLng(-37.8219782333, 175.2265028333),
        maxZoom: 17,
        tileSize: 512,
        zoomOffset: -1,
        showParagraph: false,
        mapOptions: {
          zoomSnap: 0.5
        },
        showMap: true
      },
      countMarker: 0
    };
  },
  methods: {
    updateMarker() {
      this.addressAry.forEach(item => {
        const marker = new jobMarker(new L.LatLng(item.lat, item.lng), {
          title: item.title,
          isClick: item.isClick,
          lat: item.lat,
          lng: item.lng
        });
        marker.on("click", e => {
          if(e.target.options.isClick) return
          this.clearLastMarker();
          this.countMarker = 1;
          e.target.options.isClick = true;
          markers.refreshClusters(e.target);
          this.tempMarkerAry = [e.target];
          this.tempLayer = e.target;
        });
        markers.addLayer(marker);
      });
    },
    clearLastMarker() {
      if (this.tempMarkerAry.length > 0)
        this.tempMarkerAry.forEach(marker => (marker.options.isClick = false));
      if (this.tempLayer) markers.refreshClusters(this.tempLayer);
    }
  },
  mounted() {
    map = L.map("map", this.options);
    L.tileLayer(this.url, this.attribution).addTo(map);
    markers = L.markerClusterGroup({
      iconCreateFunction: cluster => {
        const markerGroup = cluster.getAllChildMarkers();
        const isClick = markerGroup.every(item => item.options.isClick);
        let className =
          this.countMarker == markerGroup.length && isClick
            ? "myCustomMarker root"
            : isClick
            ? "myCustomMarker anchor"
            : "myCustomMarker";
        return L.divIcon({
          html: markerGroup.length,
          className,
          iconSize: L.point(40, 40)
        });
      },
      ...CLUSTER_SETTING
    });
    this.updateMarker(); // mount marker
    markers.on("clusterclick", cluster => {
      cluster.originalEvent.preventDefault();
      const markerGroup = cluster.layer.getAllChildMarkers();
      if (markerGroup.every(item => item.options.isClick)) return;
      this.countMarker = markerGroup.length;
      this.clearLastMarker();
      markerGroup.forEach(marker => (marker.options.isClick = true));
      markers.refreshClusters(cluster.layer);
      this.tempMarkerAry = markerGroup;
      this.tempLayer = cluster.layer;
    });
    map.doubleClickZoom.disable();
    map.addLayer(markers);
  }
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
