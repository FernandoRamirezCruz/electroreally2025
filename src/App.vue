<script>
import VueApexCharts from "vue3-apexcharts";
import axios from "axios";

export default {
  components: {
    apexchart: VueApexCharts,
  },
  data() {
    return {
      // Config ESP32
      esp32Ip: "192.168.43.16",

      // Último valor recibido de la ESP32
      newFly: 0,

      // Datos completos
      allFliesData: [],
      allSpidersData: [],
      allXValues: [],

      // Vector cíclico de 100 números (valores proporcionados)
      spiderPattern: [
        447, 448, 447, 446, 445, 448, 445, 447, 447, 448, 447, 447, 445, 446, 447, 447, 446, 448, 446, 446,
        448, 445, 447, 447, 448, 446, 447, 448, 448, 446, 446, 446, 448, 447, 435, 437, 435, 435, 435, 437,
        436, 435, 435, 437, 438, 434, 436, 435, 437, 434, 435, 434, 436, 436, 435, 433, 434, 435, 435, 437,
        435, 435, 433, 434, 435, 433, 435, 435
      ],
      spiderIndex: 0, // Índice actual del vector

      // Series y opciones del chart
      series: [
        { name: "Agua(datos recabados)", data: [] },
        { name: "Actual", data: [] },
      ],
      chartOptions: {
        chart: {
          id: "chart2",
          type: "line",
          toolbar: { show: false },
          animations: {
            enabled: false,
            easing: "linear",
            dynamicAnimation: { speed: 300 },
          },
        },
        colors: ["#008FFB", "#00E396"],
        stroke: { width: [2, 2], curve: "straight" },
        yaxis: {
          min: 0,
          max: 750,
          tickAmount: Math.floor(10),
          labels: { formatter: (v) => Math.round(v) },
          title: { text: "Sensor UV" },
        },
        xaxis: {
          type: "numeric",
          labels: { show: true },
          axisTicks: { show: true },
          axisBorder: { show: true },
        },
        markers: { size: 0 },
        tooltip: { x: { show: true } },
      },

      // Control de simulación y paginación
      isRunning: true,
      intervalId: null,
      pageStart: 0,
      pageSize: 10,
    };
  },
  mounted() {
    // Cada 100 ms: leer ESP32 y agregar datos al gráfico
    this.intervalId = setInterval(async () => {
      await this.obtenerMensaje();
      this.addDataPoint();
    }, 100);
  },
  methods: {
    // 1) Obtener nuevo fly desde ESP32
    async obtenerMensaje() {
      try {
        const resp = await axios.get(`http://${this.esp32Ip}/getMensaje`);
        const msg = resp.data.mensaje;
        const num = Number(msg);
        if (!isNaN(num)) this.newFly = num;
      } catch (e) {
        console.error("Error al obtener mensaje ESP32:", e);
      }
    },

    // 2) Empujar datos y refrescar gráfico
    addDataPoint() {
      if (!this.isRunning) return;

      // Obtener el número cíclico del vector
      const newSpider = this.spiderPattern[this.spiderIndex];
      this.spiderIndex = (this.spiderIndex + 1) % this.spiderPattern.length; // Reiniciar índice si llega al final

      const newIndex = this.allXValues.length + 1;
      this.allFliesData.push(this.newFly);
      this.allSpidersData.push(newSpider);

      this.allXValues.push(newIndex);

      // Mantén ventana deslizante y paginación
      const start = Math.max(0, this.allXValues.length - this.pageSize);
      this.pageStart = start;
      this.updateChart();
    },

    // 3) Actualizar series y categorías
    updateChart() {
      const start = this.pageStart;
      const end = start + this.pageSize;

      this.series = [
        { name: "Datos Actuales", data: this.allFliesData.slice(start, end) },
        { name: "Aguas destilada", data: this.allSpidersData.slice(start, end) },
      ];
      this.chartOptions = {
        ...this.chartOptions,
        xaxis: {
          ...this.chartOptions.xaxis,
          categories: this.allXValues.slice(start, end),
        },
      };
    },

    // Pausar / reanudar
    toggleSimulation() {
      this.isRunning = !this.isRunning;
      if (this.isRunning) {
        // Al reanudar, mostrar últimos datos
        this.pageStart = Math.max(0, this.allXValues.length - this.pageSize);
        this.updateChart();
      }
    },

    // Navegación manual
    prevPage() {
      if (this.pageStart - this.pageSize >= 0) {
        this.pageStart -= this.pageSize;
        this.updateChart();
      }
    },
    nextPage() {
      if (this.pageStart + this.pageSize < this.allXValues.length) {
        this.pageStart += this.pageSize;
        this.updateChart();
      }
    },
  },
};
</script>

<template>
  <div id="wrapper">
    <h2 id="chart-title">Graficas de deteccion de Rayos UV</h2>
    <div id="chart-container">
      <apexchart
        type="line"
        :height="'100%'"
        :options="chartOptions"
        :series="series"
      />
    </div>
    <div id="controls">
      <button @click="toggleSimulation">
        {{ isRunning ? "Pausar" : "Reanudar" }}
      </button>
      <div v-if="!isRunning" class="nav-buttons">
        <button @click="prevPage">← Anterior</button>
        <button @click="nextPage">Siguiente →</button>
      </div>
    </div>
  </div>
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  font-family: Arial, sans-serif;
  background-color: #fff;
}
#wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: start;
  height: 100vh;
  width: 100vw;
  padding: 20px;
  box-sizing: border-box;
}
#chart-title {
  margin: 10px 0 20px;
  font-size: 1.8rem;
  font-weight: bold;
  text-align: center;
}
#chart-container {
  flex: 1;
  width: 100%;
  height: calc(100vh - 160px);
  max-width: 1400px;
}
#controls {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
#controls button {
  margin: 5px;
  padding: 10px 20px;
  font-size: 1rem;
  cursor: pointer;
}
.nav-buttons {
  display: flex;
  gap: 10px;
}
</style>
