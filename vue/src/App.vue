<template>
  <div class="app-wrapper">
    <transition name="page-transition" mode="out-in">
      <router-view />
    </transition>
  </div>
</template>

<style>
/* 页面切换动画 */
.page-transition-enter-active,
.page-transition-leave-active {
  transition: all var(--transition-normal);
}

.page-transition-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.page-transition-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* 应用包装器 */
.app-wrapper {
  overflow-y: hidden;
  overflow-x: hidden;
  min-height: 100vh;
}
</style>
<script>
export default {
  name:"App",
}
</script>