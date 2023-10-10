<template lang="pug">
.polysynth
    h2.polysynth__title MicroShaper
    .polysynth__param-group
        h3.polysynth__param-group-name Volume
        input(
            v-model="volume" 
            type="range" 
            min="0" 
            max="1" 
            step="0.1" 
            @change="updateVolume"
        )
    .polysynth__param-group
        h3.polysynth__param-group-name Oscillator waveform
        select.polysynth__param-select(v-model="waveform" @change="updateSynth")
            option(
                v-for="waveformOption in waveformOptions" 
                :key="waveformOption.value" 
                :value="waveformOption.value"
            ) {{ waveformOption.label }}
    .polysynth__param-group
        h3.polysynth__param-group-name ADSR
        ul.polysynth__param-list
            li.polysynth__param-control(v-for="envelopeParam in envelopeParams" :key="envelopeParam.name")
                label {{ envelopeParams[envelopeParam.name].label }}
                .polysynth__param-input-group    
                    input(
                        v-model="envelopeParams[envelopeParam.name].value" 
                        :id="envelopeParam.name" 
                        type="range" 
                        min="0" 
                        max="1" 
                        step="0.1" 
                        @change="updateEnvelopeParams($event, envelopeParam.name)"
                    )
                    span {{ envelopeParams[envelopeParam.name].value }}
    .polysynth__param-group
        h3.polysynth__param-group-name Low pass filter
        label Frequency
        .polysynth__param-input-group
            input(type="range" min="20" max="20000" id="lpf-range-input" v-model="lowPassFilter.frequency" @change="updateLPFrequency")
            label {{ lowPassFilter.frequency }} Hz
        label Q
        .polysynth__param-input-group
            input(type="range" min="0" max="20" id="lpf-q-input" v-model="lowPassFilter.Q" @change="updateLPQ")
            label {{ lowPassFilter.Q }}
        .polysynth__param-group
            h3.polysynth__param-group-name Low pass filter ADSR
            ul.polysynth__param-list
                li.polysynth__param-control(v-for="envelopeParam in envelopeParams" :key="envelopeParam.name")
                    label {{ envelopeParams[envelopeParam.name].label }}
                    .polysynth__param-input-group    
                        input(
                            v-model="lowPassFilter.envelopeParams[envelopeParam.name].value" 
                            :id="envelopeParam.name" 
                            type="range" 
                            min="0" 
                            max="1" 
                            step="0.1" 
                            @change="updateLPEnvelopeParams($event, envelopeParam.name)"
                        )
                        span {{ lowPassFilter.envelopeParams[envelopeParam.name].value }}
    EnvelopeControl
    FilterModule(
        :id="lowPassFilter.id"
        :enabled="lowPassFilter.enabled"
        :frequency="lowPassFilter.frequency"
        @changeEnabled="handleEnabledChange"
        @changeFrequency="handleFrequencyChange"
    )
    .polysynth__test-tone
        button(@mousedown="playNote(testNote + testOctave)" @mouseup="releaseNote") Test {{ testNote + testOctave }}
        select(v-model="testNote")
            option(
                v-for="noteName in noteNames" 
                :key="noteName" 
                :value="noteName"
            ) {{ noteName }}
        input(type="number" id="octave-input" min="1" max="7" v-model="testOctave")
    
</template>

<script>
import NoteMixin from '../../mixins/NoteMixin.vue'
import EnvelopeControl from '../Modules/EnvelopeControl.vue'
import FilterModule from '../Modules/FilterModule.vue'

export default {
    data() {
        return {
            audioContext: null,
            oscillator: null,
            gainNode: null,
            testNote: 'C',
            testOctave: 4,
            waveform: "sawtooth",
            waveformOptions: [
                {
                    label: 'Sine',
                    value: 'sine'
                },
                {
                    label: 'Sawtooth',
                    value: 'sawtooth'
                },
                {
                    label: 'Triangle',
                    value: 'triangle'
                },
                {
                    label: 'Square',
                    value: 'square'
                }
            ],
            envelopeParams: {
                attackTime: {
                    name: 'attackTime',
                    label: 'Attack',
                    value: 0.1
                },
                decayTime: {
                    name: 'decayTime',
                    label: 'Decay',
                    value: 0.2
                },
                sustainLevel: {
                    name: 'sustainLevel',
                    label: 'Sustain',
                    value: 0.5
                },
                releaseTime: {
                    name: 'releaseTime',
                    label: 'Release',
                    value: 0.5
                }
            },
            lowPassFilter: {
                id: 'lpf',
                enabled: false,
                frequency: 5000,
                Q: 1,
                envelopeParams: {
                    attackTime: {
                        name: 'attackTime',
                        label: 'Attack',
                        value: 0.1
                    },
                    decayTime: {
                        name: 'decayTime',
                        label: 'Decay',
                        value: 0.2
                    },
                    sustainLevel: {
                        name: 'sustainLevel',
                        label: 'Sustain',
                        value: 0.5
                    },
                    releaseTime: {
                        name: 'releaseTime',
                        label: 'Release',
                        value: 0.5
                    }
                }
            },
            volume: 0.5
        };
    },
    mixins: [NoteMixin],
    components: {EnvelopeControl, FilterModule},
    mounted() {
        this.audioContext = new AudioContext();
        if (this.audioContext.state === 'running') {
            this.setupSynth();
        } else {
            document.addEventListener('click', () => {
                this.audioContext.resume().then(() => {
                    this.setupSynth();
                });
            }, { once: true });
        }
    },
    methods: {
        // Filter module handlers
        handleEnabledChange() {
            this.lowPassFilter.enabled = !this.lowPassFilter.enabled
            this.setupSynth()
        },
        handleFrequencyChange(frequency) {
            this.lowPassFilter.frequency = frequency
            if (this.lowPassFilter.enabled) {
                this.lowPassFilter.filter.frequency.setValueAtTime(this.lowPassFilter.frequency, this.audioContext.currentTime);
            }
        },
        updateEnvelopeParams(event, paramName) {
            this.envelopeParams[paramName].value = parseFloat(event.target.value);
        },
        updateSynth() {
            this.setupSynth()
        },
        updateLPFrequency() {
            if (this.lowPassFilter.enabled) {
                this.lowPassFilter.filter.frequency.setValueAtTime(this.lowPassFilter.frequency, this.audioContext.currentTime);
            }
        },
        updateLPEnvelopeParams(event, paramName) {
            this.lowPassFilter.envelopeParams[paramName].value = parseFloat(event.target.value);
        },
        updateLPQ() {
            if (this.lowPassFilter.enabled) {
                this.lowPassFilter.filter.Q.setValueAtTime(this.lowPassFilter.Q, this.audioContext.currentTime);
            }
        },
        setupLPEnvelope() {
            const now = this.audioContext.currentTime;
            const envelope = this.lowPassFilter.envelopeParams;
            const filter = this.lowPassFilter.filter;
            filter.frequency.setValueAtTime(this.lowPassFilter.frequency, now);
            filter.Q.setValueAtTime(this.lowPassFilter.Q, now);
            filter.frequency.cancelScheduledValues(now);
            filter.frequency.setValueAtTime(this.lowPassFilter.frequency, now);
            filter.frequency.linearRampToValueAtTime(this.lowPassFilter.frequency, now + envelope.attackTime.value);
            filter.frequency.exponentialRampToValueAtTime(this.lowPassFilter.frequency * envelope.sustainLevel.value, now + envelope.attackTime.value + envelope.decayTime.value);
        },
        setupSynth() {
            this.oscillator = this.audioContext.createOscillator();
            this.gainNode = this.audioContext.createGain();

            var nodeQueue = []
            var currentNode = 0
            if (this.lowPassFilter.enabled) {
                this.lowPassFilter.filter = this.audioContext.createBiquadFilter();
                nodeQueue.push(this.lowPassFilter.filter)
            }

            nodeQueue.push(this.gainNode)

            this.oscillator.connect(nodeQueue[currentNode]);

            if (this.lowPassFilter.enabled) {
                this.lowPassFilter.filter.type = 'lowpass';
                this.setupLPEnvelope();
                this.lowPassFilter.filter.frequency.setValueAtTime(this.lowPassFilter.frequency, this.audioContext.currentTime);
                this.lowPassFilter.filter.Q.setValueAtTime(this.lowPassFilter.Q, this.audioContext.currentTime);
                currentNode++
                this.lowPassFilter.filter.connect(nodeQueue[currentNode]);
            }

            this.gainNode.connect(this.audioContext.destination);

            this.oscillator.type = this.waveform;

            this.gainNode.gain.setValueAtTime(0, this.audioContext.currentTime);
            this.oscillator.start();
        },
        playNote(note) {
            if (this.gainNode) {
                const now = this.audioContext.currentTime;
                const frequency = this.getFrequencyFromNoteName(note);
                this.oscillator.frequency.setValueAtTime(frequency, now);
                this.gainNode.gain.cancelScheduledValues(now);
                this.gainNode.gain.setValueAtTime(0, now);
                this.gainNode.gain.linearRampToValueAtTime(1 * this.volume, now + this.envelopeParams.attackTime.value);
                if (this.volume != 0) {
                    this.gainNode.gain.exponentialRampToValueAtTime(
                        this.envelopeParams.sustainLevel.value * this.volume,
                        now + this.envelopeParams.attackTime.value + this.envelopeParams.decayTime.value
                    );
                }
            }
        },
        releaseNote() {
            if (this.gainNode) {
                const now = this.audioContext.currentTime;

                this.gainNode.gain.cancelScheduledValues(now);
                this.gainNode.gain.setValueAtTime(this.gainNode.gain.value, now);
                this.gainNode.gain.linearRampToValueAtTime(0, now + this.envelopeParams.releaseTime.value);
            }
        },
    },
};
</script>

<style scoped lang="sass">
.polysynth
    &__param-group
    &__param-group-name
    &__param-select
        display: block
        margin-top: calc(var(--margin-increment) * 2)
    &__param-list
        padding: 0
    &__param-control
        list-style: none
        label
            display: block
    &__param-input-group
        display: flex
        align-items: center
        flex-direction: row
    &__test-tone
        margin-top: calc(var(--margin-increment) * 2)
        display: flex
        height: fit-content
        button
            margin-right: calc(var(--margin-increment) * 2)

</style>
