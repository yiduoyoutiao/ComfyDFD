<template>
  <div class="splide-container">
    <!-- 轮播 -->
    <Splide ref="splideRef" :options="options">
      <SplideSlide v-for="(slide, index) in allSlides" :key="index">
        <img :src="slide.url" class="slide-image" />
      </SplideSlide>
    </Splide>

    <!-- 切换按钮 -->
    <div class="switch-button-container">
      <button @click="switchType" class="switch-button">
        {{ switchButtonText }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { Splide, SplideSlide } from '@splidejs/vue-splide'
import '@splidejs/vue-splide/css'

// 1. 将猫狗图片都放进同一个数组，并标记它是 cats 还是 dogs
const allSlides = [
  { type: 'cats', url: '/cats/cat1.jpg' },
  { type: 'cats', url: '/cats/cat2.jpg' },
  { type: 'cats', url: '/cats/cat3.jpg' },
  { type: 'dogs', url: '/dogs/dog1.jpg' },
  { type: 'dogs', url: '/dogs/dog2.jpg' },
  { type: 'dogs', url: '/dogs/dog3.jpg' },
]

// 2. 定义 Splide 的引用，用来控制轮播跳转
const splideRef = ref(null)

// 3. 配置轮播
const options = {
  type: 'loop',
  autoplay: true,
  interval: 3000,
}

// 4. 计算按钮文字：根据“当前显示的图片”来决定文字
const switchButtonText = computed(() => {
  // 若 splide 还没挂载完成，则先返回一个占位
  if (!splideRef.value) return '切换中...'

  // 获取当前的索引
  const currentIndex = splideRef.value.splide.index
  const currentSlide = allSlides[currentIndex]

  // 如果当前是猫图，就显示“切换到狗狗”，否则反之
  return currentSlide.type === 'cats'
    ? '切换到狗狗🐶'
    : '切换到猫猫🐱'
})

// 5. 点击切换按钮的逻辑
function switchType() {
  const splide = splideRef.value.splide
  const currentIndex = splide.index
  const currentSlide = allSlides[currentIndex]

  // 如果当前是猫图，则找到第一张狗图的索引
  if (currentSlide.type === 'cats') {
    const dogIndex = allSlides.findIndex(item => item.type === 'dogs')
    if (dogIndex !== -1) {
      splide.go(dogIndex)
    }
  } else {
    // 如果当前是狗图，则找到第一张猫图的索引
    const catIndex = allSlides.findIndex(item => item.type === 'cats')
    if (catIndex !== -1) {
      splide.go(catIndex)
    }
  }
}
</script>

<style>
.splide-container {
  max-width: 600px;
  margin: 40px auto;
  position: relative;
}

.slide-image {
  width: 100%;
  height: 350px;
  object-fit: cover;
  border-radius: 12px;
}

/* 切换按钮样式喵 */
.switch-button-container {
  position: absolute;
  top: 450px;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
}

.switch-button {
  background-color: #ffb6c1;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 1rem;
  border-radius: 20px;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(255, 105, 180, 0.3);
  transition: all 0.3s;
}

.switch-button:hover {
  background-color: #ff69b4;
  transform: scale(1.05);
}

/* 箭头整体样式 */
.splide__arrow {
  width: 40px;
  height: 40px;
  background-color: rgba(255, 182, 193, 0.8);
  border-radius: 50%;
  transition: background-color 0.3s ease;
}

.splide__arrow svg {
  fill: #fff;
}

.splide__arrow:hover {
  background-color: rgba(255, 105, 180, 1);
}

.splide__pagination {
  bottom: -30px;
}

.splide__pagination__page {
  background-color: #ffd7e9;
  width: 12px;
  height: 12px;
  margin: 0 6px;
  opacity: 1;
  transition: transform 0.3s, background-color 0.3s;
}

.splide__pagination__page.is-active {
  background-color: #ff69b4;
  transform: scale(1.5);
}
</style>
