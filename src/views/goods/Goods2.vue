<template>
  <div>
    <el-card class="card">
      <!-- 面包屑 -->
      <my-breadcrumb level1="玩家管理" level2="动态管理"></my-breadcrumb>
      <!-- 自动审核 -->
      <div style="padding-top: 10px; padding-bottom: 10px">
        是否自动审核：
        <el-switch
          v-model="isAutoAuditValue"
          active-color="#13ce66"
          inactive-color="#ff4949"
          @change="handleChange"
        >
        </el-switch>
        (开启状态所有动态自动通过)
      </div>
      <!-- 搜索框 -->
      <el-row class="search">
        <el-col :span="24">
          <el-select
            v-model="selectValue"
            @change="handleChangeStatus"
            placeholder="请选择"
          >
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
          <!-- <el-button @click="handleSearch" type="success" plain>搜索</el-button> -->
        </el-col>
      </el-row>
      <!-- table表格 -->
      <template>
        <el-table center stripe border :data="list" style="width: 100%">
          <el-table-column prop="id" label="id"></el-table-column>
          <el-table-column prop="name" label="name"></el-table-column>
          <el-table-column prop="fbid" label="fbid"></el-table-column>
          <el-table-column label="图片" width="180">
            <template slot-scope="scope">
              <div style="height: 80px">
                <img
                  style="height: 100%; width: 100%"
                  :src="scope.row.imgurl"
                  alt=""
                />
              </div>
            </template>
          </el-table-column>
          <el-table-column label="status">
            <template slot-scope="scope">
              {{ scope.row.status ? "审核通过" : "审核不通过" }}
            </template>
          </el-table-column>
          <el-table-column label="好友可见/世界可见" width="100">
            <template slot-scope="scope">
              {{ scope.row.type == 1 ? "世界可见" : "好友可见" }}
            </template>
          </el-table-column>
          <el-table-column label="创建时间">
            <template slot-scope="scope">
              <!-- 格式化日期 -->
              {{ scope.row.add_time | formDate("YYYY-MM-DD") }}
            </template>
          </el-table-column>
          <el-table-column label="操作" width="240">
            <template slot-scope="scope">
              <el-button
                type="primary"
                @click="handleUpdate(scope.row)"
                icon="el-icon-edit"
                >{{ selectValue == 1 ? "不通过" : "通过" }}</el-button
              >
              <el-button
                type="danger"
                @click="handleDelete(scope.row)"
                icon="el-icon-delete"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </template>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="pageNo"
        :page-size="pageSize"
        layout="total, prev, pager, next"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      searchValue: "",
      list: [],
      isAutoAuditValue: "",
      pageNo: 1,
      pageSize: 10,
      total: 0,
      selectValue: 1,
      options: [
        {
          label: "已通过",
          value: 1,
        },
        {
          label: "未通过",
          value: 2,
        },
      ],
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
    async handleUpdate(row) {
      console.log(row);
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "updateStatusById",
        token: sessionStorage.getItem("token"),
        id: row.id,
        status: row.status == 1 ? 0 : 1,
      });
      if (res.data.code == 1) {
        console.log(res.data);
        //  请求成功
        this.closeLoading(letter); //关闭loading
        this.loadList();
      } else {
        this.closeLoading(letter);
        this.$message.error("更新失败");
      }
    },
    async handleDelete(row) {
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "deleteById",
        token: sessionStorage.getItem("token"),
        id: row.id,
      });
      if (res.data.code == 1) {
        console.log(res.data);
        //  请求成功
        this.closeLoading(letter); //关闭loading
        this.loadList();
      } else {
        this.closeLoading(letter);
        this.$message.error("删除失败");
      }
    },
    handleChangeStatus(val) {
      if (val == 1) {
        this.selectValue = 1;
        console.log(this.selectValue);
        this.loadList();
      }
      if (val == 2) {
        this.selectValue = 2;
        console.log(this.selectValue);
        this.loadList();
      }
    },
    async getAutioInfo() {
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "getAutoSatus",
        token: sessionStorage.getItem("token"),
      });
      if (res.data.code == 1) {
        console.log(res.data);
        if (res.data.status == "0") {
          this.isAutoAuditValue = false;
          this.loadList();
        }
        if (res.data.status == "1") {
          this.isAutoAuditValue = true;
          this.loadList();
        }
        //  请求成功
        this.closeLoading(letter); //关闭loading
      } else {
        this.closeLoading(letter);
      }
    },
    async handleChange() {
      console.log(this.isAutoAuditValue);
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "updateAutoStatus",
        token: sessionStorage.getItem("token"),
        status: this.isAutoAuditValue ? 1 : 0,
      });
      if (res.data.code == 1) {
        console.log(res.data);
        this.loadList();
        // this.isAutoAuditValue = true;
        //  请求成功
        this.closeLoading(letter); //关闭loading
        // this.loadList();
      } else {
        // this.isAutoAuditValue = false;
        this.closeLoading(letter);
      }
    },
    handleSearch() {},
    // 获取商品列表
    async loadList() {
      console.log(this.isAutoAuditValue);
      let letter = this.startLoading(); //loading开启
      let res = await this.$http.post("/admin", {
        cmd: "getNews",
        token: sessionStorage.getItem("token"),
        pagesize: this.pageSize,
        pagenum: this.pageNo,
        status: this.selectValue == 1 ? 1 : 0,
      });
      if (res.data.code == 1) {
        //  请求成功
        this.closeLoading(letter); //关闭loading
        this.list = res.data.data;
        this.total = Number(res.data.total);
      } else {
        this.closeLoading(letter);
        this.$message.error(msg);
      }
    },
  },
  created() {
    this.getAutioInfo();
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