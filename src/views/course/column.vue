<template>
  <div class="app-container">
    <!-- 头部筛选部分 -->
    <div class="filter-container">
      <el-button type="primary" icon="el-icon-edit" @click="addColumn">
        新增专栏
      </el-button>

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
        placeholder="标题"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-select
        v-model="listQuery.importance"
        placeholder="商品状态"
        clearable
        style="width: 120px; margin-right: 10px"
        class="filter-item">
        <el-option
         v-for="item in importanceOptions"
          :key="item"
          :label="item"
          :value="item"/>
      </el-select>
    </div>
     <!--数据表格 -->
    <el-table
    :key="tableKey"
    >
      
      
    </el-table>
  </div>
 
</template>

<script>
export default {
  data () {
    return {
            tableKey: 0,
            searchTitle:'',
            listQuery: {//数据表格请求参数
              page: 1,
              limit: 20,
              importance: undefined,
              title: undefined,
              type: undefined,
              sort: "+id",
            },
            importanceOptions:['已上架','已下架']

    }
  },
  methods:{
    //添加专栏
    addColumn() {
      console.log('addColumn');
    },
    //搜索专栏
    handleCreate() {
      console.log('search column');
    }
  }

}
</script>

</script>
样式
<style scoped>
.filter-item {
  float: right;
}
.img {
  float: left;
  width: 100px;
  height: 50px;
  margin-top: 10px;
}
.imgDiv {
  float: left;
  margin-top: 10px;
  margin-left: 10px;
}
</style>