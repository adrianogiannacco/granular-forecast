<template>
  <v-container>

    <p class="text-center my-12 text-h4">
      Is it 🌞 today?
    </p>

    <div class="mx-12">
      <highcharts :options="chartOptions"></highcharts>
    </div>
  </v-container>
</template>

<script>
import {Chart} from 'highcharts-vue'

export default {
  components: {
    highcharts: Chart
  },
  data() {
    return {
      chartOptions: {
        title: 'Example',
        series: [{
          data: [4, 2, 1.25, 1, 1.25, 2, 4] // sample data
        }]
      }
    }
  },
  props: {
    msg: String
  }
}
</script>
