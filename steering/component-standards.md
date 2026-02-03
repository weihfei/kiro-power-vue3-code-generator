# 组件使用规范

生成代码时，**必须遵循本规范的组件使用规则**。

## 核心原则

当 `src/components/ELESUI/` 中有可用的项目封装组件时，**必须使用封装组件**，不得直接使用 Element UI 原生组件。

## 封装组件列表

### 自动导入的组件（无需 import）

以下组件已在 `src/components/ELESUI/index.ts` 中导出，会被自动导入：

| 封装组件 | 替代的 Element UI 组件 | 说明 |
|---------|----------------------|------|
| `<e-button>` | `<el-button>` | 按钮组件 |
| `<e-input>` | `<el-input>` | 输入框 |
| `<e-select>` | `<el-select>` | 下拉选择框（支持 multiple） |
| `<e-table>` | `<el-table>` | 表格组件 |
| `<e-dialog>` | `<el-dialog>` | 对话框 |
| `<e-pagination>` | `<el-pagination>` | 分页组件 |
| `<e-date-time-range>` | `<el-date-picker type="datetimerange">` | 日期时间范围选择器 |
| `<e-checkbox-group>` | `<el-checkbox-group>` | 复选框组 |
| `<e-radio-group>` | `<el-radio-group>` | 单选框组 |
| `<e-tooltip>` | `<el-tooltip>` | 提示组件 |
| `<e-dialog-button>` | - | 对话框按钮组件 |

### 需要手动导入的组件

以下组件未在 index.ts 中导出，使用时需要手动导入：

```typescript
import ESteps from '@/components/ELESUI/ESteps.vue';
import ESwitch from '@/components/ELESUI/ESwitch.vue';
import ETabs from '@/components/ELESUI/ETabs.vue';
import EAutoComplete from '@/components/ELESUI/EAutoComplete.vue';
import EEditor from '@/components/ELESUI/EEditor.vue';
import EProgress from '@/components/ELESUI/EProgress.vue';
import EButtonTabs from '@/components/ELESUI/EButtonTabs.vue';
```

| 封装组件 | 替代的 Element UI 组件 | 说明 |
|---------|----------------------|------|
| `<e-steps>` | `<el-steps>` | 步骤条 |
| `<e-switch>` | `<el-switch>` | 开关 |
| `<e-tabs>` | `<el-tabs>` | 标签页 |
| `<e-autocomplete>` | `<el-autocomplete>` | 自动完成输入框 |
| `<e-editor>` | - | 富文本编辑器 |
| `<e-progress>` | `<el-progress>` | 进度条 |
| `<e-button-tabs>` | - | 按钮式标签页 |

### 可直接使用 Element UI 的组件

以下组件尚未封装，可以直接使用 Element UI 原生组件：

- `<el-form>` / `<el-form-item>` - 表单组件
- `<el-card>` - 卡片容器
- `<el-date-picker>` - 单日期选择器（非范围选择）
- `<el-collapse>` / `<el-collapse-item>` - 折叠面板
- `<el-upload>` - 文件上传
- `<el-table-column>` - 表格列（配合 e-table 使用）
- `<el-tab-pane>` - 标签页面板（配合 e-tabs 使用）

## 代码示例

### ❌ 错误示例

```vue
<template>
  <!-- ❌ 错误：使用 Element UI 原生组件 -->
  <el-button type="primary" @click="handleSubmit">提交</el-button>
  
  <el-input v-model="keyword" placeholder="请输入关键词" />
  
  <el-select v-model="status" placeholder="请选择状态">
    <el-option label="启用" value="1" />
    <el-option label="禁用" value="0" />
  </el-select>
  
  <el-table :data="tableData" border>
    <el-table-column prop="name" label="名称" />
  </el-table>
  
  <el-dialog v-model="visible" title="提示">
    <p>对话框内容</p>
  </el-dialog>
  
  <el-tabs v-model="activeTab">
    <el-tab-pane label="标签1" name="1">内容1</el-tab-pane>
  </el-tabs>
  
  <el-pagination
    :total="total"
    :page-size="pageSize"
    @current-change="handlePageChange"
  />
</template>
```

### ✅ 正确示例（自动导入的组件）

```vue
<template>
  <!-- ✅ 正确：使用封装组件 -->
  <e-button type="primary" @click="handleSubmit">提交</e-button>
  
  <e-input v-model="keyword" placeholder="请输入关键词" />
  
  <e-select 
    v-model="status" 
    :options="statusOptions"
    placeholder="请选择状态"
  />
  
  <!-- 多选下拉框 -->
  <e-select 
    v-model="selectedValues" 
    :options="options"
    multiple
    placeholder="请选择多个"
  />
  
  <e-table :table-data="tableData" border>
    <el-table-column prop="name" label="名称" />
  </e-table>
  
  <e-dialog v-model="visible" title="提示">
    <p>对话框内容</p>
  </e-dialog>
  
  <e-pagination
    :total="total"
    :page-size="pageSize"
    @change="handlePageChange"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue';

// 这些组件自动导入，无需 import
const keyword = ref('');
const status = ref('');
const selectedValues = ref<string[]>([]);
const visible = ref(false);
const total = ref(0);
const pageSize = ref(10);

const statusOptions = [
  { label: '启用', value: '1' },
  { label: '禁用', value: '0' }
];

const options = [
  { label: '选项1', value: '1' },
  { label: '选项2', value: '2' }
];

const tableData = ref([]);

const handleSubmit = () => {};
const handlePageChange = (page: number) => {};
</script>
```

### ✅ 正确示例（需要手动导入的组件）

```vue
<template>
  <!-- ✅ 使用需要手动导入的封装组件 -->
  <e-tabs v-model="activeTab">
    <el-tab-pane label="标签1" name="1">内容1</el-tab-pane>
    <el-tab-pane label="标签2" name="2">内容2</el-tab-pane>
  </e-tabs>
  
  <e-steps :active="activeStep" :steps="steps" />
  
  <e-switch v-model="switchValue" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
// ✅ 这些组件需要手动导入
import ETabs from '@/components/ELESUI/ETabs.vue';
import ESteps from '@/components/ELESUI/ESteps.vue';
import ESwitch from '@/components/ELESUI/ESwitch.vue';

const activeTab = ref('1');
const activeStep = ref(0);
const switchValue = ref(false);

const steps = [
  { title: '步骤1' },
  { title: '步骤2' },
  { title: '步骤3' }
];
</script>
```

### ✅ 正确示例（混合使用）

```vue
<template>
  <!-- ✅ 未封装的组件可以直接使用 Element UI -->
  <el-card shadow="hover">
    <template #header>
      <span>卡片标题</span>
    </template>
    
    <el-form ref="formRef" :model="form" :rules="rules">
      <el-form-item label="用户名" prop="username">
        <!-- ✅ 输入框使用封装组件 -->
        <e-input v-model="form.username" />
      </el-form-item>
      
      <el-form-item label="状态" prop="status">
        <!-- ✅ 下拉框使用封装组件 -->
        <e-select v-model="form.status" :options="statusOptions" />
      </el-form-item>
      
      <el-form-item label="生日" prop="birthday">
        <!-- ✅ 单日期选择器未封装，使用 Element UI -->
        <el-date-picker
          v-model="form.birthday"
          type="date"
          placeholder="请选择"
        />
      </el-form-item>
      
      <el-form-item>
        <!-- ✅ 按钮使用封装组件 -->
        <e-button type="primary" @click="handleSubmit">提交</e-button>
        <e-button @click="handleReset">重置</e-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import type { FormInstance, FormRules } from 'element-plus';

interface FormData {
  username: string;
  status: string;
  birthday: string;
}

const formRef = ref<FormInstance>();
const form = reactive<FormData>({
  username: '',
  status: '',
  birthday: ''
});

const statusOptions = [
  { label: '启用', value: '1' },
  { label: '禁用', value: '0' }
];

const rules: FormRules<FormData> = {
  username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
  status: [{ required: true, message: '请选择状态', trigger: 'change' }]
};

const handleSubmit = async () => {
  if (!formRef.value) return;
  
  try {
    await formRef.value.validate();
    // 提交逻辑
  } catch {
    // 校验失败
  }
};

const handleReset = () => {
  formRef.value?.resetFields();
};
</script>
```

## $Modal 工具使用

### 获取 $Modal 实例

```typescript
import { getCurrentInstance } from 'vue';

const {
  proxy: { $Modal }
} = getCurrentInstance() as any;
```

### $Modal 方法

```typescript
// 成功提示
$Modal.msgSuccess('操作成功');

// 警告提示
$Modal.msgWarning('请填写必填项');

// 错误提示
$Modal.msgError('操作失败，请稍后重试');

// 确认对话框
$Modal.confirm('确定要删除吗？').then(
  () => {
    // 确认逻辑
  },
  () => {
    // 取消逻辑
  }
);

// 带配置的确认对话框
$Modal.confirm({
  title: '确认操作',
  content: '您确定要执行此操作吗？',
  onOk: () => {
    // 确认逻辑
  }
});
```

## 业务组件

### 业务组件列表

项目中的业务组件使用 `Db` 前缀，位于 `src/components/business/` 目录：

| 组件名 | 用途 |
|--------|------|
| `<db-type-switcher>` | 类型切换器（Tab 样式） |
| `<db-anchor>` | 锚点导航组件 |
| `<db-import-file>` | 文件导入组件 |
| `<db-custom-select>` | 自定义下拉选择 |
| `<db-approval-history>` | 审批历史组件 |
| `<db-current-approval>` | 当前审批环节 |
| `<db-approval-flow-overview>` | 流程全景组件 |

### 业务组件使用示例

```vue
<template>
  <db-type-switcher 
    v-model="tabActiveName" 
    :tabs="pageTabs" 
    @tab-click="handleTabClick"
  >
    <template #custom-button>
      <e-button @click="handleCancel">返回</e-button>
      <e-button @click="handleSave" type="primary">保存草稿</e-button>
    </template>
  </db-type-switcher>
  
  <db-anchor
    :container-ref="containerRef"
    :anchor-info="anchorInfo"
    :is-back-top="true"
    v-model:open-module-list="openModuleList"
  />
  
  <db-import-file 
    ref="dbImportFileRef" 
    :import-api="materialImportApi" 
    :on-import-success="handleImportSuccess" 
  />
</template>
```

## 检查清单

生成代码后，检查以下项目：

- [ ] 没有使用 `<el-button>`，应使用 `<e-button>`
- [ ] 没有使用 `<el-input>`，应使用 `<e-input>`
- [ ] 没有使用 `<el-select>`，应使用 `<e-select>`
- [ ] 没有使用 `<el-table>`，应使用 `<e-table>`
- [ ] 没有使用 `<el-dialog>`，应使用 `<e-dialog>`
- [ ] 没有使用 `<el-pagination>`，应使用 `<e-pagination>`
- [ ] 没有使用 `<el-tabs>`，应使用 `<e-tabs>`（需手动导入）
- [ ] 没有使用 `<el-switch>`，应使用 `<e-switch>`（需手动导入）
- [ ] 需要手动导入的组件已正确导入
- [ ] 使用 `$Modal` 进行消息提示和确认框
