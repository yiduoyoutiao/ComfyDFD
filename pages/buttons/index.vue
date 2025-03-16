<template>
  <div class="splide-container">
    <!-- Splide轮播容器 -->
    <Splide ref="splideRef" :options="options">
      <!-- 每个按钮作为一个幻灯片 -->
      <SplideSlide v-for="(slide, index) in slides" :key="index">
        <button
          @click="selectSlide(index)"
          class="menu-button"
          :class="{ active: currentIndex === index }"
        >
          {{ slide.label }}
        </button>
      </SplideSlide>
    </Splide>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Splide, SplideSlide } from '@splidejs/vue-splide'
import '@splidejs/vue-splide/css'

// 定义按钮数据数组喵～
const slides = [
  { label: '🐱 猫咪1' },
  { label: '🐱 猫咪2' },
  { label: '🐱 猫咪3' },
  { label: '🐶 狗狗1' },
  { label: '🐶 狗狗2' },
  { label: '🐶 狗狗3' },
]

// 当前选中的按钮索引，默认选中第一个按钮
const currentIndex = ref(0)

// 获取Splide组件的引用，用于后续控制
const splideRef = ref(null)

// Splide的配置选项，具体含义见注释喵～
const options = {
  type: 'slide',      // 普通滑动类型
  perPage: 'auto',    // 每页自动根据内容宽度显示（自由滑动）
  gap: '1rem',        // 增加按钮之间的间距
  pagination: false,  // 隐藏分页指示器（小圆点）
  arrows: false,      // 隐藏左右切换箭头
  drag: 'free',       // 启用自由拖动模式
  focus: 'center',    // 点击的按钮自动居中
  speed: 200,
}

// 点击按钮时的事件处理函数
function selectSlide(index) {
  currentIndex.value = index
  splideRef.value.splide.go(index)
}

// 组件挂载后自动选中第一个按钮居中显示
onMounted(() => {
  selectSlide(0)
})
</script>

<style>
/* 外部轮播容器样式，增加padding防止按钮裁切 */
.splide-container {
  max-width: 90vw;             /* 宽度限制 */
  margin: 40px auto;            /* 上下外边距，居中显示 */
  position: relative;           /* 相对定位 */
  padding: 15px 20px;           /* 左右和上下额外padding防止按钮裁切 */
  box-sizing: border-box;       /* 防止padding导致宽度溢出 */
  overflow: hidden;             /* 隐藏溢出部分 */
}

/* Splide轨道，允许按钮显示完整不被裁切 */
.splide__track {
  overflow: visible !important; /* 防止按钮被裁切 */
}

/* 每个幻灯片自动宽度，适应按钮内容 */
.splide__slide {
  width: auto !important;       /* 幻灯片宽度自动适应按钮宽度 */
}

/* 按钮通用样式喵 */
.menu-button {
  width: 33vw;
  background-color: #ffd7e9;    /* 按钮默认背景色 */
  border: none;                 /* 无边框设计 */
  color: #333;                  /* 文字颜色 */
  padding: 16px 28px;           /* 按钮更大更好点击 */
  font-size: 1.2rem;            /* 字体大小增加 */
  border-radius: 24px;          /* 圆角设计，更柔和可爱 */
  cursor: pointer;              /* 鼠标移上去变成手型 */
  white-space: nowrap;          /* 防止按钮文字换行 */
  transition: transform 0.3s, background-color 0.3s; /* 平滑过渡动画 */
}

/* 按钮被激活选中时的样式 */
.menu-button.active {
  background-color: #ff69b4;    /* 选中后的亮色背景 */
  color: white;                 /* 选中后文字白色 */
  transform: scale(1.15);       /* 稍微放大突出选中状态 */
  box-shadow: 0 6px 12px rgba(255, 105, 180, 0.5); /* 添加阴影效果 */
}

/* 鼠标悬停到按钮上的效果 */
.menu-button:hover {
  background-color: #ffb6c1;    /* 悬停时按钮变浅粉色 */
  transform: scale(1.08);       /* 稍微放大 */
}

/* 隐藏默认箭头样式（防止出现意外） */
.splide__arrow {
  display: none;                /* 完全隐藏轮播箭头 */
}
</style>
