<template>
  <div class="dashboard-container">
    <!-- 页面标题 -->
    <div class="page-header">
      <h1 class="page-title">📊 数据仪表板</h1>
      <p class="page-subtitle">实时监控图书馆运营数据和统计信息</p>
    </div>

    <!-- 时间显示 -->
    <div id="myTimer" class="timer"></div>

    <!-- 统计卡片 -->
    <el-row :gutter="20" class="cards-row">
      <el-col :xs="12" :sm="12" :md="6" v-for="item in cards" :key="item.title">
        <el-card class="stat-card fade-in" shadow="hover">
          <template #header>
            <div class="card-header">
              <span class="card-title">{{ item.title }}</span>
            </div>
          </template>
          <div class="card-content">
            <div class="card-icon">
              <svg class="icon" aria-hidden="true">
                <use :xlink:href="item.icon"></use>
              </svg>
            </div>
            <span class="card-data">{{ item.data }}</span>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- ECharts 图表 -->
    <el-card class="chart-card fade-in" shadow="hover">
      <template #header>
        <div class="card-header">
          <span class="card-title">📈 运营统计图表</span>
        </div>
      </template>
      <div id="main" class="chart-container"></div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import { ElMessage } from "element-plus";
import request from "../utils/request";
import router from "@/router";

export default {
  data() {
    return {
      cards: [
        { title: '已借阅', data: 100, icon: '#iconlend-record-pro' },
        { title: '总访问', data: 100, icon: '#iconvisit' },
        { title: '图书数', data: 100, icon: '#iconbook-pro' },
        { title: '用户数', data: 1000, icon: '#iconpopulation' }
      ],
      data: {}
    }
  },
  created() {
    let userJson = sessionStorage.getItem("user")
    if (!userJson) {
      router.push("/login")
    }
  },
  mounted() {
    this.circleTimer()

    request.get("/dashboard").then(res => {
      if (res.code == 0) {
        this.cards[0].data = res.data.lendRecordCount
        this.cards[1].data = res.data.visitCount
        this.cards[2].data = res.data.bookCount
        this.cards[3].data = res.data.userCount
      } else {
        ElMessage.error(res.msg)
      }

      // 基于准备好的dom，初始化echarts实例
      var myChart = echarts.init(document.getElementById('main'))

      // 绘制图表
      myChart.setOption({
        title: {
          text: '统计数据',
          textStyle: {
            fontSize: 16,
            fontWeight: '600',
            color: '#2c3e50'
          }
        },
        tooltip: {
          trigger: 'axis',
          backgroundColor: 'rgba(255, 255, 255, 0.95)',
          borderColor: '#e2e8f0',
          borderWidth: 1,
          textStyle: {
            color: '#2c3e50'
          },
          shadowBlur: 10,
          shadowColor: 'rgba(0, 0, 0, 0.1)'
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'category',
          data: this.cards.map(item => item.title),
          axisTick: {
            alignWithLabel: true
          },
          axisLine: {
            lineStyle: {
              color: '#e2e8f0'
            }
          },
          axisLabel: {
            color: '#64748b'
          }
        },
        yAxis: {
          type: 'value',
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          },
          splitLine: {
            lineStyle: {
              color: '#f1f5f9'
            }
          },
          axisLabel: {
            color: '#64748b'
          }
        },
        series: [
          {
            type: 'bar',
            label: {
              show: true,
              position: 'top',
              textStyle: {
                color: '#64748b',
                fontSize: 12
              }
            },
            barWidth: '40%',
            itemStyle: {
              borderRadius: [4, 4, 0, 0],
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                { offset: 0, color: '#667eea' },
                { offset: 1, color: '#764ba2' }
              ])
            },
            data: [
              this.cards[0].data,
              this.cards[1].data,
              this.cards[2].data,
              this.cards[3].data
            ]
          }
        ]
      })

      // 响应式调整
      window.addEventListener('resize', () => {
        myChart.resize()
      })
    })
  },
  methods: {
    circleTimer() {
      this.getTimer()
      setInterval(() => {
        this.getTimer()
      }, 1000)
    },
    getTimer() {
      var d = new Date()
      var t = d.toLocaleString()
      document.getElementById('myTimer').innerHTML = t
    }
  }
}
</script>

<style scoped>
/* 仪表板容器 */
.dashboard-container {
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

/* 时间显示 */
.timer {
  font-weight: 550;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 20px;
  font-size: 14px;
  text-align: center;
  font-style: italic;
}

/* 卡片行 */
.cards-row {
  margin-bottom: 20px;
}

/* 统计卡片 */
.stat-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  transition: all 0.3s ease;
  margin-bottom: 20px;
  border: none;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.15);
}

/* 卡片头部 */
.card-header {
  font-size: 14px;
  font-weight: 500;
  color: #64748b;
  text-align: center;
  padding: 16px;
}

/* 卡片内容 */
.card-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

/* 卡片图标 */
.card-icon {
  margin-bottom: 16px;
}

.card-icon .icon {
  width: 60px;
  height: 60px;
  fill: #667eea;
  transition: transform 0.3s ease;
}

.stat-card:hover .card-icon .icon {
  transform: scale(1.1) rotate(5deg);
}

/* 卡片数据 */
.card-data {
  font-size: 32px;
  font-weight: 700;
  color: #2d3748;
  background: linear-gradient(45deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 图表卡片 */
.chart-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: none;
  margin-top: 20px;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

.chart-card:hover {
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.15);
}

/* 图表容器 */
.chart-container {
  width: 100%;
  height: 400px;
  min-height: 300px;
}

/* 动画效果 */
.fade-in {
  animation: fadeIn 0.6s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 页面标题样式 */
.page-header {
  text-align: center;
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  background: linear-gradient(45deg, #fff, #f0f8ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.page-subtitle {
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.8);
  margin: 10px 0 0 0;
  font-weight: 300;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .dashboard-container {
    padding: 16px;
  }

  .cards-row {
    margin-bottom: 16px;
  }

  .card-data {
    font-size: 24px;
  }

  .card-icon .icon {
    width: 50px;
    height: 50px;
  }

  .chart-container {
    height: 300px;
  }

  .page-title {
    font-size: 2rem;
  }
}

@media (max-width: 480px) {
  .dashboard-container {
    padding: 12px;
  }

  .card-content {
    padding: 16px;
  }

  .card-data {
    font-size: 20px;
  }

  .card-icon .icon {
    width: 40px;
    height: 40px;
  }

  .chart-container {
    height: 250px;
  }

  .page-title {
    font-size: 1.5rem;
  }

  .page-subtitle {
    font-size: 1rem;
  }
}
</style>