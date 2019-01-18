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
          <span class="beat" :style="{backgroundColor: beatColor('main')}"></span>
        </div>
        <div class="input-group">
          <input type="radio" name="role" id="offbeat" value="off"  v-model="role">
          <label for="offbeat">add offbeats</label>
          <span class="beat" :style="{backgroundColor: beatColor('off')}"></span>
        </div>
        <button @click="resetBeats">reset beats</button>
      </div>
    </div>
    <p>click on the dots to add beats</p>
    <button class="big-button" @click="playPause">play / pause</button>
    <button class="big-button" @click="stopPlaying">stop</button>
    <button class="big-button" @mouseup="rotate('right')">rotate right</button>
    <button class="big-button" @mouseup="rotate('left')">rotate left</button>
    <vue-p5 ref="rounds" @setup="setup"
          @draw="draw"
          @mouseclicked="createBeats"
    ></vue-p5>
    <audio ref="hitSound" autobuffer>
      <source src="../assets/hit.mp3" type="audio/mp3">
    </audio>
    <audio ref="offSound" autobuffer>
      <source src="../assets/tamb.mp3" type="audio/mp3">
    </audio>
    <p>inspired by <a href="https://www.youtube.com/watch?v=2UphAzryVpY" target="_blank">John Varney's wheel method</a></p>
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
        role: ROLES.main,
        isRotating: false,
        rotationDir: ''
      }
    },
    computed: {
      isMobile() {
        return window.innerWidth < 480
      },
      canvasSize () {
        return this.isMobile ? window.innerWidth * 0.9 : window.innerHeight * 0.6
//        return initSize < 300 ? 300 : initSize
      },
      angles() {
        return parseInt(this.beatTempo, 10) * 3 * parseInt(this.notes, 10) / 4
      },
      beatColor() {
        return (role) => {
          const colors = {
            main: '#da3369',
            off: '#8c63da'
          }

          return colors[role]
        }
      },
      rotation() {
        const dir = this.rotationDir === 'right' ? -1 : 1
        return 360 / this.angles * dir
      }
    },
    methods: {
      playPause() {
        this.started = !this.started
      },
      stopPlaying() {
        this.started = false
        this.currentAngle = 0
      },
      rotate(dir) {
        this.rotationDir = dir
        this.isRotating = true
      },
      isAreaDetected (pointer, target) {
        const n = 10
        return (pointer.x <= target.x + n && pointer.x >= target.x - n
          && pointer.y <= target.y + n && pointer.y >= target.y - n)
      },
      setup (sketch) {
        sketch.createCanvas(this.canvasSize, this.canvasSize)
        sketch.background('#fafafa')
        sketch.strokeWeight(6)
        this.radius = sketch.width * 0.44
        this.circles.push(this.radius * 2)
        this.center = {
          x: sketch.width / 2,
          y: sketch.height / 2,
        }
        this.createDots(this.radius)
      },
      createDots(radius, update=false) {
        if (update) {
          this.dots = this.dots.map((e, i) => this.getDots([], radius * (1 - (0.25 * i))))
        } else {
          this.dots.push(this.getDots([], radius))
        }
      },
      getDots(array, radius) {
        for (let i = 0; i < this.angles; i++) {
          const angle = i * 2 * Math.PI / this.angles * -1
          const x = this.center.x - (radius * Math.sin(angle))
          const y = this.center.y - (radius * Math.cos(angle))
          array.push({x, y, angle})
        }
        return array
      },
      getBeat(dot, circle) {
        return {
          x: dot.x,
          y: dot.y,
          angle: dot.angle / Math.PI * 180,
          circle,
          size: this.beatSize * (1 - (circle * 0.08)),
          role: this.role
        }
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
              this.drawFullDot(sketch, beat, this.beatColor(beat.role), this.beatSize * 1.6)
              beatSound.play()
            }
          })
        })
        sketch.angleMode(sketch.DEGREES)
        let delta = this.started ? this.tempo * 1.5 / sketch.frameRate() : 0
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
          group.forEach(dot => this.drawFullDot(sketch, dot, '#321fb2'))
        })
      },
      drawFullDot (sketch, dot, color, size) {
        const drawSize = size || dot.size || this.dotSize
        sketch.fill(color)
        sketch.noStroke()
        sketch.ellipse(dot.x, dot.y, drawSize)
      },
      drawBeats (sketch) {
        if (this.isRotating === true) {
          this.beats.forEach(beat => {
            beat = this.rotateBeats(sketch, beat)
            this.drawFullDot(sketch, beat, this.beatColor(beat.role))
          })
          this.isRotating = false
        } else {
          this.beats.forEach(beat => {
            this.drawFullDot(sketch, beat, this.beatColor(beat.role))
          })
        }
      },
      rotateBeats(sketch, beat) {
        sketch.angleMode(sketch.DEGREES)
        const radius = this.radius * (1 - (0.25 * beat.circle))
        const angle = beat.angle + this.rotation

        const x = this.center.x - (radius * sketch.sin(angle))
        const y = this.center.y - (radius * sketch.cos(angle))
        return Object.assign(beat, {x, y, angle})
      },
      createBeats ({mouseX, mouseY}) {
        let mouse = {
          x: mouseX,
          y: mouseY,
        }

        this.dots.forEach((dotGroup, circle) => {
          dotGroup.forEach(dot => {
            if (this.isAreaDetected(mouse, dot)) {
              let beatFound = this.beats.find(item => item.x === dot.x && item.y === dot.y)
              if (beatFound) {
                this.beats = this.beats.filter(beat => beat !== beatFound)
              } else {
                this.beats.push(this.getBeat(dot, circle))
              }
            }
          })
        })
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
    padding: 15px 5px;
    margin: 0 auto 15px;
    display: flex;
    border-radius: 7px;
    border: 1px solid #3f298c;
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
    transition: color .2s;
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
    color: #4f34ad;
    cursor: pointer;
    border-color: #3f298c;
  }

  button:hover {
    color: #3f298c;
  }

  button.big-button {
    font-size: 16px;
    padding: 5px 10px;
  }

  p {
    margin: 0 0 10px;
  }
</style>
