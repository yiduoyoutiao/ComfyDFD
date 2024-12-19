<template>
  <div ref="canvasContainer" class="canvas-container"></div>
  <div ref="debugInfo" class="debug-info"></div>
</template>

<script setup>
import { onMounted, ref, onUnmounted } from 'vue';
import * as THREE from 'three';
import Stats from 'stats.js';

const canvasContainer = ref(null);
const debugInfo = ref(null);

onMounted(() => {
  // 1️⃣ 创建 Stats (显示 FPS)
  const stats = new Stats();
  stats.showPanel(0); // 显示FPS
  document.body.appendChild(stats.dom);
  stats.dom.style.left = "calc(100% - 100px)";
  stats.dom.style.top = "50px";
  stats.dom.classList.add('custom-stats');

  // 2️⃣ 创建场景
  const scene = new THREE.Scene();

  // 3️⃣ 创建正交相机
  let aspect = window.innerWidth / window.innerHeight;
  let zoomFactor = 1;
  const minZoom = 0.5;
  const maxZoom = 3;

  const camera = new THREE.OrthographicCamera(
      -aspect * 5,
      aspect * 5,
      5,
      -5,
      0.1,
      100
  );
  camera.position.z = 10;

  // 4️⃣ 创建渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0xf0f0f0);
  canvasContainer.value.appendChild(renderer.domElement);

  // 5️⃣ 添加网格背景 (柔和灰色)
  const gridHelper = new THREE.GridHelper(10, 40, 0xaaaaaa, 0xaaaaaa);
  gridHelper.rotation.x = Math.PI / 2;
  scene.add(gridHelper);

  // 原始方块的几何与材质
  const geometry = new THREE.PlaneGeometry(1, 1);
  const material = new THREE.MeshBasicMaterial({ color: 0x42b983, side: THREE.DoubleSide });

  // 创建方块的函数（每个方块有红色边框）
  function createSquareAt(position) {
    const sq = new THREE.Mesh(geometry, material);
    sq.position.copy(position);
    const eGeo = new THREE.EdgesGeometry(geometry);
    const eMat = new THREE.LineBasicMaterial({ color: 0xff0000 });
    const ed = new THREE.LineSegments(eGeo, eMat);
    ed.visible = false;
    sq.add(ed);
    return sq;
  }

  // 6️⃣ 创建初始方块
  const square = createSquareAt(new THREE.Vector3(0, 0, 0));
  scene.add(square);

  // 方块数组管理所有方块
  const squares = [square];

  // 用于 Raycaster 的辅助对象
  const raycaster = new THREE.Raycaster();

  // 禁止默认右键菜单
  window.addEventListener('contextmenu', e => e.preventDefault());

  // 7️⃣ 窗口大小变化
  function onWindowResize() {
    aspect = window.innerWidth / window.innerHeight;
    camera.left = -aspect * 5 * zoomFactor;
    camera.right = aspect * 5 * zoomFactor;
    camera.top = 5 * zoomFactor;
    camera.bottom = -5 * zoomFactor;
    camera.updateProjectionMatrix();

    renderer.setSize(window.innerWidth, window.innerHeight);
  }
  window.addEventListener('resize', onWindowResize);

  // 8️⃣ 滚轮缩放
  function handleScroll(event) {
    if (event.deltaY > 0) {
      zoomFactor *= 1.1;
    } else {
      zoomFactor /= 1.1;
    }

    zoomFactor = Math.min(maxZoom, Math.max(minZoom, zoomFactor));

    camera.left = -aspect * 5 * zoomFactor;
    camera.right = aspect * 5 * zoomFactor;
    camera.top = 5 * zoomFactor;
    camera.bottom = -5 * zoomFactor;
    camera.updateProjectionMatrix();
  }
  window.addEventListener('wheel', handleScroll);

  // 9️⃣ 中键拖拽移动画面
  let isPanning = false;
  let lastMouseX = 0;
  let lastMouseY = 0;

  // 🔟 左键选中及拖拽方块
  let selectedObject = null;
  let isDraggingObject = false;
  let dragOffset = new THREE.Vector3();

  function getWorldPositionFromScreen(x, y) {
    const rect = renderer.domElement.getBoundingClientRect();
    const ndcX = ((x - rect.left) / rect.width) * 2 - 1;
    const ndcY = -((y - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera({ x: ndcX, y: ndcY }, camera);
    const planeZ = 0;
    const t = (planeZ - raycaster.ray.origin.z) / raycaster.ray.direction.z;
    const worldPos = new THREE.Vector3().copy(raycaster.ray.origin).add(raycaster.ray.direction.clone().multiplyScalar(t));
    return worldPos;
  }

  // 鼠标按下事件
  function onMouseDown(event) {
    if (event.button === 1) {
      // 中键拖拽画面
      isPanning = true;
      lastMouseX = event.clientX;
      lastMouseY = event.clientY;

    } else if (event.button === 0) {
      // 左键尝试选中方块
      const rect = renderer.domElement.getBoundingClientRect();
      const ndcX = ((event.clientX - rect.left) / rect.width) * 2 - 1;
      const ndcY = -((event.clientY - rect.top) / rect.height) * 2 + 1;

      raycaster.setFromCamera({ x: ndcX, y: ndcY }, camera);
      const intersects = raycaster.intersectObjects(squares, false);

      if (intersects.length > 0) {
        // 点击到了方块
        // 先取消所有方块的选中
        for (const sq of squares) {
          sq.children[0].visible = false;
        }

        selectedObject = intersects[0].object;
        selectedObject.children[0].visible = true;
        isDraggingObject = true;
        const worldPos = getWorldPositionFromScreen(event.clientX, event.clientY);
        dragOffset.copy(selectedObject.position).sub(worldPos);
      } else {
        // 空白处点击: 取消选中所有方块
        for (const sq of squares) {
          sq.children[0].visible = false;
        }
        selectedObject = null;
      }

    } else if (event.button === 2) {
      // 右键：在鼠标位置新建方块
      const pos = getWorldPositionFromScreen(event.clientX, event.clientY);
      const newSquare = createSquareAt(pos);
      scene.add(newSquare);
      squares.push(newSquare);
    }
  }

  // 鼠标移动事件
  function onMouseMove(event) {
    // 处理中键平移画面
    if (isPanning) {
      const deltaX = event.clientX - lastMouseX;
      const deltaY = event.clientY - lastMouseY;
      lastMouseX = event.clientX;
      lastMouseY = event.clientY;

      const worldWidth = (camera.right - camera.left);
      const worldHeight = (camera.top - camera.bottom);

      const moveX = (deltaX / window.innerWidth) * worldWidth;
      const moveY = (deltaY / window.innerHeight) * worldHeight;

      camera.position.x -= moveX;
      camera.position.y += moveY;
    }

    // 左键拖拽方块
    if (isDraggingObject && selectedObject) {
      const worldPos = getWorldPositionFromScreen(event.clientX, event.clientY);
      selectedObject.position.copy(worldPos).add(dragOffset);
    }
  }

  // 鼠标抬起事件
  function onMouseUp(event) {
    if (event.button === 1) {
      isPanning = false;
    } else if (event.button === 0) {
      isDraggingObject = false;
    }
  }

  // 鼠标离开窗口事件
  function onMouseLeave() {
    isPanning = false;
    isDraggingObject = false;
  }

  window.addEventListener('mousedown', onMouseDown);
  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('mouseup', onMouseUp);
  window.addEventListener('mouseleave', onMouseLeave);

  // 渲染循环
  function animate() {
    stats.begin();
    renderer.render(scene, camera);
    const drawCalls = renderer.info.render.calls;
    debugInfo.value.innerText = `Draw Calls: ${drawCalls}`;
    stats.end();
    requestAnimationFrame(animate);
  }
  animate();

  // 组件卸载时移除事件监听
  onUnmounted(() => {
    window.removeEventListener('resize', onWindowResize);
    window.removeEventListener('wheel', handleScroll);
    window.removeEventListener('mousedown', onMouseDown);
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', onMouseUp);
    window.removeEventListener('mouseleave', onMouseLeave);
    window.removeEventListener('contextmenu', e => e.preventDefault());
  });
});
</script>

<style scoped>
.canvas-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  cursor: grab;
}

.canvas-container:active {
  cursor: grabbing;
}

.debug-info {
  width: 200px;
  height: 50px;
  position: absolute;
  top: 50px;
  right: 10px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  font-size: 14px;
  padding: 5px 10px;
  border-radius: 5px;
}
</style>
