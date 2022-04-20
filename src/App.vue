<template>
  <el-form
      ref="ruleFormRef"
      :model="ruleForm"
      :rules="rules"
      label-width="700px"
      class="demo-ruleForm"
      :size="formSize">
    <h2>QQ帳號查詢工具</h2>
    <el-form-item label="QQ帳號" prop="qqNumber" required>
      <el-input v-model="ruleForm.qqNumber" style="width:250px"/>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm(ruleFormRef)">查詢</el-button>
      <el-button @click="resetForm(ruleFormRef)">重置</el-button>
    </el-form-item>
  </el-form>
  <el-dialog v-model="dialogVisible" title="查詢到的資料" width="30%" draggable>
    <span>QQ號碼:{{userInfo.qqNumber}}</span><br>
    <span>QQ名字:{{userInfo.userName}}</span><br>
    <span>QQ頭像:<el-avatar :size="50" :src="userInfo.avatar"></el-avatar></span>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="dialogVisible = false">確認</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
import axios from 'axios'

const dialogVisible = ref(false)
axios.defaults.baseURL = "https://api.usuuu.com/qq/"
// 前置攔截
axios.interceptors.request.use(config => {
  return config;
});

// 後置攔截
axios.interceptors.response.use(response => {
  let res = response.data;
  if (res.code === 200) {
    return response;
  }
}, error => {
  if (error.response.data.code === 500) {
    ElMessage('請輸入正確的QQ帳號!');
  }
  return Promise.reject('');
});

const formSize = ref('default');
const ruleFormRef = ref<FormInstance>();
const ruleForm = reactive({
  id: ''
});

const userInfo = reactive({
  qqNumber: '',
  userName: '',
  avatar: ''
});

const rules = reactive<FormRules>({
  qqNumber: [
    { required: true, message: '請輸入QQ帳號', trigger: 'blur' },
  ],
});

const submitForm = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      const { qqNumber } = ruleForm
      axios({
        method: 'get',
        url: qqNumber,
      }).then(({ data }) => {
        userInfo.qqNumber = data.data.qq;
        userInfo.userName = data.data.name;
        userInfo.avatar = data.data.avatar;
        dialogVisible.value = true;
      })
    } else {
    }
  })
}

const resetForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.resetFields()
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>