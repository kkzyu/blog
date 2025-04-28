---
title: pinia重构
date: 2025-04-28 22:05:46
tags: web前端
---
### Pinia 重构过程记录
#### 重构步骤

1. 安装必要依赖
```bash
npm install pinia
```

2. 创建 Pinia Store
```js
// stores/cvStore.js
import { defineStore } from 'pinia'
import { message } from 'ant-design-vue'
import cvDataStructure from '@/assets/data/cv-info.json'
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'

export const useCvStore = defineStore('cv', {
  state: () => ({
    currentLang: 'cn',
    activeSections: [],
    commentContent: '',
    isDownloadingPdf: false,
    defaultAvatar: '/images/kkz.jpg',
    manualToggleStates: {}
  }),
  
  actions: {
    // 业务逻辑方法...
  },
  
  getters: {
    // 计算属性...
  }
})
```

3. 修改组件结构
```vue
<script setup>
import { storeToRefs } from 'pinia'
import { useCvStore } from '@/stores/cvStore'

const store = useCvStore()
const { 
  currentLang,
  activeSections,
  isDownloadingPdf,
  cvData
} = storeToRefs(store)

// 方法代理
const handleLangChange = ({ key }) => store.handleLangChange(key)
const downloadPDF = () => store.downloadPDF(cvContainerRef.value)
</script>
```

4. 状态管理迁移
| 原位置 | 新位置 | 示例 | |--------|--------|------| | data() | store state | currentLang | | computed | store getters | cvData | | methods | store actions | downloadPDF |

5. 遇到的问题及解决方案
问题1: 图标不显示
原因: <script setup> 需要显式导入组件
解决:
```vue
import { GlobalOutlined } from '@ant-design/icons-vue'
```
问题2: DOM 元素访问
原因: Store 无法直接访问组件 ref
解决: 通过参数传递 DOM 元素
```js
// 组件中
store.downloadPDF(cvContainerRef.value)

// Store 中
downloadPDF(cvElement) {
  // 使用 cvElement
}
```
问题3: 响应式丢失
原因: 直接解构 store 会导致响应式丢失
解决: 使用 storeToRefs
```js
const { currentLang } = storeToRefs(store)
```