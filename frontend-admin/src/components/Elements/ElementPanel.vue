<template>
  <div class="element-panel card">
    <div class="section-title">元件库</div>
    <div class="element-filter">
      <el-input
        v-model="filterKeyword"
        placeholder="按名称搜索元件"
        clearable
        :prefix-icon="Search"
      />
    </div>
    <div class="element-list">
      <div
        v-for="item in filteredElementTypes"
        :key="item.type"
        class="element-item"
        draggable="true"
        @dragstart="handleDragStart($event, item)"
      >
        <el-icon :size="24"><component :is="item.icon" /></el-icon>
        <span>{{ item.label }}</span>
      </div>
    </div>
    <el-empty
      v-if="filteredElementTypes.length === 0"
      description="没有匹配的元件"
      :image-size="60"
    />

    <div class="section-title">图层列表</div>
    <div class="layer-list">
      <div
        v-for="element in sortedElements"
        :key="element.id"
        class="layer-item"
        :class="{ active: store.selectedElementId === element.id }"
        @click="selectElement(element.id)"
      >
        <el-icon :size="16"><component :is="getElementIcon(element.type)" /></el-icon>
        <span class="layer-name">{{ getElementName(element) }}</span>
        <div class="layer-actions">
          <el-icon @click.stop="toggleVisibility(element)">
            <View v-if="element.visible" />
            <Hide v-else />
          </el-icon>
          <el-icon @click.stop="deleteElement(element.id)"><Delete /></el-icon>
        </div>
      </div>
      <el-empty v-if="store.elements.length === 0" description="暂无元件" :image-size="60" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Search } from '@element-plus/icons-vue'
import { useCanvasStore } from '@/stores/canvas'

const store = useCanvasStore()

const elementTypes = [
  { type: 'text', label: '文本', icon: 'Document', defaultProps: { content: '双击编辑', fontSize: 14, fontFamily: 'Arial', color: '#000000', bold: false, italic: false } },
  { type: 'rect', label: '矩形', icon: 'FullScreen', defaultProps: { fillColor: '#ffffff', strokeColor: '#000000', strokeWidth: 1 } },
  { type: 'circle', label: '圆形', icon: 'CircleCheck', defaultProps: { fillColor: '#ffffff', strokeColor: '#000000', strokeWidth: 1 } },
  { type: 'line', label: '线条', icon: 'Minus', defaultProps: { strokeColor: '#000000', strokeWidth: 2 } },
  { type: 'image', label: '图片', icon: 'Picture', defaultProps: { src: 'https://picsum.photos/100/100' } },
  { type: 'barcode', label: '条码', icon: 'Postcard', defaultProps: { content: '123456789', format: 'CODE128', showText: true } },
  { type: 'qrcode', label: '二维码', icon: 'Grid', defaultProps: { content: 'https://example.com', errorLevel: 'M' } },
  { type: 'table', label: '表格', icon: 'Grid', defaultProps: { rows: 3, cols: 3, borderWidth: 1, borderColor: '#000000', cellFontSize: 12, cellFontFamily: 'Arial', cellFontColor: '#000000', cellTextAlign: 'center', cells: {} } }
]

const filterKeyword = ref('')

// 按最近使用次数从多到少排列；次数相同时保持元件库原有顺序
const sortedElementTypes = computed(() =>
  elementTypes
    .map((item, index) => ({ item, index }))
    .sort(
      (a, b) =>
        store.getUsageCount(b.item.type) - store.getUsageCount(a.item.type) ||
        a.index - b.index
    )
    .map(entry => entry.item)
)

// 名称过滤在排序后的列表上进行
const filteredElementTypes = computed(() => {
  const keyword = filterKeyword.value.trim()
  if (!keyword) return sortedElementTypes.value
  return sortedElementTypes.value.filter(item => item.label.includes(keyword))
})

// 图层顺序与元件库一致：同类型按使用次数排序，次数相同时保持原来的新增在前
const sortedElements = computed(() =>
  [...store.elements]
    .reverse()
    .map((element, index) => ({ element, index }))
    .sort(
      (a, b) =>
        store.getUsageCount(b.element.type) - store.getUsageCount(a.element.type) ||
        a.index - b.index
    )
    .map(entry => entry.element)
)

const handleDragStart = (e, item) => {
  e.dataTransfer.effectAllowed = 'copy'
  e.dataTransfer.setData('application/json', JSON.stringify(item))
}

const selectElement = (id) => store.selectElement(id)
const deleteElement = (id) => store.deleteElement(id)
const toggleVisibility = (el) => store.updateElement(el.id, { visible: !el.visible })

const getElementIcon = (type) => elementTypes.find(e => e.type === type)?.icon || 'Document'
const getElementName = (el) => {
  const names = { text: '文本', rect: '矩形', circle: '圆形', line: '线条', image: '图片', barcode: '条码', qrcode: '二维码', table: '表格' }
  return names[el.type] || el.type
}
</script>

<style lang="scss" scoped>
.element-panel {
  width: 200px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.element-filter {
  padding: 0 12px 8px;
}

.element-list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
  padding: 12px;
}

.element-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 12px 8px;
  background: #f5f7fa;
  border-radius: 6px;
  cursor: grab;
  transition: all 0.2s;
  font-size: 12px;
  color: #606266;
  user-select: none;
  
  &:hover {
    background: #ecf5ff;
    color: #409eff;
  }
  
  &:active {
    cursor: grabbing;
  }
}

.layer-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.layer-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  
  &:hover { background: #f5f7fa; }
  &.active { background: #ecf5ff; color: #409eff; }
  
  .layer-name { flex: 1; font-size: 13px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .layer-actions { display: flex; gap: 8px; opacity: 0; transition: opacity 0.2s; }
  &:hover .layer-actions { opacity: 1; }
}
</style>
