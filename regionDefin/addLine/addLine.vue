<template>
  <div class="addLine">
    <div class="left">
      <define-map :configObj="configObj" @sendPath="getPath"/>
    </div>
    <div class="right">
      <el-form label-position="right" label-width="80px" :model="form">
        <el-form-item label="区域名称">
          <el-input v-model="form.pathName"></el-input>
        </el-form-item>
        <el-form-item label="区域类型">
          <el-select v-model="form.pathType" placeholder="请选择区域类型" @change="getSelectType">
            <el-option :label="getTypeName(v)" :value="v" v-for="(v,i) in selectItem" :key="i"></el-option>
          </el-select>
        </el-form-item>
        <div class="viewsAddField1">
          <el-form-item label="超速阀值" v-if="selectV==1">
            <el-input-number v-model="form.iLimitValue" controls-position="right" :min="0"></el-input-number>
          </el-form-item>
          <el-form-item label="附加参数" v-if="selectV===2||selectV===3">
            <el-select v-model="form.addField" placeholder="请选择" @change="getSelectType">
              <el-option value="进入线路报警" label="进入线路报警"></el-option>
              <el-option value="偏离线路报警" label="偏离线路报警"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="报警下发信息内容">
            <el-input v-model="form.marks" type="textarea"></el-input>
          </el-form-item>
        </div>
      </el-form>

      <div class="border-top" v-if="selectV!==6&&selectV!==10&&selectV!==47">
        <el-form :inline="true" :model="lngAndlat" class="demo-form-inline">
          <div>
            <el-form-item label="是否附加">
              <el-radio-group v-model="addition">
                <el-radio label="是">是</el-radio>
                <el-radio label="否">否</el-radio>
              </el-radio-group>
            </el-form-item>
          </div>
          <el-form-item label="开始经度">
            <el-input disabled v-model="lngAndlat.sMinX" type="number" class="input_num"></el-input>
          </el-form-item>
          <el-form-item label="开始纬度">
            <el-input disabled v-model="lngAndlat.sMinY" type="number" class="input_num"></el-input>
          </el-form-item>
          <el-form-item label="结束经度">
            <el-input disabled v-model="lngAndlat.sMaxX" type="number" class="input_num"></el-input>
          </el-form-item>
          <el-form-item label="结束纬度">
            <el-input disabled v-model="lngAndlat.sMaxY" type="number" class="input_num"></el-input>
          </el-form-item>
        </el-form>
        <el-button type="primary" size="small" @click="submit">保存</el-button>
        <!-- <el-button type="danger" size="small">退出</el-button> -->
      </div>
    </div>
  </div>
</template>

<script>
import defineMap from "@/components/defineMap/defineMap.vue";
import { savePathInfo } from "@/api/api";

export default {
  props: {
    configObj: {
      type: Object
    }
  },
  data() {
    return {
      pathId:null,
      addition: "",
      selectV: "",
      form: {
        pathName: "",
        pathType: "",
        iLimitValue: 0,
        addField: "",
        marks: ""
        // iRequestID: 0,
        // iOrderID: 0, // 0-添加线路   1-更新线路  2-删除线路
        // iLineID: 2223,
        // iPathType: 4,
        // sRunTime: "runtime",
      },
      pathInfo: [],
      lngAndlat: {
        sMaxX: "",
        sMinX: "",
        sMaxY: "",
        sMinY: ""
      },

      selectItem: [1, 2, 3, 4, 6, 10, 47]
    };
  },
  components: {
    "define-map": defineMap
  },
  computed: {},
  mounted() {
    //判断下拉框显示隐藏
    this.selectV = this.getTypeName(this.form.iPathType);

    //传过来的iO为1  编辑地图
    if (this.configObj.iOrderID == 1) {
      console.log(this.configObj);
      this.form = this.configObj.selectObj;
      this.pathId = this.configObj.selectObj.pathId;
      this.lngAndlat.sMaxX = this.configObj.selectObj.dMaxLongitude;
      this.lngAndlat.sMinX = this.configObj.selectObj.dMinLongitude;
      this.lngAndlat.sMaxY = this.configObj.selectObj.dMaxLatitude;
      this.lngAndlat.sMinY = this.configObj.selectObj.dMinLatitude;
    }
    this.selectV = this.getTypeName(this.form.iPathType);
  },
  methods: {
    getSelectType(e) {
      console.log(e);
    },
    getPath(v) {
      this.lngAndlat.sMinX = v[0].split(",")[0];
      this.lngAndlat.sMaxX = v[0].split(",")[1];
      this.lngAndlat.sMaxY = v[v.length - 1].split(",")[0];
      this.lngAndlat.sMinY = v[v.length - 1].split(",")[1];
      //console.log(v, "v");
      v.map(ele => {
        let obj = {};
        obj.longitude = ele.split(",")[0];
        obj.latitude = ele.split(",")[1];
        this.pathInfo.push(obj);
      });
      console.log(this.pathInfo);
    },
    getTypeName(v) {
      if (v === 1) {
        return "分段限速";
      } else if (v === 2) {
        return "路线偏移";
      } else if (v === 3) {
        return "区域监测报警";
      } else if (v === 4) {
        return "中途返回检测";
      } else if (v === 6) {
        return "排班区域检测";
      } else if (v === 10) {
        return "混泥土卸料区域";
      } else if (v === 47) {
        return "山区公路禁行";
      }
    },

    submit() {
      let data = {
        pathName: this.form.pathName,
        pathType: this.form.pathType,
        userId: 1,
        marks: this.form.marks,
        addField: this.addition,
        pathDetails: this.pathInfo
      };
      if(this.pathId){
        data.pathId = this.pathId
      }
      savePathInfo(data).then(res => {
        if (res.success) {
          this.$emit("closedia");
        }
      });
    }
  }
};
</script>

<style lang="scss">
.addLine {
  width: 100%;
  height: 100%;
  display: flex;
  .left {
    flex: 1.5;
  }
  .right {
    flex: 1;
    .viewsAddField1 {
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      // padding-left: 28px;
    }
    .border-top {
      border-top: 1px solid #ccc;
      margin-left: 10px;
      .input_num {
        width: 100px;
      }
    }
  }
}
</style>
