<template>
  <div>
    <el-card class="card">
      <!-- 面包屑 -->
      <my-breadcrumb level1="玩家管理" level2="玩家数据"></my-breadcrumb>
      <!-- 搜索框 -->
      <el-row class="search">
        <el-col :span="24">
          <el-input
            v-model="searchValue"
            clearable
            placeholder="搜索玩家id"
            class="searchInput"
            @change="handleChange"
          >
            <el-button
              @click="handleSearch"
              slot="append"
              icon="el-icon-search"
            ></el-button>
          </el-input>
          <!-- <el-button @click="$router.push('goods/add')" type="success" plain
            >搜索玩家id</el-button
          > -->
        </el-col>
      </el-row>
      <!-- table表格 -->
      <template>
        <el-table stripe border :data="list" style="width: 100%">
          <!-- <el-table-column type="index" width="80"> </el-table-column> -->
          <el-table-column prop="id" label="id"></el-table-column>
          <el-table-column prop="ip" label="ip" width="150"> </el-table-column>
          <el-table-column
            prop="coin"
            label="coin"
            width="150"
          ></el-table-column>
          <el-table-column prop="exp" label="exp" width="150"></el-table-column>
          <el-table-column label="创建时间">
            <template slot-scope="scope">
              <!-- 格式化日期 -->
              {{ scope.row.createTime }}
            </template>
          </el-table-column>
          <el-table-column label="登录时间">
            <template slot-scope="scope">
              <!-- 格式化日期 -->
              {{ scope.row.loginTime }}
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                type="primary"
                icon="el-icon-edit"
                circle
                @click="handleShowDialog(scope.row)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          v-if="isShowPage"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="pageNo"
          :page-size="pageSize"
          layout="total, prev, pager, next"
          :total="total"
        >
        </el-pagination>
      </template>
    </el-card>
    <el-dialog title="修改" :visible.sync="isShowDialog" width="30%" center>
      <div style="padding: 20px">
        <el-row style="margin-top: 10px">
          <el-col :span="4">
            <div style="height: 40px; line-height: 40px">金币：</div>
          </el-col>
          <el-col :span="20">
            <el-input
              v-model="rowData.coin"
              clearable
              placeholder=""
              @change="handleChange"
            >
            </el-input>
          </el-col>
        </el-row>
        <el-row style="margin-top: 10px">
          <el-col :span="4">
            <div style="height: 40px; line-height: 40px">经验：</div>
          </el-col>
          <el-col :span="20">
            <el-input
              v-model="rowData.exp"
              clearable
              placeholder=""
              @change="handleChange"
            >
            </el-input>
          </el-col>
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isShowDialog = false">取 消</el-button>
        <el-button type="primary" @click="handleEdit">修改</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      searchValue: "",
      list: [],
      isShowDialog: false,
      rowData: {},
      pageNo: 1,
      pageSize: 10,
      total: 0,
      isShowPage: true,
    };
  },
  methods: {
    handleSizeChange(val) {
      this.pageSize = val;
      this.loadList();
    },
    handleCurrentChange(val) {
      console.log(val);
      this.pageNo = val;
      this.loadList();
    },
    async handleEdit() {
      console.log(this.rowData.coin);
      console.log(this.rowData.exp);
      let letter = this.startLoading(); //loading开启
      let res = await this.$http
        .post("/admin", {
          cmd: "updateSingleplayer",
          token: sessionStorage.getItem("token"),
          ...this.rowData,
        })
        .catch((err) => {
          console.log(err);
          this.closeLoading(letter);
          this.$message.error("请检查输入数据是否合法");
        });
      if (res.data.code == 1) {
        //  请求成功
        this.closeLoading(letter); //关闭loading
        this.isShowDialog = false;
        this.loadList();
      } else {
        this.closeLoading(letter);
      }
    },
    handleShowDialog(row) {
      this.rowData = row;
      console.log(this.rowData);
      this.isShowDialog = true;
    },
    handleChange(value) {
      console.log(value);
    },
    // 获取商品列表
    async loadList() {
      console.log(this.pageNo);
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "searchPlayer",
        token: sessionStorage.getItem("token"),
        pagesize: this.pageSize,
        pagenum: this.pageNo,
      });
      if (res.data.code == 1) {
        //  请求成功
        this.closeLoading(letter); //关闭loading
        this.list = res.data.data;
        this.total = Number(res.data.totalcount);
      } else {
        this.closeLoading(letter);
        this.$message.error(msg);
      }
    },

    async handleSearch() {
      if (this.searchValue !== "") {
        this.isShowPage = false;
        let letter = this.startLoading(); //loading开启
        let res = await this.$http
          .post("/admin", {
            cmd: "searchSiglePlayer",
            token: sessionStorage.getItem("token"),
            id: this.searchValue,
          })
          .catch((err) => {
            console.log(err);
            this.closeLoading(letter);
            this.$message.error("请检查玩家id是否合法");
          });
        if (res.data.code == 1) {
          this.list = [];
          this.list.push(res.data.data);
          //  请求成功
          this.closeLoading(letter); //关闭loading
          // this.list = res.data.data;
        } else {
          this.closeLoading(letter);
          // this.$message.error(msg);
        }
      } else {
        this.loadList();
        this.isShowPage = true;
      }
    },
  },
  created() {
    this.loadList();
  },
};
</script>
<style>
.searchInput {
  margin-top: 10px;
  width: 350px;
}
.el-table {
  margin-top: 10px;
}
</style>