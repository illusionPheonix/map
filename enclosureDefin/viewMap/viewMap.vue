<template>
  <div id="container" class="mymap"></div>
</template>

<script>
import { compareAscSort } from "@/assets/js/utils";

export default {
  props: {
    configObj: {
      type: Object
    }
  },
  data() {
    return {
      map: {},
      rectangleEditor: {},
      rectangle: {},
      lineArr: [],
      marker: {},
      markers: [],
      color: "#66B1FF",
      title: ""
    };
  },
  mounted() {
    this.loadmap(); //加载地图和相关组件;
  },
  watch: {
    configObj: {
      handler(newName, oldName) {
        console.log(newName, oldName);
        this.viewMask();
      },
      deep: true
    }
  },
  methods: {
    loadmap() {
      this.map = new AMap.Map("container", {
        center: [116.395577, 39.892257],
        zoom: 14
      });
      if (this.configObj.selectObj) {
        this.viewMask();
      }
    },

    viewMask() {
      let _this = this;
      if (_this.marker) {
        _this.map.clearMap();
        if (this.configObj.selectObj.longitudel) {
          let southWest = new AMap.LngLat(
            this.configObj.selectObj.longitudel,
            this.configObj.selectObj.latitudel
          );
          // latitudel: "39.99482"
          // latitudes: "39.95167"
          // longitudel: "116.55729"
          // longitudes: "116.50648"
          let northEast = new AMap.LngLat(
            this.configObj.selectObj.longitudes,
            this.configObj.selectObj.latitudes
          );
          let bounds = new AMap.Bounds(southWest, northEast);
          _this.rectangle = new AMap.Rectangle({
            bounds: bounds,
            strokeColor: "red",
            strokeWeight: 6,
            strokeOpacity: 0.5,
            strokeDasharray: [30, 10],
            // strokeStyle还支持 solid
            strokeStyle: "dashed",
            fillColor: "blue",
            fillOpacity: 0.5,
            cursor: "pointer",
            zIndex: 50
          });
          _this.rectangle.setMap(_this.map);
          _this.map.setFitView([_this.rectangle]);
        }
      }
    }
  }
};
</script>

<style lang="scss">
.map_content {
  width: 100%;
  height: 100%;

  .button {
    display: flex;
  }

  .mymap {
    width: 100%;
    height: 500px;
  }
}
</style>
