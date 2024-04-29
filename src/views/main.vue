<template>
  <!-- 根据是否显示悬浮页面来决定是否渲染 IHeader -->
  <IHeader @goLevitate="goLevitate" v-if="!isLevitating && routeName !== 'editor'" />

  <!-- 根据是否显示悬浮页面来决定是否渲染主内容 -->
  <div v-if="!isLevitating" class="bg-white">
    <router-view v-slot="{ Component }" :key="routeName">
      <transition name="main-fade">
        <div class="transition" :key="routeName" :data-title="routeName">
          <keep-alive>
            <component :is="Component" />
          </keep-alive>
        </div>
      </transition>
    </router-view>
  </div>
  <!-- 悬浮页面，始终显示在最上层 " -->
  <teleport v-if="isLevitating" to="body > *:first-child">
    <div class="levitate-page-overlay">
      <GoLevitate @close="hideLevitate" :isLevitating="isLevitating" />
    </div>
  </teleport>
</template>

<script setup lang="ts">
import { remote } from 'electron';
import { ref, onBeforeUpdate } from 'vue';
import { useRoute } from 'vue-router';
import IHeader from '@/components/IHeader.vue';
import GoLevitate from '@/components/IGoLevitate.vue'; // 悬浮组件

// 当前路由名称
const routeName = ref(useRoute().name as string);

// 获取窗口固定状态
let isAlwaysOnTop = ref(false);
const currentWindow = remote.getCurrentWindow();
isAlwaysOnTop.value = currentWindow.isAlwaysOnTop();

// 是否显示悬浮页面的状态
const isLevitating = ref(false);

// 触发悬浮页面显示的方法
const showLevitate = () => {
  isLevitating.value = true;
};

// 触发悬浮页面隐藏的方法
const hideLevitate = () => {
  isLevitating.value = false;
  currentWindow.setSize(600, 600);

  let [currentX, currentY] = currentWindow.getPosition();
  currentWindow.setPosition(currentX - 600, currentY - 300);
};

// 悬浮
const goLevitate = () => {
  showLevitate();

  const { screen } = remote;
  // 获取屏幕的宽度和高度
  const { width, height } = screen.getPrimaryDisplay().workAreaSize;

  // 设置窗口的位置到屏幕右侧
  // 设置窗口的新位置
  let newX = width - 30;
  let newY = (height - 100) / 2;
  currentWindow.setSize(30, 30);

  // 初始位置
  let [startCurrentX, startCurrentY] = currentWindow.getPosition();

  // 初始位置Y轴方向的上面还是下面
  let currentYFlag = true;
  // 判断窗口y轴的位置与要更新的位置大小
  if (startCurrentY > newY) currentYFlag = false;

  // 逐步移动窗口到新位置
  let interval = setInterval(() => {
    let [currentX, currentY] = currentWindow.getPosition();
    if (currentX < newX) currentX = currentX + 30;
    if (currentYFlag) {
      if (currentY < newY) currentY = currentY + 1;
    } else {
      if (currentY > newY) currentY = currentY - 1;
    }
    currentWindow.setPosition(currentX, currentY);
    // 当窗口到达新位置时停止动画
    if (currentX >= newX && (currentYFlag ? currentY >= newY : newY >= currentY)) {
      clearInterval(interval);
      currentWindow.setPosition(newX, newY);
    }
  }, 1); // 10毫秒更新一次位置
  currentWindow.setAlwaysOnTop(true, 'normal');
};
// 在路由变化前更新 routeName
onBeforeUpdate(() => {
  routeName.value = useRoute().name as string;
});
</script>

<style lang="less" scoped>
.main-fade-enter,
.main-fade-leave-to {
  display: none;
  opacity: 0;
  animation: main-fade 0.4s reverse;
}
.main-fade-enter-active,
.main-fade-leave-active {
  opacity: 0;
  animation: main-fade 0.4s;
}
@keyframes main-fade {
  from {
    opacity: 0;
    transform: scale(0.96);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
