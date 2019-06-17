<template>
  <div id="container" class="mymap" @click="addPoint"></div>
</template>

<script>
import { colorTorgb } from "@/assets/js/utils";
export default {
  props: {
    selectObj: {
      type: Object,
      required: false
    },
    iOrderID: {
      type: Number, //type 1:编辑,
      required: false
    }
  },
  data() {
    return {
      map: {},
      marker: {},
      markers: [],
      color: "rgb(89, 162, 239)",
      title: ""
    };
  },
  mounted() {
    this.loadmap(); //加载地图和相关组件;
  },
  watch: {
    selectObj: {
      handler(newVal, oldVal) {
        this.drawPoint(
          [newVal.longitude, newVal.latitude],
          colorTorgb(newVal.color),
          newVal.name
        );
      },
      deep: true
    }
  },

  methods: {
    loadmap() {
      let map = new AMap.Map("container", {
        resizeEnable: true
      });
      this.map = map;

      //编辑地图时  回显位置地点
      if (this.selectObj.longitude) {
        this.color = colorTorgb(this.selectObj.color);
        this.drawPoint(
          [this.selectObj.longitude, this.selectObj.latitude],
          colorTorgb(this.selectObj.color),
          this.selectObj.name
        );
      }
    },
    addPoint() {
      let _this = this;
      console.log(this.color);
      if (_this.iOrderID != 2) {
        //   添加点
        this.map.on("click", function(e) {
          if (_this.marker) {
            _this.map.clearMap();
          }
          let position = [e.lnglat.getLng(), e.lnglat.getLat()];
          _this.$emit("sendLngAlat", {
            lng: e.lnglat.getLng(),
            lat: e.lnglat.getLat()
          });
          _this.drawPoint(position, _this.color, _this.title);
        });
      }
    },

    drawPoint(position, color, title) {
      let map = this.map;
      this.map.clearMap();
      let marker = new AMap.Marker({
        position: position ? position : map.getCenter(),
        content:
          '<div style="background-color: ' +
          color +
          "; height: 24px; width: 24px; border: 1px solid " +
          color +
          "; border-radius: 50%; box-shadow:" +
          color +
          ' 0px 0px 1px;"></div>',
        offset: new AMap.Pixel(-13, -30)
      });
      this.marker = marker;
      marker.setMap(map);

      // 设置鼠标划过点标记显示的文字提示
      marker.setTitle(title);
      this.map.setFitView([marker]);

      // 设置label标签
      // label默认蓝框白底左上角显示，样式className为：amap-marker-label
      if (title) {
        marker.setLabel({
          //修改label相对于maker的位置
          offset: new AMap.Pixel(20, 20),
          content: `<div class='info'>${title}</div>`
        });
      }
    }
  }
};
</script>

<style>
</style>
