<template>
  <div>
    <div class="header">

      <el-button v-show=false type="primary" @click="getServiceList">查询</el-button>
      <el-button v-show="getUser().role == '0'" @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
      <el-table-column prop="name" label="服务名" />
      <el-table-column prop="description" label="描述" />
      <el-table-column prop="serverName" label="医护人员" />

      <el-table-column label="操作">

        <template #default="scope">
          <div v-if="getUser().role == '0'">
            <el-button text type="primary" @click="handleEdit(scope.$index, scope.row)">
              编辑
            </el-button>

            <el-button text type="danger" @click="handleDel(scope.$index, scope.row)">
              删除
            </el-button>
          </div>
          <div v-else>
            <el-button text type="primary" @click="handleReserve(scope.$index, scope.row)">
              预约
            </el-button>
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
        <el-form-item label="医护人员">
          <el-select v-model="form.server" placeholder="请选择">
            <el-option v-for="item in state.userList" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
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
    <el-dialog v-model="dialogVisible2" width="500" @close="clearData2">
      <el-form class="form" :model="reserveForm" label-width="auto" style="max-width: 600px">
        <el-form-item label="预约时间">
          <el-date-picker v-model="reserveForm.serviceDate" type="datetime" value-format="YYYY-MM-DD HH:mm:ss"
            placeholder="请选择" />
        </el-form-item>


      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible2 = false">取消</el-button>
          <el-button type="primary" @click="saveReserve">
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

const dialogVisible1 = ref(false)
const dialogVisible2 = ref(false)
let mode = '0'
const form = reactive({
  id: null,
  name: '',
  description: '',
  server: ''


})

const reserveForm = reactive({
  serviceId: null,
  username: '',
  serviceDate: ''
})

const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,

})


const state = reactive({
  data: {},
  curService: {},
  userList: [],

})

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
  form.description = ''
  form.server = ''

  mode = '0'

}
const clearData2 = () => {
  reserveForm.serviceId = null
  reserveForm.serviceDate = ''
  reserveForm.username = ''
}



const getServiceList = () => {

  axios.get('service/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}


const getUserList = () => {
  axios.get('user/list', { params: { pageNum: 1, pageSize: 10000, role: '1' } }).then(res => {
    state.userList = res.list.map(item => {
      return {
        value: item.username,
        label: item.nickname
      }
    })
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

}

const saveOrUpdate = () => {
  if (mode == '0') {
    axios.post('service', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getServiceList()

    })
  } else {
    axios.put('service', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getServiceList()

    })
  }


}

const saveReserve = () => {
  reserveForm.username = getUser().username
  axios.post('reserve', reserveForm).then(res => {
    ElMessage.success('预约成功')
    dialogVisible2.value = false
  })
}


const handleDel = (index, row) => {
  axios.delete('service/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getServiceList()
  })
}

const handleReserve = (index, row) => {
  dialogVisible2.value = true
  reserveForm.serviceId = row.id
  
}


const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getServiceList()
}

getServiceList()
getUserList()


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