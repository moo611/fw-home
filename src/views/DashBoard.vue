<template>
  <div class="main">
    <div class="header">
      <img src="../assets/yly.jpg" style="width: 60px; height: 60px;margin-left: 30px;"/>
      <h3 style="margin-left: 30px;">养老院管理系统</h3>
      
      <div class="btn-logout">
        <el-popover v-model:visible="popoverVisible" placement="bottom" width="200">
          <el-button type="text" @click="logout">退出登录</el-button>
          <template #reference>
            <div @click="popoverVisible = !popoverVisible" style="display: flex; align-items: center; cursor: pointer;">

              <img style="width: 40px; height: 40px;" src="../assets/avt_default.png" />
              <span style="margin-left: 8px;">{{ curName }}</span>
            </div>
          </template>
        </el-popover>
      </div>

    </div>
    <div style="height: 1px; width: 100%; background-color: #f0f0f0;"></div>
    <div class="content">
      <div class="sidebar">

        <el-menu active-text-color="#ffd04b" background-color="#545c64" text-color="#fff" :default-active="activeMenu"
          router class="custom-menu">

          <el-menu-item v-for="item, index in menuList" :index="item.value">
            <span>{{ item.label }}</span>
          </el-menu-item>
        </el-menu>
      </div>

      <router-view class="container">
      </router-view>


    </div>

  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router';
import { clear, setUser } from '../utils/auth';
import axios from '../axios'
const popoverVisible = ref(false)
const menuList = ref([])
const router = useRouter();
const route = useRoute();
const activeMenu = ref(route.path);
const curName = ref('')
watch(route, (newRoute) => {
  activeMenu.value = newRoute.path;
});
function logout() {
  clear(); // 清除缓存的用户名
  router.push('/login'); // 重定向到登录页
}


const getInfo = () => {

  axios.get('user/info').then(res => {
    const user = res
    console.log(user)
    setUser(user)
    curName.value = user.nickname

    //动态获取路由
    let menus = []
    if (user.role == '0' || user.role == '1') {

      menus.push({ label: '信息管理', value: '/dashboard/user' })
      menus.push({ label: '健康档案', value: '/dashboard/health' })
      menus.push({ label: '活动管理', value: '/dashboard/activity' })
      menus.push({ label: '服务管理', value: '/dashboard/service' })
    } else {

      menus.push({ label: '活动列表', value: '/dashboard/activity' })
      menus.push({ label: '我的活动', value: '/dashboard/join' })
      menus.push({ label: '服务列表', value: '/dashboard/service' })
      menus.push({ label: '我的预约', value: '/dashboard/reserve' })
    }
    menuList.value = menus

  })
}

getInfo()

</script>
<style lang="css" scoped>
/* 使用 ::v-deep 来覆盖 el-menu 的样式 */

.custom-menu{
  height: 100%;
}
::v-deep .custom-menu .el-menu-item{
  font-size: 16px;
  font-weight: 600;
}
.main {
  /* max-width: 1280px;
  margin: 0 auto;
  background-color: transparent; */
  width: 100%;

}

.header {
  height: 80px;
  /* background-color: #409eff; */
  position: relative;
  background-color: transparent;
  display: flex;
  align-items: center;
}

.btn-logout {
  position: absolute;
  right: 30px;

}

.sidebar {
  width: 20%;
  max-width: 200px;
}

.content {
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  height: calc(100vh - 100px);
}

.container {

  overflow-y: auto;
  width: 100%;
  height: 100%;
  padding: 20px;
  /* margin: 30px; */
  border-radius: 12px;
  box-sizing: border-box;
}
</style>
