<template>
   <div class="app-container">
      <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch">
         <el-form-item label="MenuName" prop="menuName">
            <el-input
               v-model="queryParams.menuName"
               placeholder="MenuName"
               clearable
               style="width: 200px"
               @keyup.enter="handleQuery"
            />
         </el-form-item>
         <el-form-item label="Status" prop="status">
            <el-select v-model="queryParams.status" placeholder="Status" clearable style="width: 200px">
               <el-option
                  v-for="dict in sys_normal_disable"
                  :key="dict.value"
                  :label="dict.label"
                  :value="dict.value"
               />
            </el-select>
         </el-form-item>
         <el-form-item>
            <el-button type="primary" icon="Search" @click="handleQuery">Query</el-button>
            <el-button icon="Refresh" @click="resetQuery">Reset</el-button>
         </el-form-item>
      </el-form>

      <el-row :gutter="10" class="mb8">
         <el-col :span="1.5">
            <el-button
               type="primary"
               plain
               icon="Plus"
               @click="handleAdd"
               v-hasPermi="['system:menu:add']"
            >Add</el-button>
         </el-col>
         <el-col :span="1.5">
            <el-button 
               type="info"
               plain
               icon="Sort"
               @click="toggleExpandAll"
            >unfold/fold</el-button>
         </el-col>
         <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
      </el-row>

      <el-table
         v-if="refreshTable"
         v-loading="loading"
         :data="menuList"
         row-key="menuId"
         :default-expand-all="isExpandAll"
         :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
      >
         <el-table-column prop="menuName" label="MenuName" :show-overflow-tooltip="true" width="160"></el-table-column>
         <el-table-column prop="icon" label="Icon" align="center" width="100">
            <template #default="scope">
               <svg-icon :icon-class="scope.row.icon" />
            </template>
         </el-table-column>
         <el-table-column prop="orderNum" label="OrderNum" width="90"></el-table-column>
         <el-table-column prop="perms" label="permissions" :show-overflow-tooltip="true"></el-table-column>
         <el-table-column prop="component" label="Component" :show-overflow-tooltip="true"></el-table-column>
         <el-table-column prop="status" label="Status" width="80">
            <template #default="scope">
               <dict-tag :options="sys_normal_disable" :value="scope.row.status" />
            </template>
         </el-table-column>
         <el-table-column label="CreateTime" align="center" width="160" prop="createTime">
            <template #default="scope">
               <span>{{ parseTime(scope.row.createTime) }}</span>
            </template>
         </el-table-column>
         <el-table-column label="Operation" align="center" width="280" class-name="small-padding fixed-width">
            <template #default="scope">
               <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['system:menu:edit']">Update</el-button>
               <el-button link type="primary" icon="Plus" @click="handleAdd(scope.row)" v-hasPermi="['system:menu:add']">Add</el-button>
               <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['system:menu:remove']">Delete</el-button>
            </template>
         </el-table-column>
      </el-table>

      <!-- 添加或修改菜单对话框 -->
      <el-dialog :title="title" v-model="open" width="980px" append-to-body>
         <el-form ref="menuRef" :model="form" :rules="rules" label-width="200px">
            <el-row>
               <el-col :span="24">
                  <el-form-item label="Parent menu">
                     <el-tree-select
                        v-model="form.parentId"
                        :data="menuOptions"
                        :props="{ value: 'menuId', label: 'menuName', children: 'children' }"
                        value-key="menuId"
                        placeholder="Parent menu"
                        check-strictly
                     />
                  </el-form-item>
               </el-col>
               <el-col :span="24">
                  <el-form-item label="MenuType" prop="menuType">
                     <el-radio-group v-model="form.menuType">
                        <el-radio value="M">Directory</el-radio>
                        <el-radio value="C">Menu</el-radio>
                        <el-radio value="F">Button</el-radio>
                     </el-radio-group>
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType != 'F'">
                  <el-form-item label="Menu icon" prop="icon">
                     <el-popover
                        placement="bottom-start"
                        :width="540"
                        trigger="click"
                     >
                        <template #reference>
                           <el-input v-model="form.icon" placeholder="Menu icon" @blur="showSelectIcon" readonly>
                              <template #prefix>
                                 <svg-icon
                                    v-if="form.icon"
                                    :icon-class="form.icon"
                                    class="el-input__icon"
                                    style="height: 32px;width: 16px;"
                                 />
                                 <el-icon v-else style="height: 32px;width: 16px;"><search /></el-icon>
                              </template>
                           </el-input>
                        </template>
                        <icon-select ref="iconSelectRef" @selected="selected" :active-icon="form.icon" />
                     </el-popover>
                  </el-form-item>
               </el-col>
               <el-col :span="12">
                  <el-form-item label="OrderNum" prop="orderNum">
                     <el-input-number v-model="form.orderNum" controls-position="right" :min="0" />
                  </el-form-item>
               </el-col>
               <el-col :span="12">
                  <el-form-item label="MenuName" prop="menuName">
                     <el-input v-model="form.menuName" placeholder="请输入菜单名称" />
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType == 'C'">
                  <el-form-item prop="routeName">
                     <template #label>
                        <span>
                           <el-tooltip content="If the address is' user ', the name is' User '(Note: because the router will delete routes with the same name, in order to avoid name conflict, please customize in special cases to ensure uniqueness)." placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           RouteName
                        </span>
                     </template>
                     <el-input v-model="form.routeName" placeholder="RouteName" />
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType != 'F'">
                  <el-form-item>
                     <template #label>
                        <span>
                           <el-tooltip content="If external link is selected, the routing address needs to start with 'http(s)://'" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>Outside chain or not
                        </span>
                     </template>
                     <el-radio-group v-model="form.isFrame">
                        <el-radio value="0">Yes</el-radio>
                        <el-radio value="1">No</el-radio>
                     </el-radio-group>
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType != 'F'">
                  <el-form-item prop="path">
                     <template #label>
                        <span>
                           <el-tooltip content="The routing address of the access, such as: 'user', if the external network address requires internal link access, start with 'http(s)://'" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           RoutePath
                        </span>
                     </template>
                     <el-input v-model="form.path" placeholder="RoutePath" />
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType == 'C'">
                  <el-form-item prop="component">
                     <template #label>
                        <span>
                           <el-tooltip content="The access component path, such as' system/user/index ', defaults to the 'views' directory" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           Component
                        </span>
                     </template>
                     <el-input v-model="form.component" placeholder="Component" />
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType != 'M'">
                  <el-form-item>
                     <el-input v-model="form.perms" placeholder="Permission identification" maxlength="100" />
                     <template #label>
                        <span>
                           <el-tooltip content="Permission characters defined in the controller, such as: @PreAuthorize(' @ss.hasPermi('system:user:list') ')" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           Permission character
                        </span>
                     </template>
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType == 'C'">
                  <el-form-item>
                     <el-input v-model="form.query" placeholder="Route parameter" maxlength="255" />
                     <template #label>
                        <span>
                           <el-tooltip content='Default pass parameters for access routes, such as `{"id": 1, "name": "ry"}`' placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           Route parameter
                        </span>
                     </template>
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType == 'C'">
                  <el-form-item>
                     <template #label>
                        <span>
                           <el-tooltip content="If yes is selected, it will be cached by 'keep-alive', and the name and address of the matching component must be consistent" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           Cache or not
                        </span>
                     </template>
                     <el-radio-group v-model="form.isCache">
                        <el-radio value="0">Cache</el-radio>
                        <el-radio value="1">Uncached</el-radio>
                     </el-radio-group>
                  </el-form-item>
               </el-col>
               <el-col :span="12" v-if="form.menuType != 'F'">
                  <el-form-item>
                     <template #label>
                        <span>
                           <el-tooltip content="Select Hide and the route will not appear in the sidebar, but will still be accessible" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           Display status
                        </span>
                     </template>
                     <el-radio-group v-model="form.visible">
                        <el-radio
                           v-for="dict in sys_show_hide"
                           :key="dict.value"
                           :value="dict.value"
                        >{{ dict.label }}</el-radio>
                     </el-radio-group>
                  </el-form-item>
               </el-col>
               <el-col :span="12">
                  <el-form-item>
                     <template #label>
                        <span>
                           <el-tooltip content="If disabled is selected, the route will not appear in the sidebar and cannot be accessed" placement="top">
                              <el-icon><question-filled /></el-icon>
                           </el-tooltip>
                           MenuStatus
                        </span>
                     </template>
                     <el-radio-group v-model="form.status">
                        <el-radio
                           v-for="dict in sys_normal_disable"
                           :key="dict.value"
                           :value="dict.value"
                        >{{ dict.label }}</el-radio>
                     </el-radio-group>
                  </el-form-item>
               </el-col>
            </el-row>
         </el-form>
         <template #footer>
            <div class="dialog-footer">
               <el-button type="primary" @click="submitForm">Confirm</el-button>
               <el-button @click="cancel">Cancel</el-button>
            </div>
         </template>
      </el-dialog>
   </div>
</template>

<script setup name="Menu">
import { addMenu, delMenu, getMenu, listMenu, updateMenu } from "@/api/system/menu";
import SvgIcon from "@/components/SvgIcon";
import IconSelect from "@/components/IconSelect";

const { proxy } = getCurrentInstance();
const { sys_show_hide, sys_normal_disable } = proxy.useDict("sys_show_hide", "sys_normal_disable");

const menuList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const title = ref("");
const menuOptions = ref([]);
const isExpandAll = ref(false);
const refreshTable = ref(true);
const iconSelectRef = ref(null);

const data = reactive({
  form: {},
  queryParams: {
    menuName: undefined,
    visible: undefined
  },
  rules: {
    menuName: [{ required: true, message: "The menu name cannot be empty", trigger: "blur" }],
    orderNum: [{ required: true, message: "The menu orderNum cannot be empty", trigger: "blur" }],
    path: [{ required: true, message: "The route path cannot be empty", trigger: "blur" }]
  },
});

const { queryParams, form, rules } = toRefs(data);

/** 查询菜单列表 */
function getList() {
  loading.value = true;
  listMenu(queryParams.value).then(response => {
    menuList.value = proxy.handleTree(response.data, "menuId");
    loading.value = false;
  });
}

/** 查询菜单下拉树结构 */
function getTreeselect() {
  menuOptions.value = [];
  listMenu().then(response => {
    const menu = { menuId: 0, menuName: "Main category", children: [] };
    menu.children = proxy.handleTree(response.data, "menuId");
    menuOptions.value.push(menu);
  });
}

/** 取消按钮 */
function cancel() {
  open.value = false;
  reset();
}

/** 表单重置 */
function reset() {
  form.value = {
    menuId: undefined,
    parentId: 0,
    menuName: undefined,
    icon: undefined,
    menuType: "M",
    orderNum: undefined,
    isFrame: "1",
    isCache: "0",
    visible: "0",
    status: "0"
  };
  proxy.resetForm("menuRef");
}

/** 展示下拉图标 */
function showSelectIcon() {
  iconSelectRef.value.reset();
}

/** 选择图标 */
function selected(name) {
  form.value.icon = name;
}

/** 搜索按钮操作 */
function handleQuery() {
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

/** 新增按钮操作 */
function handleAdd(row) {
  reset();
  getTreeselect();
  if (row != null && row.menuId) {
    form.value.parentId = row.menuId;
  } else {
    form.value.parentId = 0;
  }
  open.value = true;
  title.value = "Add menu";
}

/** 展开/折叠操作 */
function toggleExpandAll() {
  refreshTable.value = false;
  isExpandAll.value = !isExpandAll.value;
  nextTick(() => {
    refreshTable.value = true;
  });
}

/** 修改按钮操作 */
async function handleUpdate(row) {
  reset();
  await getTreeselect();
  getMenu(row.menuId).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "Update menu";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["menuRef"].validate(valid => {
    if (valid) {
      if (form.value.menuId != undefined) {
        updateMenu(form.value).then(response => {
          proxy.$modal.msgSuccess("Modified successfully");
          open.value = false;
          getList();
        });
      } else {
        addMenu(form.value).then(response => {
          proxy.$modal.msgSuccess("New success");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  proxy.$modal.confirm('Do you want to delete the name as"' + row.menuName + '"data item?').then(function() {
    return delMenu(row.menuId);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("Successfully deleted");
  }).catch(() => {});
}

getList();
</script>
