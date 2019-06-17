<template>
  <div class="positionDefin">
    <div class="left">
      <el-form :inline="true" :model="formInline" class="demo-form-inline">
        <el-form-item>
          <el-input v-model="formInline.keyword" placeholder="请输入名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="search" icon="el-icon-search">搜索区域</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="add">添加区域</el-button>
        </el-form-item>
      </el-form>
      <div class="table_cont">
        <el-table :data="tableData" style="width: 100%" height="500">
          <el-table-column fixed="left" prop="pathName" label="线路名称" width="100"></el-table-column>
          <el-table-column :formatter="formatType" prop="pathType" label="线路类型"></el-table-column>
          <el-table-column prop="addField" label="附加参数"></el-table-column>
          <el-table-column prop="marks" label="备注"></el-table-column>
          <el-table-column label="操作" width="200" fixed="right">
            <template slot-scope="scope">
              <el-button @click="view(scope.row)" type="text" size="small" icon="el-icon-view">查看</el-button>
              <el-button type="text" size="small" icon="el-icon-edit" @click="edit(scope.row)">编辑</el-button>
              <el-button
                type="text"
                size="small"
                icon="el-icon-delete"
                @click="del(scope.row.pathId)"
              >删除</el-button>
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
        <add-line @closedia="closedia" :configObj="configObj"/>
      </el-dialog>
    </div>
    <div class="right">
      <my-map :configObj="configObj"/>
    </div>
  </div>
</template>

<script>
import AMap from "AMap"; //在页面中引入高德地图
import addLine from "./addLine/addLine.vue";
import map from "./map/map.vue";
import { getKeyWord } from "@/assets/js/utils";
import { getPathList, deletePathInfo } from "@/api/api";

export default {
  components: {
    "add-line": addLine,
    "my-map": map
  },
  data() {
    return {
      formInline: {
        keyword: null
      },
      // 配置对象
      configObj: {
        type: 1,
        selectObj: {},
        iOrderID: 1
      },
      iOrderID: 0,
      dialogTableVisible: false,
      selectObj: {},
      tableData: [],
      tableDataAll: {}
    };
  },
  computed: {},
  created() {
    this.getLinePath();
  },
  methods: {
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
    getLinePath(arr) {},
    /*
getDataSuccess() {
      let arr = [
        {
          iPathID: 1,
          sPathName: "线路名称",
          iType: 4,
          iUserID: "用户id",
          iLimitValue: 44,
          sMarks: "备注",
          sAddField1: "0", //附带参数  山区禁行的持续时间
          sRunTime: "runtime",
          sMaxX: "maxx",
          sMinX: "minx",
          sMaxY: "maxy",
          sMinY: "miny"
        }
      ];
      for (const iterator of arr) {
        iterator.iType = this.getTypeName(iterator.iType);
      }
      setTimeout(() => {
        this.tableData = arr;
        // 缓存拿到的本地数据
        this.tableDataAll = [...arr];
      }, 300);
    }, 
*/
    view(row) {
      this.configObj.selectObj = row;
      this.configObj.iOrderID = 2;
    },
    getAllTableData() {
      this.tableData = [...this.tableDataAll];
    },
    search() {
      // console.log(this.formInline.name);
      // if (this.formInline.name) {
      //   this.tableData = getKeyWord(
      //     this.tableData,
      //     this.formInline.name,
      //   );
      // }
      this.getPathListFn();
    },
    add() {
      this.dialogTableVisible = true;
      this.title = "新增区域";
      this.configObj.iOrderID = 0;
      this.configObj.selectObj = {};
    },
    edit(v) {
      this.dialogTableVisible = true;
      this.title = "编辑区域";
      this.configObj.selectObj = v;
      this.configObj.iOrderID = 1;
      console.log(v);
    },
    del(id) {
      console.log(id);
      this.configObj.iOrderID = 2;
      this.$confirm("此操作将删除该条数据, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          // for (let i = 0; i < this.tableData.length; i++) {
          //   let element = this.tableData[i];
          //   if (element.iNodeID == v.iNodeID) {
          //     this.tableData.splice(i, 1);
          //     this.tableDataAll.splice(i, 1);
          //   }
          //}
          deletePathInfo({ pathId: id }).then(res => {
            this.$message.success("删除成功");
            this.getPathListFn();
          });
        })

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

        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    closedia() {
      this.dialogTableVisible = false;
      this.getPathListFn();
    },
    getPathListFn() {
      getPathList({
        pathName: this.formInline.keyword,
        currentPage: 1,
        pageSize: 999
      }).then(res => {
        this.tableData = res.list;
      });
    },

    /* 路线类型根据数字返回内容 */
    formatType: function(row, cloumn) {
      switch (row.pathType) {
        case 1:
          return "分段限速";
          break;
        case 2:
          return "路线偏移";
          break;
        case 3:
          return "区域监测报警";
          break;
        case 4:
          return "中途返回检测";
          break;
        case 6:
          return "排班区域检测";
          break;
        case 10:
          return "混泥土卸料区域";
          break;
        case 47:
          return "山区公路禁行";
          break;
      }
    }
  },

  created() {
    this.getPathListFn();
  }
};
</script>

<style lang="scss">
.positionDefin {
  width: 100%;
  box-sizing: border-box;
  .amap-marker-label {
    border: none;
  }
  .info {
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
