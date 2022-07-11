<template>
  <div v-if="bestCity" class="mt-16">
    <p>
      On the selected date, with the given criteria, it seems that the best city
      to be located in is {{ bestCity }}
    </p>
  </div>
</template>

<script>
export default {
  props: {
    stats: {
      type: Array,
      default: () => [],
    },
    criteria: {
      type: String,
      default: '',
    },
    date: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      ideals: {
        temp: 23,
        temp_min: 16,
        temp_max: 25,
        humidity: 20,
        wind_speed: 3,
        clouds: 20,
        rain: 0,
      },
    }
  },
  computed: {
    bestCity() {
      this.stats.map((stat) => {
        return {
          name: stat.city.name,
        }
      })

      const averageItems = []
      this.stats.forEach((stat) => {
        const data = stat.list
          .filter((i) => i.dt_txt.startsWith(this.date))
          .map((i) => {
            if (this.criteria === 'clouds') return i.clouds ? i.clouds.all : 0
            if (this.criteria === 'rain')
              return i.rain && i.rain['3h'] ? i.rain['3h'] : 0
            if (this.criteria === 'wind_speed') return i.wind ? i.wind.speed : 0
            if (
              this.criteria === 'temp' ||
              this.criteria === 'temp_min' ||
              this.criteria === 'temp_max'
            )
              return +(i.main[this.criteria] - 273.15).toFixed(2) // from Kelvin to Celsius
            return i.main[this.criteria]
          })

        const average = data.reduce((a, b) => a + b) / data.length
        const item = {
          name: stat.city.name,
          average,
        }
        averageItems.push(item)
      })

      return averageItems.length
        ? averageItems.reduce((prev, curr) =>
            Math.abs(curr.average - this.ideals[this.criteria]) <
            Math.abs(prev.average - this.ideals[this.criteria])
              ? curr
              : prev
          ).name
        : ''
    },
  },
}
</script>
