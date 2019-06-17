<template>
  <div class="addPosition">
    <div class="left">
      <map-point :type="1" @sendLngAlat="getLngAlat" :iOrderID="iOrderID" :selectObj="selectObj"/>
    </div>
    <div class="right">
      <el-form label-position="right" label-width="80px" :model="positionData">
        <el-form-item label="名称">
          <el-input v-model="positionData.name"></el-input>
        </el-form-item>
        <el-form-item label="经度">
          <el-input v-model="positionData.longitude" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="纬度">
          <el-input v-model="positionData.latitude" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="选取颜色">
          <el-color-picker color-format="rgb" v-model="positionData.color"></el-color-picker>
          <!-- <input type="color" @change="getColor" class="color-picker" value="#fffff"> -->
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="small" @click="save">保存</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import mapPoint from "../mapPoint/mapPoint.vue";
// import { Photoshop } from "vue-color";
import { savePoint } from "@/api/api";
import { toRGB,colorTorgb } from "@/assets/js/utils";

export default {
  props: {
    selectObj: {
      type: Object,
      required: true
    },
    potionData: {
      type: Object,
      required: false
    },
    iOrderID: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      colors: "#333",
      positionData: {
        iRequestID: 0,
        iOrderID: this.iOrderID, //操作类型,    0-添加位置    1-更新位置    2-删除位置
        iPointID: 0, //需要操作的位置id
        longitude: 0, //经度      删除位置是可以传0
        latitude: 0, //纬度	     删除位置是可以传0
        name: "", //位置名称  删除位置是可以传""
        iType: 0,
        color: "rgb(89, 162, 239)"
      }
    };
  },

  mounted() {
    //iOrderID为1时 修改位置 表单回显 下一层地图回显
    if (this.iOrderID == 1) {
      this.positionData.longitude = this.selectObj.longitude;
      this.positionData.latitude = this.selectObj.latitude;
      this.positionData.name = this.selectObj.name;
      this.positionData.color = colorTorgb(this.selectObj.color)
    }
  },

  methods: {
    save() {
      //判断是否填写
      if (this.positionData.iOrderID == 2) {
        if (
          this.positionData.longitude == 0 ||
          this.positionData.latitude == 0
        ) {
          this.$message.error("请选择位置!");
          return;
        }
      }
      if (!this.positionData.name) {
        this.$message.error("请填写位置名称!");
        return;
      }

      //获取表单数据传给后台
      let data = {
        name: this.positionData.name,
        longitude: this.positionData.longitude,
        latitude: this.positionData.latitude,
        color: toRGB(this.positionData.color)
      };
      //如果为修改 多传一个id参数
      if (this.selectObj.id) {
        data.id = this.selectObj.id;
      }
      savePoint(data).then(res => {
        if (res.success) {
          this.$message.success("操作成功");
          this.$emit("closedia");
        }
      });
    },

    //根据地图传过来的经纬度显示到表单
    getLngAlat(v) {
      if (v) {
        this.positionData.longitude = v.lng;
        this.positionData.latitude = v.lat;
      }
    }
    // getColor(e) {
    //   console.log(e.target.value);
    //   this.positionData.iColor = e.target.value;
    // }
  },

  components: {
    "map-point": mapPoint
    // "color-picker": Photoshop
  }
};
</script>

<style lang="scss" >
.addPosition {
  width: 100%;
  height: 100%;
  display: flex;
  .color-picker {
    border: none;
    color: aquamarine;
    background: #fff;
    outline: none;
  }

  .left {
    // width: 300px;
    height: 100%;
    flex: 1.5;
  }
  .right {
    height: 100%;
    flex: 1;
  }
}
</style>
