<template>
  <div class="container">
    <div class="main-content">
      <el-card class="mb-4">
        <template #header>
          <div class="card-header">
            <span class="title">钓鱼日志分析</span>
            <el-button type="primary" @click="exportToExcel" size="small">
              <el-icon><Download /></el-icon>
              导出Excel
            </el-button>
          </div>
        </template>
        <el-input
          v-model="logText"
          type="textarea"
          :rows="7"
          placeholder="请输入钓鱼日志..."
          @input="parseLog"
        />
      </el-card>

      <el-card>
        <template #header>
          <div class="card-header">
            <span class="title">捕获记录</span>
            <div class="filters">
              <el-select 
                v-model="filters.name" 
                placeholder="选择鱼种" 
                clearable 
                @change="handleFilter"
                class="filter-select"
              >
                <el-option
                  v-for="item in fishTypes"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
              <el-select 
                v-model="filters.category" 
                placeholder="选择品质" 
                clearable 
                @change="handleFilter"
                class="filter-select"
              >
                <el-option
                  v-for="item in categories"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
              <el-select 
                v-model="filters.bait" 
                placeholder="选择鱼饵" 
                clearable 
                @change="handleFilter"
                class="filter-select"
              >
                <el-option
                  v-for="item in baits"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
            </div>
          </div>
        </template>

        <el-table 
          :data="paginatedData" 
          style="width: 100%" 
          stripe 
          border
          height="440"
          :default-sort="{ prop: 'time', order: 'descending' }"
        >
          <el-table-column prop="time" label="捕获时间" sortable>
            <template #default="scope">
              <el-tag size="small">{{ scope.row.time }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="name" label="鱼名" sortable>
            <template #default="scope">
              <el-tag type="success" size="small">{{ scope.row.name }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="category" label="品质" sortable>
            <template #default="scope">
              <el-tag :type="getCategoryType(scope.row.category)" size="small">
                {{ scope.row.category }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="weight" label="重量" sortable>
            <template #default="scope">
              <span class="weight">{{ scope.row.weight }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="length" label="长度" sortable />
          <el-table-column prop="exp" label="经验" sortable>
            <template #default="scope">
              <el-tag type="warning" size="small">{{ scope.row.exp }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column 
            prop="time_cost" 
            label="耗时" 
            sortable
            :sort-method="(a, b) => a.time_seconds - b.time_seconds"
          >
            <template #default="scope">
              <el-tag type="info" size="small">{{ scope.row.time_cost }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="bait" label="鱼饵">
            <template #default="scope">
              <el-tag type="info" size="small">{{ scope.row.bait }}</el-tag>
            </template>
          </el-table-column>
        </el-table>

        <div class="pagination-container">
          <el-pagination
            v-model:current-page="currentPage"
            v-model:page-size="pageSize"
            :page-sizes="[10, 20, 50, 100]"
            layout="total, sizes, prev, pager, next, jumper"
            :total="filteredData.length"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
        </div>
      </el-card>
    </div>

    <div class="stats-container">
      <el-card class="stats-card">
        <template #header>
          <div class="card-header">
            <span class="title">基础统计</span>
          </div>
        </template>
        <div class="stats-grid">
          <div class="stat-item">
            <el-statistic title="总捕获数" :value="fishData.length">
              <template #suffix>
                <el-icon><List /></el-icon>
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic title="总经验" :value="totalExp">
              <template #suffix>
                <el-icon><Histogram /></el-icon>
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic title="达标数量" :value="qualifiedCount">
              <template #suffix>
                <el-icon><Medal /></el-icon>
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic title="平均耗时" :value="averageTimeValue">
              <template #suffix>
                {{ averageTimeSuffix }}
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <div 
              class="clickable-stat" 
              @click="handleCategoryFilter('星级')"
            >
              <el-statistic title="星级数量" :value="starCount">
                <template #suffix>
                  <el-tag 
                    type="warning" 
                    size="small" 
                    :effect="filters.category === '星级' ? 'dark' : 'light'"
                  >星级</el-tag>
                </template>
              </el-statistic>
            </div>
          </div>
          <div class="stat-item">
            <div 
              class="clickable-stat" 
              @click="handleCategoryFilter('蓝冠')"
            >
              <el-statistic title="蓝冠数量" :value="blueCount">
                <template #suffix>
                  <el-tag 
                    type="primary" 
                    size="small"
                    :effect="filters.category === '蓝冠' ? 'dark' : 'light'"
                  >蓝冠</el-tag>
                </template>
              </el-statistic>
            </div>
          </div>
        </div>
      </el-card>

      <el-card class="stats-card mt-4">
        <template #header>
          <div class="card-header">
            <span class="title">详细统计</span>
          </div>
        </template>
        <div class="stats-grid">
          <div class="stat-item">
            <el-statistic 
              title="最大重量" 
              :value="maxWeight.weight"
              :formatter="(value) => value.toString()"
            >
              <template #suffix>
                {{ maxWeight.unit }} {{ maxWeight.name }}
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic 
              title="最高经验" 
              :value="maxExp.exp"
            >
              <template #suffix>
                {{ maxExp.name }}
              </template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic 
              title="达标率" 
              :value="qualifiedRate"
              :precision="1"
            >
              <template #suffix>%</template>
            </el-statistic>
          </div>
          <div class="stat-item">
            <el-statistic 
              title="平均经验" 
              :value="averageExp"
              :precision="1"
            />
          </div>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { List, Histogram, Medal, Timer, Download } from '@element-plus/icons-vue'
import * as XLSX from 'xlsx'

const logText = ref('')
const fishData = ref([])
const currentPage = ref(1)
const pageSize = ref(10)
const filters = ref({
  name: '',
  category: '',
  bait: ''
})

// 计算属性：获取所有鱼种、品质和鱼饵类型
const fishTypes = computed(() => [...new Set(fishData.value.map(item => item.name))])
const categories = computed(() => [...new Set(fishData.value.map(item => item.category))])
const baits = computed(() => [...new Set(fishData.value.map(item => item.bait))])
//测试
// 计算属性：统计信息
const totalExp = computed(() => 
  fishData.value.reduce((sum, item) => sum + parseInt(item.exp), 0)
)

const qualifiedCount = computed(() => 
  fishData.value.filter(item => item.category === '达标').length
)

const qualifiedRate = computed(() => 
  fishData.value.length ? (qualifiedCount.value / fishData.value.length * 100) : 0
)

const averageExp = computed(() => 
  fishData.value.length ? totalExp.value / fishData.value.length : 0
)

const maxWeight = computed(() => {
  if (!fishData.value.length) return { weight: 0, name: '-', unit: '克' }
  return fishData.value.reduce((max, current) => {
    const currentWeight = parseFloat(current.weight.replace(/[公克]/, ''))
    const maxWeight = parseFloat(max.weight.toString().replace(/[公克]/, ''))
    return currentWeight > maxWeight ? {
      weight: currentWeight,
      name: current.name,
      unit: current.weight.includes('公斤') ? '公斤' : '克'
    } : max
  }, { weight: 0, name: '-', unit: '克' })
})

const maxExp = computed(() => {
  if (!fishData.value.length) return { exp: 0, name: '-' }
  return fishData.value.reduce((max, current) => 
    parseInt(current.exp) > parseInt(max.exp) ? current : max
  )
})

const averageTimeValue = computed(() => {
  const times = fishData.value.map(item => item.time_seconds)
  if (!times.length) return 0
  return Math.floor(times.reduce((a, b) => a + b) / times.length)
})

const averageTimeSuffix = computed(() => {
  const seconds = averageTimeValue.value
  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const remainingSeconds = seconds % 60
  
  let result = ''
  if (hours > 0) result += `${hours}时`
  if (minutes > 0) result += `${minutes}分`
  if (remainingSeconds > 0 || (!hours && !minutes)) result += `${remainingSeconds}秒`
  return result
})

const starCount = computed(() => 
  fishData.value.filter(item => item.category === '星级').length
)

const blueCount = computed(() => 
  fishData.value.filter(item => item.category === '蓝冠').length
)

// 计算属性：过滤后的数据
const filteredData = computed(() => {
  return fishData.value.filter(item => {
    const nameMatch = !filters.value.name || item.name === filters.value.name
    const categoryMatch = !filters.value.category || item.category === filters.value.category
    const baitMatch = !filters.value.bait || item.bait === filters.value.bait
    return nameMatch && categoryMatch && baitMatch
  })
})

// 计算属性：分页后的数据
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return filteredData.value.slice(start, end)
})

// 方法：解析日志
const parseLog = () => {
  const lines = logText.value.split('\n')
  const parsedData = []
  
  lines.forEach(line => {
    if (line.includes('捕获：')) {
      const timeMatch = line.match(/(\d{2}:\d{2}:\d{2})/)
      const fishNameMatch = line.match(/【(.+?)】/)
      const categoryMatch = line.match(/【(.+?)】.*?【(.+?)】/)
      const weightMatch = line.match(/(\d+(?:\.\d+)?(?:克|公斤))/)
      const lengthMatch = line.match(/(\d+(?:\.\d+)?)\s*(?:厘米|分米|米)/)
      const lengthUnitMatch = line.match(/(?:厘米|分米|米)/)
      const expMatch = line.match(/总经验:(\d+)/)
      const timeSpentMatch = line.match(/耗时:(?:(\d+)时)?(?:(\d+)分)?(?:(\d+)秒)?/)
      const baitMatch = line.match(/鱼饵:(.+)$/)

      if (timeMatch && fishNameMatch && categoryMatch && weightMatch && lengthMatch && expMatch && timeSpentMatch && baitMatch) {
        // 处理长度单位转换为厘米
        let lengthValue = parseFloat(lengthMatch[1])
        const lengthUnit = lengthUnitMatch ? lengthUnitMatch[0] : '厘米'
        switch (lengthUnit) {
          case '分米':
            lengthValue *= 10
            break
          case '米':
            lengthValue *= 100
            break
        }

        // 处理耗时转换为秒
        const hours = timeSpentMatch[1] ? parseInt(timeSpentMatch[1]) : 0
        const minutes = timeSpentMatch[2] ? parseInt(timeSpentMatch[2]) : 0
        const seconds = timeSpentMatch[3] ? parseInt(timeSpentMatch[3]) : 0
        const totalSeconds = hours * 3600 + minutes * 60 + seconds

        // 格式化耗时显示
        let formattedTime = ''
        if (hours > 0) formattedTime += `${hours}时`
        if (minutes > 0) formattedTime += `${minutes}分`
        if (seconds > 0 || (!hours && !minutes)) formattedTime += `${seconds}秒`

        parsedData.push({
          time: timeMatch[1],
          name: fishNameMatch[1],
          category: categoryMatch[2],
          weight: weightMatch[1],
          length: `${lengthValue.toFixed(1)}厘米`,
          exp: expMatch[1],
          time_cost: formattedTime,
          time_seconds: totalSeconds,
          bait: baitMatch[1]
        })
      }
    }
  })

  fishData.value = parsedData
  currentPage.value = 1
}

// 方法：处理分页
const handleSizeChange = (val) => {
  pageSize.value = val
  currentPage.value = 1
}

const handleCurrentChange = (val) => {
  currentPage.value = val
}

// 方法：处理筛选
const handleFilter = () => {
  currentPage.value = 1
}

// 方法：获取品质标签类型
const getCategoryType = (category) => {
  switch (category) {
    case '蓝冠':
      return 'primary'
    case '星级':
      return 'warning'
    case '达标':
      return 'success'
    case '普通':
      return 'info'
    default:
      return ''
  }
}

// 方法：导出Excel
const exportToExcel = () => {
  const data = fishData.value.map(item => ({
    '捕获时间': item.time,
    '鱼名': item.name,
    '品质': item.category,
    '重量': item.weight,
    '长度': item.length,
    '经验': item.exp,
    '耗时': item.time_cost,
    '鱼饵': item.bait
  }))

  const ws = XLSX.utils.json_to_sheet(data)
  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, ws, '钓鱼记录')
  XLSX.writeFile(wb, '钓鱼记录.xlsx')
}

// 方法：处理类别筛选
const handleCategoryFilter = (category) => {
  if (filters.value.category === category) {
    // 如果当前已经是这个筛选，则清除筛选
    filters.value.category = ''
  } else {
    // 否则设置新的筛选
    filters.value.category = category
  }
  currentPage.value = 1 // 重置页码
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.container {
  padding: 20px;
  display: flex;
  gap: 20px;
  background-color: #f5f7fa;
  height: 100%;
}

.main-content {
  flex: 1;
  max-width: calc(100% - 360px);
  margin-right: 320px;
}

.stats-container {
  width: 320px;
  position: fixed;
  top: 20px;
  right: 20px;
  height: calc(100vh - 40px);
  overflow-y: auto;
  align-self: flex-start;
}

.stats-card {
  background: #fff;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.stats-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0,0,0,0.1);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.stat-item {
  padding: 12px;
  border-radius: 6px;
  background-color: #f8f9fa;
  transition: all 0.3s ease;
}

.stat-item:hover {
  background-color: #f0f2f5;
  transform: translateY(-2px);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.filters {
  display: flex;
  gap: 12px;
  align-items: center;
}

.filter-select {
  width: 160px;
}

.mb-4 {
  margin-bottom: 20px;
}

.mt-4 {
  margin-top: 20px;
}

.pagination-container {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

.weight {
  color: #67c23a;
  font-weight: bold;
}

:deep(.el-card) {
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1);
}

:deep(.el-table) {
  border-radius: 4px;
  overflow: hidden;
}

:deep(.el-input__wrapper) {
  box-shadow: 0 0 0 1px #dcdfe6 inset;
}

:deep(.el-card__header) {
  background-color: #f5f7fa;
  border-bottom: 1px solid #e4e7ed;
}

:deep(.el-select-dropdown__item) {
  padding: 0 12px;
  min-width: 160px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

:deep(.el-select) {
  --el-select-input-focus-border-color: var(--el-color-primary);
}

:deep(.el-select .el-input__wrapper) {
  box-shadow: 0 0 0 1px #dcdfe6 inset;
}

:deep(.el-select .el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 1px var(--el-color-primary) inset;
}

:deep(.el-statistic__content) {
  display: flex;
  align-items: center;
  gap: 8px;
}

:deep(.el-statistic__suffix) {
  margin-left: 4px;
}

.clickable-stat {
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 8px;
  border-radius: 4px;
}

.clickable-stat:hover {
  background-color: #f0f2f5;
}

@media (max-width: 1200px) {
  .container {
    flex-direction: column;
  }
  
  .main-content {
    max-width: 100%;
    margin-right: 0;
  }
  
  .stats-container {
    width: 100%;
    position: static;
    height: auto;
    overflow-y: visible;
  }
  
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
