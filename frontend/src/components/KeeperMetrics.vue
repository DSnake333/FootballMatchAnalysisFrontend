<template>
  <div ref="plotlyChart" style="width: 100%; height: 100%;"></div>
</template>

<script>
import Plotly from "plotly.js-dist";

export default {
  props: {
    plotlyData: {
      type: Object,
      required: true,
    },
  },
  mounted() {
    // Initial chart rendering
    if (this.plotlyData) {
      Plotly.newPlot(this.$refs.plotlyChart, this.plotlyData.data, this.plotlyData.layout || {});
    }
  },
  watch: {
    plotlyData: {
      handler(newData, oldData) {
        // Check if the data has actually changed before re-rendering
        if (JSON.stringify(newData) !== JSON.stringify(oldData)) {
          // Use Plotly.react for efficient updates
          Plotly.react(this.$refs.plotlyChart, newData.data, newData.layout || {});
        }
      },
      deep: true,
    },
  },
};
</script>

<style>
/* Add any required styling here */
</style>
