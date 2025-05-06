/* App.vue */
<script>
  //import * as CanvasJS from '@canvasjs/charts';
  export default {
    data() {
      return {
        options: {
          theme: "light2",
          exportEnabled: true,
          title: {
            text: "Medicion del Espectometro UV VIS expectrofotometro"
          },
          axisY: {
            title: "Fertitlity Rate"
          },
          toolTip: {
            shared: true
          },
          legend: {
            cursor: "pointer",
            itemclick: function (e) {
                if (typeof (e.dataSeries.visible) === "undefined" || e.dataSeries.visible) {
                    e.dataSeries.visible = false;
                } else {
                    e.dataSeries.visible = true;
                }
                e.chart.render();
            }
          },
          data: [{
            type: "line",
            name: "Germany",
            showInLegend: true,
            color: "#F7C705",
            toolTipContent: "<img src=\"https://canvasjs.com/wp-content/uploads/images/gallery/javascript-column-bar-charts/germany.png\" style=\"height:11px;width:18px;\"> {name}: {y}",
            dataPoints: [
              { label: "2014", y: 1.47 },
              { label: "2015", y: 1.5 },
              { label: "2016", y: 1.6 },
              { label: "2017", y: 1.57 },
              { label: "2018", y: 1.57 },
              { label: "2019", y: 1.54 }
            ]
          }, {
            type: "line",
            name: "United Kingdom",
            showInLegend: true,
            color: "#012066",
            toolTipContent: "<img src=\"https://canvasjs.com/wp-content/uploads/images/gallery/javascript-column-bar-charts/uk.png\" style=\"height:11px;width:18px;\"> {name}: {y}",
            dataPoints: [
              { label: "2014", y: 1.81 },
              { label: "2015", y: 1.80 },
              { label: "2016", y: 1.79 },
              { label: "2017", y: 1.74 },
              { label: "2018", y: 1.68 },
              { label: "2019", y: 1.65 }
            ]
          }]
        },
        styleOptions: {
          width: "100%",
          height: "360px"
        }
      }
    }
  }
</script>
<template>
  <CanvasJSChart :options="options" :styles="styleOptions"/>
</template>                              
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
