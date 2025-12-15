<template>
  <header class="modern-header">
    <!-- Logo区域 -->
    <div class="header-left">
      <div class="logo-container">
        <img :src="imgUrl" class="logo-icon">
        <h1 class="logo-text">图书馆管理系统</h1>
      </div>
    </div>
    
    <!-- 中间占位 -->
    <div class="header-center"></div>
    
    <!-- 用户信息区域 -->
    <div class="header-right">
      <el-dropdown trigger="click" placement="bottom-end">
        <div class="user-info" @click="$event.stopPropagation()">
          <div class="user-avatar">
            <svg class="avatar-icon" aria-hidden="true">
              <use xlink:href="#icon-mingpian"></use>
            </svg>
          </div>
          <div class="user-details">
            <span class="user-name">{{ user.nickName || user.username }}</span>
            <span class="user-role">{{ user.role === 1 ? '管理员' : '普通用户' }}</span>
          </div>
          <el-icon class="dropdown-arrow"><arrow-down /></el-icon>
        </div>
        <template #dropdown>
          <el-dropdown-menu class="user-dropdown-menu">
            <el-dropdown-item @click="exit" class="dropdown-item">
              <el-icon class="item-icon"><SwitchButton /></el-icon>
              <span>退出系统</span>
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </header>
</template>

<script>
import { ElMessage } from "element-plus";
import { ArrowDown, SwitchButton } from '@element-plus/icons-vue';

export default {
  name: "Header",
  components: {
    ArrowDown,
    SwitchButton
  },
  props: ['user'],
  data() {
    return {
      user: {},
      imgUrl: require("../assets/icon/login.png")
    }
  },
  created() {
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
  },
  methods: {
    exit() {
      sessionStorage.removeItem("user")
      this.$router.push("/login")
      ElMessage.success("退出系统成功")
    }
  }
}
</script>

<style scoped>
/* 现代化Header样式 */
.modern-header {
  height: 64px;
  background: var(--primary-gradient);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 var(--spacing-lg);
  box-shadow: var(--shadow-md);
  position: sticky;
  top: 0;
  z-index: 100;
  transition: all var(--transition-normal);
}

/* Header左侧 - Logo区域 */
.header-left {
  display: flex;
  align-items: center;
}

.logo-container {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
}

.logo-icon {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
  background: rgba(255, 255, 255, 0.2);
  padding: var(--spacing-xs);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  transition: transform var(--transition-normal);
}

.logo-icon:hover {
  transform: scale(1.1);
}

.logo-text {
  font-size: 20px;
  font-weight: 600;
  color: white;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  letter-spacing: 1px;
}

/* Header中间占位 */
.header-center {
  flex: 1;
}

/* Header右侧 - 用户信息区域 */
.header-right {
  display: flex;
  align-items: center;
}

.user-info {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  transition: all var(--transition-normal);
  color: white;
}

.user-info:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

/* 用户头像 */
.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.avatar-icon {
  width: 20px;
  height: 20px;
  fill: white;
}

/* 用户详情 */
.user-details {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 2px;
}

.user-name {
  font-size: 14px;
  font-weight: 500;
  color: white;
  margin: 0;
}

.user-role {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.8);
  margin: 0;
  background: rgba(255, 255, 255, 0.2);
  padding: 2px 6px;
  border-radius: 10px;
}

/* 下拉箭头 */
.dropdown-arrow {
  font-size: 12px;
  color: white;
  transition: transform var(--transition-normal);
}

.user-info:hover .dropdown-arrow {
  transform: rotate(180deg);
}

/* 用户下拉菜单 */
.user-dropdown-menu {
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-lg);
  border: none;
  overflow: hidden;
  min-width: 160px;
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  padding: var(--spacing-sm) var(--spacing-md);
  font-size: 14px;
  transition: all var(--transition-fast);
}

.dropdown-item:hover {
  background-color: rgba(102, 126, 234, 0.1);
  color: var(--primary-color);
}

.item-icon {
  font-size: 16px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .modern-header {
    padding: 0 var(--spacing-md);
  }
  
  .logo-text {
    font-size: 18px;
  }
  
  .user-details {
    display: none;
  }
  
  .user-info {
    padding: var(--spacing-sm);
  }
}

@media (max-width: 480px) {
  .logo-text {
    display: none;
  }
  
  .logo-container {
    gap: var(--spacing-sm);
  }
}
</style>