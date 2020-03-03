<template>
  <div ref="map" id="map" style="height:1000px"></div>
</template>

<script>
import addressPoints from "../maker";
import L from "leaflet";
import "leaflet.markercluster";
let markers;
let map;
let countMarker = 0;
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
      markerAry: [],
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
      polygon: {
        latlngs: [
          [47.2263299, -1.6222],
          [47.21024000000001, -1.6270065],
          [47.1969447, -1.6136169],
          [47.18527929999999, -1.6143036],
          [47.1794457, -1.6098404],
          [47.1775788, -1.5985107],
          [47.1676598, -1.5753365],
          [47.1593731, -1.5521622],
          [47.1593731, -1.5319061],
          [47.1722111, -1.5143967],
          [47.1960115, -1.4841843],
          [47.2095404, -1.4848709],
          [47.2291277, -1.4683914],
          [47.2533687, -1.5116501],
          [47.2577961, -1.5531921],
          [47.26828069, -1.5621185],
          [47.2657179, -1.589241],
          [47.2589612, -1.6204834],
          [47.237287, -1.6266632],
          [47.2263299, -1.6222]
        ],
        color: "#ff00ff"
      }
    };
  },
  methods: {
    updateMarker() {
      this.removeMarker();
      this.addressAry.forEach(item => {
        const marker = L.marker(new L.LatLng(item.lat, item.lng), {
          title: item.title,
          isClick: item.isClick,
          lat: item.lat,
          lng: item.lng
        });
        marker.bindPopup(item[2]);
        markers.addLayer(marker);
        this.markerAry.push(marker);
      });
    },
    removeMarker() {
      this.markerAry.forEach(marker => markers.removeLayer(marker));
      this.markerAry = [];
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
          countMarker == markerGroup.length && isClick
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
      const markerGroup = cluster.layer.getAllChildMarkers();
      if (markerGroup.every(item => item.options.isClick)) return;
      countMarker = markerGroup.length;
      this.addressAry.forEach(item => (item.isClick = false));
      this.updateMarker();
      this.addressAry.forEach(item => {
        const index = markerGroup.findIndex(marker => {
          return item.lat == marker.options.lat;
        });
        if (index >= 0) {
          item.isClick = true;
        }
      });
      this.updateMarker();
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
  // &::before {
  //   content: "";
  //   display: inline-block;
  //   position: absolute;
  //   left: 50%;
  //   transform: translateX(-50%);
  //   bottom: -15px;
  //   width: 0;
  //   height: 0;
  //   border-style: solid;
  //   border-width: 30px 15px 0 15px;
  //   border-color: white transparent transparent transparent;
  //   z-index: -1;
  // }
  // &::after {
  //   content: "";
  //   display: inline-block;
  //   position: absolute;
  //   left: 50%;
  //   transform: translateX(-50%);
  //   bottom: -15px;
  //   width: 0;
  //   height: 0;
  //   border-style: solid;
  //   border-width: 32px 15px 0 15px;
  //   border-color: rgb(255, 145, 0) transparent transparent transparent;
  //   z-index: -1;
  // }
}
</style>