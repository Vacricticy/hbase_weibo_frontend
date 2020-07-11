<template>
  <div class="">
    <el-card>
      <el-row :gutter="10">
        <el-col :span="3">
          <el-button @click="getAllData" type="primary">获取所有数据</el-button>
        </el-col>
        <el-col :span="3">
          <el-button @click="getFirstLevelDept" type="primary"
            >获取所有一级部门</el-button
          >
        </el-col>
        <el-col :span="9">
          <el-input
            placeholder="请输入父部门row_key获取其直接子部门数据"
            v-model="fatherDeptId"
            class="input-with-select"
            clearable
            @clear="getChildDept"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getChildDept"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="3">
          <el-button @click="addDialogVisible = true" type="primary"
            >添加部门</el-button
          >
        </el-col>
        <el-col :span="4">
          <el-button @click="goBackToHome" type="info"
            >返回主页</el-button
          >
        </el-col>
      </el-row>

      <el-table :data="deptList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="row_key" label="行键"></el-table-column>
        <el-table-column prop="column_family" label="列族"></el-table-column>
        <el-table-column prop="column" label="列名"></el-table-column>
        <el-table-column prop="value" label="值"></el-table-column>
        <el-table-column label="操作">
          <template v-slot:="scope">
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteDept(scope.row.row_key)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>

      <!-- 添加管理员对话框 -->
      <el-dialog title="添加部门" :visible.sync="addDialogVisible" width="30%">
        <el-form ref="addRef" :model="addForm" label-width="80px">
          <el-form-item label="行键">
            <el-input v-model="addForm.row_key"></el-input>
          </el-form-item>
          <el-form-item label="列族">
            <el-input v-model="addForm.column_family"></el-input>
          </el-form-item>
          <el-form-item label="列">
            <el-input v-model="addForm.column"></el-input>
          </el-form-item>
          <el-form-item label="值">
            <el-input v-model="addForm.value"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addDept">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      deptList: [],
      fatherDeptId: "",
      addDialogVisible: false,
      addForm: {
        row_key: "",
        column_family: "",
        column: "",
        value: "",
      },
    };
  },
  created() {
    this.getAllData();
  },
  methods: {
    async getAllData() {
      this.deptList = [];
      const { data: res } = await this.$http.post("getAllData", {});
      console.log(res);
      this.deptList = res;
    },
    async getFirstLevelDept() {
      this.deptList = [];
      const { data: res } = await this.$http.post("getFirstLevelDept", {
        row_key: "0_001",
      });
      console.log(res);
      this.deptList = res;
    },
    async getChildDept() {
      this.deptList = [];
      const { data: res } = await this.$http.post("getChildDept", {
        row_key: this.fatherDeptId,
      });
      console.log(res);
      this.deptList = res;
    },
    async addDept() {
      const { data: res } = await this.$http.post("addDept", this.addForm);
      if (res === 200) {
        this.$message.success("添加成功");
        this.addDialogVisible = false;
        this.deptList = [];
        const { data: res2 } = await this.$http.post("getAllData", {});
        console.log(res2);
        this.deptList = res2;
      } else {
        this.$message.success("添加失败");
      }
    },
    async deleteDept(row_key) {
      const isConfirm = await this.$confirm(
        "此操作将删除该部门, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      // 判断是否取消删除
      if (isConfirm == "cancel") {
        return this.$message({ message: "取消删除成功", duration: 1000 });
      }
      if (isConfirm == "confirm") {
        const { data: res } = await this.$http.post("deleteDept", {
          row_key: row_key,
        });

        if (res === 200) {
          this.$message.success("删除成功");
          this.deptList = [];
          const { data: res2 } = await this.$http.post("getAllData", {});
          console.log(res2);
          this.deptList = res2;
        } else {
          this.$message.success("删除失败");
        }
      }
    },
    goBackToHome(){
      this.$router.push('/login')
    }
  },
};
</script>
<style lang="less" scoped>
.homePage {
  height: 100%;
}
.el-header {
  background-color: #ff2741;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: aliceblue;
  font-size: 23px;
  font-weight: bold;
  letter-spacing: 0.1em;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 10px;
    }
  }
  .avatar {
    width: 45px;
    height: 45px;
    border: 1px solid #e0e0e0;
    border-radius: 50%;
    box-shadow: 0 0 12px #fff;
    padding: 1px;
    background-color: #fff;
  }
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
  }
  .el-button {
    background-color: #ff93a1;
    border: none;
  }
}
.el-aside {
  // background-color: #ff93a1;
  background-color: #fff;
  .el-menu {
    border-right: none;
  }
  .collapse {
    color: aliceblue;
    background-color: #ff93a1;
    text-align: center;
    line-height: 25px;
    letter-spacing: 0.1em;
    cursor: pointer;
  }
}
.el-main {
  background-color: #f8f8f8;
}
.iconfont {
  margin-right: 10px;
}
</style>
