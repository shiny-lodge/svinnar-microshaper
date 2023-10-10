<template lang="pug">
PolySynth
</template>

<script>
import PolySynth from './components/PolySynth/PolySynth.vue'

export default {
  name: 'App',
  components: {
    PolySynth
  }
}
</script>

<style lang="sass">
  body
    --margin-increment: 10px

  // Hide arrows
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button
    -webkit-appearance: none
    margin: 0
  input[type=number]
    -moz-appearance: textfield

</style>
