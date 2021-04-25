<template>
  <div class="app-container">
    <!-- 头部部分 -->
    <div class="filter-container">
      <el-dropdown>
            <el-button type="danger" size="mini">
                黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item @click.native="changeCommentStatus(row.user.id)">禁止评论</el-dropdown-item>
                <el-dropdown-item @click.native="changeAccessStatus(row.user.id)">禁止访问</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
      <el-button
        class="filter-item"
        style="margin-left: 0px"
        type="primary"
        icon="el-icon-search"
        @click="handleCreate"
      >
        搜索
      </el-button>
      <el-input
        v-model="searchTitle"
        placeholder="用户名"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <!-- <el-select
        v-model="listQuery.importance"
        placeholder="商品状态"
        clearable
        style="width: 120px; margin-right: 10px"
        class="filter-item"
      >
        <el-option
          v-for="item in importanceOptions"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select> -->
    </div>
    <!-- 数据表格 -->
    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="listData"
      border
      fit
      highlight-current-row
      style="width: 100%"
      @sort-change="sortChange"
    >


      <el-table-column label="用户" prop="content" width="500">
        <template slot-scope="{ row }">
          <img class="img" :src="row.user.avatar" alt="" />
          <div class="imgDiv">
            <span>{{ row.user.username }}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        label="消费总额"
        prop="total_consume"
        align="center"
        width="80"
      >
        <template slot-scope="{ row }">
          <span>{{ row.total_consume }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="创建时间"
        prop="create_time"
        align="center"
        width="200"
      >
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="300">
        <template slot-scope="{ row}">
          <el-button type="primary" size="mini" @click="openInfo(row.user.id)">
            详情
          </el-button>
          <el-button
            v-if="row.status === 1"
            size="mini"
            type="success"
            @click="handleStatus(row)"
          >
            联系用户
          </el-button>
          <el-dropdown>
            <el-button type="danger" size="mini">
                黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item @click.native="changeCommentStatus(row.user.id)">禁止评论</el-dropdown-item>
                <el-dropdown-item @click.native="changeAccessStatus(row.user.id)">禁止访问</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
   <!-- 底部导航栏 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="1"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="20"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
<!-- 编辑修改弹出框 -->
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="70px"
        style="width: 400px; margin-left: 25px"
      >
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="试看内容" prop="tryContent">
          <Tinymce :height="300" :width="600" v-model="temp.try" />
        </el-form-item>
        <el-form-item label="课程内容" prop="content">
          <Tinymce :height="300" :width="600" v-model="temp.content" />
        </el-form-item>
        <el-form-item label="课程价格">
          <el-input-number
            v-model="temp.price"
            @change="handleChange"
            :min="1"
          ></el-input-number>
        </el-form-item>
        <el-form-item label="课程价格">
          <el-radio v-model="temp.status" label="1">下架</el-radio>
          <el-radio v-model="temp.status" label="0">上架</el-radio>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button type="primary" @click="dialogStatus === 'create' ? handleAdd() : updateVideo1()"> {{ dialogStatus === "create" ? "完成添加" : "修改提交" }}</el-button>
      </div>
    </el-dialog>

    <!-- 详情页 -->
    <info ref = 'info'></info>
  </div>
</template>

<script>
// 导入和接口
import {
  // fetchList,
  createVideo,
  deleteVideo,
  updateVideo,
} from "../../api/video";
//引入详情页
import info from './info'
import {
  fetchList
} from '../../api/user'
//导入格式化时间和富文本编辑器
import { formatTime } from '@/utils'
import Tinymce from "@/components/Tinymce/index";

export default {
  data() {
    return {
      tableKey: 0,
      list: null,//初始化表格数据
      listData: [],//初始化表格数据
      total: 0,
      listLoading: true,
      searchTitle: "",
      pageSize:10,
      pageNum:1,
      listQuery: {//数据表格请求参数
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: "+id",
      },
      importanceOptions: ["已上架", "已下架"],//option的选项
      sortOptions: [
        { label: "ID Ascending", key: "+id" },
        { label: "ID Descending", key: "-id" },
      ],
      statusOptions: ["published", "draft", "deleted"],//status选项
      showReviewer: false,
      temp: {
        id: undefined,
        try: "",
        title: "",
        price: "",
        status:1,
        content:""
      },
      dialogFormVisible: false,//谈粗框是否显示
      dialogStatus: "",//弹出框的形式是为create or edit
      //判断弹出框text为修改或为新增
      textMap: {
        update: "修改",
        create: "新增",
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [
          { required: true, message: "type is required", trigger: "change" },
        ],
        timestamp: [
          {
            type: "date",
            required: true,
            message: "timestamp is required",
            trigger: "change",
          },
        ],
        title: [{ required: true, message: "标题为必填项", trigger: "blur" }],
        tryContent: [
          { required: true, message: "试看内容为必填项", trigger: "blur" },
        ],
        content: [
          { required: true, message: "课程内容为必填项", trigger: "blur" },
        ],
      },
      downloadLoading: false,
    };
  },
  created() {
    this.getlist();
  },
  methods: {
    changeCommentStatus() {

    },

     changeAccessStatus() {

    },
    // 得到表格数据
    async getlist(obj) {
      let res = await fetchList(obj);
        // console.log(res);
      if (res.code === 20000) {
        this.total = res.data.total;
        this.list = res.data.items;
        this.listData = res.data.items;
        // console.log(this.list);
        this.listLoading = false;
      }
    },
    //添加Video
    addVideo() {

      // addVideo初始化temp状态
      this.temp ={
        id: undefined,
        try: "",
        title: "",
        price: "",
        status:1,
        content:""
      }

      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    handleCreate() {
      if (this.listQuery.importance === "已上架") {
        this.listData = this.list.filter((item) => item.status === 0);
      } else if (this.listQuery.importance === "未上架") {
        this.listData = this.list.filter((item) => item.status === 1);
      }
      // console.log(this.listData);
      if (this.searchTitle) {
        console.log(this.searchTitle);
        this.listData = this.list.filter(
          (item) => item.title.indexOf(this.searchTitle) !== -1
        );
      } else {
        this.getlist();
      }
    },
    handleDownload() {},
    sortChange() {},
    handleSizeChange(v){
        this.pageSize = v
        // console.log(v);
        this.getlist({
            page:this.pageNum,
            limit:this.pageSize
        })
    },
    handleCurrentChange(v){
        this.pageNum = v
        // console.log(v);
        this.getlist({
            page:this.pageNum,
            limit:this.pageSize
        })
    },
    handleChange() {
      console.log(this.price);
    },
    //get 详情页
    openInfo(id) {
      // console.log(id,'//////');
      // this.temp = Object.assign({},obj)
      // console.log(this.temp);
      // this.dialogStatus = 'update'
      // this.dialogFormVisible = true
      this.$refs.info.open(id)
      
    },
    handleStatus(obj) {
      console.log(obj);
      if (obj.status === 0) {
        obj.status = 1;
      } else if (obj.status === 1) {
        obj.status = 0;
      }
      this.$message.success("更改状态成功");
    },
    // 删除
    async handleDelete(obj, i) {
      console.log(obj);
      console.log(i);
      let res = await deleteVideo(i);
      console.log(res);
      if (res.code === 20000) {
        this.$message.success("删除成功");
        // this.getlist()
        this.list.splice(i, 1);
      }
    },
    async handleAdd() {
      let id = parseInt(Math.random() * 100) + 1024 // mock a id
      let time = new Date();
      time = formatTime("yyyy-MM-dd hh:mm:ss", time);
      console.log(time);
      let data = {
        id:id,
        content: this.temp.content,
        cover: "http://dummyimage.com/200x100",
        price: this.temp.price,
        status: Number.parseInt(this.temp.status),
        try: this.temp.tryContent,
        created_time: time,
        sub_count: 6,
        t_price: 99,
        title: this.temp.title,
        updated_time: time,
      };
      console.log(data);
      let res = await createVideo(data);
      console.log(res);
      if (res.code === 20000) {
        this.dialogFormVisible = false;
        this.listData.unshift(data)
        console.log(this.listData);
      }
      this.updateVideo1()
    },
    //更新数据
    async updateVideo1(){
        // console.log(123);
        console.log(this.temp);
        let time = new Date();
        time = formatTime("yyyy-MM-dd hh:mm:ss", time);
        console.log(time);
        this.temp.updated_time = time
        this.temp.status = parseInt(this.temp.status)
        console.log(this.temp);
        let res = await updateVideo(this.temp)
        console.log(res);
        if(res.code === 20000){
            let index = this.listData.findIndex( item => item.id = this.temp.id )
            this.listData.splice(index,1,this.temp)
            this.dialogFormVisible = false
            this.$message.success("修改成功")
            this.temp={
                id: undefined,
                try: "",
                title: "",
                price: "",
                status:1,
                content:""
            }
        }
    }
  },
  //过滤器
  filters: {
    // status过滤
    filterStatus(v) {
      //   console.log(v);
      if (v === 0) {
        return "已上架";
      } else if (v === 1) {
        return "未上架";
      }
    },
    statusFilter(v) {
      if (v === 0) {
        return "success";
      } else if (v === 1) {
        return "danger";
      }
    },
  },
  //使用组件
  components: {
    Tinymce,
    info,
  },
};
</script>
样式
<style scoped>
.filter-item {
  float: right;
}
.img {
  float: left;
  width:  50px;
  height: 50px;
  border-radius: 50px;
  margin-top: 10px;
}
.imgDiv {
  float: left;
  margin-top: 10px;
  margin-left: 10px;
}
</style>
