<template>
  <div>
    <div id="controls">
      <div class="control-column">
        <button @click="addCircle">add circle</button>
        <button @click="resetCircles">reset circles</button>
      </div>
      <div class="control-column">
        <div class="input-group">
          <input type="radio" name="beatTempo" id="three" value="3" v-model="beatTempo">
          <label for="three">3 beat</label>
          <input type="radio" name="beatTempo" id="four" value="4"  v-model="beatTempo">
          <label for="four">4 beat</label>
        </div>
        <div class="input-group">
          <input type="radio" name="notes" id="fourth" value="4" v-model="notes">
          <label for="fourth">4 notes</label>
          <input type="radio" name="notes" id="eighth" value="8"  v-model="notes">
          <label for="eighth">8 notes</label>
        </div>
        <input type="range" id="tempo" name="tempo"
               min="30" max="180" v-model="tempo">
        <label for="tempo">tempo: {{tempo}}</label>
      </div>
      <div class="control-column">
        <div class="input-group">
          <input type="radio" name="role" id="mainbeat" value="main" v-model="role">
          <label for="mainbeat">add main beats</label>
          <span class="beat" :style="{backgroundColor: getBeatColor('main')}"></span>
        </div>
        <div class="input-group">
          <input type="radio" name="role" id="offbeat" value="off"  v-model="role">
          <label for="offbeat">add offbeats</label>
          <span class="beat" :style="{backgroundColor: getBeatColor('off')}"></span>
        </div>
        <button @click="resetBeats">reset beats</button>
      </div>
    </div>

    <button class="big-button" @click="playPause">play / pause</button>
    <button class="big-button" @click="stop">stop</button>
    <vue-p5 ref="rounds" @setup="setup"
          @draw="draw"
          @mouseclicked="mouseclicked"
    ></vue-p5>
    <audio ref="hitSound" autobuffer>
      <source src="../assets/hit.mp3" type="audio/mp3">
    </audio>
    <audio ref="offSound" autobuffer>
      <source src="../assets/tamb.mp3" type="audio/mp3">
    </audio>
    <p>inspired by <a href="https://www.youtube.com/watch?v=2UphAzryVpY">John Varney's wheel method</a></p>
  </div>
</template>

<script>
  import VueP5 from 'vue-p5'
  /* eslint-disable no-console*/

  let ROLES = {
    main: 'main',
    off: 'off'
  }

  export default {
    name: 'circleCanvas',
    data () {
      return {
        beatTempo: 4,
        notes: 4,
        currentAngle: 0,
        tempo: 60,
        radius: 0,
        center: 0,
        circles: [],
        dots: [],
        dotSize: 5,
        beats: [],
        beatSize: 25,
        sounds: {
          [ROLES.main]: null,
          [ROLES.off]: null
        },
        started: false,
        role: ROLES.main
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
        sketch.createCanvas(400, 400)
        sketch.background('#fafafa')
        sketch.strokeWeight(6)
        this.radius = sketch.width * 0.44
        this.circles.push(this.radius * 2)
        this.center = {
          x: sketch.width / 2,
          y: sketch.height / 2,
        }
        this.createDots(this.radius)
        sketch.rectMode(sketch.CENTER);
      },
      createDots(radius, update=false) {
        if (update) {
          this.dots = this.dots.map((e, i) => {
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
            const beatSound = this.sounds[beat.role]
            if (this.started && this.isAreaDetected(end, beat)) {
              this.drawFullDot(sketch, beat, this.beatSize * 1.6, this.getBeatColor(beat.role))
              beatSound.play()
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
          sketch.ellipse(this.center.x, this.center.y, radius)
        })
      },
      drawDots (sketch) {
        this.dots.forEach(group => {
          group.forEach(dot => this.drawFullDot(sketch, dot, this.dotSize, '#321fb2'))
        })
      },
      drawBeats (sketch) {
        this.beats.forEach(beat => {
          const size = this.beatSize * (1 - (beat.circle * 0.08))
          this.drawFullDot(sketch, beat, size, this.getBeatColor(beat.role))
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
                this.beats.push({x: dot.x, y: dot.y, circle: i, role: this.role})
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
        this.circles.push(radius)
        this.createDots(radius / 2)
      },
    },
    mounted () {
      this.sounds.main = this.$refs.hitSound
      this.sounds.off = this.$refs.offSound
      this.sounds.main.load()
      this.sounds.off.load()
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
  #controls {
    width: 500px;
    margin: 0 auto 30px;
    display: flex;
  }

  .control-column {
    display: flex;
    flex-flow: column wrap;
    justify-content: flex-start;
    align-items: flex-start;
    margin: 0 5px;
  }

  .control-column:first-of-type {
    width: 120px
  }

  .control-column button {
    display: block;
    margin-bottom: 10px;
  }

  .input-group {
    margin-bottom: 5px;
  }

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

  label {
    margin: 0 5px;
  }

  button {
    font-size: 14px;
    border-radius: 4px;
    margin: 0 5px;
  }

  button.big-button {
    font-size: 16px;
    padding: 5px 10px;
    border-color: #3f298c;
    color: #3f298c;
  }
</style>
