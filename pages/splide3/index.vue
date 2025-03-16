<template>
  <div class="splide-container">
    <!-- 轮播：只渲染当前种类的图片 -->
    <Splide :options="options" :key="currentType">
      <SplideSlide v-for="(slide, index) in filteredSlides" :key="index">
        <img :src="slide.url" class="slide-image" />
      </SplideSlide>
    </Splide>

    <!-- 切换按钮 -->
    <div class="switch-button-container">
      <button @click="switchType" class="switch-button">
        切换到{{ currentType === 'cats' ? '狗狗🐶' : '猫猫🐱' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { Splide, SplideSlide } from '@splidejs/vue-splide'
import '@splidejs/vue-splide/css'

// 1. 定义当前种类
const currentType = ref('cats')

// 2. 把 6 张猫狗图片放在同一个数组，并用 type 区分
const allSlides = [
  { type: 'cats', url: '/cats/cat1.jpg' },
  { type: 'cats', url: '/cats/cat2.jpg' },
  { type: 'cats', url: '/cats/cat3.jpg' },
  { type: 'dogs', url: '/dogs/dog1.jpg' },
  { type: 'dogs', url: '/dogs/dog2.jpg' },
  { type: 'dogs', url: '/dogs/dog3.jpg' },
  { type: 'dogs', url: '/dogs/dog3.jpg' },
]

// 3. 根据 currentType 过滤出对应种类的图片
const filteredSlides = computed(() => {
  return allSlides.filter(slide => slide.type === currentType.value)
})

// 4. 切换类型时，只显示另一种，并回到它的第一张
function switchType() {
  currentType.value = currentType.value === 'cats' ? 'dogs' : 'cats'
}

// 5. Splide 轮播配置
const options = {
  type: 'loop',
  autoplay: true,
  interval: 3000,
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
