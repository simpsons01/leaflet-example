<template>
  <div ref="map" id="map" style="height:1000px"></div>
</template>

<script>
import addressPoints from "../../maker";
import L from "leaflet";
import "leaflet.markercluster";
let markers;
let map;
let count = 0;
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
        maxZoom: 18,
        tileSize: 512,
        zoomOffset: -1,
        showParagraph: false,
        mapOptions: {
          zoomSnap: 0.5
        },
        showMap: true
      },
      countMarker:0
    };
  },
  methods: {
    updateMarker() {
      markers.clearLayers();
      this.addressAry.forEach(item => {
        const marker = new jobMarker(new L.LatLng(item.lat, item.lng), {
          title: item.title,
          isClick: item.isClick,
          lat: item.lat,
          lng: item.lng
        });
        marker.bindPopup(item[2]);
        markers.addLayer(marker);
      });
    }
  },
  mounted() {
    map = L.map("map", this.options);
    L.tileLayer(this.url, this.attribution).addTo(map);
    markers = L.markerClusterGroup({
      iconCreateFunction: cluster => {
        count += 1;
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
      zoomToBoundsOnClick: false,
      showCoverageOnHover: false
    });
    this.updateMarker(); // mount marker
    markers.on("clusterclick", cluster => {
      console.log(count);
      const markerGroup = cluster.layer.getAllChildMarkers();
      if (markerGroup.every(item => item.options.isClick)) return;
      this.countMarker = markerGroup.length;
      this.addressAry.forEach(item => {
        item.isClick = false
        const index = markerGroup.findIndex(marker => {
          return item.lat == marker.options.lat;
        });
        if (index >= 0) {
          item.isClick = true;
        }
      });
      this.updateMarker()
      // markers.refreshClusters(cluster.layer)
    });
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
