<template>
  <div>
    <div class="header">

      <el-button v-show=false type="primary" @click="getJoinList">查询</el-button>

    </div>
    <el-table class="my-table" :data="state.data.list">
      <el-table-column prop="activityName" label="活动名" />
      <el-table-column prop="type" label="状态" :formatter="typeFormatter" />
      <el-table-column prop="startTime" label="开始日期" />
      <el-table-column prop="endTime" label="结束日期" />

      <el-table-column label="操作">

        <template #default="scope">

          <el-button text type="primary" @click="handleEdit(scope.$index, scope.row)" :disabled="scope.row.type == '1'">
            签到
          </el-button>

        </template>
      </el-table-column>
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
  curJoin: {},

})

const typeFormatter = (row, col, value) => {
  if (value == '0') {
    return '已报名'
  }
  return '已签到'
}

const getJoinList = () => {
  queryParams.username = getUser().username
  axios.get('join/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}


const handleEdit = (index, row) => {

  row.type = '1'
  axios.put('join', row).then(res => {

    dialogVisible1.value = false
    ElMessage.success('签到成功')
    getJoinList()

  })
}



const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getJoinList()
}

getJoinList()



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
</style>