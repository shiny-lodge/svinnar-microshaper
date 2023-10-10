<template>
div
    h3 Low pass filter
    div
      input(type='checkbox', id='lpf-enabled-checkbox', v-model='enabled', @change='handleChange')
      label(for='lpf-enabled-checkbox') Enable
    div
      label(for='lpf-range-input') Frequency
      input(type='range', id='lpf-range-input', v-model='frequency', min='20', max='20000', @change='handleChange')
      span {{ frequency }} Hz
    div
      label(for='lpf-q-input') Q
      input(type='range', id='lpf-q-input', v-model='Q', min='0', max='20', @change='handleChange')
      span {{ Q }}
    EnvelopeControl(:envelope-params='envelopeParams', @change='handleEnvelopeChange')
</template>
  
<script>
import EnvelopeControl from './EnvelopeControl.vue';

export default {
    components: {
        EnvelopeControl,
    },
    props: {
        enabled: Boolean,
        frequency: Number,
        Q: Number,
        envelopeParams: Object,
    },
    methods: {
        handleChange() {
            this.$emit('change');
        },
        handleEnvelopeChange(name) {
            this.$emit('envelope-change', name);
        },
    },
};
</script>