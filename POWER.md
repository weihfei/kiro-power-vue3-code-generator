---
name: "vue3-code-generator"
displayName: "Vue 3 规范代码生成器"
description: "将需求转换为功能点，确认后按照项目规范生成 Vue 3 + TypeScript 代码，确保代码块分类有序"
keywords: ["vue3", "typescript", "code-generator", "requirements", "功能点", "代码规范"]
author: "weihfa"
---

# Vue 3 规范代码生成器

## Overview

这个 Power 帮助你将需求转换为规范的 Vue 3 + TypeScript 代码。通过交互式问答，确保：

1. **需求梳理** - 将模糊需求转换为清晰的功能点
2. **用户确认** - 功能点确认后再开始编码
3. **规范生成** - 按照项目规范生成代码
4. **有序组织** - 代码块分类有序，不随意插入

## Available Steering Files

- **requirement-to-features** - 需求转功能点的详细流程
- **code-organization** - 代码块分类和组织规范
- **component-standards** - 组件使用规范（封装组件 vs Element UI）

## 核心工作流程

```
用户输入需求
    ↓
分析需求，提取功能点
    ↓
向用户展示功能点清单
    ↓
用户确认/修改功能点
    ↓
按规范生成代码
    ↓
代码审查和优化
```

## 第一阶段：需求分析

### 需求输入格式

用户可以用任何方式描述需求：
- 自然语言描述
- 产品文档
- 设计稿说明
- 接口文档

### 功能点提取规则

将需求拆解为以下类型的功能点：

| 功能类型 | 示例 |
|---------|------|
| 页面展示 | 列表展示、详情展示、卡片布局 |
| 表单交互 | 搜索筛选、表单提交、数据校验 |
| 数据操作 | 新增、编辑、删除、批量操作 |
| 状态管理 | 加载状态、空状态、错误状态 |
| 业务逻辑 | 权限控制、流程审批、数据计算 |

### 功能点展示模板

```markdown
## 功能点清单

根据你的需求，我梳理出以下功能点：

### 📋 页面展示
- [ ] 1.1 列表页面布局
- [ ] 1.2 筛选区域（名称、状态、分类）
- [ ] 1.3 卡片列表展示
- [ ] 1.4 分页功能

### 🔄 交互功能
- [ ] 2.1 搜索筛选
- [ ] 2.2 重置筛选条件
- [ ] 2.3 创建新记录
- [ ] 2.4 编辑记录
- [ ] 2.5 启用/禁用切换

### 📡 数据处理
- [ ] 3.1 列表数据加载
- [ ] 3.2 状态更新接口
- [ ] 3.3 错误处理

### 🎨 状态展示
- [ ] 4.1 加载中状态
- [ ] 4.2 空数据状态
- [ ] 4.3 操作成功/失败提示

---

请确认以上功能点是否符合预期？
1. ✅ 确认，开始生成代码
2. ➕ 需要添加功能点
3. ➖ 需要删除某些功能点
4. ✏️ 需要修改功能点描述
```

## 第二阶段：代码生成

### 代码块分类顺序

生成的代码必须按以下顺序组织：

```vue
<template>
  <!-- 模板内容 -->
</template>

<script setup lang="ts">
// ==================== 1. 导入声明 ====================
// 1.1 Vue 核心
import { ref, reactive, computed, onMounted, watch } from 'vue';
// 1.2 路由相关
import { useRoute, useRouter } from 'vue-router';
// 1.3 组件导入（需要手动导入的）
import ESteps from '@/components/ELESUI/ESteps.vue';
// 1.4 API 导入
import { xxxRequest } from './api/index';
// 1.5 工具函数
import { formatDate, formatPrice } from '@/utils/util';
// 1.6 类型导入
import type { FormInstance } from 'element-plus';

// ==================== 2. 类型定义 ====================
interface FormData {
  name: string;
  status: number;
}

interface ListItem {
  id: number;
  name: string;
  enabled: boolean;
}

// ==================== 3. 枚举/常量定义 ====================
enum StatusEnum {
  DISABLED = 0,
  ENABLED = 1
}

const STATUS_OPTIONS = [
  { label: '全部', value: '' },
  { label: '启用', value: StatusEnum.ENABLED },
  { label: '禁用', value: StatusEnum.DISABLED }
];

// ==================== 4. Props/Emits 定义 ====================
interface Props {
  modelValue?: boolean;
  data?: ListItem[];
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: false,
  data: () => []
});

const emit = defineEmits<{
  'update:modelValue': [value: boolean];
  'change': [item: ListItem];
}>();

// ==================== 5. 响应式数据 ====================
// 5.1 基础状态
const loading = ref(false);
const visible = ref(false);

// 5.2 表单数据
const formRef = ref<FormInstance>();
const formData = reactive<FormData>({
  name: '',
  status: 0
});

// 5.3 列表数据
const dataList = ref<ListItem[]>([]);
const total = ref(0);

// ==================== 6. 计算属性 ====================
const isReadonly = computed(() => {
  return props.data.length === 0;
});

const filteredList = computed(() => {
  return dataList.value.filter(item => item.enabled);
});

// ==================== 7. API 调用 ====================
const { getListData, updateStatus, submitForm } = xxxRequest();

// ==================== 8. 方法定义 ====================
// 8.1 数据加载
const fetchData = async () => {
  loading.value = true;
  try {
    const res = await getListData({ page: 1, size: 10 });
    dataList.value = res?.list ?? [];
    total.value = res?.total ?? 0;
  } catch (error) {
    console.error('获取数据失败:', error);
    $Modal.msgError('获取数据失败');
  } finally {
    loading.value = false;
  }
};

// 8.2 搜索/筛选
const handleSearch = () => {
  fetchData();
};

const handleReset = () => {
  Object.assign(formData, { name: '', status: 0 });
  fetchData();
};

// 8.3 表单操作
const handleSubmit = async () => {
  if (!formRef.value) return;
  
  try {
    await formRef.value.validate();
    await submitForm(formData);
    $Modal.msgSuccess('提交成功');
    visible.value = false;
    fetchData();
  } catch (error) {
    console.error('提交失败:', error);
  }
};

// 8.4 状态切换
const handleToggle = async (item: ListItem, value: boolean) => {
  try {
    await $Modal.confirm(`确定要${value ? '启用' : '禁用'}吗？`);
    await updateStatus(item.id, value);
    $Modal.msgSuccess('操作成功');
    item.enabled = value;
  } catch {
    // 用户取消，不处理
  }
};

// ==================== 9. 监听器 ====================
watch(
  () => props.modelValue,
  (val) => {
    visible.value = val;
  }
);

// ==================== 10. 生命周期 ====================
onMounted(() => {
  fetchData();
});

// ==================== 11. 暴露方法 ====================
defineExpose({
  fetchData,
  formData
});
</script>

<style lang="scss" scoped>
/* 样式定义 */
</style>
```

### 组件使用规范

**必须使用封装组件（自动导入）：**
- `<e-button>` 替代 `<el-button>`
- `<e-input>` 替代 `<el-input>`
- `<e-select>` 替代 `<el-select>`
- `<e-table>` 替代 `<el-table>`
- `<e-dialog>` 替代 `<el-dialog>`
- `<e-pagination>` 替代 `<el-pagination>`

**需要手动导入的封装组件：**
```typescript
import ESteps from '@/components/ELESUI/ESteps.vue';
import ESwitch from '@/components/ELESUI/ESwitch.vue';
import ETabs from '@/components/ELESUI/ETabs.vue';
```

**可以直接使用 Element UI 的组件：**
- `<el-form>` / `<el-form-item>`
- `<el-card>`
- `<el-date-picker>`（单日期）
- `<el-collapse>`

### TypeScript 规范

**禁止使用 `any`：**
```typescript
// ❌ 错误
const data: any = ref(null);

// ✅ 正确
interface DataItem {
  id: number;
  name: string;
}
const data = ref<DataItem | null>(null);
```

**Props 类型定义：**
```typescript
// ✅ 正确
interface Props {
  title: string;
  count?: number;
}
const props = withDefaults(defineProps<Props>(), {
  count: 0
});
```

### 错误处理规范

```typescript
// ✅ 正确：完整的错误处理
const fetchData = async () => {
  try {
    loading.value = true;
    const res = await getListData(params);
    
    if (!res) {
      $Modal.msgWarning('未获取到数据');
      return;
    }
    
    dataList.value = res.list;
  } catch (error) {
    console.error('获取数据失败:', error);
    $Modal.msgError('获取数据失败，请稍后重试');
  } finally {
    loading.value = false;
  }
};
```

## 第三阶段：代码审查

生成代码后，自动进行以下检查：

### 检查清单

- [ ] TypeScript 类型安全（无 `any`）
- [ ] 使用封装组件（`e-xxx` 而非 `el-xxx`）
- [ ] 完整的错误处理
- [ ] 代码块分类有序
- [ ] 命名规范（Hooks 以 `use` 开头）
- [ ] 注释完整（复杂逻辑必须注释）

### 问题反馈格式

```markdown
## 代码审查结果

### ✅ 通过项
- TypeScript 类型定义完整
- 使用了封装组件
- 错误处理完善

### ⚠️ 建议优化
- 建议将复杂计算提取为 computed
- 建议添加防抖处理

### 🔴 需要修复
- 第 XX 行使用了 `any` 类型
- 第 XX 行使用了 `el-button` 应改为 `e-button`
```

## 使用示例

### 示例 1：简单列表页

**用户输入：**
> 我需要一个智能体列表页面，可以搜索、筛选、创建、编辑、启用禁用

**功能点输出：**
```markdown
## 功能点清单

### 📋 页面展示
- [x] 1.1 列表页面布局（标题 + 筛选区 + 卡片列表）
- [x] 1.2 筛选区域（名称输入、状态下拉、分类下拉）
- [x] 1.3 卡片列表展示（标题、描述、标签、开关、头像）
- [x] 1.4 创建按钮

### 🔄 交互功能
- [x] 2.1 搜索筛选
- [x] 2.2 重置筛选条件
- [x] 2.3 创建新智能体（跳转/弹窗）
- [x] 2.4 编辑智能体
- [x] 2.5 启用/禁用切换（带确认）

### 📡 数据处理
- [x] 3.1 列表数据加载
- [x] 3.2 状态更新接口
- [x] 3.3 错误处理和提示

确认后开始生成代码？
```

### 示例 2：复杂表单页

**用户输入：**
> 营销活动编辑页面，包含基本信息、活动对象、活动内容三个模块，支持草稿保存和提交

**功能点输出：**
```markdown
## 功能点清单

### 📋 页面结构
- [x] 1.1 Tab 切换（基本信息/活动对象/活动内容）
- [x] 1.2 锚点导航
- [x] 1.3 底部操作栏（返回/保存草稿/提交）

### 📝 基本信息模块
- [x] 2.1 活动名称（必填、重名校验）
- [x] 2.2 活动类型选择
- [x] 2.3 活动时间范围
- [x] 2.4 活动范围选择

### 👥 活动对象模块
- [x] 3.1 客户选择（省份/标签筛选）
- [x] 3.2 客户清单生成
- [x] 3.3 客户列表展示

### 📦 活动内容模块
- [x] 4.1 物料添加
- [x] 4.2 价格配置
- [x] 4.3 库存校验

### 💾 数据操作
- [x] 5.1 草稿保存
- [x] 5.2 数据提交
- [x] 5.3 表单校验
- [x] 5.4 详情数据加载（编辑模式）

确认后开始生成代码？
```

## Onboarding

### 使用方式

1. 描述你的需求（可以是模糊的）
2. 查看生成的功能点清单
3. 确认或修改功能点
4. 获取规范的代码

### 最佳实践

- **需求描述越详细越好** - 包含业务场景、交互细节
- **提供参考** - 如果有类似页面，可以提供参考
- **分步确认** - 复杂页面可以分模块确认
- **及时反馈** - 功能点不对立即修改

## Troubleshooting

### 功能点不完整

**问题：** 生成的功能点缺少某些需求
**解决：** 告诉我缺少的功能，我会补充

### 代码不符合规范

**问题：** 生成的代码使用了 Element UI 原生组件
**解决：** 我会自动检查并修正，如有遗漏请指出

### 代码块顺序混乱

**问题：** 变量或方法位置不对
**解决：** 告诉我具体问题，我会重新组织

---

**注意：** 此 Power 不包含 MCP 服务器，纯粹通过 steering 规则指导代码生成。
