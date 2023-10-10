
<script>

export default {
    name: 'FilterModule',
    data() {
        return {
            isHidden: false
        }
    },
    props: {
        enabled: {
            type: Boolean,
            required: true
        },
        id: {
            type: String,
            required: true
        },
        frequency: {
            type: Number,
            required: true
        }
    },
    methods: {
        toggleParameterVisibility () {
            this.isHidden = !this.isHidden
        },
        handleChangeEnabledStatus () {
            this.$emit('change-enabled')
        },
        handleChangeFrequency (event) {
            this.$emit('change-frequency', event.target.value)
        }
    }
};

</script>


<template lang="pug">
.filter-module
    .filter-module__header
        span Low pass filter
        div
            button(@click='handleChangeEnabledStatus') {{ enabled ? 'Disable' : 'Enable' }}
            button(@click='toggleParameterVisibility') Hide
    .filter-module__parameters(:class="{ hidden: isHidden }")
        .filter-module__parameter-block
            label.filter-module__parameter-label Frequency
            input.filter-module__range(
                type='range' 
                min='20'
                max='20000'
                :value='frequency'
                :id='id + "-frequency-range"'
                @input='handleChangeFrequency'
                orient='vertical'
            )
            .filter-module__frequency-input
                input(
                    type='number' 
                    min='20'
                    max='20000'
                    :value='frequency'
                    :id='id + "-frequency-input"'
                    @change='handleChangeFrequency'
                )
                span Hz
</template>

<style scoped lang="sass">
.filter-module
    background: wheat
    padding: 20px 30px
    &__header
        display: flex
        flex-direction: row
        justify-content: space-between
        align-items: center
        margin-bottom: 30px
        span
            font-size: 20px
    &__parameters
        &.hidden
            display: none
    &__frequency-input
        margin-left: auto
        input
            width: 40px
            text-align: center
            border: none
            outline: none
            border-bottom: 1px solid black
            background: transparent
    &__parameter-label
        margin-bottom: 10px
    &__range
        appearance: slider-vertical
        width: 20px
        margin: 0
        margin-bottom: 10px
    &__parameter-block
        width: fit-content
        display: flex
        flex-direction: column
        align-items: center
</style>