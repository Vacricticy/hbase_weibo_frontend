<template>
  <div class="container">
    <div class="left">
      <!-- 用户信息展示区 -->
      <el-card class="card1">
        <el-row class="title">
          <div class="picture">
            <img src="../assets/liuyaren.png" alt="" />
          </div>
          <div>
            <!-- 222 -->
            <el-row>
              <span class="underlinetext"
                >用户id:{{ currentUser.row_key }}
              </span>
            </el-row>
            <el-row>
              <span class="underlinetext">
                昵称:{{ currentUser.info_nickname }}
              </span>
            </el-row>
            <el-row>
              <span class="underlinetext">
                性别:{{ currentUser.info_gender }}
              </span>
            </el-row>
          </div>
        </el-row>

        <el-row class="follow">
          <el-button @click="getFollowingUsers" type="primary"
            >关注 {{ followingNum }}</el-button
          >
          <el-button @click="getFollowers" type="primary"
            >粉丝 {{ followersNum }}</el-button
          >
        </el-row>
      </el-card>

      <!-- 搜索用户区域 -->
      <el-card class="card3">
        <el-row>
          <el-input
            placeholder="请输入用户昵称查找用户"
            v-model="searchUserForm.info_nickname"
            class="input-with-select"
            clearable
            @clear="clearSearchResult"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchUser"
            ></el-button>
          </el-input>
        </el-row>
        <el-row>
          <el-table :data="searchResult" border stripe>
            <el-table-column
              prop="info_nickname"
              label="昵称"
            ></el-table-column>
            <!-- <el-table-column label="昵称" width="150px">
            <template v-slot:="scope">
              <el-button
                type="danger"
                size="mini"
                @click="seeDeatil(scope.row.row_key, scope.row)"
                >{{scope.row.info_nickname}}</el-button
              >
            </template>
          </el-table-column> -->

            <el-table-column label="状态" width="105px">
              <template v-slot:="scope">
                <el-button
                  type="primary"
                  size="mini"
                  @click="
                    changeFollowState(
                      scope.row.row_key,
                      scope.row.state,
                      scope.row
                    )
                  "
                  >{{ scope.row.state }}</el-button
                >
              </template>
            </el-table-column>
            <el-table-column label="操作" width="105px">
              <template v-slot:="scope">
                <el-button
                  type="danger"
                  size="mini"
                  @click="seeDeatil(scope.row.row_key, scope.row)"
                  >查看主页</el-button
                >
              </template>
            </el-table-column>
          </el-table>
          <el-pagination
            @size-change="handleSizeChange2"
            @current-change="handleCurrentChange2"
            :current-page="pageNo2"
            :page-sizes="[1, 2, 3]"
            :pager-count="pagerCount"
            :page-size="pageSize2"
            layout="total, sizes, prev, pager, next"
            :total="total2"
          >
          </el-pagination>
        </el-row>
      </el-card>
    </div>

    <!-- 关注列表和粉丝列表展示区 -->
    <div class="right">
      <el-card class="card2">
        <el-row type="flex" justify="space-between">
          <el-col :span="4">
            <el-tag>{{ showmessage }}</el-tag>
          </el-col>
          <el-col :span="3.5">
            <el-tag @click="goBackToMyPage" style="cursor:pointer"
              >返回我的主页</el-tag
            >
          </el-col>
        </el-row>
        <el-row>
          <!-- 从关注列表或粉丝列表搜索结果 -->
          <el-input
            :placeholder="inputMessage"
            v-model="searchFromList_nickname"
            class="input-with-select"
            clearable
            @clear="recoverData"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchFromList"
            ></el-button>
          </el-input>
        </el-row>
        <el-table :data="userList" border stripe>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column prop="row_key" label="用户id"></el-table-column>
          <el-table-column prop="info_nickname" label="昵称"></el-table-column>
          <el-table-column prop="info_gender" label="性别"></el-table-column>
          <el-table-column label="状态">
            <template v-slot:="scope">
              <el-button
                type="primary"
                size="mini"
                @click="
                  changeFollowState(
                    scope.row.row_key,
                    scope.row.state,
                    scope.row
                  )
                "
                >{{ scope.row.state }}</el-button
              >
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template v-slot:="scope">
              <el-button
                type="danger"
                size="mini"
                @click="seeDeatil(scope.row.row_key, scope.row)"
                >查看主页</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pageNo"
          :page-sizes="[1, 3, 5, 7]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </el-card>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 当前页面显示的用户列表
      userList: [],
      requestUser: {
        row_key: "100001",
      },
      searchUserForm: {},
      login_row_key: "100001",
      nickname: "LiuXiaoKang",
      gender: "男",
      showmessage: "",
      state: "",
      followersNum: 0,
      followingNum: 0,
      //   当前显示的用户的信息
      currentUser: {
        row_key: "100001",
        info_nickname: "LiuXiaoKang",
        info_gender: "男",
      },
      //   登录用户的信息
      login_user: {
        row_key: "100001",
        info_nickname: "LiuXiaoKang",
        info_gender: "男",
      },
      //   搜索到的用户数组
      searchResult: [],
      // 标记当前页面是否为主页
      flag: 1,
      // 输入框中的提示内容
      inputMessage: "请输入用户昵称查找你关注的用户",
      // 从列表搜索数据时的输入的昵称
      searchFromList_nickname: "",
      // 从列表中搜索到的结果
      searchFromList_result: [],
      // 中间结果，用于保存之前的完整数据
      middleList: [],
      pageSize: 7,
      pageNo: 1,
      total: 0,
      pageSize2: 3,
      pageNo2: 1,
      total2: 0,
      middleTotal: 0,
      // 用于分页
      pageTotalList: [],
      // 用于分页2
      pageTotalList2: [],
      pagerCount: 3,
    };
  },
  created() {
    this.getUserData();
  },
  methods: {
    // 获取主页中关注列表，关注数，粉丝数
    async getUserData() {
      this.userList = [];
      this.showmessage = "关注列表";
      this.$http.post("getFollowingUsers", this.requestUser).then((value) => {
        const res = value.data;
        // this.userList = res;
        this.followingNum = res.length; //设置关注数
        this.total = res.length;
        this.pageTotalList = res; //用于分页
        this.handleSizeChange(this.pageSize);
      });

      this.$http.post("getFollowers", this.requestUser).then((value) => {
        const res = value.data;
        this.followersNum = res.length; //设置粉丝数
      });
    },
    // 点击获取关注列表
    async getFollowingUsers() {
      this.inputMessage = "请输入用户昵称查找你关注的用户";
      this.userList = [];
      this.showmessage = "关注列表";
      //   this.requestUser.row_key = this.row_key;
      const { data: res } = await this.$http.post(
        "getFollowingUsers",
        this.requestUser
      );
      console.log(res);
      // this.userList = res;

      this.total = res.length;
      this.pageTotalList = res; //用于分页
      this.handleSizeChange(this.pageSize);
    },

    // 点击获取粉丝列表
    async getFollowers() {
      // console.log(this.requestUser);
      this.inputMessage = "请输入用户昵称查找关注你的粉丝";
      this.userList = [];
      this.showmessage = "粉丝列表";
      const { data: res } = await this.$http.post(
        "getFollowers",
        this.requestUser
      );
      console.log(res);
      // this.userList = res;
      this.total = res.length;
      this.pageTotalList = res; //用于分页
      this.handleSizeChange(this.pageSize);
    },
    // 查看用户详情页面
    seeDeatil(row_key, currentUser) {
      this.requestUser.row_key = row_key;
      this.getUserData();
      this.currentUser = currentUser;
      flag = 0;
    },
    // 返回我的主页
    goBackToMyPage() {
      this.requestUser.row_key = this.login_user.row_key;
      this.getUserData();
      this.currentUser = this.login_user;
      flag = 1;
    },
    // 按照昵称搜索用户
    async searchUser() {
      // 传入本人的id，方便查询搜索到的用户是否关注了登录用户
      this.searchUserForm.row_key = this.login_user.row_key;
      const { data: res } = await this.$http.post(
        "searchUser",
        this.searchUserForm
      );
      console.log(res);
      // this.searchResult = res;
      this.total2 = res.length;
      this.pageTotalList2 = res;
      this.handleSizeChange2(this.pageSize2);
    },
    // 修改关注状态
    async changeFollowState(row_key, state, row) {
      if (state === "关注") {
        // alert("关注成功");
        const { data: res } = await this.$http.post("addFollowing", {
          login_user_row_key: this.login_user.row_key,
          search_row_key: row_key,
        });
        console.log(res);
        if (res === 999) {
          this.$message.success("关注成功");
          row.state = "已关注";
          // 刷新当前正在显示的主页
          this.seeDeatil(this.currentUser.row_key, this.currentUser);
        }
      } else {
        const isConfirm = await this.$confirm("是否取消关注?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }).catch((err) => err);
        // 判断是否取消删除
        if (isConfirm == "cancel") {
        }
        if (isConfirm == "confirm") {
          const { data: res } = await this.$http.post("unFollowing", {
            login_user_row_key: this.login_user.row_key,
            unfollow_row_key: row_key,
          });
          if (res === 999) {
            this.clearSearchResult();
            row.state = "关注";
            this.$message.success("取关成功");
            // 刷新当前正在显示的主页
            this.seeDeatil(this.currentUser.row_key, this.currentUser);
          }
        }
      }
    },
    // 从已知列表中过滤用户
    searchFromList() {
      this.middleTotal = this.total;
      this.searchFromList_result = [];
      this.searchFromList_result = this.pageTotalList.filter((value) => {
        return (
          value.info_nickname.toLowerCase() ===
          this.searchFromList_nickname.toLowerCase()
        );
      });
      // this.middleList = this.userList;
      this.userList = this.searchFromList_result;
      this.total = this.searchFromList_result.length;
    },
    // 清除关注列表中搜索的结果
    recoverData() {
      this.pageSize = 7;
      this.pageNo = 1;
      this.total = this.middleTotal;
      this.handleSizeChange(7);
    },
    // 清除用户搜索中的搜索结果
    clearSearchResult() {
      this.searchUserForm = {};
      this.searchResult = [];
      this.total2 = 0;
    },
    handleSizeChange(pageSize) {
      this.pageNo = 1;
      this.pageSize = pageSize;
      this.searchFromList_result = this.pageTotalList.filter((value, index) => {
        return (
          index >= pageSize * (this.pageNo - 1) &&
          index <= pageSize * (this.pageNo - 1) + pageSize - 1
        );
      });
      this.userList = this.searchFromList_result;
    },
    handleCurrentChange(pageNo) {
      this.pageNo = pageNo;
      this.searchFromList_result = this.pageTotalList.filter((value, index) => {
        return (
          index >= this.pageSize * (pageNo - 1) &&
          index <= this.pageSize * (pageNo - 1) + this.pageSize - 1
        );
      });
      this.userList = this.searchFromList_result;
    },
    handleSizeChange2(pageSize2) {
      this.pageNo2 = 1;
      this.pageSize2 = pageSize2;
      this.searchResult = this.pageTotalList2.filter((value, index) => {
        return (
          index >= pageSize2 * (this.pageNo2 - 1) &&
          index <= pageSize2 * (this.pageNo2 - 1) + pageSize2 - 1
        );
      });
      // this.searchResult = this.searchFromList_result;
    },
    handleCurrentChange2(pageNo2) {
      this.pageNo2 = pageNo2;
      this.searchResult = this.pageTotalList2.filter((value, index) => {
        return (
          index >= this.pageSize2 * (pageNo2 - 1) &&
          index <= this.pageSize2 * (pageNo2 - 1) + this.pageSize2 - 1
        );
      });
      // this.userList = this.searchFromList_result;
    },
  },
};
</script>
<style lang="less" scoped>
.container {
  display: flex;
  justify-content: space-between;
  padding-left: 50px;
  padding-right: 50px;
  padding-top: 30px;
  padding-bottom: 30px;
  box-sizing: border-box;
  height: 100%;
  background-color: green;
  .left {
    height: 100%;
    flex-basis: 35%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  .right {
    flex: 1;
    height: 100%;
  }
  .card1 {
    // flex-basis: 20%;
    // width: 60%;
    margin-right: 15px;
    font-weight: bold;
    // padding-left: 50px;
    // display: flex;
    // justify-content: center;
    .title {
      display: flex;
      flex-direction: row;
      align-items: center;
      // justify-content: space-around;
    }
    .picture {
      margin-top: 15px;
      margin-left: 20px;
      margin-right: 55px;
      // flex-basis: 40%;
      height: 90px;
      width: 90px;
      border: 1px solid #e0e0e0;
      border-radius: 50%;
      padding: 5px;
      box-shadow: 0 0 5px #ddd;
      // position: absolute;
      // left: 50%;
      // transform: translate(-50%, -50%);
      background-color: #fff;
      img {
        height: 100%;
        width: 100%;
        border-radius: 50%;
      }
    }
    // img {
    //   width: 90px;
    //   height: 90px;
    //   border-radius: 50%;
    // }
  }
  .follow {
    display: flex;
    justify-content: space-around;
    margin-top: 25px;
  }
  .card3 {
    flex: 1;
    // flex-basis: 30%;
    margin-right: 15px;
  }

  .card2 {
    height: 97.9%;
    // flex-grow: 1.6;
    // margin-right: 10px;
  }
}
.underlinetext {
  padding-bottom: 3px;
  border-bottom: 1px solid rgb(161, 161, 161);
}
</style>
