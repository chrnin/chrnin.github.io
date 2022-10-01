<template>
  <div id="app">
    <h1>Tonnetz of life</h1>
    Voici un petit morceau de pavage cyclique de <a href="https://fr.wikipedia.org/wiki/Tonnetz">tonnetz</a><br/>
    Vous pouvez choisir d'activer ou désactiver les notes en cliquant dessus<br/>
    Lorsque vous cliquerez sur le bouton [play], chaque note s'activera ou disparaîtra selon une règle simple: <br/> 
    Seules les notes ayant 2 voisines actives s'activent à leur tour, les autres restent désactivée ou se désactivent si besoin.<br/>
    L'application jouera alors les notes activées en arpège à chaque cycle.<p/>

    <span class=red>Si vous souffrez d'épilepsie, n'utilisez pas ce jouet.</span><p/>
    <div style="position: relative; left: -30px" v-for="line, index in canvas" :key="index">
      <Hexagon :playing="playing" :style="'position:absolute;  top:' + index*50 + 'px; left:' + (n*60 + 30*(index%2)) + 'px'" v-for="j, n in line" :value="j" :key="n" :notes="notes"></Hexagon>
    </div>
    
    
    <button style="position: relative; top: 300px" type="button" class="btn btn-primary" @click="enable?next():stop()">{{ enable?"play":"stop" }}</button><br/>
    
  </div>
</template>

<script>
import Hexagon from './components/Hexagon.vue'
import * as Tone from 'tone'
import {MonoSynth} from 'tone'

export default {
  name: 'App',
  components: {Hexagon},
  data() {return {
    notes: [
        false, false, false,
        false, false, false,
        false, false, false,
        false, false, false,
    ],
    enable: true,
    notesNames: ['C4', 'C#4', 'D4', 'D#4', 'E4', 'F4', 'F#4', 'G4', 'G#4', 'A4', 'A#4', 'B4' ],
    canvas: [
      [0 ,7 ,2 ,9 ,4 ,11 ,6 ,1 ,8 ,3 ,10, 5, 0 ,7 ,2 ,9 ,4 ,11 ,6 ,1 ,8 ,3 ,10, 5,0 ,7 ,2 ,9 ,4 ,11 ,6 ,1 ,8 ],
       [3 ,10 ,5 ,0 ,7 ,2 ,9 ,4 ,11,6 ,1 ,8, 3 ,10 ,5 ,0 ,7 ,2 ,9 ,4 ,11,6 ,1 ,8, 3 ,10 ,5 ,0 ,7 ,2 ,9 ,4 ,11],
      [11,6 ,1 ,8 ,3 ,10,5 ,0 ,7 ,2 ,9 ,4, 11,6 ,1 ,8 ,3 ,10,5 ,0 ,7 ,2 ,9 ,4, 11,6 ,1 ,8 ,3 ,10,5 ,0 ,7],
       [2, 9 ,4 ,11 ,6 ,1 ,8 ,3 ,10,5 ,0 ,7, 2, 9 ,4 ,11 ,6 ,1 ,8 ,3 ,10,5 ,0 ,7, 2, 9 ,4 ,11 ,6 ,1 ,8 ,3 ,10],
      [10,5 ,0 ,7 ,2 ,9 ,4 ,11,6 ,1 ,8 ,3, 10,5 ,0 ,7 ,2 ,9 ,4 ,11,6 ,1 ,8 ,3, 10,5 ,0 ,7 ,2 ,9 ,4 ,11,6   ],
    ],
    started: false,
    synth: null,
    playing: null,
    async play(n) {
      if (!this.notes.some(v => v === true)) {
        this.playing = null
        this.enable = true
        return
      }
      if (n == 12) {
        await this.next()
        n=0
      }
      n = n%12
      if (this.notes[n]) {
        this.playing = n
        this.synth.triggerAttackRelease(this.notesNames[n], "32n")
        setTimeout(() => {this.play(n+1)}, 150) 
      } else {
        this.play(n+1)
      }
    },
  }},
  methods: {
    stop() {
      this.notes = [
        false, false, false,
        false, false, false,
        false, false, false,
        false, false, false,
      ]
    },
    async next() {
      if (!this.started) {
        await Tone.start();
        this.synth = new MonoSynth({
                oscillator: {
                  type: "sine"
                },
                envelope: {
                  attack: 0.005,
                  decay: 0.2,
                  sustain:0,
                  releas:0,
                }
              }).toDestination()
        this.started=true
      }
      if (this.enable) {
        this.enable = false
        this.play(0);
        return
      }
      let newNotes = [
        false, false, false,
        false, false, false,
        false, false, false,
        false, false, false,
      ]
      for (let i in newNotes) {
        // console.log("reading for", i)
        let len = this.livingNeighbours(i)
        // console.log("got", len)
        if (len == 2) {
          newNotes[i] = true
        }
      }
      this.notes = [
        false, false, false,
        false, false, false,
        false, false, false,
        false, false, false,
      ]
      this.notes = newNotes
    },
    livingNeighbours(note) {
      let n = 0
      for (let i of [3,4,5,7,8,9]) {
        if (this.notes[(parseInt(note) + parseInt(i))%12]) {
          n++
        }
      }
      return n
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
