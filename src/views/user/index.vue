<template>
  <div>
    <div class="header">
      <el-upload class="btn-inport" :headers="{ 'Authorization': 'Bearer ' + getToken() }" action="http://localhost:3000/api/user/import"
        ref="uploadRef" accept=".xls,.xlsx" :auto-upload="true" :limit="1" :on-exceed="handleExceed"
        :show-file-list="false" :on-success="handleSuccess">
        <el-button type="success" style="width: 100%; height: 100%;">导入</el-button>
      </el-upload>

      <el-button @click="handleExport" type="info" class="btn-export">导出</el-button>
      <el-button @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">

      <el-table-column prop="username" label="用户名" />
      <el-table-column prop="nickname" label="昵称" />
      <el-table-column prop="age" label="年龄" />
      <el-table-column prop="sex" label="性别" :formatter="sexFormatter" />
      <el-table-column prop="createTime" label="创建时间" />

      <el-table-column label="操作" width="180">
        <template #default="scope">
          <el-button type="primary" @click="handleEdit(scope.$index, scope.row)">
            编辑
          </el-button>
          <el-button type="danger" @click="handleDel(scope.$index, scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="用户名">
          <el-input v-model="form.username" v-if="mode == '0'" />
        </el-form-item>
        <el-form-item label="密码" v-if="mode == '0'">
          <el-input v-model="form.password" type="password" />
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model="form.nickname" />
        </el-form-item>
        <el-form-item label="性别">
          <el-select v-model="form.sex" placeholder="请选择">
            <el-option v-for="item in sexOptions" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="年龄">
          <el-input v-model="form.age" type="number" />
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
import myaxios from 'axios';
import { ElMessage } from 'element-plus';
import { getToken } from '../../utils/auth';
let mode = '0'
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  role: '2',
})
const state = reactive({
  data: {},
  curUser: {}
})

const sexFormatter = (row, col, value) => {
  if (value == '0') {
    return '男'
  }
  return '女'
}
const sexOptions = [{ value: '0', label: '男' }, { value: '1', label: '女' }]
const form = reactive({
  username: '',
  password: '',
  nickname: '',
  role: '2',
  age: null,
  sex: '0',
  id: ''

})

const dialogVisible1 = ref(false)

const clearData = () => {
  form.username = ''
  form.password = ''
  form.nickname = ''
  form.role = '2'
  form.age = null
  form.sex = '0'
  form.id = ''
  mode = '0'
}


const getUserList = () => {

  axios.get('user/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}

const saveOrUpdate = () => {
  if (mode == '0') {


    axios.post('user', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getUserList()


    })
  } else {
    axios.put('user', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getUserList()


    })
  }


}

const copyValue = (src, target) => {
  // 遍历 target 中的 key，并将 src 对应属性赋值给 target
  Object.keys(target).forEach((key) => {
    if (src[key] !== undefined) {
      target[key] = src[key] // 仅赋值存在于 src 中的属性
    }
  })
}

const handleAdd = () => {
  mode = '0'
  dialogVisible1.value = true
}

const handleEdit = (index, row) => {
  mode = '1'
  copyValue(row, form)
  dialogVisible1.value = true
}
const handleDel = (index, row) => {
  axios.delete('user/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getUserList()
  })
}

const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getUserList()
}
const uploadRef = ref(null)

const handleExceed = (files) => {
  if (uploadRef.value) {
    uploadRef.value.clearFiles();
    const file = files[0];
    file.uid = genFileId();
    uploadRef.value.handleStart(file);
  }
};

const handleSuccess = () => {
  ElMessage.success('导入成功')
  getUserList()
}

const handleExport = () => {
  let params = { role: '2' }
  myaxios({
    url: '/api/user/export',
    params: params,
    method: 'GET',
    responseType: 'blob' // 指定响应类型为 blob，处理二进制数据
  }).then(response => {
    // 创建一个 Blob 对象
    const blob = new Blob([response.data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

    // 创建下载链接
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);
    link.href = url;
    link.setAttribute('download', '老人信息.xlsx'); // 设置下载文件名

    // 触发下载
    document.body.appendChild(link);
    link.click();

    // 移除链接
    document.body.removeChild(link);
    URL.revokeObjectURL(url);
  }).catch(error => {
    console.error("Download failed:", error);
  });
}

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

.btn-inport {
  position: absolute;
  right: 180px;
}

.btn-export {
  position: absolute;
  right: 100px;
}
</style>