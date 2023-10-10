<script>
export default {
    data() {
        return {
            noteNames: ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'],
            referenceNote: {
                note: 'A4',
                frequency: 440
            }
        };
    },
    methods: {
        getFrequencyFromNoteName(noteName = 'C4') {
            // Defining notes
            console.clear()

            // Getting data from var note
            console.log("full notation: " + noteName)
            const octave = parseInt(noteName.slice(-1));
            console.log("octave: " + octave)
            const noteIndex = this.noteNames.indexOf(noteName.slice(0, -1));
            console.log("note index: " + noteIndex)
            console.log("note name: " + this.noteNames[noteIndex])
            console.log(' ')

            // Getting data from reference note
            console.log("full notation: " + this.referenceNote.note)
            const octaveRef = parseInt(this.referenceNote.note.slice(-1));
            console.log("octave: " + octaveRef)
            const noteIndexRef = this.noteNames.indexOf(this.referenceNote.note.slice(0, -1));
            console.log("note index: " + noteIndexRef)
            console.log("note name: " + this.noteNames[noteIndexRef])
            console.log(' ')
            
            // Half-steps within octave
            const noteDifference = noteIndex - noteIndexRef
            const octaveDifference = octave - octaveRef
            console.log('Note diff: ' + noteDifference)
            console.log('Octave diff: ' + octaveDifference)
            const totalDifference = noteDifference + (octaveDifference * 12)
            console.log('Total diff: ' + totalDifference)
            console.log(' ')

            // Frequency calculation
            const frequency = this.referenceNote.frequency * Math.pow(2, totalDifference/12);
            return parseFloat(frequency)
        }
    }
};
</script>
