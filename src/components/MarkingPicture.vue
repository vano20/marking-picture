<template>
  <div
    :style="{
      minHeight: `${config.canvas.height + 80}px`
    }"
  >
    <ol>
      <li>Click and hold inside picture to set initial point</li>
      <li>drag while holding to draw a line from the first initial point</li>
      <li>
        when a point is close to start point, there will be a prompt to finish
        the process
      </li>
    </ol>
    <div id="canvas1-container" style="position: relative">
      <img
        ref="backgroundImage"
        v-show="config.bgUrl"
        alt="backgroundImg"
        :class="{ 'fallback-img-ratio': !isCanvasRatio }"
      />
      <canvas ref="canvas1" v-show="config.bgUrl" style="position: absolute">
      </canvas>
      <div v-for="(shape, index) in shapes" :key="index">
        <div
          :style="{
            position: 'absolute',
            top: `${
              getY(shape.coordinate, shape.lowestPosition.y) +
              config.additional.top.y
            }px`,
            left: `${
              getX(shape.coordinate, shape.lowestPosition) +
              config.additional.top.x
            }px`,
            zIndex: 3
          }"
        >
          <slot name="top" :props="shape"> </slot>
        </div>
        <div
          :style="{
            position: 'absolute',
            top: `${
              getTopRight(
                shape.coordinate,
                shape.lowestPosition,
                shape.highestPosition
              ).y + config.additional.right.y
            }px`,
            left: `${
              getTopRight(
                shape.coordinate,
                shape.lowestPosition,
                shape.highestPosition
              ).x + config.additional.right.x
            }px`,
            zIndex: 3
          }"
        >
          <slot name="top-right" :props="shape"> </slot>
        </div>
        <div
          :style="{
            position: 'absolute',
            top: `${
              getBottomRight(shape.coordinate, shape.highestPosition).y
            }px`,
            left: `${
              getBottomRight(shape.coordinate, shape.highestPosition).x
            }px`,
            transform: 'translate(-70%, -70%)',
            zIndex: 3
          }"
        >
          <slot name="bottom-right" :props="shape"> x </slot>
        </div>
        <div
          :style="{
            position: 'absolute',
            transform: 'translate(-50%, -50%)',
            top: `${
              getMiddle(
                shape.coordinate,
                shape.lowestPosition,
                shape.highestPosition
              ).y
            }px`,
            left: `${
              getMiddle(
                shape.coordinate,
                shape.lowestPosition,
                shape.highestPosition
              ).x
            }px`,
            zIndex: 3
          }"
        >
          <slot name="middle" :props="shape"> </slot>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { isNotUndefined } from '../utils'

export default {
  props: {
    options: {
      type: Object,
      default: () => ({})
    },
    confirmation: {
      type: Function,
      default: () => confirm('Are you finishing drawing?')
    },
    ids: {
      type: Function,
      default: () => null
    },
    data: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      image: null,
      canvas: null,
      context2d: null,
      isDown: false,
      isFinished: false,
      start: {
        x: '',
        y: ''
      },
      titlePos: {
        x: 0,
        y: 0
      },
      highest: {
        x: 0,
        y: 0
      },
      tempCoords: [],
      shapes: []
    }
  },
  computed: {
    config() {
      const config = {
        bgUrl: '',
        bgColor: 'white',
        pointMargin: 20,
        edit: true,
        additional: {
          top: {
            x: 0,
            y: 0
          },
          right: {
            x: 0,
            y: 0
          }
        },
        canvas: {
          lineWidth: 3,
          setLineDash: [10, 5],
          strokeStyle: 'black',
          height: null,
          width: null
        }
      }
      return {
        ...config,
        ...this.options,
        canvas: {
          ...config.canvas,
          ...this.options.canvas
        },
        additional: {
          ...config.additional,
          ...this.options.additional
        }
      }
    },
    isCanvasRatio() {
      return (
        this.config.canvas.height !== null && this.config.canvas.width !== null
      )
    }
  },
  mounted() {
    this.init()
  },
  watch: {
    config(value) {
      if (value.bgUrl) {
        this.init()
      }
    }
  },
  methods: {
    init() {
      this.canvas = this.$refs.canvas1
      this.context2d = this.canvas.getContext('2d')
      if (this.config.bgUrl) {
        this.image = this.$refs.backgroundImage
        this.image.onload = () => {
          if (this.config.canvas.width && this.config.canvas.height) {
            this.image.width = this.config.canvas.width
            this.image.height = this.config.canvas.height
            this.canvas.width = this.config.canvas.width
            this.canvas.height = this.config.canvas.height
          } else {
            this.canvas.width = this.image.width
            this.canvas.height = this.image.height
            this.$emit('update:options', {
              ...this.options,
              canvas: {
                ...this.options.canvas,
                width: this.image.width,
                height: this.image.height
              }
            })
          }

          if (this.data.length) {
            this.shapes = this.data
            this.redrawCanvas()
          }
        }
        this.image.src = this.config.bgUrl
      }
      if (this.config.edit) this.setCanvasHandler()
    },
    setCanvasHandler() {
      this.canvas.addEventListener('mousedown', this.handlerMouseDown)
      this.canvas.addEventListener('mousemove', this.handlerMouseMove)
      this.canvas.addEventListener('mouseup', this.handleMouseUp)
    },
    snapping({ x, y }) {
      const margin = this.config.pointMargin
      const currX = x
      const currY = y
      const checkMargin = (toX, toY) =>
        currX <= toX + margin &&
        currX >= toX - margin &&
        currY <= toY + margin &&
        currY >= toY - margin
      const point = this.tempCoords.find(point => {
        const aroundXY1 = checkMargin(point.x1, point.y1)
        const aroundXY2 = checkMargin(point.x2, point.y2)
        return aroundXY1 || aroundXY2
      })
      if (point) {
        if (checkMargin(point.x1, point.y1)) {
          return {
            x: point.x1,
            y: point.y1
          }
        } else if (checkMargin(point.x2, point.y2)) {
          return {
            x: point.x2,
            y: point.y2
          }
        }
      }
    },
    redrawAllPosLine() {
      this.isFinished = true
      for (let indexPos = 0; indexPos < this.shapes.length; indexPos++) {
        this.redrawStoredLines(this.shapes[indexPos].coordinate)
      }
      this.isFinished = false
    },
    redrawCanvas() {
      this.context2d.clearRect(0, 0, this.canvas.width, this.canvas.height)
      if (this.shapes.length) {
        if (!this.isFinished) {
          this.redrawStoredLines(this.tempCoords)
        }
        this.redrawAllPosLine()
      } else {
        this.redrawStoredLines(this.tempCoords)
      }
    },
    redrawStoredLines(currentCoords) {
      if (!currentCoords || !currentCoords?.length) return
      for (let i = 0; i < currentCoords.length; i++) {
        const { x1, y1, x2, y2 } = currentCoords[i]
        this.drawLines({
          x1,
          y1,
          x2,
          y2
        })
      }
    },
    drawLines({ x1, y1, x2, y2 }) {
      const ctx = this.context2d
      ctx.beginPath()
      if (isNotUndefined(this.config.canvas.strokeStyle)) {
        ctx.strokeStyle = this.config.canvas.strokeStyle
      }
      if (isNotUndefined(this.config.canvas.setLineDash)) {
        ctx.setLineDash(this.config.canvas.setLineDash)
      }
      if (isNotUndefined(this.config.canvas.lineWidth)) {
        ctx.lineWidth = this.config.canvas.lineWidth
      }
      ctx.moveTo(x1, y1)
      ctx.lineTo(x2, y2)
      ctx.stroke()
    },
    finishingShape() {
      if (!(this.tempCoords.length >= 3)) {
        this.$emit('alert', 'Please create shapes first before finishing')
        return
      }
      if (this.confirmation()) {
        this.isFinished = true
        const generatedId = this.ids()
        this.shapes.push({
          id: generatedId || this.shapes.length + 1,
          name: `title ${this.shapes.length + 1}`,
          coordinate: this.tempCoords,
          lowestPosition: this.titlePos,
          highestPosition: this.highest
        })
        this.$emit('update:data', this.shapes)
      }
      this.titlePos = {
        x: 0,
        y: 0
      }
      this.highest = {
        x: 0,
        y: 0
      }
      this.redrawCanvas()
      this.tempCoords = []
    },
    handlerMouseDown(event) {
      this.isDown = true
      this.isFinished = false
      this.start = {
        x: event.layerX,
        y: event.layerY
      }
      const findSimilar = this.snapping({ ...this.start })
      if (findSimilar) {
        this.start = findSimilar
      }
      this.titlePos = this.isLowestXY(this.start, this.tempCoords.length)
      this.highest = this.isHighestXY(this.start, this.tempCoords.length)
    },
    handlerMouseMove(event) {
      if (!this.isDown) return
      this.redrawCanvas()
      this.drawLines({
        x1: this.start.x,
        y1: this.start.y,
        x2: event.layerX,
        y2: event.layerY
      })
    },
    handleMouseUp(event) {
      this.isDown = false
      let mouse = {
        x: event.layerX,
        y: event.layerY
      }
      if (mouse.x >= this.start.x - 50 && mouse.x <= this.start.x + 50) {
        mouse.x = this.start.x
      }
      if (mouse.y >= this.start.y - 50 && mouse.y <= this.start.y + 50) {
        mouse.y = this.start.y
      }
      const findSimilar = this.snapping({ ...mouse })
      if (findSimilar) {
        mouse = findSimilar
      }
      this.titlePos = this.isLowestXY(mouse, this.tempCoords.length)
      this.highest = this.isHighestXY(mouse, this.tempCoords.length)
      this.tempCoords.push({
        x1: this.start.x,
        y1: this.start.y,
        x2: mouse.x,
        y2: mouse.y
      })
      this.redrawCanvas()
    },
    isLowestXY({ x, y }, index) {
      let selectedX = this.titlePos.x
      let selectedY = this.titlePos.y
      if (
        index === selectedX ||
        this.tempCoords[selectedX].x1 > x ||
        this.tempCoords[selectedX].x2 > x
      )
        selectedX = index
      if (
        index === selectedY ||
        this.tempCoords[selectedY].y1 > y ||
        this.tempCoords[selectedY].y2 > y
      )
        selectedY = index
      return {
        x: selectedX,
        y: selectedY
      }
    },
    isHighestXY({ x, y }, index) {
      let selectedX = this.highest.x
      let selectedY = this.highest.y
      if (
        index === selectedX ||
        this.tempCoords[selectedX].x1 < x ||
        this.tempCoords[selectedX].x2 < x
      )
        selectedX = index
      if (
        index === selectedY ||
        this.tempCoords[selectedY].y1 < y ||
        this.tempCoords[selectedY].y2 < y
      )
        selectedY = index
      return {
        x: selectedX,
        y: selectedY
      }
    },
    lowestPoint(coords, index, prop) {
      return coords[index][`${prop}1`] > coords[index][`${prop}2`]
        ? coords[index][`${prop}2`]
        : coords[index][`${prop}1`]
    },
    getY(coords, index) {
      return this.lowestPoint(coords, index, 'y')
    },
    getX(coords, { x: index, y: yIndex }) {
      let y1 = this.getY(coords, yIndex)
      let y2 = this.getY(coords, index)
      const yDifference = y2 - y1
      return this.lowestPoint(coords, yDifference > 10 ? yIndex : index, 'x')
    },
    getTopRight(coords, { y }, { x }) {
      const yLowest = this.lowestPoint(coords, y, 'y')
      const yHighest = this.lowestPoint(coords, x, 'y')
      const yDifference = yHighest - yLowest
      const selectedIndex = yDifference < 10 ? x : y
      return {
        x:
          coords[selectedIndex].x1 > coords[selectedIndex].x2
            ? coords[selectedIndex].x1
            : coords[selectedIndex].x2,
        y: this.lowestPoint(coords, y, 'y')
      }
    },
    getBottomRight(coords, { y }) {
      return {
        x: coords[y].x1 > coords[y].x2 ? coords[y].x1 : coords[y].x2,
        y: coords[y].y1 > coords[y].y2 ? coords[y].y1 : coords[y].y2
      }
    },
    getMiddle(coords, lowest, highest) {
      const topRight = this.getTopRight(coords, lowest, highest)
      const bottomRight = this.getBottomRight(coords, highest)
      const x1 = this.getX(coords, lowest)
      const x2 = topRight.x
      const y1 = topRight.y
      const y2 = bottomRight.y
      const xMiddle = Math.ceil((x2 - x1) / 2 + x1)
      const yMiddle = Math.ceil((y2 - y1) / 2 + y1)
      return {
        x: xMiddle,
        y: yMiddle
      }
    },
    undo() {
      if (this.tempCoords.length) this.tempCoords.pop()
      this.redrawCanvas()
    }
  }
}
</script>
<style lang="scss">
#canvas1-container {
  // background: var(--bgContainer);
  // background-repeat: no-repeat;
  height: 100%;
  width: 100%;
  canvas {
    z-index: 2;
  }
  img {
    z-index: 1;
    position: absolute;
  }
}
.fallback-img-ratio {
  max-width: 100%;
  height: auto;
}
</style>
