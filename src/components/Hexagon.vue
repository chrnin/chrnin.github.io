<template>
  <div>
    <button type="button" style="width: 55px !important;" :class="buttonClass" @click="toggle()">{{ notesNames[value] }}</button>
  </div>
</template>

<script>
export default {
  name: 'HexaGon',
  props: ["value", "playing"],
  data() {
    return {
      notesNames: ['C4', 'C#4', 'D4', 'D#4', 'E4', 'F4', 'F#4', 'G4', 'G#4', 'A4', 'A#4', 'B4' ],
    }
  },
  computed: {
    buttonClass() {
      if (this.playing == this.value) {
        return "btn-circle btn btn-primary"
      }
      if (this.note) {
        return "btn-circle btn btn-success"
      } else {
        return "btn-circle btn btn-secondary"
      }
    },
    note: {
      get() {return this.$parent.notes[this.value]},
      set(val) {
        this.$set(this.$parent.notes, this.value, val)
      },
    } 
  },
  methods: {
    toggle() {
      if (this.$parent.enable) {
        this.note = !this.note
      }
    },
  },
}
</script>

<style scoped>
  .btn-circle {
    width: 50px;
    height: 50px;
    padding: 1px 6px;
    border-radius: 35px;
    font-size: 24px;
    line-height: 1.33;
  }
</style>