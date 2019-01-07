<template>
  <div>
    <button @click="resetBeats">reset</button>
    <vue-p5 @setup="setup"
            @draw="draw"
            @mouseclicked="mouseclicked"
    ></vue-p5>
    <audio ref="sound" src="../assets/hit.mp3" controls="" autobuffer></audio>
  </div>
</template>

<script>
import VueP5 from 'vue-p5';
/* eslint-disable no-console*/

export default {
  name: 'circleCanvas',
  data () {
    return {
      angles: 12,
      angle: 0,
      tempo: 45,
      width: 0,
      height: 0,
      radius: 0,
      center: 0,
      dots: [],
      dotSize: 5,
      beats: [],
      beatSize: 25,
      sound: null
    }
  },
  computed: {
//    dots () {
//      return this.angles.map(item => {
//        const x = this.center.x - (this.radius * Math.sin(item * -1))
//        const y = this.center.y - (this.radius * Math.cos(item * -1))
//        return {x, y}
//      })
//    }
  },
  methods: {
    isAreaDetected(pointer ,target) {
      const n = 10
      return (pointer.x <= target.x + n && pointer.x >= target.x - n
        && pointer.y <= target.y + n && pointer.y >= target.y - n)
    },
    setup(sketch) {
      sketch.createCanvas(500, 500)
      sketch.background('#fafafa')
      sketch.strokeWeight(6)
      this.width = sketch.width
      this.height = sketch.height
      this.radius = this.width / 3
      this.center = {
        x: this.width / 2,
        y: this.height / 2
      }
      this.createDots(sketch)
    },
    createDots(sketch) {
      sketch.angleMode(sketch.DEGREES)

      for (let i = 0; i < this.angles; i++) {
        let angle = i * 360 / this.angles
        const x = this.center.x - (this.radius * sketch.sin(angle * -1))
        const y = this.center.y - (this.radius * sketch.cos(angle * -1))
        this.dots.push(sketch.createVector(x, y))
      }
    },
    draw(sketch) {
      sketch.clear()
      this.drawMainCircle(sketch)
      this.drawFinger(sketch)
      this.drawDots(sketch)
      this.drawBeats(sketch)
    },
    drawMainCircle(sketch) {
      const size = this.width * 2 / 3
      sketch.stroke('#34ebac')
      sketch.noFill()
      sketch.ellipse(this.width / 2, this.height / 2, size)
    },
    drawFinger(sketch) {
      const end = {
        x: this.center.x - (this.radius * sketch.sin(this.angle)),
        y: this.center.y - (this.radius * sketch.cos(this.angle))
      }
      sketch.angleMode(sketch.DEGREES)
      sketch.stroke('#6a44eb')
      sketch.line(this.center.x, this.center.y, end.x, end.y)

      this.angle -= this.tempo / 60

      this.beats.forEach(beat => {
        if (this.isAreaDetected(end, beat)) {
          this.drawFullDot(sketch, beat, this.beatSize * 1.6, '#d93f28')
        }
      })
    },
    drawDots(sketch) {
      this.dots.forEach(item => {
        this.drawFullDot(sketch, item, this.dotSize, '#321fb2')
      })
    },
    drawBeats(sketch) {
      this.beats.forEach(beat => {
        this.drawFullDot(sketch, beat, this.beatSize, '#d93f28')
      })
    },
    mouseclicked(sketch) {
      let mouse = {
        x: sketch.mouseX,
        y: sketch.mouseY
      }

      this.dots.forEach(dot => {
        if (sketch.mouseClicked && this.isAreaDetected(mouse, dot)) {
          let beatFound = this.beats.find(item => item.x === dot.x && item.y === dot.y)
          if (beatFound) {
            this.beats = this.beats.filter(beat => beat.x !== beatFound.x && beat.y !== beatFound.y)
          } else {
            this.beats.push({x: dot.x, y: dot.y})
          }
        }
      })
    },
    drawFullDot(sketch,dot, size, color) {
      sketch.fill(color)
      sketch.noStroke()
      sketch.ellipse(dot.x, dot.y, size)
    },
    resetBeats() {
      this.beats = []
    }
  },
  mounted() {
    this.sound = this.$el.querySelector('audio')
    window.sound = this.sound
  },
  components: {
    VueP5
  }
}
</script>
