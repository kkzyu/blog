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
