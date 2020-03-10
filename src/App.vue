<template>
  <div class="wrap">
    <canvas-drag :imgArr="imgArr"></canvas-drag>
    <div class="upload">
      <input type="file"
          accept="image/jpeg, image/png, image/jpg"
          @change="changeImage($event)"
          class="input-opacity">
      上传图片
    </div>
  </div>
</template>

<script>
import canvasDrag from './components/canvas-drag'
export default {
  name: 'App',
  data () {
    return {
      url: '',
      imgArr: []
    }
  },
  methods: {
    changeImage (e) {
      const fileObj = new FileReader()
      fileObj.readAsDataURL(e.target.files[0])
      fileObj.onload = () => {
        this.picReduce(fileObj.result, imgObj => {
          this.imgArr.push(imgObj)
        })
      }
    },
    picReduce (picObj, callback) {
      const img = new Image()
      img.src = picObj
      img.onload = () => {
        const w = img.width
        const h = img.height
        const scale = this.getScale(w, h)
        const maxW = w / scale
        const maxH = h / scale
        const imgObj = {
          url: img,
          width: maxW,
          height: maxH
        }
        callback(imgObj)
      }
    },
    getScale (width, height) {
      if (width >= height) {
        if (height <= 120) {
          return 1
        } else {
          return height / 120
        }
      } else if (height > width) {
        if (width <= 120) {
          return 1
        } else return width / 120
      }
    }
  },
  components: { canvasDrag }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
body{
  height: 100%;
}
.upload {
  position: relative;
  width: 100%;
  height: 35px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  font-weight: 800;
  border-radius: 50px;
  background-color: rgb(22,188,121);
}
.input-opacity {
  position: absolute;
  top: 0;
  width: 100%;
  height: 35px;
  opacity: 0;
}
</style>
