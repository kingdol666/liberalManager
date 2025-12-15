<template>
  <div class="sidebar-container">
    <el-menu :default-active="path" class="modern-sidebar" @open="handleOpen" @close="handleClose" router
      background-color="transparent" text-color="#2c3e50" active-text-color="#3498db" unique-opened>

      <!-- Logo区域 -->
      <div class="sidebar-header">
        <div class="logo">
          <svg class="logo-icon" aria-hidden="true">
            <use xlink:href="#iconbook"></use>
          </svg>
          <span class="logo-text">图书馆管理</span>
        </div>
      </div>

      <!-- 数据可视化 -->
      <el-menu-item index="/dashboard" class="menu-item-modern">
        <div class="menu-item-content">
          <svg class="menu-icon" aria-hidden="true">
            <use xlink:href="#icondashboard"></use>
          </svg>
          <span class="menu-text">数据可视化</span>
        </div>
      </el-menu-item>

      <!-- 个人信息 -->
      <el-sub-menu index="2" class="submenu-modern">
        <template #title>
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#icon-mingpian"></use>
            </svg>
            <span class="menu-text">个人信息</span>
            <el-icon class="arrow-icon"><arrow-right /></el-icon>
          </div>
        </template>
        <el-menu-item index="/person" class="submenu-item-modern">
          <div class="submenu-item-content">
            <svg class="submenu-icon" aria-hidden="true">
              <use xlink:href="#icon-a-bianji1"></use>
            </svg>
            <span>修改个人信息</span>
          </div>
        </el-menu-item>
        <el-menu-item index="/password" class="submenu-item-modern">
          <div class="submenu-item-content">
            <svg class="submenu-icon" aria-hidden="true">
              <use xlink:href="#icon-mima"></use>
            </svg>
            <span>修改密码</span>
          </div>
        </el-menu-item>
      </el-sub-menu>

      <!-- 管理员菜单 -->
      <template v-if="user.role == 1">
        <el-menu-item index="/user" class="menu-item-modern">
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#iconreader"></use>
            </svg>
            <span class="menu-text">读者管理</span>
          </div>
        </el-menu-item>

        <el-menu-item index="/book" class="menu-item-modern">
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#iconbook"></use>
            </svg>
            <span class="menu-text">书籍管理</span>
          </div>
        </el-menu-item>

        <el-menu-item index="/lendrecord" class="menu-item-modern">
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#iconlend-record"></use>
            </svg>
            <span class="menu-text">借阅记录</span>
          </div>
        </el-menu-item>
      </template>

      <!-- 普通用户菜单 -->
      <template v-if="user.role == 2">
        <el-menu-item index="/book" class="menu-item-modern">
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#iconbook"></use>
            </svg>
            <span class="menu-text">图书查询</span>
          </div>
        </el-menu-item>

        <el-menu-item index="/lendrecord" class="menu-item-modern">
          <div class="menu-item-content">
            <svg class="menu-icon" aria-hidden="true">
              <use xlink:href="#iconlend-record"></use>
            </svg>
            <span class="menu-text">借阅信息</span>
          </div>
        </el-menu-item>
      </template>

      <!-- 借阅状态 -->
      <el-menu-item index="/bookwithuser" class="menu-item-modern">
        <div class="menu-item-content">
          <el-icon class="menu-icon-modern">
            <grid />
          </el-icon>
          <span class="menu-text">借阅状态</span>
        </div>
      </el-menu-item>
    </el-menu>
  </div>
</template>

<script>
import { Grid, ArrowRight } from '@element-plus/icons-vue'

export default {
  name: "Aside",
  components: {
    Grid,
    ArrowRight
  },
  created() {
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
  },
  data() {
    return {
      user: {},
      path: this.$route.path
    }
  }
}
</script>

<style scoped>
/* 侧边栏容器 */
.sidebar-container {
  width: 280px;
  height: 100%;
  background: var(--primary-gradient);
  box-shadow: var(--shadow-md);
  position: relative;
  overflow: hidden;
  transition: all var(--transition-normal);
}

/* 背景装饰 */
.sidebar-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background:
    radial-gradient(circle at 20% 80%, rgba(255, 255, 255, 0.1) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 255, 255, 0.1) 0%, transparent 50%);
  pointer-events: none;
}

/* 现代化菜单样式 */
.modern-sidebar {
  border: none !important;
  background: transparent !important;
  padding: 0 var(--spacing-md);
  position: relative;
  z-index: 1;
  height: 100%;
  overflow-y: auto;
}

/* Logo区域 */
.sidebar-header {
  padding: var(--spacing-xl) var(--spacing-md) var(--spacing-lg);
  text-align: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  margin-bottom: var(--spacing-lg);
}

.logo {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-md);
}

.logo-icon {
  width: 32px;
  height: 32px;
  fill: white;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
  transition: transform var(--transition-normal);
}

.logo-icon:hover {
  transform: scale(1.1);
}

.logo-text {
  font-size: 20px;
  font-weight: 600;
  color: white;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  letter-spacing: 1px;
}

/* 菜单项样式 */
.menu-item-modern {
  margin: var(--spacing-sm) 0;
  border-radius: var(--radius-md) !important;
  transition: all var(--transition-normal);
  background: rgba(255, 255, 255, 0.1) !important;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.menu-item-modern:hover {
  background: rgba(255, 255, 255, 0.2) !important;
  transform: translateX(4px);
  box-shadow: var(--shadow-md);
}

.menu-item-modern.is-active {
  background: linear-gradient(135deg, var(--secondary-color), #2980b9) !important;
  box-shadow: 0 4px 15px rgba(52, 152, 219, 0.4);
  transform: translateX(8px);
}

/* 子菜单样式 */
.submenu-modern {
  background: rgba(255, 255, 255, 0.05) !important;
  border-radius: var(--radius-md);
  margin: var(--spacing-sm) 0;
  backdrop-filter: blur(10px);
}

.submenu-item-modern {
  margin: var(--spacing-xs) 0;
  border-radius: var(--radius-sm) !important;
  background: transparent !important;
  padding-left: 48px !important;
}

.submenu-item-modern:hover {
  background: rgba(255, 255, 255, 0.1) !important;
}

.submenu-item-modern.is-active {
  background: rgba(231, 76, 60, 0.2) !important;
  color: var(--danger-color) !important;
  font-weight: 600;
}

/* 菜单项内容布局 */
.menu-item-content {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  padding: var(--spacing-sm) var(--spacing-md);
  position: relative;
}

.submenu-item-content {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  padding: var(--spacing-xs) 0;
}

/* 图标样式 */
.menu-icon {
  width: 20px;
  height: 20px;
  fill: currentColor;
  transition: transform var(--transition-normal);
}

.submenu-icon {
  width: 16px;
  height: 16px;
  fill: currentColor;
}

.menu-icon-modern {
  font-size: 20px;
  color: currentColor;
}

.menu-item-modern:hover .menu-icon {
  transform: scale(1.1);
}

/* 菜单文本 */
.menu-text {
  font-size: 15px;
  font-weight: 500;
  letter-spacing: 0.5px;
  color: white;
  transition: color var(--transition-normal);
}

/* 选中状态的菜单文本颜色 */
.menu-item-modern.is-active .menu-text {
  color: white !important;
  font-weight: 600;
}

/* 箭头图标 */
.arrow-icon {
  margin-left: auto;
  transition: transform var(--transition-normal);
  font-size: 12px;
  color: white;
}

.el-sub-menu.is-opened .arrow-icon {
  transform: rotate(90deg);
}

/* 滚动条美化 */
.modern-sidebar::-webkit-scrollbar {
  width: 6px;
}

.modern-sidebar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.modern-sidebar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.modern-sidebar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* 动画效果 */
.menu-item-modern,
.submenu-modern {
  animation: slideInLeft var(--transition-normal);
}

@keyframes slideInLeft {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* 激活状态指示器 */
.menu-item-modern.is-active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 60%;
  background: linear-gradient(to bottom, var(--secondary-color), #2980b9);
  border-radius: 0 4px 4px 0;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .sidebar-container {
    width: 100%;
    height: auto;
  }

  .logo-text {
    font-size: 18px;
  }

  .menu-text {
    font-size: 14px;
  }
}

@media (max-width: 480px) {
  .sidebar-header {
    padding: var(--spacing-lg) var(--spacing-sm) var(--spacing-md);
  }

  .logo-icon {
    width: 28px;
    height: 28px;
  }

  .logo-text {
    font-size: 16px;
  }
}
</style>