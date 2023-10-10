<template>
    <div style="background: steelblue" class="polysynth__param-group">
      <h3 class="polysynth__param-group-name">ADSR</h3>
      <ul class="polysynth__param-list">
        <li v-for="(param, name) in envelopeParams" :key="name" class="polysynth__param-control">
          <label>{{ param.label }}</label>
          <div class="polysynth__param-input-group">
            <input type="range" :id="name" :value="param.value" min="0" max="1" step="0.1" @change="handleChange(name)">
            <span>{{ param.value }}</span>
          </div>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      envelopeParams: Object,
    },
    methods: {
      handleChange(name) {
        this.$emit('change', name);
      },
    },
  };
  </script>