<template>
  <div>
    <div id="container" class="mymap"></div>
  </div>
</template>

<script>
export default {
  props: {
    potionData: {
      type: Array,
      required: false
    },
    type: {
      type: Number, //type 1:编辑,
      required: false
    },
    configObj: {
      type: Object
    }
  },
  data() {
    return {
      map: {},
      lineArr: [],
      marker: {},
      markers: [],
      color: "#66B1FF",
      title: ""
    };
  },
  mounted() {
    //加载地图和相关组件;
    this.loadmap();
  },

  watch: {
    configObj: {
      handler(newName, oldName) {
        this.ivewMap();
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
      this.ivewMap();
    },

    //画线
    ivewMap() {
      let _this = this;
      _this.map.clearMap();

      //根据传过来的数据获取数组格式的path
      let path = [];
      if (this.configObj.selectObj.notes) {
        let len = this.configObj.selectObj.notes.length;
        let pointArr = this.configObj.selectObj.notes;
        for (let i = 0; i < len; i++) {
          let point = [];
          point.push(pointArr[i].longitude, pointArr[i].latitude);
          //console.log(point);
          path.push(point);
        }
      }

      if (path) {
        //console.log("开始画图",path);
        let polyline = new AMap.Polyline({
          path: path,
          isOutline: true,
          outlineColor: "#ffeeff",
          borderWeight: 3,
          strokeColor: "#3366FF",
          strokeOpacity: 1,
          strokeWeight: 6,
          strokeStyle: "solid",
          strokeDasharray: [10, 5],
          lineJoin: "round",
          lineCap: "round",
          zIndex: 50
        });

        //this.polyline = polyline;
        polyline.setMap(_this.map);
        // 缩放地图到合适的视野级别
        _this.map.setFitView([polyline]);
      }
    }
  }
};
</script>

<style lang="scss">
.positionDefin .mymap {
  width: 100%;
  height: 500px;
}
</style>
