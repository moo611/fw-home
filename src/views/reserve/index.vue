<template>
  <div>
    <div class="header">
      
      <span style="margin-right: 5px;">今日预约提醒</span>
      <div style="width: 15px;height: 15px; background-color: var(--el-color-warning-light-5);"></div>
    </div>
    <el-table class="my-table" :data="state.data.list" :row-class-name="tableRowClassName">
      <el-table-column prop="serviceName" label="服务名" />
      <el-table-column prop="serverName" label="医护人员"  />
      <el-table-column prop="serviceDate" label="预约时间" />
      
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />


  </div>
</template>

<script lang="js" setup>

import { reactive } from 'vue';
import axios from '../../axios';
import { ElMessage } from 'element-plus';
import { getUser } from '../../utils/auth';


const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,

})

const state = reactive({
  data: {},
  curReserve: {},

})

const tableRowClassName = ({
  row,
  rowIndex,
}) => {
  console.log('row',row)
  const date = new Date(row.serviceDate);
  const today = new Date();
  if (date.toLocaleDateString() === today.toLocaleDateString()) {
    console.log('--------')
    return 'warning-row'
  } 
  return ''
}


const getReserveList = () => {
  queryParams.username = getUser().username
  axios.get('reserve/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}



const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getReserveList()
}

getReserveList()



</script>

<style lang="css" scoped>
.header {
  height: 50px;
  position: relative;
  display: flex;
  align-items: center;
  margin-bottom: 30px;

}

.btn-add {
  position: absolute;
  right: 20px;
}

.avatar {
  width: 150px;
  height: 150px;
  object-fit: contain;
}
::v-deep .el-table .warning-row {
  --el-table-tr-bg-color: var(--el-color-warning-light-5);
  /* background-color: red !important; */
}
</style>