<template>
  <div class="app-container">
    <!-- 头部组件 -->
    <Header />
    
    <!-- 主体内容区域 -->
    <main class="main-content">
      <!-- 侧边栏 -->
      <Aside class="sidebar" />
      
      <!-- 主要内容 -->
      <div class="content-wrapper">
        <router-view />
      </div>
    </main>
  </div>
</template>

<script>
import Header from "../components/Header";
import Aside from "../components/Aside";

export default {
  name: "Layout",
  components: {
    Header,
    Aside
  }
}
</script>

<style scoped>
/* 应用容器 */
.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: var(--bg-primary);
}

/* 主体内容区域 */
.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
  min-height: calc(100vh - 64px);
}

/* 侧边栏 */
.sidebar {
  flex-shrink: 0;
  width: 280px;
  background: var(--primary-gradient);
  transition: all var(--transition-normal);
  box-shadow: 4px 0 20px rgba(0, 0, 0, 0.1);
  z-index: 50;
}

/* 内容包装器 */
.content-wrapper {
  flex: 1;
  padding: var(--spacing-lg);
  overflow-y: auto;
  background-color: var(--bg-primary);
  transition: all var(--transition-normal);
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .sidebar {
    width: 240px;
  }
  
  .content-wrapper {
    padding: var(--spacing-md);
  }
}

@media (max-width: 768px) {
  .main-content {
    flex-direction: column;
  }
  
  .sidebar {
    width: 100%;
    height: auto;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  }
  
  .content-wrapper {
    padding: var(--spacing-sm);
  }
}

@media (max-width: 480px) {
  .content-wrapper {
    padding: var(--spacing-xs);
  }
}
</style>