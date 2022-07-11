<template>
  <v-app>
    <v-app-bar color="secondary" clipped-left fixed app dark flat>
      <v-toolbar-title>
        <img alt="Granular logo" src="./assets/granular-logo.png" />
      </v-toolbar-title>
    </v-app-bar>

    <v-main>
      <WeatherForecast />
    </v-main>

    <v-footer app>
      <div class="text-center" style="width: 100%">
        Granular Energy &copy; {{ new Date().getFullYear() }}
      </div>
    </v-footer>
  </v-app>
</template>

<script>
import WeatherForecast from './components/WeatherForecast.vue'

export default {
  name: 'App',
  components: {
    WeatherForecast,
  },
}
</script>

<style lang="scss">
.v-app-bar {
  img {
    max-height: 2.5rem;
    display: block;
  }
}
</style>
