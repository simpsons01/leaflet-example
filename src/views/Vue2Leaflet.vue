<template>
  <div>
    <v-map ref="map" v-bind="options" style="height:1000px" @click="mapClick" @zoom="zoomChange">
      <v-tilelayer :url="url"></v-tilelayer>
      <v-polyon :lat-lngs="polygon.latlngs" :color="polygon.color" />
      <v-marker :lat-lng="mapLatlng" :icon="customIcon" />
      <v-marker-cluster ref="clusterRef" :options="clusterOptions" @clusterclick="clusterclick">
        <v-marker
          v-for="address in addressAry"
          :key="address.lat"
          :lat-lng="address.latlng"
          :options="address"
          ref="marker"
          @click="markerClick"
        />
      </v-marker-cluster>
    </v-map>
    <div class="card">
      <div
        class="card-items"
        v-for="(card,index) in cardGroup"
        :key="index"
        @mouseleave="cardMouseleave"
        @click="flyToCluster(card.markerInstance)"
        @mouseover="cardMouseover(card.markerInstance)"
      >
        <p>lat: {{card.lat}}</p>
        <p>lng: {{card.lng}}</p>
        <p>title: {{card.title}}</p>
      </div>
    </div>
  </div>
</template>

<script>
import iconUrl from "leaflet/dist/images/marker-icon.png";
import shadowUrl from "leaflet/dist/images/marker-shadow.png";
import * as Vue2Leaflet from "vue2-leaflet";
import { Icon, icon, latLng, divIcon, point } from "leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";
import addressPoints from "../../maker";

const CLUSTER_SETTING = {
  spiderfyOnMaxZoom: false,
  zoomToBoundsOnClick: false,
  showCoverageOnHover: false,
  singleMarkerMode: true
};

export default {
  name: "Example",
  components: {
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "v-marker": Vue2Leaflet.LMarker,
    "v-marker-cluster": Vue2LeafletMarkerCluster,
    "v-polyon": Vue2Leaflet.LPolygon
  },
  data() {
    return {
      zoomLevel: 0,
      customIcon: icon(
        Object.assign({}, Icon.Default.prototype.options, {
          iconUrl,
          shadowUrl
        })
      ),
      mapLatlng: {
        lat: -3.82042121980573,
        lng: 175.22205007158524
      },
      cardGroup: [],
      addressAry: addressPoints.map(item => {
        return {
          lat: item[0],
          lng: item[1],
          title: item[2],
          isClick: false,
          isMouseOver: false,
          isPicked: false,
          latlng: latLng(item[0], item[1])
        };
      }),
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      options: {
        zoom: 15,
        center: latLng(-37.8219782333, 175.2265028333),
        maxZoom: 17,
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
      tempLayer: null,
      tempHoverMarker: null,
      tempHoverLayer: null,
      tempPickedMarker: null,
      tempPickedLayer: null,
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
        color: "green"
      }
    };
  },
  methods: {
    zoomChange(map) {
      this.zoomLevel = map.target._zoom;
    },
    mapClick(map) {
      this.mapLatlng = map.latlng;
    },
    markerClick(marker) {
      this.clearLastClickMarker();
      this.countMarker = 1;
      marker.target.options.isClick = true;
      this.$refs.clusterRef.mapObject.refreshClusters(marker.target);
      this.cardGroup = [
        {
          markerInstance: marker.target,
          ...marker.target.options
        }
      ];
      this.tempMarkerAry = [marker.target];
      this.tempLayer = marker.target;
    },
    cardMouseover(marker) {
      this.clearLastHoverMarker();
      marker.options.isMouseOver = true;
      this.$refs.clusterRef.mapObject.refreshClusters(marker.__parent);
      this.tempHoverMarker = marker;
      this.tempHoverLayer = marker.__parent;
    },
    clusterclick(cluster) {
      this.clearLastClickMarker();
      const markerGroup = cluster.layer.getAllChildMarkers();
      this.cardGroup = markerGroup.map(item => {
        return {
          markerInstance: item,
          ...item.options
        };
      });
      this.countMarker = markerGroup.length;
      markerGroup.forEach(marker => (marker.options.isClick = true));
      this.$refs.clusterRef.mapObject.refreshClusters(cluster.layer);
      this.tempMarkerAry = markerGroup;
      this.tempLayer = cluster.layer;
    },
    clearLastClickMarker() {
      if (this.tempMarkerAry.length > 0)
        this.tempMarkerAry.forEach(marker => {
          marker.options.isClick = false;
          marker.options.isMouseOver = false;
          marker.options.isPicked = false;
        });
      if (this.tempLayer)
        this.$refs.clusterRef.mapObject.refreshClusters(this.tempLayer);
    },
    clearLastHoverMarker() {
      if (this.tempHoverMarker)
        this.tempHoverMarker.options.isMouseOver = false;
      if (this.tempHoverLayer)
        this.$refs.clusterRef.mapObject.refreshClusters(this.tempHoverLayer);
    },
    clearLastPickedMarker() {
      if (this.tempPickedMarker) this.tempPickedMarker.options.isPicked = false;
      if (this.tempPickedLayer)
        this.$refs.clusterRef.mapObject.refreshClusters(this.tempPickedLayer);
    },
    cardMouseleave() {
      this.clearLastHoverMarker();
    },
    flyToCluster(marker) {
      this.clearLastPickedMarker();
      if (this.zoomLevel < 17) {
        this.$refs.map.mapObject.flyTo(marker.getLatLng(), 17);
      }
      marker.options.isPicked = true;
      this.$refs.clusterRef.mapObject.refreshClusters(marker.__parent);
      this.tempPickedMarker = marker;
      this.tempPickedLayer = marker.__parent;
    }
  },
  created() {
    this.clusterOptions = {
      iconCreateFunction: cluster => {
        const markerGroup = cluster.getAllChildMarkers();
        const statu = {
          isClick: true,
          isMouseOver: false,
          isPicked: false
        };
        markerGroup.forEach(marker => {
          if (!marker.options.isClick) statu.isClick = false;
          if (marker.options.isPicked) statu.isPicked = true;
          if (marker.options.isMouseOver) statu.isMouseOver = true;
        });
        const spidfyClass =
          this.countMarker == markerGroup.length && statu.isClick
            ? "root"
            : statu.isClick
            ? "anchor"
            : "myCustomMarker";
        const mouseOverClass = statu.isMouseOver ? "over" : "";
        const isPickedClass = statu.isPicked ? "picked" : "";
        return divIcon({
          html: `<div>${markerGroup.length}</div>`,
          className: `${spidfyClass} ${mouseOverClass} ${isPickedClass}`,
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
  &:hover {
    background: #ffc16f;
    border: 2px solid rgb(255, 145, 0);
  }
}

.card {
  z-index: 100000;
  position: absolute;
  right: 10px;
  top: 20px;
  max-height: 900px;
  overflow-y: auto;
  border: 1px solid black;
  .card-items {
    background: white;
    padding: 20px 35px;
    > p {
      margin: 4px;
    }
    &:hover {
      background: #ffc16f;
      color: white;
    }
  }
  .card-items + .card-items {
    border-top: 1px solid black;
  }
}

.root {
  text-align: center;
  border-radius: 50%;
  line-height: 40px;
  color: white;
  font-size: 18px;
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
  text-align: center;
  border-radius: 50%;
  line-height: 40px;
  font-size: 18px;
  background: white;
  border: 2px solid rgb(255, 145, 0);
  color: rgb(255, 145, 0);
  position: relative;
  overflow: hidden;
  transition: 0.3s all ease-in-out;
  &:hover {
    background: #ffc16f;
    color: white;
  }
  &.over {
    background: #ffc16f;
    color: white;
    border: none;
  }
  &.picked {
    background: yellowgreen;
    border: none;
    color: white;
  }
}
</style>