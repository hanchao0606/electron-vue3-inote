<template>
  <!-- 根据是否显示悬浮页面来决定是否渲染 IHeader -->
  <IHeader v-if="routeName !== 'editor'" />

  <!-- 根据是否显示悬浮页面来决定是否渲染主内容 -->
  <div class="bg-white">
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
</template>

<script setup lang="ts">
import { remote } from 'electron';
import { ref, onBeforeUpdate } from 'vue';
import { useRoute } from 'vue-router';
import IHeader from '@/components/IHeader.vue';

// 当前路由名称
const routeName = ref(useRoute().name as string);

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
