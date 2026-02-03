# 代码块分类和组织规范

生成代码时，**必须按照本规范的顺序组织代码块**，不要随意插入新的代码逻辑或变量。

## 代码块顺序规范

### Vue 3 SFC 整体结构

```vue
<template>
  <!-- 模板内容 -->
</template>

<script setup lang="ts">
// 按以下顺序组织代码
</script>

<style lang="scss" scoped>
/* 样式定义 */
</style>
```

### Script 内部代码块顺序

```typescript
// ==================== 1. 导入声明 ====================
// 1.1 Vue 核心导入
import { ref, reactive, computed, onMounted, watch, nextTick } from 'vue';

// 1.2 Vue Router 导入
import { useRoute, useRouter } from 'vue-router';

// 1.3 Pinia Store 导入
import { storeToRefs } from 'pinia';
import { useUserStore } from '@/store/modules/user';

// 1.4 组件导入（需要手动导入的封装组件）
import ESteps from '@/components/ELESUI/ESteps.vue';
import ETabs from '@/components/ELESUI/ETabs.vue';

// 1.5 业务组件导入
import BasicInfo from './components/BasicInfo/index.vue';
import ContentList from './components/ContentList/index.vue';

// 1.6 API 导入
import { xxxRequest } from './api/index';

// 1.7 Hooks 导入
import { useCommitSave } from './hooks/useCommitSave';
import { useTableData } from './hooks/useTableData';

// 1.8 工具函数导入
import { formatDate, formatPrice, isEmptyStr } from '@/utils/util';
import { positiveIntegerReg } from '@/utils/validate';

// 1.9 类型导入
import type { FormInstance, FormRules } from 'element-plus';
import type { ListItem, FormData } from './interface';

// ==================== 2. 类型定义 ====================
// 2.1 接口类型
interface IFormData {
  name: string;
  status: number;
  type: string;
}

interface IListItem {
  id: number;
  name: string;
  enabled: boolean;
  createTime: string;
}

interface IQueryParams {
  page: number;
  size: number;
  keyword?: string;
}

// 2.2 组件 Ref 类型
type BasicInfoRef = InstanceType<typeof BasicInfo> | null;
type ContentListRef = InstanceType<typeof ContentList> | null;

// ==================== 3. 枚举/常量定义 ====================
// 3.1 状态枚举
enum StatusEnum {
  DISABLED = 0,
  ENABLED = 1,
  PENDING = 2
}

enum OperationTypeEnum {
  CREATE = 'create',
  EDIT = 'edit',
  VIEW = 'view'
}

// 3.2 下拉选项常量
const STATUS_OPTIONS = [
  { label: '全部', value: '' },
  { label: '启用', value: StatusEnum.ENABLED },
  { label: '禁用', value: StatusEnum.DISABLED }
] as const;

const TYPE_OPTIONS = [
  { label: '类型A', value: 'A' },
  { label: '类型B', value: 'B' }
] as const;

// 3.3 表单校验规则
const FORM_RULES: FormRules<IFormData> = {
  name: [
    { required: true, message: '请输入名称', trigger: 'blur' },
    { max: 50, message: '名称不能超过50个字符', trigger: 'blur' }
  ],
  type: [
    { required: true, message: '请选择类型', trigger: 'change' }
  ]
};

// ==================== 4. Props/Emits 定义 ====================
// 4.1 Props 定义
interface Props {
  modelValue?: boolean;
  data?: IListItem[];
  readonly?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: false,
  data: () => [],
  readonly: false
});

// 4.2 Emits 定义
const emit = defineEmits<{
  'update:modelValue': [value: boolean];
  'change': [item: IListItem];
  'submit': [data: IFormData];
}>();

// ==================== 5. 路由/Store 实例 ====================
// 5.1 路由实例
const route = useRoute();
const router = useRouter();

// 5.2 Store 实例
const userStore = useUserStore();
const { userInfo, permissions } = storeToRefs(userStore);

// 5.3 路由参数
const operationType = computed(() => route.query?.type as string);
const recordId = computed(() => route.query?.id as string);

// ==================== 6. 响应式数据 ====================
// 6.1 基础状态
const loading = ref(false);
const submitting = ref(false);
const visible = ref(false);

// 6.2 组件 Ref
const formRef = ref<FormInstance>();
const basicInfoRef = ref<BasicInfoRef>(null);
const contentListRef = ref<ContentListRef>(null);

// 6.3 表单数据
const formData = reactive<IFormData>({
  name: '',
  status: StatusEnum.ENABLED,
  type: ''
});

// 6.4 列表数据
const dataList = ref<IListItem[]>([]);
const total = ref(0);
const currentPage = ref(1);
const pageSize = ref(10);

// 6.5 筛选条件
const filterForm = reactive<IQueryParams>({
  page: 1,
  size: 10,
  keyword: ''
});

// 6.6 其他状态
const activeTab = ref('basic');
const selectedIds = ref<number[]>([]);

// ==================== 7. 计算属性 ====================
// 7.1 权限相关
const isReadonly = computed(() => {
  return props.readonly || operationType.value === OperationTypeEnum.VIEW;
});

const canEdit = computed(() => {
  return permissions.value.includes('edit');
});

// 7.2 数据相关
const filteredList = computed(() => {
  return dataList.value.filter(item => item.enabled);
});

const isEmpty = computed(() => {
  return dataList.value.length === 0;
});

// 7.3 表单相关
const isFormValid = computed(() => {
  return !isEmptyStr(formData.name) && !isEmptyStr(formData.type);
});

// ==================== 8. API 实例化 ====================
const { getListData, getDetailData, submitFormData, updateStatus } = xxxRequest();

// ==================== 9. Hooks 实例化 ====================
const { saveDraftData, commitData } = useCommitSave();
const { tableData, fetchTableData } = useTableData();

// ==================== 10. 方法定义 ====================
// 10.1 数据加载方法
/**
 * 获取列表数据
 */
const fetchList = async () => {
  loading.value = true;
  try {
    const params = { ...filterForm };
    const res = await getListData(params);
    
    if (!res) {
      $Modal.msgWarning('未获取到数据');
      return;
    }
    
    dataList.value = res.list ?? [];
    total.value = res.total ?? 0;
  } catch (error) {
    console.error('获取列表失败:', error);
    $Modal.msgError('获取列表失败');
  } finally {
    loading.value = false;
  }
};

/**
 * 获取详情数据
 */
const fetchDetail = async (id: string) => {
  loading.value = true;
  try {
    const res = await getDetailData(id);
    
    if (!res) {
      $Modal.msgWarning('未获取到详情');
      return;
    }
    
    Object.assign(formData, res);
  } catch (error) {
    console.error('获取详情失败:', error);
    $Modal.msgError('获取详情失败');
  } finally {
    loading.value = false;
  }
};

// 10.2 搜索/筛选方法
/**
 * 搜索
 */
const handleSearch = () => {
  filterForm.page = 1;
  fetchList();
};

/**
 * 重置筛选条件
 */
const handleReset = () => {
  Object.assign(filterForm, {
    page: 1,
    size: 10,
    keyword: ''
  });
  fetchList();
};

// 10.3 表单操作方法
/**
 * 表单提交
 */
const handleSubmit = async () => {
  if (!formRef.value) return;
  
  try {
    await formRef.value.validate();
    
    submitting.value = true;
    await submitFormData(formData);
    
    $Modal.msgSuccess('提交成功');
    visible.value = false;
    fetchList();
  } catch (error) {
    console.error('提交失败:', error);
    $Modal.msgError('提交失败');
  } finally {
    submitting.value = false;
  }
};

/**
 * 保存草稿
 */
const handleSaveDraft = async () => {
  try {
    submitting.value = true;
    await saveDraftData(formData);
    $Modal.msgSuccess('草稿保存成功');
  } catch (error) {
    console.error('保存草稿失败:', error);
    $Modal.msgError('保存草稿失败');
  } finally {
    submitting.value = false;
  }
};

// 10.4 状态切换方法
/**
 * 切换启用/禁用状态
 */
const handleToggleStatus = async (item: IListItem, value: boolean) => {
  try {
    await $Modal.confirm(`确定要${value ? '启用' : '禁用'}「${item.name}」吗？`);
    
    await updateStatus(item.id, value);
    $Modal.msgSuccess('操作成功');
    item.enabled = value;
  } catch {
    // 用户取消，不处理
  }
};

// 10.5 导航方法
/**
 * 返回列表页
 */
const handleBack = () => {
  router.back();
};

/**
 * 跳转编辑页
 */
const handleEdit = (id: number) => {
  router.push({
    path: '/edit',
    query: { id, type: OperationTypeEnum.EDIT }
  });
};

// 10.6 分页方法
/**
 * 分页变化
 */
const handlePageChange = (page: number) => {
  filterForm.page = page;
  fetchList();
};

/**
 * 每页条数变化
 */
const handleSizeChange = (size: number) => {
  filterForm.size = size;
  filterForm.page = 1;
  fetchList();
};

// 10.7 弹窗方法
/**
 * 打开新增弹窗
 */
const handleOpenCreate = () => {
  Object.assign(formData, { name: '', status: StatusEnum.ENABLED, type: '' });
  visible.value = true;
};

/**
 * 打开编辑弹窗
 */
const handleOpenEdit = async (item: IListItem) => {
  await fetchDetail(String(item.id));
  visible.value = true;
};

/**
 * 关闭弹窗
 */
const handleClose = () => {
  visible.value = false;
  formRef.value?.resetFields();
};

// 10.8 子组件方法调用
/**
 * 校验所有子组件
 */
const validateAllComponents = async (): Promise<boolean> => {
  const results = await Promise.all([
    basicInfoRef.value?.validate() ?? true,
    contentListRef.value?.validate() ?? true
  ]);
  
  return results.every(Boolean);
};

// ==================== 11. 监听器 ====================
// 11.1 Props 监听
watch(
  () => props.modelValue,
  (val) => {
    visible.value = val;
  }
);

// 11.2 路由参数监听
watch(
  () => route.query.id,
  (newId) => {
    if (newId) {
      fetchDetail(newId as string);
    }
  },
  { immediate: true }
);

// 11.3 表单数据监听
watch(
  () => formData.type,
  (newType) => {
    // 类型变化时的联动逻辑
    console.log('类型变化:', newType);
  }
);

// ==================== 12. 生命周期 ====================
onMounted(async () => {
  // 初始化数据
  await fetchList();
  
  // 编辑模式加载详情
  if (recordId.value) {
    await fetchDetail(recordId.value);
  }
});

// ==================== 13. 暴露方法 ====================
defineExpose({
  fetchList,
  formData,
  validate: validateAllComponents
});
```

## 代码块说明

### 1. 导入声明

按以下顺序导入：
1. Vue 核心（ref, reactive, computed 等）
2. Vue Router（useRoute, useRouter）
3. Pinia Store
4. 封装组件（需要手动导入的）
5. 业务组件
6. API 函数
7. Hooks
8. 工具函数
9. 类型定义

### 2. 类型定义

- 接口类型以 `I` 开头（如 `IFormData`）
- 组件 Ref 类型使用 `InstanceType<typeof Component>`
- 类型定义放在一起，不要分散

### 3. 枚举/常量定义

- 枚举使用 PascalCase + Enum 后缀（如 `StatusEnum`）
- 常量使用 UPPER_SNAKE_CASE（如 `STATUS_OPTIONS`）
- 表单校验规则放在常量区域

### 4. Props/Emits 定义

- 使用泛型语法定义 Props 和 Emits
- 使用 `withDefaults` 设置默认值

### 5. 路由/Store 实例

- 路由和 Store 实例放在一起
- 从路由提取的参数使用 computed

### 6. 响应式数据

按以下顺序定义：
1. 基础状态（loading, visible 等）
2. 组件 Ref
3. 表单数据
4. 列表数据
5. 筛选条件
6. 其他状态

### 7. 计算属性

- 权限相关的计算属性放在前面
- 数据相关的计算属性放在中间
- 表单相关的计算属性放在后面

### 8-9. API/Hooks 实例化

- API 和 Hooks 的实例化放在方法定义之前
- 便于在方法中直接使用

### 10. 方法定义

按以下顺序定义方法：
1. 数据加载方法（fetchList, fetchDetail）
2. 搜索/筛选方法（handleSearch, handleReset）
3. 表单操作方法（handleSubmit, handleSaveDraft）
4. 状态切换方法（handleToggleStatus）
5. 导航方法（handleBack, handleEdit）
6. 分页方法（handlePageChange）
7. 弹窗方法（handleOpen, handleClose）
8. 子组件方法调用

### 11. 监听器

- Props 监听放在前面
- 路由参数监听放在中间
- 表单数据监听放在后面

### 12. 生命周期

- 只使用 `onMounted`，避免使用其他生命周期
- 初始化逻辑放在 onMounted 中

### 13. 暴露方法

- 使用 `defineExpose` 暴露给父组件的方法和数据
- 放在最后

## 禁止事项

### ❌ 不要随意插入代码

```typescript
// ❌ 错误：在方法中间定义新变量
const handleSubmit = async () => {
  // ...
};

const newVariable = ref(''); // 不要在这里定义

const handleReset = () => {
  // ...
};
```

### ❌ 不要混合不同类型的代码

```typescript
// ❌ 错误：枚举和响应式数据混在一起
const loading = ref(false);
enum StatusEnum { ... } // 不要在这里定义枚举
const dataList = ref([]);
```

### ❌ 不要在方法中定义类型

```typescript
// ❌ 错误：在方法中定义类型
const handleSubmit = async () => {
  interface SubmitData { ... } // 不要在这里定义
  // ...
};
```

## 检查清单

生成代码后，检查以下项目：

- [ ] 导入声明按顺序排列
- [ ] 类型定义集中在一起
- [ ] 枚举和常量集中在一起
- [ ] 响应式数据按类型分组
- [ ] 方法按功能分类排列
- [ ] 没有在方法中间插入新变量
- [ ] 没有在方法中定义类型
- [ ] 使用了分隔注释标识代码块
