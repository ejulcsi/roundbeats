<template>
  <div>
    <button @click="resetBeats">reset beats</button>
    <button @click="resetCircles">reset circles</button>
    <button @click="addCircle">add circle</button>
    <input type="range" id="tempo" name="tempo"
           min="30" max="180" v-model="tempo">
    <label for="tempo">tempo: {{tempo}}</label>
    <div>
      <label for="three">3 beat</label>
      <input type="radio" name="beatTempo" id="three" value="3" v-model="beatTempo">
      <label for="four">4 beat</label>
      <input type="radio" name="beatTempo" id="four" value="4"  v-model="beatTempo">
    </div>
    <div>
      <label for="fourth">4 notes</label>
      <input type="radio" name="notes" id="fourth" value="4" v-model="notes">
      <label for="eighth">8 notes</label>
      <input type="radio" name="notes" id="eighth" value="8"  v-model="notes">
    </div>
    <div>
      <span class="beat" :style="{backgroundColor: getBeatColor('main')}"></span>
      <label for="mainbeat">add main beats</label>
      <input type="radio" name="role" id="mainbeat" value="main" v-model="role">
      <span class="beat" :style="{backgroundColor: getBeatColor('off')}"></span>
      <label for="offbeat">add offbeats</label>
      <input type="radio" name="role" id="offbeat" value="off"  v-model="role">
    </div>
    <button @click="playPause">play / pause</button>
    <button @click="stop">stop</button>
    <vue-p5 ref="rounds" @setup="setup"
            @draw="draw"
            @mouseclicked="mouseclicked"
    ></vue-p5>
    <audio ref="sound" controls="" autobuffer>
      <source src="../assets/hit.mp3" type="audio/mp3">
    </audio>
  </div>
</template>

<script>
  import VueP5 from 'vue-p5'
  /* eslint-disable no-console*/

  export default {
    name: 'circleCanvas',
    data () {
      return {
        beatTempo: 4,
        notes: 4,
        currentAngle: 0,
        tempo: 60,
        width: 0,
        height: 0,
        radius: 0,
        center: 0,
        circles: [],
        dots: [],
        dotSize: 5,
        beats: [],
        beatSize: 25,
        sound: null,
        started: false,
        role: 'main'
      }
    },
    computed: {
      angles() {
        return parseInt(this.beatTempo, 10) * 3 * parseInt(this.notes, 10) / 4
      },
      increment() {
        const beat = parseInt(this.beatTempo, 10) % 4 === 0 ? 4 : parseInt(this.beatTempo, 10)
        return 6 / beat
      },
      getBeatColor() {
        return (role) => {
          const colors = {
            main: '#da3369',
            off: '#8c63da'
          }

          return colors[role]
        }
      }
    },
    methods: {
      playPause() {
        this.started = !this.started
      },
      stop() {
        this.started = false
        this.currentAngle = 0
      },
      isAreaDetected (pointer, target) {
        const n = 10
        return (pointer.x <= target.x + n && pointer.x >= target.x - n
          && pointer.y <= target.y + n && pointer.y >= target.y - n)
      },
      setup (sketch) {
        sketch.createCanvas(500, 500)
        sketch.background('#fafafa')
        sketch.strokeWeight(6)
        this.width = sketch.width
        this.height = sketch.height
        this.radius = this.width / 3
        this.circles.push(this.radius * 2)
        this.center = {
          x: this.width / 2,
          y: this.height / 2,
        }
        this.createDots(this.radius)
        sketch.rectMode(sketch.CENTER);
      },
      createDots(radius, update=false) {
        if (update) {
          this.dots = this.dots.map((e, i) => {
//            return this.getDots([], radius * (0.66 ** i) )
            return this.getDots([], radius * (1 - (0.25 * i)) )
          })
        } else {
          this.dots.push([])
          this.getDots(this.dots[this.dots.length - 1], radius)
        }
      },
      getDots(array, radius) {
        for (let i = 0; i < this.angles; i++) {
          const angle = i * 2 * Math.PI / this.angles
          const x = this.center.x - (radius * Math.sin(angle * -1))
          const y = this.center.y - (radius * Math.cos(angle * -1))
          array.push({x, y})
        }
        return array
      },
      draw (sketch) {
        sketch.clear()

        this.drawCircles(sketch)
        this.drawFinger(sketch)
        this.drawDots(sketch)
        this.drawBeats(sketch)
      },
      drawFinger (sketch) {
        this.circles.forEach(radius => {
          const end = {
            x: this.center.x - (radius / 2 * sketch.sin(this.currentAngle)),
            y: this.center.y - (radius / 2 * sketch.cos(this.currentAngle)),
          }
          sketch.stroke('#6a44eb')
          sketch.line(this.center.x, this.center.y, end.x, end.y)

          this.beats.forEach(beat => {
            if (this.started && this.isAreaDetected(end, beat)) {
              this.drawFullDot(sketch, beat, this.beatSize * 1.6, beat.color)
              this.sound.play()
            }
          })
        })
        sketch.angleMode(sketch.DEGREES)
        let delta = this.started ? this.tempo * this.increment / sketch.frameRate() : 0
        this.currentAngle -= delta
      },
      drawCircles (sketch) {
        this.circles.forEach(radius => {
          sketch.stroke('#34ebac')
          sketch.noFill()
          sketch.ellipse(this.width / 2, this.height / 2, radius)
        })
      },
      drawDots (sketch) {
        this.dots.forEach(group => {
          group.forEach(dot => this.drawFullDot(sketch, dot, this.dotSize, '#321fb2'))
        })
      },
      drawBeats (sketch) {
        this.beats.forEach(beat => {
          this.drawFullDot(sketch, beat, this.beatSize * (1 - (beat.circle * 0.08)), beat.color)
        })
      },
      mouseclicked ({mouseX, mouseY}) {
        let mouse = {
          x: mouseX,
          y: mouseY,
        }

        this.dots.forEach((dotGroup, i) => {
          dotGroup.forEach(dot => {
            if (this.isAreaDetected(mouse, dot)) {
              let beatFound = this.beats.find(item => item.x === dot.x && item.y === dot.y)
              if (beatFound) {
                this.beats = this.beats.filter(beat => beat !== beatFound)
              } else {
                this.beats.push({x: dot.x, y: dot.y, circle: i, color: this.getBeatColor(this.role)})
              }
            }
          })
        })
      },
      drawFullDot (sketch, dot, size, color) {
        sketch.fill(color)
        sketch.noStroke()
        sketch.ellipse(dot.x, dot.y, size)
      },
      resetBeats () {
        this.beats = []
      },
      resetCircles() {
        this.circles = [this.circles[0]]
        this.dots = [this.dots[0]]
        this.resetBeats()
      },
      addCircle () {
        let radius = this.circles[0] * (1 - (0.25 * this.circles.length))
//        let radius = this.circles[this.circles.length - 1] * 0.66
        this.circles.push(radius)
        this.createDots(radius / 2)
      },
    },
    mounted () {
      this.sound = this.$el.querySelector('audio')
      this.sound.load()
    },
    components: {
      VueP5,
    },
    watch: {
      angles() {
        this.createDots(this.radius, true)
      },
    }
  }
</script>

<style>
  audio {
    display: none;
  }

  .beat {
    display: inline-block;
    width: 14px;
    height: 14px;
    margin: 0 10px;
    border-radius: 100%;
  }
</style>
