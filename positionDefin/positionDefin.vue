<template>
  <div class="positionDefin">
    <div class="left">
      <el-form :inline="true" :model="formInline" class="demo-form-inline">
        <el-form-item>
          <el-input v-model="formInline.keyword" placeholder="请输入名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="search" icon="el-icon-search">搜索位置</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="add">添加位置</el-button>
        </el-form-item>
      </el-form>
      <div class="table_cont">
        <el-table :data="tableData" style="width: 100%" height="500">
          <el-table-column fixed="left" prop="name" label="名称" width="100"></el-table-column>
          <el-table-column prop="longitude" label="经度"></el-table-column>
          <el-table-column prop="latitude" label="纬度"></el-table-column>
          <el-table-column prop="sAlarmType" label="操作" width="200" fixed="right">
            <template slot-scope="scope">
              <el-button @click="view(scope.row)" type="text" size="small" icon="el-icon-view">查看</el-button>
              <el-button type="text" size="small" icon="el-icon-edit" @click="edit(scope.row)">编辑</el-button>
              <el-button type="text" size="small" icon="el-icon-delete" @click="del(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-dialog :title="title" width="60%" :visible.sync="dialogTableVisible" v-if="dialogTableVisible">
        <add-position @closedia='closedia' :selectObj="selectObj" :iOrderID="iOrderID"/>
      </el-dialog>
    </div>
    <div class="right">
      <map-point :selectObj="selectObj" :iOrderID="iOrderID" />
    </div>
  </div>
</template>

<script>
import AMap from "AMap"; //在页面中引入高德地图
import addPosition from "./addPosition/addPosition.vue";
import mapPoint from "./mapPoint/mapPoint.vue";
import { getKeyWord } from "@/assets/js/utils";
import {getPointList,deletePoint} from '@/api/api'
export default {
  components: {
    "add-position": addPosition,
    "map-point":mapPoint
  },
  data() {
    return {
      formInline: {
        keyword: null
      },
      iOrderID: 2,
      dialogTableVisible: false,
      selectObj: {},
      tableData: [],
      tableDataAll: []
    };
  },
  mounted() {
    this.getPointListFn();
  },
  methods: {
    closedia(){
      this.dialogTableVisible = false;
      this.getPointListFn();
    },
    getPointListFn() {
      let data = {
        currentPage:1,
        pageSize:999,
        name:this.formInline.keyword
      }
      getPointList(data).then(res => {
        this.tableData = res.list
      })
      // setTimeout(() => {
      //   this.tableData = arr;
      //   // 缓存拿到的本地数据
      //   this.tableDataAll = [...arr];
      // }, 300);
    },
    view(row) {
      this.selectObj = row;
      this.iOrderID = 2;
    },
    getAllTableData() {
      this.tableData = [...this.tableDataAll];
    },
    search() {
      this.getPointListFn()
    },
    add() {
      this.dialogTableVisible = true;
      this.title = "新增位置";
      this.iOrderID = 0;
    },
    edit(v) {
      this.dialogTableVisible = true;
      this.title = "编辑位置";
      this.selectObj = v;
      this.iOrderID = 1;
    },
    del(v) {
      this.iOrderID = 2;
      this.$confirm("此操作将删除该条数据, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          deletePoint({id:v.id}).then(res => {
            this.$message.success('删除成功');
            this.getPointListFn()
          })
          console.log(v, this.iOrderID);

          // this.form = {
          //   iRequestID: 0,
          //   iPositionId: v.iNodeID,
          //   sVelocity: v.sVelocity,
          //   iOrderID: this.iOrderID,
          //   dMaxLongitude: v.dMaxLongitude,
          //   dMaxLatitude: v.dMaxLatitude,
          //   dMinLongitude: v.dMinLongitude,
          //   dMinLatitude: v.dMinLatitude,
          //   sName: v.sName,
          //   sAlarmType: v.sAlarmType
          // };
          // 在此处向后台发起删除指令
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },

  }
};
</script>

<style lang="scss">
.positionDefin {
  width: 100%;
  box-sizing: border-box;
  .amap-marker-label{
    border:none;
  }
  .info{
    border: none;
  }
  .el-dialog__header {
    // width: 800px;
    background: #fff;
  }
  .el-dialog__body {
    height: 500px;
    overflow-y: scroll;
    // width: 800px;
    background: #fff;
  }

  .el-dialog__body::-webkit-scrollbar {
    display: none;
  }
  display: flex;
  box-sizing: border-box;
  .left {
    flex: 1;
  }
  .table_cont {
    width: 100%;
    overflow-x: hidden;
  }
  .right {
    flex: 1;
    width: 500px;
    // height: 100%;
  }
  .mymap {
    width: 100%;
    height: 100%;
  }
}
</style>
