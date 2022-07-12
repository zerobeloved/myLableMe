<template>
  <div class="img-box" @mousedown="mousedown"
       style="width: 1000px; height:2000px;">
    <canvas id="canvas"  class="canvas-box" style="background: lightgrey"></canvas>
    <img src="../assets/404.png" ref="canvasImg" width="100%" id="targetImg" >
  </div>
</template>

<script>

export default {
  name: 'Page404',
  data() {
    return {
      lineList: [],
      downFlag: false,
      lineWid: 30,
      roundrr: 10
    }
  },
  mounted() {
    setTimeout(() => {
      this.canvasWid = this.$refs.canvasImg.getBoundingClientRect().width
      this.canvasHig = this.$refs.canvasImg.getBoundingClientRect().height
      this.scaleX = this.canvasWid / 300
      this.scaleY = this.canvasHig /150
      this.smlImgScl = 3000 / this.canvasWid
      this.initEvent()
    },300)
    this.windowToCanvasPosition()
  },
  watch: {
    lineList: {
      handler (newVal) {
        if (newVal.length > 0) {
          this.downFlag = true
        } else {
          this.downFlag = false
        }
      },
      deep: true
    }
  },
  methods: {
    // draw() {
    //   var canvas = document.getElementById('canvas');
    //   canvas.width='300';
    //   canvas.height='300';
    //   var ctx = canvas.getContext('2d');
    //   //先将笔尖移动到0,0处
    //   ctx.moveTo(0,0);
    //   //先将笔滑到200,200处
    //   ctx.lineTo(200,200);
    //   //执行绘画的动作，如果没有执行stroke函数不会有任何的效果
    //   ctx.stroke();
    // },
    initEvent() {
      let canvas = document.getElementById('canvas')
      canvas.width = this.canvasWid
      canvas.height = this.canvasHig
      this.context = canvas.getContext('2d')
      this.context.drawImage(this.$refs.canvasImg,0,0)
    },
    windowToCanvasPosition(){
      let imgPosition = this.$refs.canvasImg.getBoundingClientRect()
      this.posiX = imgPosition.x
      this.posiY = imgPosition.y
    },
    drawArc (x, y) {
      console.log("画了")
      this.context.strokeStyle = 'red'
      this.context.beginPath()
      this.context.arc(x, y, this.roundrr, 360, Math.PI * 2, true)
      this.context.closePath()
      this.context.stroke()
    },
    drawLine (startX, startY, endX, endY) {
      this.context.strokeStyle = 'blue'
      this.context.lineWidth = this.lineWid
      this.context.moveTo(startX, startY)
      this.context.lineTo(endX, endY)
      this.context.stroke()
    },
    equalStartPoint (x, y) {
      let point = this.lineList[this.lineList.length - 1].points
      if (Math.abs((x - point[0].x) * (x - point[0].x)) + Math.abs((y - point[0].y) * (y - point[0].y)) <= this.roundrr * this.roundrr) {
        this.lineList[this.lineList.length - 1].isClose = true
        return true
      } else {
        this.lineList[this.lineList.length - 1].isClose = false
        return false
      }
    },
    mousedown(e){
      this.pointX = e.clientX - this.posiX
      this.pointY = e.clientY -this.posiY
      console.log(e.clientY)
      if (!this.downFlag) {
        // 第一次画图
        let lineObj = {
          points: [],
          type: '',
          isClose: false
        }
        this.lineList.push(lineObj)
        this.lineList[0].points.push({
          x: this.pointX,
          y: this.pointY
        })
        this.drawArc(this.lineList[0].points[0].x, this.lineList[0].points[0].y)
      } else {
        // 判断最后一个图形是否闭合
        if (this.lineList[this.lineList.length - 1].isClose) {
          // 重新开始添加图形点
          let lineObj = {
            points: [],
            type: '',
            isClose: false
          }
          this.lineList.push(lineObj)
          let len = this.lineList.length
          this.lineList[len - 1].points.push({
            x: this.pointX,
            y: this.pointY
          })
          this.drawArc(this.lineList[len - 1].points[0].x, this.lineList[len - 1].points[0].y)
        } else {
          // 最后一条线上继续添加点
          // 判断是否和起始点重合
          if (this.equalStartPoint(this.pointX, this.pointY)) {
            // 重合
            let num = this.lineList.length
            this.lineList[num - 1].points.push({
              x: this.lineList[num - 1].points[0].x,
              y: this.lineList[num - 1].points[0].y
            })
            let lineObj = this.lineList[num - 1]
            let n = this.lineList[num - 1].points.length
            this.drawLine(lineObj.points[n - 2].x, lineObj.points[n - 2].y, lineObj.points[n - 1].x, lineObj.points[n - 1].y)
            this.dialogVisible = true
            this.nowLine = num - 1
          } else {
            // 不重合
            let num = this.lineList.length
            this.lineList[num - 1].points.push({
              x: this.pointX,
              y: this.pointY
            })
            let n = this.lineList[num - 1].points.length
            let lineObj = this.lineList[num - 1]
            this.drawLine(lineObj.points[n - 2].x, lineObj.points[n - 2].y, lineObj.points[n - 1].x, lineObj.points[n - 1].y)
            this.drawArc(lineObj.points[n - 1].x, lineObj.points[n - 1].y)
          }
        }
      }
    }
  }
}
</script>

