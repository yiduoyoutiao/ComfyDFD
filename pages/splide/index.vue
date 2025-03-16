<template>
  <div class="splide-container">
    <Splide :options="options" :key="currentType">
      <SplideSlide v-for="(slide, index) in slides" :key="index">
        <img :src="slide" class="slide-image" />
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

// 定义初始展示类型为猫猫喵～
const currentType = ref('cats')

// 定义图片路径数据
const images = {
  cats: ['/cats/cat1.jpg', '/cats/cat2.jpg', '/cats/cat3.jpg'],
  dogs: ['/dogs/dog1.jpg', '/dogs/dog2.jpg', '/dogs/dog3.jpg'],
}

// 根据当前类型计算展示的图片喵
const slides = computed(() => images[currentType.value])

// 点击切换类型
function switchType() {
  currentType.value = currentType.value === 'cats' ? 'dogs' : 'cats'
}

// 轮播配置
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
