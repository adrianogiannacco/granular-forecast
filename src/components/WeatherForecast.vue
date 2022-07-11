<template>
  <v-container>
    <p class="text-center my-12 text-h4">Is it 🌞 today?</p>

    <v-row class="mb-16">
      <v-col cols="4">
        <CitiesAutocomplete v-model="cities" />
      </v-col>
      <v-col cols="4">
        <v-select
          v-model="criteria"
          :items="criteriaItems"
          label="Select criteria"
        ></v-select>
      </v-col>
      <v-col cols="4">
        <v-menu
          v-model="showDate"
          :close-on-content-click="false"
          :nudge-right="40"
          transition="scale-transition"
          offset-y
          min-width="290px"
        >
          <template v-slot:activator="{ on }">
            <v-text-field
              v-model="date"
              label="Select Date"
              readonly
              v-on="on"
            ></v-text-field>
            <!-- data-vv-as = 'Country of Incorporation'
                    data-vv-scope="form2"
                    data-vv-name="shareholders_${index}_DOB"  -->
          </template>
          <v-date-picker
            v-model="date"
            :min="new Date().toISOString()"
            :max="
              new Date(
                new Date().getTime() + 5 * 24 * 60 * 60 * 1000
              ).toISOString()
            "
            @input="showDate = false"
          ></v-date-picker>
        </v-menu>
      </v-col>
    </v-row>
    <div>
      <highcharts :options="chartOptions"></highcharts>
    </div>
    <div>
      <BestCityToBe :stats="citiesStats" :criteria="criteria" :date="date" />
    </div>
  </v-container>
</template>

<script>
import { Chart } from 'highcharts-vue'
import CitiesAutocomplete from './CitiesAutocomplete.vue'
import BestCityToBe from './BestCityToBe.vue'

export default {
  components: {
    CitiesAutocomplete,
    BestCityToBe,
    highcharts: Chart,
  },
  props: {
    msg: String,
  },
  data() {
    return {
      cities: [],
      citiesStats: [],
      criteria: 'temp',
      criteriaItems: [
        {
          text: 'Temperature',
          value: 'temp',
        },
        {
          text: 'Minimum temperature',
          value: 'temp_min',
        },
        {
          text: 'Maximum temperature',
          value: 'temp_max',
        },
        {
          text: 'Humidity',
          value: 'humidity',
        },
        {
          text: 'Wind speed',
          value: 'wind_speed',
        },
        {
          text: 'Clouds',
          value: 'clouds',
        },
        {
          text: 'Rain',
          value: 'rain',
        },
      ],
      showDate: false,
      date: new Date().toISOString().split('T')[0],
    }
  },
  computed: {
    chartOptions() {
      const options = {
        title: 'Example',
        yAxis: {
          title: {
            text: this.criteriaItems.find((i) => i.value === this.criteria)
              .text,
          },
        },
        xAxis: {
          categories: [],
        },
        series: [],
      }

      if (this.citiesStats.length) {
        options.xAxis.categories = this.citiesStats[0].list
          .filter((i) => i.dt_txt.startsWith(this.date))
          .map((i) => i.dt_txt.split(' ')[1])
      }

      this.citiesStats.forEach((stat) => {
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
        const item = {
          name: stat.city.name,
          data,
        }
        options.series.push(item)
      })

      return options
    },
  },
  watch: {
    cities(newValue, oldValue) {
      if (newValue.length > oldValue.length) {
        const newSelectedCities = newValue.filter((newEl) => {
          return !oldValue.some((oldEl) => {
            return newEl.name === oldEl.name
          })
        })

        newSelectedCities.forEach(async (el) => {
          const stat = await this.fetchCityMeteo(el)
          this.citiesStats.push(stat)
        })
      }
      if (newValue.length < oldValue.length) {
        this.citiesStats = this.citiesStats.filter((stat) => {
          return newValue.some((newEl) => {
            return stat.city.name === newEl.name
          })
        })
      }
    },
  },
  methods: {
    async fetchCityMeteo(city) {
      const res = await fetch(
        `https://corsanywhere.herokuapp.com/https://api.openweathermap.org/data/2.5/forecast?q=${city.name_encoded},${city.country_code}&appid=9ca60a667e3c849942c2b0261f7aa955`
      )
      return res.json()
    },
  },
}
</script>
