<template>
  <div class="card-container">
    <div class="card">
      <div class="border"></div>
      <div class="content">
        <h1>Sine Graph</h1>
        <div class="graph-container" @wheel.prevent="handleScroll">
          <canvas ref="sineGraphCanvas"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import Chart from 'chart.js/auto';

export default {
  name: 'GraphComponent',
  setup() {
    const sineGraphCanvas = ref(null);
    let chartInstance;
    let currentScrollX = 0;
    const pointsToDisplay = 1000; 

    onMounted(() => {
      chartInstance = drawSineGraph();
      adjustCanvasSize();
      window.addEventListener('resize', adjustCanvasSize);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('resize', adjustCanvasSize);
      if (chartInstance) {
        chartInstance.destroy();
      }
    });

    const drawSineGraph = () => {
      const ctx = sineGraphCanvas.value.getContext('2d');
      const data = {
        labels: Array.from({ length: 46080 }, (_, i) => i),
        datasets: [
          {
            label: 'Sine Graph',
            data: Array.from({ length: 46080 }, (_, i) => Math.sin((i * Math.PI) / 180)),
            borderColor: 'blue',
            borderWidth: 0.5,
            fill: false,
          },
        ],
      };
      const options = {
        scales: {
          x: {
            type: 'linear',
            position: 'bottom',
            min: 0,
            max: pointsToDisplay,
            ticks: {
              maxRotation: 0
            },
          },
          y: {
            type: 'linear',
            position: 'left',
            min: -1.5,
            max: 1.5,
          },
        },
      };
      return new Chart(ctx, {
        type: 'line',
        data,
        options,
      });
    };

    const adjustCanvasSize = () => {
      const cardRect = sineGraphCanvas.value.getBoundingClientRect();
      sineGraphCanvas.value.width = cardRect.width;
      sineGraphCanvas.value.height = cardRect.height;
      if (chartInstance) {
        chartInstance.resize();
      }
    };

    const handleScroll = (event) => {
      const delta = Math.max(-1, Math.min(1, (event.deltaY || -event.detail)));
      console.log("delta: "+delta);
      if(event.deltaY) console.log("deltaY: "+event.deltaY);
      if(event.detail) console.log("detail: "+event.detail);
      currentScrollX += delta * 50; // speed
      const maxScrollX = 46080 - pointsToDisplay;
      currentScrollX = Math.max(0, Math.min(currentScrollX, maxScrollX));

      if (chartInstance) {
        chartInstance.options.scales.x.min = currentScrollX;
        chartInstance.options.scales.x.max = currentScrollX + pointsToDisplay;
        chartInstance.update();
      }
      event.preventDefault();
    };

    return { sineGraphCanvas, drawSineGraph, handleScroll };
  },
};
</script>

<style scoped>
.card-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 81vh;
}

.card {
  position: relative;
  width: 50vw;
  height: 71vh;
  background-color: #ffffff;
  border-radius: 10px;
  overflow: hidden;
}

.border {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border: 2px solid #ccc; 
  pointer-events: none;
}

.content {
  padding: 20px;
}

.graph-container {
  overflow: hidden; 
  position: relative;
  height: 100%;
}

h1 {
  margin-bottom: 10px;
}

p {
  margin: 0;
}
</style>