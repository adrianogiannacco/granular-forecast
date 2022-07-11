<template>
  <v-autocomplete
    v-model="model"
    :items="items"
    chips
    clearable
    deletable-chips
    multiple
    small-chips
    hide-no-data
    outline
    label="Select Capitals"
    @input="handleInput"
  ></v-autocomplete>
</template>

<script>
export default {
  props: {
    value: {
      type: null,
      default: null,
    },
  },
  data: () => ({
    items: [],
  }),
  computed: {
    model: {
      get() {
        return this.value
      },
      set(newValue) {
        return newValue
      },
    },
  },
  async mounted() {
    const res = await fetch(
      'https://countriesnow.space/api/v0.1/countries/capital'
    )
    const data = await res.json()
    this.items = data.data
      .filter((el) => el.capital)
      .map((el) => {
        const capital = el.capital.trim()
        return {
          text: capital,
          value: {
            name: capital,
            name_encoded: encodeURI(capital),
            country_code: el.iso2,
          },
        }
      })
      .sort((a, b) => a.text.localeCompare(b.text))
  },
  methods: {
    handleInput(ev) {
      this.$emit('input', ev)
    },
  },
}
</script>
