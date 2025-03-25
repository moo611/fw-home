<template>
  <div>
    <div class="header">

      <el-button v-show=false type="primary" @click="getActivityList">查询</el-button>
      <el-button v-show="getUser().role=='0'" @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
      <el-table-column prop="name" label="活动名" />
      <el-table-column prop="description" label="描述" />
      <el-table-column prop="startTime" label="开始日期" />
      <el-table-column prop="endTime" label="结束日期" />

      <el-table-column label="操作">

        <template #default="scope">
          <div v-if="getUser().role == '0' || getUser().role == '1'">
            <el-button text type="primary" @click="handleEdit(scope.$index, scope.row)">
              编辑
            </el-button>

            <el-button text type="danger" @click="handleDel(scope.$index, scope.row)">
              删除
            </el-button>
          </div>
          
          <div v-if="getUser().role == '2'">
            <el-button text type="primary" @click="join(scope.row)">报名</el-button>
          </div>


        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">

        <el-form-item label="名称">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="form.description" />
        </el-form-item>
        <el-form-item label="起止时间">
          <el-date-picker v-model="dateRange" type="daterange" range-separator="至" start-placeholder="开始时间"
            end-placeholder="截止时间" @change="onDateChange" value-format="YYYY-MM-DD" />
        </el-form-item>


      </el-form>

      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible1 = false">取消</el-button>
          <el-button type="primary" @click="saveOrUpdate">
            确定
          </el-button>
        </div>
      </template>
    </el-dialog>

  </div>
</template>

<script lang="js" setup>

import { reactive, ref } from 'vue';
import axios from '../../axios';
import { ElMessage } from 'element-plus';
import { getUser } from '../../utils/auth';

const dateRange = ref([])

const dialogVisible1 = ref(false)
let mode = '0'
const form = reactive({
  id: null,
  name: '',
  description: '',
  startTime: '',
  endTime: ''

})

const onDateChange = (value) => {
  console.log(value)
  form.startTime = value[0]
  form.endTime = value[1]
  console.log(form)
}

const copyValue = (src, target) => {
  // 遍历 target 中的 key，并将 src 对应属性赋值给 target
  Object.keys(target).forEach((key) => {
    if (src[key] !== undefined) {
      target[key] = src[key] // 仅赋值存在于 src 中的属性
    }
  })
}


const clearData = () => {
  form.id = null
  form.name = ''
  form.description = null
  form.startTime = ''
  form.endTime = ''
  mode = '0'

  dateRange.value = []

}
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,

})



const state = reactive({
  data: {},
  curActivity: {},

})


const getActivityList = () => {

  axios.get('activity/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}



const handleAdd = (index, row) => {
  dialogVisible1.value = true
  mode = '0'
}



const handleEdit = (index, row) => {
  mode = '1'
  //form = { ...row }
  copyValue(row, form)
  dialogVisible1.value = true
  dateRange.value[0] = form.startTime
  dateRange.value[1] = form.endTime
}

const join=(row)=>{

  let params = {}
  params.activityId = row.id
  params.username = getUser().username
  params.type = '0'
  axios.post('join',params).then(res=>{
    ElMessage.success('报名成功')
  })
}

const saveOrUpdate = () => {
  if (mode == '0') {
    axios.post('activity', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getActivityList()

    })
  } else {
    axios.put('activity', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getActivityList()

    })
  }


}


const handleDel = (index, row) => {
  axios.delete('activity/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getActivityList()
  })
}


const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getActivityList()
}

getActivityList()


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