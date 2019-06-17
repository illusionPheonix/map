<template>
  <div class="enclosureDefin">
    <div class="left">
      <el-form :inline="true" :model="form" class="demo-form-inline">
        <el-form-item>
          <el-input v-model="form.nodeName" placeholder="请输入名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="search" icon="el-icon-search">搜索围栏</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="add">添加围栏</el-button>
        </el-form-item>
      </el-form>
      <div class="table_cont">
        <el-table :data="tableData" style="width: 100%" height="500">
          <el-table-column fixed="left" prop="name" label="名称" width="100"></el-table-column>
          <el-table-column prop="longitudel" label="最大经度"></el-table-column>
          <el-table-column prop="latitudel" label="最大纬度"></el-table-column>
          <el-table-column prop="longitudes" label="最小经度"></el-table-column>
          <el-table-column prop="latitudes" label="最小纬度"></el-table-column>
          <el-table-column prop="type" label="阈值"></el-table-column>
          <el-table-column prop="alarm_type" label="报警类型"></el-table-column>
          <el-table-column label="操作" width="200" fixed="right">
            <template slot-scope="scope">
              <el-button @click="view(scope.row)" type="text" size="small" icon="el-icon-view">查看</el-button>
              <el-button type="text" size="small" icon="el-icon-edit" @click="edit(scope.row)">编辑</el-button>
              <el-button type="text" size="small" icon="el-icon-delete" @click="del(scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-dialog
        :title="title"
        width="60%"
        :visible.sync="dialogTableVisible"
        v-if="dialogTableVisible"
      >
        <add-enclousre :configObj="configObj" @closedia="closedia"/>
      </el-dialog>
    </div>
    <div class="right">
      <view-map :configObj="configObj"/>
      <!-- <div id="container" class="mymap"></div> -->
    </div>
  </div>
</template>

<script>
import AMap from "AMap"; //在页面中引入高德地图
// 使用excel导出
import JsonExcel from "vue-json-excel";
import { getKeyWord } from "@/assets/js/utils";
import addEnclousre from "./addEnclousre/addEnclousre.vue";
import defineMap from "@/components/defineMap/defineMap.vue";
import viewMap from "./viewMap/viewMap.vue";
import { getNodeTableInfoByUserGroup, deleteNodeTable } from "../../api/api";
export default {
  components: {
    "define-map": defineMap,
    "add-enclousre": addEnclousre,
    downloadExcel: JsonExcel,
    "view-map": viewMap
  },
  data() {
    return {
      form: {
        currentPage: 1,
        nodeName: null,
        pageSize: 100000
      },
      // 配置对象
      configObj: {
        type: 2,
        selectObj: {},
        iOrderID: 2
      },
      Visible: true,
      dialogTableVisible: false,
      tableData: [],
      tableDataAll: []
    };
  },
  mounted() {
    this.getNodeTableInfoByUserGroup();
    //this.configObj.selectObj = this.tableData[0];
  },
  methods: {
    closedia(){
      this.dialogTableVisible = false;
      this.getNodeTableInfoByUserGroup()
    },
    getNodeTableInfoByUserGroup() {
      let _this = this;
      getNodeTableInfoByUserGroup(this.form).then(res => {
        if (res) {
          _this.tableData = res.nodeTableList;
        }
      });
    },
    getAllTableData() {
      this.tableData = [...this.tableDataAll];
    },
    view(row) {
      console.log(row);
      this.configObj.selectObj = row;
      this.configObj.iOrderID = 2;
    },
    search() {
      this.getNodeTableInfoByUserGroup();
      console.log(this.form);
    },
    add() {
      this.dialogTableVisible = true;
      this.title = "添加围栏";
      this.configObj.selectObj = {};
      this.configObj.iOrderID = 0;
    },
    edit(v) {
      this.dialogTableVisible = true;
      this.title = "编辑围栏";
      this.configObj.selectObj = v;
      this.configObj.iOrderID = 1;
    },
    del(v) {
      this.configObj.iOrderID = 2;
      let _this = this;
      this.$confirm("此操作将删除该条数据, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          deleteNodeTable({ id: v.id }).then(res => {
            if (res) {
              this.$message({
                type: "success",
                message: "删除成功!"
              });

              _this.getNodeTableInfoByUserGroup();
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style lang="scss">
.enclosureDefin {
  width: 100%;
  box-sizing: border-box;
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
