<template>
  <div class="go-levitate-page">
    <!-- 悬浮页面的内容  -->
    <button @mouseover="$emit('close')">关闭悬浮</button>
  </div>
</template>

<script setup>
import { toRefs, defineProps } from 'vue';

const emit = defineEmits(['close']);

const props = defineProps({
  // 子组件接收父组件传递过来的值
  isLevitating: Boolean
});

// 使用父组件传递过来的值
const { isLevitating } = toRefs(props);

console.log('IGolevitate', isLevitating);

const funisLevitating = () => {
  this.isLevitating = !this.isLevitating;
};
</script>

<style scoped>
.go-levitate-page {
  /* 悬浮页面的样式 */
}
</style>
