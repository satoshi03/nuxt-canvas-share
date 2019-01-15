<template>
  <v-layout
    row
    justifiy-center
    align-center
    >
    <v-flex
      xs12
      sm6
    >
      <v-card>
        <v-card-media>
          <div ref="container">
            <canvas ref="canvas" width=600 height=400>
            </canvas>
          </div>
        </v-card-media>
        <v-card-actions>
          <v-btn @click="reset()">RESET</v-btn>
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import Konva from "konva"
import firebase from '~/plugins/firebase'

export default {
  data () {
    return {
      canvas: {},
      stage: {},
      drawLayer: {},
      image: {},
      context: {},
      canvasConfig: {
        width: 600,
        height: 400,
      },
      drawConfig: {
        strokeStyle: "#df4b26",
        lineJoin: "round",
        lineWidth: 5,
      },
      isPaint: false,
      lastPointerPosition: {},
      compositeOperation: "source-over",
      events: [],
      eventRef: null,
    }
  },
  created: function () {
    this.database = firebase.database()
    this.eventRef = this.database.ref('events/1')
    this.eventRef.on('child_added', snapshot => {
      const ss = snapshot.val()
      this.events.push({
        fromX: ss.fromX,
        fromY: ss.fromY,
        posX: ss.posX,
        posY: ss.posY,
        toX: ss.toX,
        toY: ss.toY,
      })
      //this.draw(ss.fromX, ss.fromY, ss.posX, ss.posY, ss.toX, ss.toY)
    })
  },
  mounted () {
    this.canvas = this.$refs.canvas;
    this.context = this.canvas.getContext("2d")

    this.context.strokeStyle = this.drawConfig.strokeStyle
    this.context.lineJoin = this.drawConfig.lineJoin
    this.context.lineWidth = this.drawConfig.lineWidth

    var container = this.$refs.container
    this.stage = new Konva.Stage({
      container,
      width: this.canvasConfig.width,
      height: this.canvasConfig.height,
    })

    this.image = new Konva.Image({
      image: this.canvas,
      x: 0,
      y: 0,
    })

    this.drawLayer = new Konva.Layer();
    this.drawLayer.add(this.image)
    this.stage.add(this.drawLayer)
    this.stage.draw()

    this.stage.on("contentMousedown.proto", this.mousedown);
    this.stage.on("contentTouchstart.proto", this.mousedown);

    this.stage.on("contentMouseup.proto", this.mouseup);
    this.stage.on("contentTouchend.proto", this.mouseup);

    this.stage.on("contentMousemove.proto", this.mousemove);
    this.stage.on("contentTouchmove.proto", this.mousemove);
  },
  watch: {
    events: function (events) {
      for (let val of events) {
        this.draw(val.fromX, val.fromY, val.posX, val.posY, val.toX, val.toY)
      }
    }
  },
  methods: {
    reset () {
      this.events = []
      this.eventRef.remove()
    },
    mousedown (event) {
      this.isPaint = true
      this.lastPointerPosition = this.stage.getPointerPosition()
    },
    mouseup (event) {
      this.isPaint = false
    },
    mousemove (event) {
      if (!this.isPaint) {
        return
      }

      var localPosFrom = {
        x: this.lastPointerPosition.x - this.image.x(),
        y: this.lastPointerPosition.y - this.image.y(),
      }

      var pos = this.stage.getPointerPosition()
      var localPosTo = {
        x: pos.x - this.image.x(),
        y: pos.y - this.image.y(),
      }

      this.eventRef.push({
        fromX: localPosFrom.x,
        fromY: localPosFrom.y,
        posX: pos.x,
        posY: pos.y,
        toX: localPosTo.x,
        toY: localPosTo.y,
      })

      this.events.push({
        fromX: localPosFrom.x,
        fromY: localPosFrom.y,
        posX: pos.x,
        posY: pos.y,
        toX: localPosTo.x,
        toY: localPosTo.y,
      })

      // this.draw(localPosFrom.x, localPosFrom.y, pos.x, pos.y, localPosTo.x, localPosTo.y)
    },
    draw (posFromX, posFromY, posX, posY, posToX, posToY) {
      this.context.globalCompositeOperation = this.compositeOperation
      this.context.beginPath()

      this.context.moveTo(posFromX, posFromY);
      this.context.lineTo(posToX, posToY);
      this.context.closePath()
      this.context.stroke()

      this.lastPointerPosition = {
        x: posX,
        y: posY,
      }
      this.drawLayer.batchDraw()
    }
  }
}
</script>

<style>
</style>
