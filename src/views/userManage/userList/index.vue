<template>
  <div class="table-box">
    <ProTable
      ref="proTable"
      title="用户列表"
      :columns="columns"
      :request-api="getTableList"
      :init-param="initParam"
      :data-callback="dataCallback"
    >
      <!-- 表格 header 按钮 -->
      <template #tableHeader="scope">
        <el-button v-auth="'add'" type="primary" :icon="CirclePlus" @click="openDrawer('新增')"> 添加会员 </el-button>
        <el-button v-auth="'batchAdd'" type="primary" :icon="Upload" plain @click="batchAdd"> 导入用户 </el-button>
        <el-button v-auth="'export'" type="primary" :icon="Download" plain @click="downloadFile"> 会员导出 </el-button>
        <el-button type="primary" plain @click="toDetail"> To 子集详情页面 </el-button>
        <el-button type="danger" :icon="Delete" plain :disabled="!scope.isSelected" @click="batchDelete(scope.selectedListIds)">
          批量删除用户
        </el-button>
      </template>
      <!-- 表格操作 -->
      <template #operation="scope">
        <el-button type="primary" link :icon="View" @click="openDrawer('查看', scope.row)"> 聊天 </el-button>
        <el-button type="primary" link :icon="View" @click="openDrawer('查看', scope.row)"> 禁言 </el-button>
        <el-button type="primary" link :icon="EditPen" @click="openDrawer('编辑', scope.row)"> 注销 </el-button>
        <el-button type="primary" link :icon="Refresh" @click="resetPass(scope.row)"> 内容清除 </el-button>
        <el-button type="primary" link :icon="Delete" @click="deleteAccount(scope.row)"> 详情 </el-button>
      </template>
    </ProTable>
    <UserDrawer ref="drawerRef" />
    <ImportExcel ref="dialogRef" />
  </div>
</template>

<script setup lang="tsx" name="userList">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import { User } from "@/api/interface";
import { useHandleData } from "@/hooks/useHandleData";
import { useDownload } from "@/hooks/useDownload";
import { ElMessageBox } from "element-plus";
import ProTable from "@/components/ProTable/index.vue";
import ImportExcel from "@/components/ImportExcel/index.vue";
import UserDrawer from "@/views/proTable/components/UserDrawer.vue";
import { ProTableInstance, ColumnProps } from "@/components/ProTable/interface";
import { CirclePlus, Delete, EditPen, Download, Upload, View, Refresh } from "@element-plus/icons-vue";
import {
  getUserList,
  deleteUser,
  editUser,
  addUser,
  resetUserPassWord,
  exportUserInfo,
  BatchAddUser,
  getUserStatus,
  getUserGender
} from "@/api/modules/user";

const router = useRouter();

// 跳转详情页
const toDetail = () => {
  router.push(`/proTable/useProTable/detail/${Math.random().toFixed(3)}?params=detail-page`);
};

// 获取 ProTable 元素，调用其获取刷新数据方法（还能获取到当前查询参数，方便导出携带参数）
const proTable = ref<ProTableInstance>();

// 如果表格需要初始化请求参数，直接定义传给 ProTable(之后每次请求都会自动带上该参数，此参数更改之后也会一直带上，改变此参数会自动刷新表格数据)
const initParam = reactive({ type: 1 });

// dataCallback 是对于返回的表格数据做处理，如果你后台返回的数据不是 list && total && pageNum && pageSize 这些字段，那么你可以在这里进行处理成这些字段
// 或者直接去 hooks/useTable.ts 文件中把字段改为你后端对应的就行
const dataCallback = (data: any) => {
  return {
    list: data.list,
    total: data.total,
    pageNum: data.pageNum,
    pageSize: data.pageSize
  };
};

// 如果你想在请求之前对当前请求参数做一些操作，可以自定义如下函数：params 为当前所有的请求参数（包括分页），最后返回请求列表接口
// 默认不做操作就直接在 ProTable 组件上绑定	:requestApi="getUserList"
const getTableList = (params: any) => {
  let newParams = JSON.parse(JSON.stringify(params));
  newParams.createTime && (newParams.startTime = newParams.createTime[0]);
  newParams.createTime && (newParams.endTime = newParams.createTime[1]);
  delete newParams.createTime;
  return getUserList(newParams);
};

// 表格配置项
const columns: ColumnProps<User.ResUserList>[] = [
  { type: "selection", fixed: "left", width: 80 },
  { prop: "id", label: "ID" },
  { prop: "id", label: "用户ID", search: { el: "input" } },
  { prop: "username", label: "用户姓名" },
  { prop: "username", label: "用户名" },
  {
    prop: "gender",
    label: "性别",
    // 字典数据
    // enum: genderType,
    // 字典请求不带参数
    enum: [
      { label: "男", value: 1 },
      { label: "女", value: 2 }
    ],
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "username",
    label: "状态",
    enum: [
      { label: "待审", value: 1 },
      { label: "通过", value: 2 },
      { label: "不通过", value: 3 },
      { label: "在审核", value: 4 }
    ],
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "username",
    label: "隐私设置",
    enum: [
      { label: "完全公开", value: 1 },
      { label: "私密", value: 2 },
      { label: "仅好友", value: 3 }
    ],
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "username",
    label: "推荐",
    enum: [
      { label: "是", value: 0 },
      { label: "否", value: 1 }
    ],
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "username",
    label: "注册地",
    enum: getUserGender,
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "username",
    label: "注册来源",
    enum: [
      { label: "APP", value: 1 },
      { label: "小程序", value: 2 },
      { label: "群组", value: 3 },
      { label: "二维码", value: 4 }
    ],
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    search: { el: "select", props: { filterable: true } },
    fieldNames: { label: "label", value: "value" }
  },
  {
    prop: "createTime",
    label: "注册时间",
    width: 180,
    search: {
      el: "date-picker",
      span: 2,
      props: { type: "datetimerange", valueFormat: "YYYY-MM-DD HH:mm:ss" },
      defaultValue: ["2022-11-12 11:35:00", "2022-12-12 11:35:00"]
    }
  },
  { prop: "username", label: "使用时长" },
  { prop: "username", label: "登录次数" },
  {
    prop: "level",
    label: "等级",
    // 字典数据
    // enum: genderType,
    // 字典请求不带参数
    enum: getUserStatus,
    // 字典请求携带参数
    // enum: () => getUserGender({ id: 1 }),
    fieldNames: { label: "genderLabel", value: "genderValue" }
  },
  { prop: "operation", label: "操作", fixed: "right", width: 330 }
];

// 删除用户信息
const deleteAccount = async (params: User.ResUserList) => {
  await useHandleData(deleteUser, { id: [params.id] }, `删除【${params.username}】用户`);
  proTable.value?.getTableList();
};

// 批量删除用户信息
const batchDelete = async (id: string[]) => {
  await useHandleData(deleteUser, { id }, "删除所选用户信息");
  proTable.value?.clearSelection();
  proTable.value?.getTableList();
};

// 重置用户密码
const resetPass = async (params: User.ResUserList) => {
  await useHandleData(resetUserPassWord, { id: params.id }, `重置【${params.username}】用户密码`);
  proTable.value?.getTableList();
};

// 导出用户列表
const downloadFile = async () => {
  ElMessageBox.confirm("确认导出用户数据?", "温馨提示", { type: "warning" }).then(() =>
    useDownload(exportUserInfo, "用户列表", proTable.value?.searchParam)
  );
};

// 批量添加用户
const dialogRef = ref<InstanceType<typeof ImportExcel> | null>(null);
const batchAdd = () => {
  const params = {
    title: "用户",
    tempApi: exportUserInfo,
    importApi: BatchAddUser,
    getTableList: proTable.value?.getTableList
  };
  dialogRef.value?.acceptParams(params);
};

// 打开 drawer(新增、查看、编辑)
const drawerRef = ref<InstanceType<typeof UserDrawer> | null>(null);
const openDrawer = (title: string, row: Partial<User.ResUserList> = {}) => {
  const params = {
    title,
    isView: title === "查看",
    row: { ...row },
    api: title === "新增" ? addUser : title === "编辑" ? editUser : undefined,
    getTableList: proTable.value?.getTableList
  };
  drawerRef.value?.acceptParams(params);
};
</script>
