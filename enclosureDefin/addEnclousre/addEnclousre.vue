<template>
  <div class="formContent">
    <div class="left">
      <define-map :configObj="configObj" @sendPath="getPath"/>
      <!-- <define-map  @sendPath="getPath"  :configObj="configObj" /> -->
      <map/>
    </div>
    <div class="right">
      <el-form
        :model="form"
        :rules="rules"
        ref="ruleForm"
        :inline="true"
        class="demo-form-inline"
        label-position="left"
      >
        <el-form-item v-if="false" label="围栏编号">
          <el-input v-model="form.id" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="围栏名称" prop="name">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="类型" prop="alarm_type">
          <el-select v-model="form.alarm_type" placeholder="请选择">
            <el-option :label="v" :value="v" v-for="(v,i) in sAlarmType" :key="i"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="阀值" prop="type">
          <el-input-number
            v-model="form.type"
            controls-position="right"
            @change="handleChange"
            :min="1"
            :max="10"
          ></el-input-number>
        </el-form-item>

        <div class="flex_item">
          <el-form-item label="最小经度" prop="dMinLatitude">
            <el-input v-model="form.dMinLatitude" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="最大经度" prop="dMinLatitude">
            <el-input v-model="form.dMaxLongitude" :disabled="true"></el-input>
          </el-form-item>
        </div>
        <div class="flex_item">
          <el-form-item label="最小纬度" prop="dMinLongitude">
            <el-input v-model="form.dMinLongitude" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="最大纬度" prop="dMaxLatitude">
            <el-input v-model="form.dMaxLatitude" :disabled="true"></el-input>
          </el-form-item>
        </div>

        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">保存</el-button>
          <!-- <el-button type="primary" @click="close">关闭</el-button> -->
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
import defineMap from "@/components/defineMap/defineMap.vue";

import { saveNodeTable, updateNodeTable } from "@/api/api";

export default {
  components: {
    "define-map": defineMap
  },
  props: {
    configObj: {
      type: Object
    }
  },
  data() {
    return {
      //  0-添加节点   1-更新节点   2-删除节点,
      //   iOrderID: 0,
      form: {
        id: null,
        type: "",
        dMaxLongitude: "",
        dMaxLatitude: "",
        dMinLongitude: "",
        dMinLatitude: "",
        name: "",
        alarm_type: ""
      },
      type: 2,
      sAlarmType: [
        "驶入围栏报警",
        "驶出围栏报警",
        "驶入驶出围栏报警",
        "禁止围栏报警",
        "围栏超速报警"
      ],
      rules: {
        sName: [{ required: true, message: "请输入围栏名称", trigger: "blur" }],
        sAlarmType: [
          { required: true, message: "请输入类型", trigger: "blur" }
        ],
        type: [{ required: true, message: "请输入阀值", trigger: "blur" }],
        dMaxLongitude: [
          { required: true, message: "请绘制区域", trigger: "blur" }
        ],
        dMaxLatitude: [
          { required: true, message: "请绘制区域", trigger: "blur" }
        ],
        dMinLongitude: [
          { required: true, message: "请绘制区域", trigger: "blur" }
        ],
        dMinLatitude: [
          { required: true, message: "请绘制区域", trigger: "blur" }
        ]
      }
    };
  },
  mounted() {
    if (this.configObj.iOrderID == 1) {
      let selectObj = this.configObj.selectObj;
      this.form = {
        id: selectObj.id,
        type: selectObj.type,
        iOrderID: this.iOrderID,
        dMaxLongitude: selectObj.longitudel,
        dMaxLatitude: selectObj.latitudel,
        dMinLongitude: selectObj.longitudes,
        dMinLatitude: selectObj.latitudes,
        name: selectObj.name,
        alarm_type: selectObj.alarm_type
      };
    }else if(this.configObj.iOrderID == 0){
      this.configObj = {};
    }
  },
  methods: {
    handleChange(value) {
      console.log(value);
    },
    submitForm(formName) {
      this.$emit("closedia");
      this.$refs[formName].validate(valid => {
        if (valid) {
          if (this.configObj.iOrderID == 0) {
            let data = this.form;
            saveNodeTable(data).then(res => {});
          } else if (this.configObj.iOrderID == 1) {
            console.log(this.$store.state);
            let data = this.form;
            data.delflag = false;
            data.userid = 1;
            updateNodeTable(data).then(res => {
              console.log(res);
            });
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    getPath(v) {
      this.form.dMaxLongitude = v.dMaxLongitude;
      this.form.dMaxLatitude = v.dMaxLatitude;
      this.form.dMinLongitude = v.dMinLongitude;
      this.form.dMinLatitude = v.dMinLatitude;
      console.log(v);
    },
    // loadmap() {
    //   this.map = new AMap.Map("container", {
    //     center: [116.434381, 39.898515],
    //     zoom: 14
    //   });
    //   let mouseTool = new AMap.MouseTool(this.map);
    // },

    save() {
      console.log(this.form);
    },
    close() {}
  }
};
</script>

<style  lang="scss">
.formContent {
  .small {
    width: 100px;
  }
  .el-form-item__label {
    width: 80px;
  }
  width: 100%;
  height: 100%;
  display: flex;
  .left {
    flex: 1;
  }
  .right {
    flex: 1;
    box-sizing: border-box;
    padding: 28px;
    .flex_item {
      // display: flex;
    }
  }
  #addMap {
    width: 100%;
    height: 100%;
  }
}
</style>
