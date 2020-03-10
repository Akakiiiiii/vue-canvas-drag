<template>
<canvas id="myCanvas" class="canvas" style="border:1px solid #000000;" @touchstart.prevent="start($event)"
@touchmove.prevent="move($event)"> </canvas>
</template>

<script>
import CloseIcon from './image/close.png'
import ScaleIcon from './image/放大.png'
var close = new Image(); var scale = new Image()
close.src = CloseIcon
scale.src = ScaleIcon

class DragImg {
  constructor (img, canvas) {
    this.x = 30
    this.y = 30
    this.w = img.width
    this.h = img.height
    this.url = img.url
    this.ctx = canvas
    this.rotate = 0
    this.selected = true
  }

  paint () {
    this.ctx.save()
    this.centerX = this.x + this.w / 2
    this.centerY = this.y + this.h / 2
    // 旋转元素
    this.ctx.translate(this.centerX, this.centerY)
    this.ctx.rotate(this.rotate * Math.PI / 180)
    this.ctx.translate(-this.centerX, -this.centerY)
    // 渲染元素
    this.ctx.drawImage(this.url, this.x, this.y, this.w, this.h)
    // 如果是选中状态，绘制选择虚线框，和缩放图标、删除图标
    if (this.selected) {
      this.ctx.setLineDash([10, 10])
      this.ctx.lineWidth = 2
      this.ctx.strokeStyle = 'red'
      this.ctx.lineDashOffset = 10
      this.ctx.strokeRect(this.x, this.y, this.w, this.h)
      this.ctx.drawImage(close, this.x - 12, this.y - 12, 24, 24)
      this.ctx.drawImage(scale, this.x + this.w - 12, this.y + this.h - 12, 24, 24)
    }
    this.ctx.restore()
  }

  isInWhere (x, y) {
    // 变换区域左上角的坐标和区域的高度宽度
    const transformW = 24
    const transformH = 24
    let transformX = this.x + this.w
    let transformY = this.y + this.h
    const transformAngle = Math.atan2(transformY - this.centerY, transformX - this.centerX) / Math.PI * 180 + this.rotate
    const transformXY = this.getTransform(transformX, transformY, transformAngle)
    transformX = transformXY.x; transformY = transformXY.y
    // 删除区域左上角的坐标和区域的高度宽度
    const delW = 24
    const delH = 24
    let delX = this.x
    let delY = this.y
    const delAngle = Math.atan2(delY - this.centerY, delX - this.centerX) / Math.PI * 180 + this.rotate
    const delXY = this.getTransform(delX, delY, delAngle)
    delX = delXY.x; delY = delXY.y
    // 移动区域的坐标
    const moveX = this.x
    const moveY = this.y
    if (x - transformX >= 0 && y - transformY >= 0 && transformX + transformW - x >= 0 && transformY + transformH - y >= 0) {
      // 缩放区域
      return 'transform'
    } else if (x - delX >= 0 && y - delY >= 0 && delX + delW - x >= 0 && delY + delH - y >= 0) {
      // 删除区域
      return 'del'
    } else if (x - moveX >= 0 && y - moveY >= 0 && moveX + this.w - x >= 0 && moveY + this.h - y >= 0) {
      // 移动区域
      return 'move'
    }
    // 不在选择区域里面
    return false
  }

  getTransform (x, y, rotate) {
    // 将角度化为弧度
    var angle = Math.PI / 180 * rotate
    // 初始坐标与中点形成的直线长度不管怎么旋转都是不会变的，用勾股定理求出然后将其作为斜边
    var r = Math.sqrt(Math.pow(x - this.centerX, 2) + Math.pow(y - this.centerY, 2))
    // 斜边乘sin值等于即可求出y坐标
    var a = Math.sin(angle) * r
    // 斜边乘cos值等于即可求出x坐标
    var b = Math.cos(angle) * r
    // 目前的xy坐标是相对于图片中点为原点的坐标轴，而我们的主坐标轴是canvas的坐标轴，所以要加上中点的坐标值才是标准的canvas坐标
    return {
      x: this.centerX + b - 12,
      y: this.centerY + a - 12
    }
  }
}
export default {
  name: 'canvas-drag',
  props: {
    imgArr: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    return {
      ctx: null,
      dragArr: [],
      clickedkArr: [],
      c: null,
      lastImg: null,
      initial: null,
      startTouch: null
    }
  },
  methods: {
    draw () {
      this.ctx.clearRect(0, 0, this.c.width, this.c.height)
      this.dragArr.forEach((item) => {
        item.paint()
      })
    },
    start (e) {
      // 初始化一个数组用于存放所有被点击到的图片对象
      this.clickedkArr = []
      const x = e.touches[0].clientX
      const y = e.touches[0].clientY
      this.dragArr.forEach((item, index) => {
        const place = item.isInWhere(x, y)
        item.place = place
        item.index = index
        // 先将所有的item的selected变为flase
        item.selected = false
        if (place) {
          this.clickedkArr.push(item)
        }
      })
      const length = this.clickedkArr.length
      if (length) {
        // 我们知道cavans绘制的图片的层级是越来越高的，因此我们取这个数组的最后一项，保证取到的图片实例是层级最高的
        const lastImg = this.clickedkArr[length - 1]
        // 如果是删除的话就移除
        if (lastImg.place === 'del') {
          this.dragArr.splice(lastImg.index, 1)
          // 重新绘制
          this.draw()
          return
        }
        // 将该实例的被选值设为true，下次重新绘制将绘制边框
        lastImg.selected = true
        // 保存这个选中的实例
        this.lastImg = lastImg
        // 保存这个实例的初始值，以后会用上
        this.initial = {
          initialX: lastImg.x,
          initialY: lastImg.y,
          initialH: lastImg.h,
          initialW: lastImg.w,
          initialRotate: lastImg.rotate
        }
      }
      // 重新绘制
      this.draw()
      // 保存点击的坐标，move时要用
      this.startTouch = { startX: x, startY: y }
    },
    move (e) {
      const x = e.touches[0].clientX
      const y = e.touches[0].clientY
      const { initialX, initialY } = this.initial
      const { startX, startY } = this.startTouch
      const lastImg = this.lastImg
      if (this.clickedkArr.length) {
        if (this.lastImg.place === 'move') {
          // 算出移动后的xy坐标与点击时xy坐标的差（即平移量）与图片对象的初始坐标相加即可
          lastImg.x = initialX + (x - startX)
          lastImg.y = initialY + (y - startY)
        }
        if (this.lastImg.place === 'transform') {
          const { centerX, centerY } = lastImg
          // 旋转部分
          const { initialRotate } = this.initial
          const angleBefore = Math.atan2(startY - centerY, startX - centerX) / Math.PI * 180
          const angleAfter = Math.atan2(y - centerY, x - centerX) / Math.PI * 180
          // 旋转的角度
          lastImg.rotate = initialRotate + angleAfter - angleBefore
          // 缩放部分
          const { initialH, initialW } = this.initial
          // 用勾股定理算出距离
          const lineA = Math.sqrt(Math.pow(centerX - startX, 2) + Math.pow(centerY - startY, 2))
          const lineB = Math.sqrt(Math.pow(centerX - x, 2) + Math.pow(centerY - y, 2))
          const w = initialW + (lineB - lineA)
          // 由于是等比缩放，所以乘一个宽高比例。
          const h = initialH + (lineB - lineA) * (initialH / initialW)
          // 定义最小宽高
          lastImg.w = w <= 5 ? 5 : w
          lastImg.h = h <= 5 ? 5 : h
          if (w > 5 && h > 5) {
            // 放大 或 缩小
            lastImg.x = initialX - (lineB - lineA) / 2
            lastImg.y = initialY - (lineB - lineA) / 2
          }
        }
        this.draw()
      }
    }
  },
  watch: {
    imgArr (arr) {
      if (arr.length) {
        const newImg = arr.slice(-1)[0]
        const item = new DragImg(newImg, this.ctx)
        this.dragArr.push(item)
        console.log(this.dragArr)
        this.draw()
      }
    }
  },
  mounted () {
    const c = document.getElementById('myCanvas')
    this.c = c
    const { width, height } = window.screen
    c.width = width - 2
    c.height = height * 0.75
    this.ctx = c.getContext('2d')
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
