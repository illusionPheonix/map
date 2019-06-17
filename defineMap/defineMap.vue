<template>
  <div class="map_content">
    <div class="button" v-if="configObj.iOrderID==1">
      <el-button type="primary" size="small" @click="mapEdit">点击编辑</el-button>
      <el-button type="primary" size="small" @click="rectangleEditor.close()">结束编辑</el-button>
    </div>
    <div class="button" v-if="configObj.iOrderID==0">
      <el-button type="primary" class="button" size="small" @click="drawMap">点击绘制</el-button>
      <el-button type="primary" class="button" size="small" @click="clear">点击清除</el-button>
    </div>

    <div id="container" class="mymap"></div>
  </div>
</template>

<script>
import { compareAscSort } from "@/assets/js/utils";
// type=1 画线
// type=2 画矩形
// type=3 画多边形
// type=4 画圆形
export default {
  props: {
    configObj: {
      type: Object
    }
  },
  data() {
    return {
      map: {},
      polyEditor: {},
      rectangleEditor: {},
      mouseTool: {},
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
  methods: {
    clear() {
      this.map.clearMap();
    },
    handleClear() {
      this.map.clearMap();
      this.loadmap();
    },
    getPath(arr) {
      console.log(arr);
      let path;

      //画线
      if (this.configObj.type == 1) {
        let newArr = [];
        for (let i of arr) {
          let v = `${i.lng},${i.lat}`;
          newArr.push(v);
        }
        this.$emit("sendPath", newArr);

        //画矩形
      } else if (this.configObj.type == 2) {
        path = {
          dMaxLongitude: arr.sort(compareAscSort("lng"))[3].lng,
          dMaxLatitude: arr.sort(compareAscSort("lat"))[3].lat,
          dMinLongitude: arr.sort(compareAscSort("lng"))[0].lng,
          dMinLatitude: arr.sort(compareAscSort("lat"))[0].lat
        };
        this.$emit("sendPath", path);
      } else if (this.configObj.type == 3) {
        let newArr = [];
        for (const v of arr) {
          let obj = {
            dLongitude: v.lng,
            dLatitude: v.lat
          };
          newArr.push(obj);
        }
        this.$emit("sendPath", newArr);
      } else if (this.configObj.type == 4) {
        path = arr;
        this.$emit("sendPath", path);
      }
    },
    getCenter(v) {},
    //画图
    drawMap() {
      let _this = this;
      _this.map.plugin(["AMap.MouseTool"], function() {
        let mouseTool = new AMap.MouseTool(_this.map);
        //监听draw件可获取画好的覆盖物
        mouseTool.on("draw", function(e) {
          if (_this.configObj.type == 4) {
            _this.getPath(e.obj);
          } else {
            _this.getPath(e.obj.getPath());
          }
        });
        _this.clear();
        if (_this.configObj.type == 1) {
          // 画线
          _this.rectangle = mouseTool.polyline({
            strokeColor: "#3366FF",
            strokeOpacity: 1,
            strokeWeight: 6,
            strokeStyle: "solid"
          });
        } else if (_this.configObj.type == 2) {
          // 画矩形
          _this.rectangle = mouseTool.rectangle({
            strokeColor: "red",
            strokeOpacity: 0.5,
            strokeWeight: 6,
            fillColor: "blue",
            fillOpacity: 0.5,
            strokeStyle: "dashed"
          });
        } else if (_this.configObj.type == 3) {
          // 多边形
          _this.rectangle = mouseTool.polygon({
            strokeColor: "#FF33FF",
            strokeOpacity: 1,
            strokeWeight: 6,
            strokeOpacity: 0.2,
            fillColor: "#1791fc",
            fillOpacity: 0.4,
            strokeStyle: "solid"
          });
        } else if (_this.configObj.type == 4) {
          // 画圆
          _this.rectangle = mouseTool.circle({
            strokeColor: "#FF33FF",
            strokeOpacity: 1,
            strokeWeight: 6,
            strokeOpacity: 0.2,
            fillColor: "#1791fc",
            fillOpacity: 0.4,
            strokeStyle: "solid"
          });
        }
      });
    },

    save() {
      console.log(this.lineArr);
    },
    loadmap() {
      console.log(this.configObj);

      let map = new AMap.Map("container", {
        center: [116.395577, 39.892257],
        zoom: 14
      });
      this.map = map;
      if (this.configObj) {
        this.mapEdit();
      }
    },

    // 路径
    editPolyline() {
      let _this = this;
      let path = [];

      if (this.configObj.selectObj.notes) {
        let len = this.configObj.selectObj.notes.length;
        let pointArr = this.configObj.selectObj.notes;
        for (let i = 0; i < len; i++) {
          let point = [];
          point.push(pointArr[i].longitude, pointArr[i].latitude);
          console.log(point);
          path.push(point);
        }
      }

      this.polyline = new AMap.Polyline({
        path: path,
        isOutline: true,
        outlineColor: "#ffeeff",
        borderWeight: 3,
        strokeColor: "#3366FF",
        strokeOpacity: 1,
        strokeWeight: 6,
        // 折线样式还支持 'dashed'
        strokeStyle: "solid",
        // strokeStyle是dashed时有效
        strokeDasharray: [10, 5],
        lineJoin: "round",
        lineCap: "round",
        zIndex: 50
      });

      this.polyline.setMap(this.map);
      // 缩放地图到合适的视野级别
      this.map.setFitView([this.polyline]);
      var polyEditor = new AMap.PolyEditor(this.map, this.polyline);
      polyEditor.open();
      this.rectangleEditor = polyEditor;
      polyEditor.on("end", function(event) {
        console.log(event.target);
        _this.getPath(_this.polyline.getPath());
        // event.target 即为编辑后的折线对象
      });
    },

    editRectangle() {
      let _this = this;
      let selectObj = this.configObj.selectObj;
      let southWest = new AMap.LngLat(
        selectObj.longitudes,
        selectObj.latitudes
      );
      let northEast = new AMap.LngLat(
        selectObj.longitudel,
        selectObj.latitudel
      );
      let bounds = new AMap.Bounds(southWest, northEast);
      let rectangle = new AMap.Rectangle({
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
      rectangle.setMap(this.map);
      _this.rectangle = rectangle;
      // 缩放地图到合适的视野级别

      this.map.setFitView([rectangle]);
      this.rectangleEditor = new AMap.RectangleEditor(this.map, rectangle);
      this.rectangleEditor.open();
      this.rectangleEditor.on("end", function(event) {
        console.log(event.target);
        _this.getPath(_this.rectangle.getPath());
      });
    },
    mapEdit() {
      // console.log();
      if (this.configObj) {
        this.map.clearMap();
      }
      if (this.configObj.type == 1) {
        this.editPolyline();
      } else if (this.configObj.type == 2) {
        this.editRectangle();
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
    height: 50vh;
  }
}
</style>
