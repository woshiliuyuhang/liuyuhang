<template>
  <div class="container">
    <div class="find">
      <div class="top">
        <el-input
          v-model="input"
          :placeholder="search"
          size="small"
          style="width:200px;"
          @keyup.enter.native="goSearch"
        >
          <i
            slot="prefix"
            class="el-input__icon el-icon-search"
            @click="goSearch"
          ></i>
        </el-input>
        <el-button
          type="primary"
          size="small"
          class="add"
          @click="goAdd"
        >新增资讯</el-button>
      </div>
    </div>
    <el-table
      :data="tableData"
      border
      style="margin-top:20px"
      :row-style="{height:'30px'}"
      ref="multipleTable"
      tooltip-effect="dark"
      @selection-change="handleSelectionChange"
      @select="change"
      @select-all="change"
      @sort-change="sortChange"
      @filter-change="filterChange"
      id="#out-table"
    >
      <el-table-column
        type="selection"
        width="55"
      >
      </el-table-column>
      <el-table-column
        prop="id"
        :show-overflow-tooltip="true"
        label="ID"
      >
      </el-table-column>
      <el-table-column
        prop="title"
        :show-overflow-tooltip="true"
        width="300"
        label="标题"
      >
        <template slot-scope="scope">
          <span
            @click="edit(scope.row.id)"
            class="title"
          >{{scope.row.title}}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="view_num"
        label="浏览数"
        sortable
      >
      </el-table-column>
      <el-table-column
        prop="collect_num"
        label="收藏数"
        sortable
      >
      </el-table-column>
      <el-table-column
        prop="reply_num"
        label="回复数"
        sortable
      >
      </el-table-column>
      <el-table-column
        prop="share_num"
        label="转发数"
        sortable
      >
      </el-table-column>
      <el-table-column
        prop="weight"
        column-key="weight"
        label="权重"
        :filters="[{text: 1, value:1}, {text:2, value:2}, {text:3, value:3}, {text:4, value:4},{text:5, value:5} ]"
        :filter-multiple="false"
      >
      </el-table-column>
      <el-table-column
        prop="release_status"
        label="状态"
        column-key="release_status"
        :filters="filters"
        :filter-multiple="false"
        :filter-method="filterState"
      >
        <template slot-scope="scope">
          <!-- <el-tag :type="scope.row.release_status === 0 ? 'primary' : 'success'" disable-transitions>{{scope.row.release_status}}</el-tag> -->
          <span v-if="scope.row.release_status === 0">已发布</span>
          <span v-if="scope.row.release_status === 1">审核中</span>
          <span v-if="scope.row.release_status === 2">已删除</span>
        </template>
      </el-table-column>
      <!-- <el-table-column show-overflow-tooltip prop="release_status" label="状态" :render-header="renderState"></el-table-column> -->
      <!-- <el-table-column prop="pub_date" label="创建时间" sortable width="150">
      </el-table-column> -->
      <el-table-column
        :render-header="renderHeader"
        label="创建时间"
        width="200"
      >
        <template slot-scope="scope">
          {{scope.row.pub_date}}
        </template>
      </el-table-column>
    </el-table>
    <div style="margin-top: 20px">
      <el-button
        @click="toggleSelect(tableData)"
        size="mini"
        type="primary"
      >全选</el-button>
      <el-button
        :disabled="id.length==0?true:false"
        @click="del(id)"
        size="mini"
        type="primary"
      >删除</el-button>
      <el-button
        :disabled="id.length==0||id.length>1?true:false"
        @click="edit(id)"
        size="mini"
        type="primary"
      >编辑</el-button>
      <el-button
        :disabled="id.length==0?true:false"
        @click="putUp(id)"
        size="mini"
        type="primary"
      >发布</el-button>
      <el-button
        :disabled="id.length==0?true:false"
        @click="repeal(id)"
        size="mini"
        type="primary"
      >撤销</el-button>
      <el-button
        :disabled="id.length==0||id.length>1?true:false"
        @click="banner(id)"
        size="mini"
        type="primary"
      >生成Banner</el-button>
      <el-pagination
        class="page"
        background
        layout="prev, pager, next"
        :total="count"
        :page-size="rows"
        :current-page.sync="pn"
        @current-change="getPn"
        :pager-count="7"
      >
      </el-pagination>
    </div>
  </div>
</template>
<script>
import exportFile from "../../../components/export";
export default {
  name: "newsList",
  components: {
    exportFile
  },
  data() {
    return {
      tableData: [], //表格数据
      id: [], //表格id
      options: [], //板块数据
      formData: {},
      input: "",
      order: "",
      sort: "",
      startTime: "",
      endTime: "",
      status: "",
      weigh: "",
      rows: 20,
      pn: 1,
      count: 1,
      allSelect: false, //是否全选
      search: this.$t("layout.search"),
      filters: [
        { text: 0, value: 0 },
        { text: 1, value: 1 },
        { text: 2, value: 2 }
      ],
      options: [
        { label: 1, value: 1 },
        { label: 2, value: 2 },
        { label: 3, value: 3 },
        { label: 4, value: 4 },
        { label: 5, value: 5 }
      ],
      date: "", //日期
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            }
          }
        ]
      }
    };
  },
  methods: {
    filterHandler(value, row, column) {
      const property = column["property"];
      return row[property] === value;
    },
    sortChange(val) {
      this.order = val.prop;
      if (val.order == "ascending") {
        this.sort = "asc";
      }
      if (val.order == "descending") {
        this.sort = "desc";
      }
      this.getData();
    },
    filterChange(filters) {
      console.log(filters);
    },
    //获取数据
    getData() {
      this.tableData = [];
      var data = {
        keyword: this.input,
        page: this.pn,
        type: 0,
        order: this.order,
        sort: this.sort,
        startTime: this.startTime,
        endTime: this.endTime,
        status: this.status,
        weigh: this.weigh
      };
      this.$axios.get("getNews", data).then(res => {
        console.log(res);
        if (res.code == 200) {
          this.tableData = res.data.news_list;
          this.count = res.data.total_record;
        }
      });
    },
    //搜索功能
    goSearch() {
      this.pn = 1;
      this.getData();
    },
    //全选功能
    toggleSelect(rows) {
      this.id = [];
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row, !this.allSelect);
          this.id.push(row.id);
        });
        this.allSelect = !this.allSelect;
      }
      if (!this.allSelect) {
        this.id = [];
      }
    },
    //获取id
    change(selection, row) {
      this.id = [];
      if (selection.length > 0) {
        for (var i = 0; i < selection.length; i++) {
          this.id[i] = selection[i].id;
        }
      } else {
        console.log("请选择数据");
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    //分页功能
    getPn(pn) {
      this.pn = pn;
      this.getData();
      window.scrollTo(0, 0);
    },
    //新增功能
    goAdd() {
      this.$router.push("/news/news/add");
    },
    //删除功能
    del(val) {
      val = val.toString();
      this.$confirm("此操作将永久删除该新闻, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$axios
            .get("delNews", {
              news_ids: val
            })
            .then(res => {
              if (res.code == 200) {
                this.$message({
                  type: "info",
                  message: "已成功删除"
                });
              }
              this.getData();
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //编辑功能
    edit(val) {
      if (val instanceof Array) {
        this.$router.push({ path: "/news/news/edit", query: { id: val[0] } });
      } else {
        this.$router.push({ path: "/news/news/edit", query: { id: val } });
      }
    },
    //搜索
    find() {
      this.pn = 1;
      this.getData();
    },
    //发布
    putUp(val) {
      val = val.toString();
      this.$axios.get("multiNews", { news_ids: val, status: 1 }).then(res => {
        if (res.code == 200) {
          this.getData();
        }
      });
    },
    //撤销
    repeal(val) {
      val = val.toString();
      this.$axios.get("multiNews", { news_ids: val, status: 2 }).then(res => {
        if (res.code == 200) {
          this.getData();
        }
      });
    },
    banner(val) {
      if (val instanceof Array) {
        this.$router.push({ path: "/news/banner/edit", query: { id: val[0] } });
      } else {
        this.$router.push({ path: "/news/banner/edit", query: { id: val } });
      }
    },
    filterState(value, row, column) {
      // console.log(value);
      // console.log(row);
      // console.log(column);
    },
    handleDataRangeChange() {
      if (this.date) {
        this.startTime = this.date[0];
        this.endTime = this.date[1];
        this.getData();
      }
    },
    renderHeader(h, context, vnode) {
      return (
        <span class="dateColumn">
          <span class="columeLable">{context.column.label}</span>
          <span class="columeIcon">
            <el-date-picker
              class="dataTimePick"
              size="mini"
              style="display:none"
              style="width:0;visibility:hidden"
              ref={"timer"}
              v-model={this.date}
              type="datetimerange"
              picker-options={this.pickerOptions}
              format="yyyy-MM-dd HH:mm"
              value-format="yyyy-MM-dd HH:mm"
              on-change={this.handleDataRangeChange}
              align="center"
            />
            <i
              class="el-icon-date dataIcon"
              style="position:absolute;top:8px;left:68px"
              onClick={e => {
                console.log(context);
                console.log(context.column);
                context.store.table.$refs.tableHeader.$refs.timer.focus();
              }}
            />
          </span>
        </span>
      );
    }
  },
  created() {
    this.getData();
  }
};
</script>
<style>
/* .zujian {
  position: absolute;
  top: 6px;
  left: 70px;
  cursor: pointer;
}
.datePick {
  opacity: 1 !important;
  position: absolute;
  left: 0;
  top: -2px;
  display: none;
} */
</style>

<style lang="css" scoped>
.img-item {
  max-height: 100px;
  width: 150px;
}

.find {
  margin: 10px 0;
}

.input {
  width: 193px;
}

.first,
.second {
  display: flex;
  margin-bottom: 20px;
}

.page {
  float: right;
}
.page,
.add {
  float: right;
}
.title {
  cursor: pointer;
}
.title:hover {
  color: #409eff;
}
.block,
.source,
.label {
  margin-right: 20px;
}

.btn {
  margin-right: 10px;
}
</style>
