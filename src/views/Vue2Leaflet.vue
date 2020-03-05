<template>
  <div>
    <v-map v-bind="options" style="height:1000px" 
          @click="mapClick"
          @zoom="zoomChange"
    >
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
        @mouseover="cardMouseover(card.groupId)"
      >
        <p>lat: {{card.lat}}</p>
        <p>lng: {{card.lng}}</p>
        <p>title: {{card.title}}</p>
        <p>groupId: {{card.groupId}}</p>
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
let lastHover = null

function random() { // min and max included 
  return Math.floor(Math.random() * (100000000 - 1 + 1) + 1);
}

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
      addressAry: addressPoints.map((item) => {
        return {
          groupId:0,
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
      tempLayer: null,
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
    zoomChange() {
       this.$refs.clusterRef.mapObject.refreshClusters(this.tempLayer)
    },
    mapClick(map) {
      this.mapLatlng = map.latlng;
    },
    clusterclick(cluster) {
      const markerGroup = cluster.layer.getAllChildMarkers();
      this.cardGroup = markerGroup.map(item => {
        return { ...item.options };
      });
      this.countMarker = markerGroup.length;
      this.clearLastMarker();
      markerGroup.forEach(marker => (marker.options.isClick = true));
      this.$refs.clusterRef.mapObject.refreshClusters(cluster.layer);
      this.tempMarkerAry = markerGroup;
      this.tempLayer = cluster.layer;
    },
    clearLastMarker() {
      if (this.tempMarkerAry.length > 0)
        this.tempMarkerAry.forEach(marker => {
          marker.options.isClick = false;
        });
      if (this.tempLayer)
        this.$refs.clusterRef.mapObject.refreshClusters(this.tempLayer);
    },
    markerClick(marker) {
      this.clearLastMarker();
      this.countMarker = 1;
      marker.target.options.isClick = true;
      this.$refs.clusterRef.mapObject.refreshClusters(marker.target);
      this.cardGroup = [marker.target.options];
      this.tempMarkerAry = [marker.target];
      this.tempLayer = marker.target;
    },
    cardMouseover(id) {
       if(lastHover) lastHover.classList.remove('anchor-over')
       const anchor =  document.querySelector(`div[data-group-id="${id}"]`)
       if(anchor) anchor.classList.add('anchor-over')
       lastHover = anchor
    },
    cardMouseleave() {
       if(lastHover) lastHover.classList.remove('anchor-over')
    },
    // clusterMouseOver() {
    //   console.log('clustermouseover');
    // },
    // markerMouseOver() {
    //   console.log('markermouseover');
    // },
  },
  created() {
    this.clusterOptions = {
      iconCreateFunction: cluster => {
        const markerGroup = cluster.getAllChildMarkers();
        const groupId = random()
        markerGroup.forEach(item => item.options.groupId = groupId)
        this.cardGroup.forEach(item => {
          const index = markerGroup.findIndex(marker => {
            return marker.options.lat === item.lat
          })
          if(index >= 0) item.groupId = groupId
        }) 
        const isClick = markerGroup.every(item => item.options.isClick);
        const className =
          this.countMarker == markerGroup.length && isClick
            ? "root"
            : isClick
            ? "anchor"
            : "myCustomMarker";
        return divIcon({
          html: `<div data-group-id=${groupId}>${markerGroup.length}</div>`,
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
  max-height: 600px;
  overflow-y: auto;
  .card-items {
    background: white;
    padding:15px 40px;
    > p {
      margin: 2px;
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
  transition: .3s all ease-in-out;
  &:hover {
    background: #ffc16f;
    color: white;
  }
  .anchor-over {
    background: #ffc16f;
    color: white;
  }
}
</style>