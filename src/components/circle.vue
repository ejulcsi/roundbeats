<template>
  <div>
    <button @click="resetBeats">reset beats</button>
    <button @click="resetCircles">reset circles</button>
    <button @click="addCircle">add circle</button>
    <input type="range" id="tempo" name="tempo"
           min="30" max="180" v-model="tempo">
    <label for="tempo">tempo: {{tempo}}</label>
    <div>
      <label for="three">thirds</label>
      <input type="radio" name="layout" id="three" value="3" v-model="layout">
      <label for="four">fourths</label>
      <input type="radio" name="layout" id="four" value="4"  v-model="layout">
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
        layout: 4,
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
        started: false
      }
    },
    computed: {
       angles() {
        return parseInt(this.layout, 10) * 3
       },
      increment() {
         return 6 / parseInt(this.layout, 10)
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
              this.drawFullDot(sketch, beat, this.beatSize * 1.6, '#da3369')
              this.sound.play()
            }
          })
        })
        sketch.angleMode(sketch.DEGREES)

        this.currentAngle = this.started ? this.currentAngle - (this.tempo * this.increment / sketch.frameRate()) : 0
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
          this.drawFullDot(sketch, beat, this.beatSize * (1 - (beat.circle * 0.08)), '#da3369')
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
                this.beats = this.beats.filter(beat => beat.x !== beatFound.x && beat.y !== beatFound.y)
              } else {
                this.beats.push({x: dot.x, y: dot.y, circle: i})
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
</style>