<template>
  <header class="header flex-between">
    <!--悬浮到右侧-->
    <button class="icon flex-center" @click="goLevitate" title="悬浮">
      <i class="iconfont flex-center icon-refresh"></i>
    </button>
    <!-- 悬浮页面，始终显示在最上层 " -->
    <div v-if="isLevitating" class="levitate-page-overlay">
      <GoLevitate @close="hideLevitate" :isLevitating="isLevitating" :levitatingContent="levitatingContent" />
    </div>
    <template v-if="currentRouteName === 'setting'">
      <button class="icon flex-center" title="返回">
        <router-link class="flex-center" to="/">
          <i class="iconfont flex-center icon-back"></i>
        </router-link>
      </button>
    </template>
    <template v-else-if="currentRouteName !== 'imagePreview'">
      <!-- 打开新窗口 -->
      <button class="icon flex-center" @click="openNewWindow" title="新窗口">
        <i class="iconfont flex-center icongit-add"></i>
      </button>
    </template>

    <!-- 标题拖动 -->
    <div class="drag-header flex1 flex-center" :style="computedPaddingLeft" @mousedown="a">
      <transition name="header-fadein" v-if="platformWindows">
        <span :key="title">{{ title }}</span>
      </transition>
    </div>
    <!-- 右侧操作 -->
    <div class="operation-btn flex-items">
      <!-- 设置 -->
      <template v-if="currentRouteName === 'index'">
        <button class="icon flex-center" title="设置">
          <router-link class="flex-center" to="/setting">
            <i class="iconfont flex-center icon-setting"></i>
          </router-link>
        </button>
      </template>
      <template v-else-if="currentRouteName === 'editor'">
        <!-- 固定 -->
        <div class="thepin" :class="isAlwaysOnTop ? 'thepin-active' : ''">
          <div class="absolute-box">
            <button class="icon flex-center" @click="drawingPin" title="置顶">
              <i class="iconfont flex-center icon-thepin"></i>
            </button>
            <button class="icon flex-center" @click="drawingPin" title="取消置顶">
              <i class="iconfont flex-center icon-thepin-active"></i>
            </button>
          </div>
        </div>
        <!-- 更多 -->
        <button class="icon flex-center" @click="clickOptions" title="选项">
          <i class="iconfont flex-center icon-more"></i>
        </button>
      </template>
      <!-- 关闭 -->
      <button v-if="platformWindows" class="icon flex-center close-window" @click="closeWindow" title="关闭">
        <i class="iconfont flex-center icon-close"></i>
      </button>
    </div>
  </header>

  <!-- 悬浮页面，始终显示在最上层 " -->
  <Teleport v-if="isLevitating" to="body > *:first-child">
    <div class="levitate-page-overlay">
      <GoLevitate @close="hideLevitate" :isLevitating="isLevitating" />
    </div>
  </Teleport>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { onBeforeRouteUpdate, useRoute } from 'vue-router';
import { browserWindowOption } from '@/config';
import { createBrowserWindow, transitCloseWindow } from '@/utils';
import { remote } from 'electron';
import GoLevitate from '@/components/IGoLevitate.vue'; // 悬浮组件
const { screen } = remote;

// 是否显示悬浮页面的状态
const isLevitating = ref(false);

// 悬浮时的内容
const levitatingContent = '哈哈哈';

window.isLevitating_ = isLevitating;

// 悬浮切换要存在一定时间间隔
let levitatingInterval = true;

// 触发悬浮页面显示的方法
const showLevitate = () => {
  isLevitating.value = true;
};

// 触发悬浮页面隐藏的方法
const hideLevitate = () => {
  if (!levitatingInterval) return;
  levitatingInterval = false;
  setTimeout(() => {
    levitatingInterval = true;
  }, 1000);
  isLevitating.value = false;
  currentWindow.setSize(600, 600);

  let [currentX, currentY] = currentWindow.getPosition();
  currentWindow.setPosition(currentX - 600, currentY - 300);
  // 透明度
  currentWindow.setOpacity(0.97);
};

// 悬浮
const goLevitate = () => {
  if (!levitatingInterval) return;

  levitatingInterval = false;
  setTimeout(() => {
    levitatingInterval = true;
  }, 1000);

  showLevitate();
  // 获取屏幕的宽度和高度
  const { width, height } = screen.getPrimaryDisplay().workAreaSize;

  // 设置窗口的位置到屏幕右侧
  // 设置窗口的新位置
  let newX = width - 30;
  let newY = (height - 100) / 2;

  // 透明度
  currentWindow.setOpacity(0.1);
  // 初始位置
  let [startCurrentX, startCurrentY] = currentWindow.getPosition();

  // 初始位置在Y轴方向的上面还是下面
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
  // 窗口大小
  currentWindow.setSize(30, 70);
};
const emits = defineEmits(['optionClick', 'onClose']);

const platformWindows = process.platform === 'win32';

const editorWinOptions = browserWindowOption('editor');
// 打开新窗口
const openNewWindow = () => {
  createBrowserWindow(editorWinOptions, '/editor', false);
};

// 获取窗口固定状态
let isAlwaysOnTop = ref(false);
const currentWindow = remote.getCurrentWindow();
isAlwaysOnTop.value = currentWindow.isAlwaysOnTop();

// 固定前面
const drawingPin = () => {
  if (isAlwaysOnTop.value) {
    currentWindow.setAlwaysOnTop(false);
    isAlwaysOnTop.value = false;
  } else {
    currentWindow.setAlwaysOnTop(true);
    isAlwaysOnTop.value = true;
  }
};

const currentRouteName = ref(useRoute().name);

// 获取首页的内边距
const computedPaddingLeft = computed(() => {
  return currentRouteName.value === 'index' ? 'padding-left: 40px;' : '';
});

const title = ref(useRoute().meta.title as string);

onBeforeRouteUpdate((to, from, next) => {
  title.value = to.meta.title as string;
  document.title = title.value;
  currentRouteName.value = to.name;
  next();
});

const clickOptions = () => {
  emits('optionClick');
};

// 关闭窗口
const closeWindow = () => {
  emits('onClose');
  transitCloseWindow();
};

const a = () => {
  console.log(1);
};
</script>

<style lang="less" scoped>
.header-fadein-enter,
.header-fadein-leave-to {
  display: none;
  opacity: 0;
  animation: header-fadein 0.4s reverse;
}

.header-fadein-enter-active,
.header-fadein-leave-active {
  opacity: 0;
  animation: header-fadein 0.4s;
}

@keyframes header-fadein {
  from {
    opacity: 0;
    margin-right: -14px;
  }

  to {
    opacity: 1;
    margin-right: 0;
  }
}

.header {
  height: @iconSize;
  background-color: @white-color;
  position: relative;
  z-index: 99;

  button {
    padding: 0;
    outline: none;
    border: none;
    background-color: transparent;
  }

  a {
    color: initial;
    width: 100%;
    height: 100%;
    outline: none;
  }

  .icon {
    width: @iconSize;
    height: @iconSize;

    .iconfont {
      // 头部icon大小在这里设置
      font-size: @headerIconFontSize;
      padding-bottom: 2px;
    }
  }

  .close-window:hover {
    background-color: @error-color;
    color: @white-color;
  }

  @keyframes fades {
    30% {
      opacity: 0;
    }

    100% {
      opacity: 1;
    }
  }

  .drag-header {
    -webkit-app-region: drag;
    height: 36px;
    margin-top: 5px;
    padding-bottom: 5px;
    color: @text-sub-color;
    font-size: 15px;
    font-weight: bold;
    box-sizing: border-box;
  }
}

.thepin {
  width: 40px;
  height: 40px;
  overflow: hidden;
  position: relative;
  transition: all 0.4s;

  .absolute-box {
    position: absolute;
    top: 0;
    transition: all 0.4s;
  }
}

.thepin-active .absolute-box {
  top: -40px;
}
</style>
