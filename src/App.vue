<script setup lang="ts">
import { reactive, ref, computed, onMounted, watch } from "vue";
import type { FormInstance, FormRules } from "element-plus";
import dayjs from "dayjs";
import axios from "axios";
const formRef = ref<FormInstance>();
interface User {
  id: string;
  date: string;
  name: string;
}
const formInline = reactive({
  todo: "",
});
const unfinishNum = ref(0);
const finishedNum = ref(0);
if (localStorage.getItem("finishedNum")) {
  finishedNum.value = parseInt(localStorage.getItem("finishedNum")!, 10);
}
watch(finishedNum, (newValue) => {
  localStorage.setItem("finishedNum", newValue.toString());
});
const getList = async () => {
  const res = await axios.get("http://localhost:3000/data");
  tableData.value = res.data;
  unfinishNum.value = res.data.length;
};
const tableData = ref<User[]>([]);
onMounted(() => {
  getList();
});
const rules = reactive<FormRules>({
  todo: [{ required: true, message: "请填写ToDo事项", trigger: "blur" }],
});
const todoData = reactive({
  date: "",
  name: "",
  id: 0,
});
const resetForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.resetFields();
};
const onSubmit = async (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  await formEl.validate((valid, fields) => {
    if (valid) {
      todoData.date = dayjs(Date.now()).format("YYYY-MM-DD");
      todoData.name = formInline.todo;
      const date = new Date();
      todoData.id = date.getTime();
      axios.post("http://localhost:3000/data", todoData).then(() => {
        getList();
        formInline.todo = "";
      });
    } else {
      alert("未输入内容");
    }
  });
};

const search = ref("");
const filterTableData = computed(() =>
  tableData.value.filter(
    (data) =>
      !search.value ||
      data.name.toLowerCase().includes(search.value.toLowerCase())
  )
);
const handleDelete = async (id) => {
  await axios.delete(`http://localhost:3000/data/${id}`);
  getList();
};
const handleFinish = async (id) => {
  await axios.delete(`http://localhost:3000/data/${id}`);
  getList();
  finishedNum.value++;
};
const dialogFormVisible = ref(false)
const formLabelWidth = '140px'
const formId = ref(0)
const formDialog = reactive({
  todo: "",
});
const handleEdit=(row)=>{
  dialogFormVisible.value=true
  formId.value = row.id
  formDialog.todo = row.name
}
const dialogSubmit=async ()=>{
  const id = formId.value
  await axios.patch(`http://localhost:3000/data/${id}`,{
    name:formDialog.todo
  })
  formDialog.todo=''
  getList()
  dialogFormVisible.value=false
}
</script>

<template>
  <div class="common-layout">
    <el-container>
      <el-header>
        <h3>TodoList</h3>
        <el-form
          ref="formRef"
          :inline="true"
          :model="formInline"
          class="demo-form-inline"
          :rules="rules"
        >
          <el-form-item prop="todo">
            <el-input v-model="formInline.todo" placeholder="添加ToDo" />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit(formRef)"
              >提交</el-button
            >
          </el-form-item>
        </el-form>
      </el-header>
      <el-main>
        <h1>未完成（{{ unfinishNum }}）---- 已完成（{{ finishedNum }}）</h1>
        <el-table :data="filterTableData" style="width: 50%">
          <el-table-column label="Date" prop="date" />
          <el-table-column label="Name" prop="name" />
          <el-table-column align="right">
            <template #header>
              <el-input v-model="search" size="small" placeholder="搜索" />
            </template>
            <template #default="scope">
              <el-button
                size="small"
                type="danger"
                @click="handleDelete(scope.row.id)"
                >删除</el-button
              >
              <el-button
                size="small"
                type="primary"
                @click="handleFinish(scope.row.id)"
                >完成</el-button
              >
              <el-button
                size="small"
                type="primary"
                @click="handleEdit(scope.row)"
                >修改</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <el-dialog v-model="dialogFormVisible" title="修改Todo事项名称">
          <el-form :model="formDialog">
            <el-form-item label="Todo" :label-width="formLabelWidth">
              <el-input v-model="formDialog.todo" autocomplete="off" />
            </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="dialogFormVisible = false">取消</el-button>
              <el-button type="primary" @click="dialogSubmit()"
                >确认</el-button
              >
            </span>
          </template>
        </el-dialog>
      </el-main>
    </el-container>
  </div>
</template>

<style scoped lang="less">
* {
  margin: 0;
  box-sizing: border-box;
}
.common-layout {
  text-align: center;
  .el-header {
    background-color: #333333;
    display: flex;
    justify-content: space-between;
    h3 {
      color: #dddddd;
      line-height: 60px;
    }
    .el-form {
      vertical-align: middle;
      display: flex;
      align-items: center;
      .el-form-item:first-child {
        width: 800px;
      }
      .el-form-item--label-right {
        margin: 0;
      }
    }
  }
}
</style>
