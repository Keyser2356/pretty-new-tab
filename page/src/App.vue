<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';

const mouse = ref({ x: 0, y: 0 });

const onMouseMove = (e: MouseEvent) => {

  mouse.value.x = (e.clientX / window.innerWidth) * 2 - 1;
  mouse.value.y = (e.clientY / window.innerHeight) * 2 - 1;
};

onMounted(() => {
  window.addEventListener('mousemove', onMouseMove);
});

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove);
});

const wallpaperModules = import.meta.glob('@/assets/wallpapers/*.{jpg,jpeg,png,webp,gif}', {
  eager: true,
  import: 'default',
  as: 'url'
});

const wallpaperUrls = Object.values(wallpaperModules) as string[];
const randomWallpaper = wallpaperUrls[Math.floor(Math.random() * wallpaperUrls.length)];

const backgroundStyle = computed(() => {
  const intensity = 25; 
  const moveX = mouse.value.x * -intensity;
  const moveY = mouse.value.y * -intensity;

  return {
    backgroundImage: `url(${randomWallpaper})`,
    transform: `translate3d(${moveX}px, ${moveY}px, 0) scale(1.1)`
  };
});

const currentDate = ref(new Date());

const hours = computed(() => currentDate.value.getHours().toString().padStart(2, '0'));
const minutes = computed(() => currentDate.value.getMinutes().toString().padStart(2, '0'));

const timeDigits = computed(() => ({
  hours: hours.value.split(''),
  minutes: minutes.value.split(''),
}));

const blockOrder = ['hours', 'minutes'] as const;
type TimeBlock = (typeof blockOrder)[number];
const digitsPerBlock = 2;

const getDigitDelay = (block: TimeBlock, idx: number) => {
  const blockIndex = blockOrder.indexOf(block);
  const base = (blockOrder.length - blockIndex - 1) * digitsPerBlock;
  const digitOffset = digitsPerBlock - idx - 1;
  return `${(base + digitOffset) * 0.03}s`;
};

const day = computed(() => currentDate.value.getDate().toString().padStart(2, '0'));
const month = computed(() => (currentDate.value.getMonth() + 1).toString().padStart(2, '0'));
const weekday = computed(() => currentDate.value.toLocaleString('en-US', { weekday: 'long' }).toLowerCase());
const year = computed(() => currentDate.value.getFullYear());

onMounted(() => {
  setInterval(() => {
    currentDate.value = new Date();
  }, 1000);
});
</script>

<template>
  <div class="home">
    <div class="parallax-bg" :style="backgroundStyle"></div>

    <div class="main-container">
      <div class="time">
        <template v-for="block in blockOrder" :key="block">
          <span
            v-for="(digit, idx) in timeDigits[block]"
            :key="`${block}-${idx}`"
            class="digit-wrapper"
            :style="{ '--digit-delay': getDigitDelay(block, idx) }"
          >
            <transition-group name="digit" tag="span" class="digit-stack">
              <span class="digit-char" :key="`${block}-${idx}-${digit}`">
                {{ digit }}
              </span>
            </transition-group>
          </span>
          <span v-if="block !== blockOrder[blockOrder.length - 1]" class="time-divider">:</span>
        </template>
      </div>
      <div class="date">
        <div>{{ weekday }}, {{ day }}.{{ month }}.{{ year }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>

:global(html, body) {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
  background-color: #000;
}

.home {
  position: relative;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: 'Zalando Sans Expanded', 'Azeret Mono', monospace;
  color: #ffffff;
  overflow: hidden;
}

.parallax-bg {
  position: absolute;
  top: -5%;
  left: -5%;
  width: 110%;
  height: 110%;
  background-size: cover;
  background-position: center;
  z-index: 0;
  transition: transform 1s cubic-bezier(0.075, 0.82, 0.165, 1);
  will-change: transform;
}

.main-container {
  position: relative;
  z-index: 1;
  padding: 2rem 4rem;
  text-align: center;
}

.time {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 8rem;
  font-weight: bold;
  letter-spacing: 10px;
  line-height: 1;
}

.time-divider {
  color: rgba(255, 255, 255, 0.4);
  margin-bottom: 15px;
}

.date {
  font-size: 1.8rem;
  margin-top: 1rem;
  text-transform: uppercase;
  letter-spacing: 4px;
  opacity: 0.9;
}

.digit-wrapper {
  display: inline-flex;
  justify-content: center;
  width: 1ch;
}
.digit-stack {
  position: relative;
  display: inline-flex;
}
.digit-enter-active,
.digit-leave-active {
  transition: transform 0.4s ease, opacity 0.4s ease;
  transition-delay: var(--digit-delay, 0s);
}
.digit-enter-from { transform: translateY(-100%); opacity: 0; }
.digit-leave-to { transform: translateY(100%); opacity: 0; position: absolute; }
</style>
