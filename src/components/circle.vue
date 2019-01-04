<template>
  <div>
    <vue-p5 @setup="setup"
            @draw="draw"
    ></vue-p5>
  </div>
</template>

<script>
import VueP5 from 'vue-p5';

export default {
  name: 'circleCanvas',
  data () {
    return {
      angles: [0, 90, 180, 270],
      angle: 0,
      tempo: 60,
      width: 0,
      height: 0,
      radius: 0,
      center: 0,
      dots: []
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
      this.dots = this.createDots(sketch)
    },
    createDots(sketch) {
      sketch.angleMode(sketch.DEGREES)

      return this.angles.map(item => {
        const x = this.center.x - (this.radius * sketch.sin(item * -1))
        const y = this.center.y - (this.radius * sketch.cos(item * -1))
        return sketch.createVector(x, y)
      })
    },
    draw(sketch) {
      sketch.clear()
      this.drawMainCircle(sketch)
      this.drawFinger(sketch)
      this.drawDots(sketch)
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


//
      this.dots.forEach((dot, i) => {
        const n = 10
        if (end.x <= dot.x + n && end.x >= dot.x - n
          && end.y <= dot.y + n && end.y >= dot.y - n) {
          sketch.fill('#d93f28')
          sketch.noStroke()
          sketch.ellipse(dot.x, dot.y, 50)
        }
      })
    },
    drawDots(sketch) {
      const size = 30
      this.dots.forEach(item => {
        sketch.fill('#d93f28')
        sketch.noStroke()
        sketch.ellipse(item.x, item.y, size)
      })
    }
  },
  components: {
    VueP5
  }
}
</script>
