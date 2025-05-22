<template>
  <div class="model-debugger">
    <el-card class="model-selection">
      <template #header>
        <span>模型选择</span>
      </template>
      <el-select v-model="selectedModel" placeholder="请选择模型">
        <el-option
          v-for="model in modelOptions"
          :key="model.value"
          :label="model.label"
          :value="model.value"
        >
          <span>{{ model.label }}</span>
          <el-tooltip placement="right" effect="light">
            <template #content>
              <p>{{ model.description }}</p>
            </template>
            <el-icon style="margin-left: 8px"><QuestionFilled /></el-icon>
          </el-tooltip>
        </el-option>
      </el-select>
    </el-card>

    <el-card class="parameter-tuning">
      <template #header>
        <span>参数调整</span>
      </template>
      <el-form label-width="120px">
        <el-form-item label="学习率">
          <el-slider v-model="learningRate" :min="0.0001" :max="0.1" :step="0.0001" show-input />
        </el-form-item>
        <el-form-item label="批量大小">
          <el-input-number v-model="batchSize" :min="1" :max="256" />
        </el-form-item>
        <el-form-item label="训练轮次">
          <el-input-number v-model="epochs" :min="1" :max="1000" />
        </el-form-item>
      </el-form>
    </el-card>

    <el-card class="training-control">
      <template #header>
        <span>训练控制</span>
      </template>
      <el-button type="primary" @click="startTraining" class="training-button">开始训练</el-button>
      <el-button @click="stopTraining" class="training-button">停止训练</el-button>
      <el-button @click="resetParameters" class="training-button">重置参数</el-button>
    </el-card>

    <el-card class="training-progress">
      <template #header>
        <span>训练进度</span>
      </template>
      <el-progress :percentage="progress" :status="progressStatus" />
      <el-divider />
      <el-tag type="info">当前损失: {{ currentLoss }}</el-tag>
      <el-tag type="success">当前准确率: {{ currentAccuracy }}</el-tag>
    </el-card>

    <el-card class="result-visualization">
      <template #header>
        <span>结果可视化</span>
      </template>
      <div class="chart-container">
        <div class="loss-chart" ref="lossChart"></div>
        <div class="accuracy-chart" ref="accuracyChart"></div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import * as echarts from 'echarts'

const selectedModel = ref('cnn')
const modelOptions = [
  { 
    value: 'cnn', 
    label: '卷积神经网络',
    description: '适合处理图像数据，通过卷积核提取局部特征'
  },
  { 
    value: 'rnn', 
    label: '循环神经网络',
    description: '适合处理序列数据，具有记忆功能'
  },
  { 
    value: 'transformer', 
    label: 'Transformer',
    description: '基于注意力机制，适合处理长序列数据'
  },
]

const learningRate = ref(0.001)
const batchSize = ref(32)
const epochs = ref(10)

const progress = ref(0)
const progressStatus = ref('')
const currentLoss = ref(0)
const currentAccuracy = ref(0)
const trainingInterval = ref(null)

const startTraining = () => {
  progressStatus.value = 'success'
  
  // 检查是否已选择模型
  if (!selectedModel.value) {
    ElMessage.error('请先选择模型')
    return
  }
  
  // 重置进度和指标
  progress.value = 0
  currentLoss.value = 0
  currentAccuracy.value = 0
  
  // 初始化训练数据
  const lossData = []
  const accuracyData = []
  const totalSteps = 20
  
  // 模拟训练过程
  const interval = setInterval(() => {
    if (progress.value >= 100) {
      clearInterval(interval)
      ElMessage.success('训练完成')
      
      // 绘制训练曲线
      drawChart(lossData, accuracyData)
      return
    }
    
    // 模拟更真实的训练数据
    const step = progress.value / (100 / totalSteps)
    const loss = 10 * Math.exp(-0.2 * step) + Math.random() * 0.5
    const accuracy = 100 * (1 - Math.exp(-0.3 * step)) + Math.random() * 2
    
    progress.value += 100 / totalSteps
    currentLoss.value = loss.toFixed(4)
    currentAccuracy.value = accuracy.toFixed(2)
    
    // 记录数据点
    lossData.push(currentLoss.value)
    accuracyData.push(currentAccuracy.value)
  }, 1000)
  
  // 保存训练任务ID以便停止
  trainingInterval.value = interval
}

const stopTraining = () => {
  if (trainingInterval.value) {
    clearInterval(trainingInterval.value)
    trainingInterval.value = null
    progressStatus.value = 'exception'
    ElMessage.warning('训练已停止')
  } else {
    ElMessage.warning('没有正在运行的训练任务')
  }
}

const resetParameters = () => {
  learningRate.value = 0.001
  batchSize.value = 32
  epochs.value = 10
  progress.value = 0
  progressStatus.value = ''
  currentLoss.value = 0
  currentAccuracy.value = 0
}

const drawChart = (lossData = [], accuracyData = []) => {
  // 确保DOM元素已加载
  if (!lossChartRef.value || !accuracyChartRef.value) {
    console.error('图表容器未找到')
    return
  }
  
  // 如果没有数据，使用示例数据
  if (lossData.length === 0) {
    lossData = Array.from({length: 10}, (_, i) => (10 - i) + Math.random() * 0.5)
    accuracyData = Array.from({length: 10}, (_, i) => i * 10 + Math.random() * 2)
  }
  
  // 初始化ECharts实例
  const lossChart = echarts.init(lossChartRef.value)
  const accuracyChart = echarts.init(accuracyChartRef.value)
  
  // 损失曲线配置
  lossChart.setOption({
    title: { text: '损失曲线', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: { 
      type: 'category',
      data: Array.from({length: lossData.length}, (_, i) => `步骤${i+1}`)
    },
    yAxis: { type: 'value', name: '损失值' },
    series: [{
      name: '损失值',
      data: lossData.map(Number),
      type: 'line',
      smooth: true,
      symbol: 'circle',
      symbolSize: 8,
      itemStyle: {
        color: '#ff4d4f'
      },
      animationDuration: 2000,
      lineStyle: { color: '#ff4d4f' },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: 'rgba(255, 77, 79, 0.3)' },
          { offset: 1, color: 'rgba(255, 77, 79, 0.1)' }
        ])
      }
    }]
  })
  
  // 准确率曲线配置
  accuracyChart.setOption({
    title: { text: '准确率曲线', left: 'center' },
    tooltip: { trigger: 'axis' },
    xAxis: { 
      type: 'category',
      data: Array.from({length: accuracyData.length}, (_, i) => `步骤${i+1}`)
    },
    yAxis: { type: 'value', name: '准确率(%)', max: 100 },
    series: [{
      name: '准确率',
      data: accuracyData.map(Number),
      type: 'line',
      smooth: true,
      symbol: 'circle',
      symbolSize: 8,
      itemStyle: {
        color: '#52c41a'
      },
      animationDuration: 2000,
      lineStyle: { color: '#52c41a' },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: 'rgba(82, 196, 26, 0.3)' },
          { offset: 1, color: 'rgba(82, 196, 26, 0.1)' }
        ])
      }
    }]
  })
  
  // 响应式调整图表大小
  window.addEventListener('resize', () => {
    lossChart.resize()
    accuracyChart.resize()
  })
}

// 声明图表ref
const lossChartRef = ref(null)
const accuracyChartRef = ref(null)

// 组件挂载时初始化示例图表
onMounted(() => {
  // 添加延迟确保DOM完全加载
  setTimeout(() => {
    drawChart()
  }, 100)
})
</script>

<style scoped>
.model-debugger {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.el-card {
  margin-bottom: 20px;
}

.chart-container {
  display: flex;
  justify-content: space-between;
}

.loss-chart,
.accuracy-chart {
  width: 48%;
  height: 300px;
  background-color: #f5f7fa;
}

.training-button {
  margin: 0 10px;
  padding: 10px 20px;
  border-radius: 4px;
  font-weight: bold;
  transition: all 0.3s;
}

.training-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}
</style>