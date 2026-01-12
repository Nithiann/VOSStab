<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  timeFormat: {
    type: String,
    default: '24h'
  }
});

const timeData = ref({ hours: '', minutes: '', ampm: '' });
let timeInterval;

const updateTime = () => {
  const now = new Date();
  const is12h = props.timeFormat === '12h';
  
  if (is12h) {
    let hours = now.getHours();
    const ampm = hours >= 12 ? 'PM' : 'AM';
    hours = hours % 12;
    hours = hours ? hours : 12;
    const minutes = now.getMinutes().toString().padStart(2, '0');
    
    timeData.value = {
      hours: hours.toString(),
      minutes: minutes,
      ampm: ampm
    };
  } else {
    timeData.value = {
      hours: now.getHours().toString().padStart(2, '0'),
      minutes: now.getMinutes().toString().padStart(2, '0'),
      ampm: ''
    };
  }
};

onMounted(() => {
  updateTime();
  timeInterval = setInterval(updateTime, 1000);
});

onUnmounted(() => {
  if (timeInterval) clearInterval(timeInterval);
});
</script>

<template>
  <div class="time-display">
    <span class="hours">{{ timeData.hours }}</span>
    <span class="separator">:</span>
    <span class="minutes">{{ timeData.minutes }}</span>
    <span v-if="timeData.ampm" class="ampm">{{ timeData.ampm }}</span>
  </div>
</template>

<style scoped>
.time-display {
  position: fixed;
  top: 10%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: baseline;
  line-height: 1;
  z-index: 40;
}

.hours {
  font-size: 5.5rem;
  font-weight: 700;
  -webkit-text-stroke: 3px var(--primary-color);
  color: var(--on-background);
}

.separator {
  font-size: 4rem;
  color: var(--on-background);
  opacity: 0.5;
  margin: 0 10px;
  animation: blink 1s infinite;
}

.minutes {
  font-size: 4rem;
  font-weight: 300;
  color: var(--primary-color);
}

.ampm {
  font-size: 2rem;
  font-weight: 500;
  color: var(--on-background);
  margin-left: 15px;
  opacity: 0.8;
}

@keyframes blink {
  50% { opacity: 0.2; }
}
</style>
