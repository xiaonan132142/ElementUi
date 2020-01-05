<template>
  <div class="hello">
    <el-table ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100%"
        @select="handleSelected"
        @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        prop="id"
        label="ID"
        width="120">
      </el-table-column>
      <el-table-column
        label="日期"
        width="120">
        <template slot-scope="scope">{{ scope.row.createdAt }}</template>
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="120">
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="currentPage"
      :page-sizes="[10, 20, 40]"
      :page-size="pageSize"
      layout="total,sizes, prev, pager, next"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
import cache from '../utils/cache'
export default {
  name: 'demo1',
  data () {
    return {
      currentPage:1,
      pageSize:10,
      total:50,
      mutipleData:[],
      tableData:[],
    }
  },
  created() {
    this.initData()
    cache.setItem('localTD','')
  },
  methods:{
    // 获取表格数据
    initData(){
      let params={
        page:this.currentPage,
        limit:this.pageSize,
      }
      this.axios.get('/api/v1/users',{params})
      .then((res)=>{
        let {data} = res
        this.tableData = data
        this.handleCheckedRows()
      })
    },
    // 存储checked
    handleSelected(rows,row){
      let localData = cache.getItem('localTD')||''
      let localCur = localData.split(',')
      let selected = rows.length&&rows.indexOf(row)!==-1
      if(selected){
        if(!localCur.includes(row.id)){
          localCur.push(row.id)
        }
      }else{
        let idx = localCur.findIndex(item=>item==row.id)
        localCur.splice(idx,1)
      }
      localCur=localCur.filter((item)=>{return item!==''})
      console.log(localCur)
      cache.setItem('localTD',localCur.toString())
    },
    // 渲染checked
    handleCheckedRows(){
      let that = this
      let localData = cache.getItem('localTD')
      if(!localData) return
      let localCur = localData.split(',')
      console.log(localCur)
      // 解决toggleRowSelection 失效问题
      this.$nextTick(()=>{
        if (that.$refs.multipleTable) {
          that.$refs.multipleTable.clearSelection();
        }
        // 设置checked
        that.tableData.forEach((item,index)=>{
          that.$refs.multipleTable.toggleRowSelection(that.tableData[index],localCur.includes(item.id))
        })
      })
    },
    handleSelectionChange(val){
      this.mutipleData = val
    },
    // 分页
    handleSizeChange(val) {
      this.pageSize = val
      this.initData()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.initData()
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
