<template>
  <div>
    <div class="header">

      <el-button v-show=false type="primary" @click="getHealthList">查询</el-button>
      <el-button @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
      <el-table-column prop="nickname" label="姓名" />
      <el-table-column prop="sex" label="性别" :formatter="sexFormatter" />
      <el-table-column prop="age" label="年龄" />
      <el-table-column prop="age" label="年龄" />
      <el-table-column prop="height" label="身高" />
      <el-table-column prop="weight" label="体重" />
      <el-table-column prop="bloodPressure" label="血压" />
      <el-table-column prop="heartRate" label="心率" />
      <el-table-column prop="bloodSugar" label="血糖" />
      <el-table-column label="操作">

        <template #default="scope">

          <el-button text type="primary" @click="handleEdit(scope.$index, scope.row)">
            编辑
          </el-button>

          <el-button text type="danger" @click="handleDel(scope.$index, scope.row)">
            删除
          </el-button>

        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="用户">
          <el-select v-model="form.username" placeholder="请选择" :disabled="mode == '1'">
            <el-option v-for="item in state.userList" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="体重(kg)">
          <el-input v-model="form.weight" type="number" />
        </el-form-item>
        <el-form-item label="身高(cm)">
          <el-input v-model="form.height" type="number" />
        </el-form-item>
        <el-form-item label="血压">
          <el-input v-model="form.bloodPressure" />
        </el-form-item>
        <el-form-item label="心率">
          <el-input v-model="form.heartRate" type="number" />
        </el-form-item>
        <el-form-item label="血糖">
          <el-input v-model="form.bloodSugar" type="number" />
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

const dialogVisible1 = ref(false)
let mode = '0'
const form = reactive({
  id: null,
  username: '',
  weight: null,
  bloodPressure: '',
  height: null,
  heartRate: null,
  bloodSugar: null,
  

})


const copyValue = (src, target) => {
  // 遍历 target 中的 key，并将 src 对应属性赋值给 target
  Object.keys(target).forEach((key) => {
    if (src[key] !== undefined) {
      target[key] = src[key] // 仅赋值存在于 src 中的属性
    }
  })
}


const dateRange = ref([])
const clearData = () => {
  form.id = null
  form.username = ''
  form.weight = null
  form.height = null
  form.bloodPressure = ''
  form.heartRate = null
  form.bloodSugar = null
 

}
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,

})



const state = reactive({
  data: {},
  curHealth: {},
  userList: []
})

const sexFormatter = (row, col, value) => {
  if (value == '0') {
    return '男'
  }
  return '女'
}

const getHealthList = () => {

  axios.get('health/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}

const getUserList = () => {
  axios.get('user/list', { params: { pageNum: 1, pageSize: 10000, role: '2' } }).then(res => {
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
    axios.post('health', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getHealthList()

    })
  } else {
    axios.put('health', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getHealthList()

    })
  }


}


const handleDel = (index, row) => {
  axios.delete('health/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getHealthList()
  })
}


const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getHealthList()
}

getHealthList()
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