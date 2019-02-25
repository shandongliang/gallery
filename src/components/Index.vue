<template>
  <div class="index-wrap">
    <div class="image-wrap" ref="stage">
      <div class="image-item"
           v-for="(item,index) in imgsArrangeArr"
           :key="item.fileName" :ref="`image${index}`"
           :style="`left:${item.pos?item.pos.left:0}px;top:${item.pos?item.pos.top:0}px;transform:rotate(${item.rotate?item.rotate:0}deg) ;zIndex:${item.isCenter?11:1}`"
           @click="center(index,item)"
      >
        <div class="img" :style="`transform: ${item.isInverse?'rotateY(180deg)':'rotateY(0deg)'};`">
          <img :src="require(`../images/${item.fileName}`)" />
          <p>{{item.title}}</p>
        </div>
        <div class="img-back" :style="`transform: ${item.isInverse?'rotateY(0deg)':'rotateY(-180deg)'};`">
          <p>{{item.desc}}</p>
        </div>
      </div>
    </div>
    <div class="controller-nav">
      <span :class="`controller-unit ${item.isCenter?'dos-center':''} ${item.isInverse?'dos-inverse':''}`"
            v-for="(item,index) in imgsArrangeArr"
            :key="item.fileName"
            @click="center(index,item)">
      </span>
    </div>
  </div>
</template>

<script>
import {imageDatas, getRangeRandom, get30DegRandom} from '../data/imageDatas.js'
export default {
  data () {
    return {
      imgsArrangeArr: imageDatas,
      Constant: {
        centerPos: {
          left: 0,
          right: 0
        },
        hPosRange: {// 水平方向的取值范围
          leftSecX: [0, 0],
          rightSecX: [0, 0],
          y: [0, 0]
        },
        vPosRange: {// 垂直方向的取值范围
          x: [0, 0],
          topY: [0, 0]
        }
      }
    }
  },
  mounted () {
    // 首先拿到舞台的大小
    let stageDOM = this.$refs.stage
    let stageW = stageDOM.scrollWidth
    let stageH = stageDOM.scrollHeight
    let halfStageW = Math.ceil(stageW / 2)
    let halfStageH = Math.ceil(stageH / 2)
    // 拿到一个imageFigure的大小
    let imgFigureDOM = this.$refs.image0[0]
    let imgW = imgFigureDOM.scrollWidth
    let imgH = imgFigureDOM.scrollHeight
    let halfImgW = Math.ceil(imgW / 2)
    let halfImgH = Math.ceil(imgH / 2)
    // 计算中心图片的位置点
    this.Constant.centerPos = {
      left: halfStageW - halfImgW,
      top: halfStageH - halfImgH
    }

    // 计算左侧，右侧区域图片排布位置的取值范围
    this.Constant.hPosRange.leftSecX[0] = -halfImgW
    this.Constant.hPosRange.leftSecX[1] = halfStageW - halfImgW * 3
    this.Constant.hPosRange.rightSecX[0] = halfStageW + halfImgW
    this.Constant.hPosRange.rightSecX[1] = stageW - halfImgW
    this.Constant.hPosRange.y[0] = -halfImgH
    this.Constant.hPosRange.y[1] = stageH - halfImgH

    // 计算上侧区域图片排布位置的取值范围
    this.Constant.vPosRange.topY[0] = -halfImgH
    this.Constant.vPosRange.topY[1] = halfStageH - halfImgH * 3
    this.Constant.vPosRange.x[0] = halfStageW - imgW
    this.Constant.vPosRange.x[1] = halfStageW
    this.rearrange(0)
  },
  methods: {
    center (index, item) {
      if (item.isCenter) {
        this.imgsArrangeArr[index].isInverse = !this.imgsArrangeArr[index].isInverse
      } else {
        this.rearrange(index)
      }
    },
    rearrange (centerIndex) {
      let imgsArrangeArr = this.imgsArrangeArr
      let Constant = this.Constant

      let centerPos = Constant.centerPos
      let hPosRange = Constant.hPosRange
      let vPosRange = Constant.vPosRange
      let hPosRangeLeftSecX = hPosRange.leftSecX
      let hPosRangeRightSecX = hPosRange.rightSecX
      let hPosRangeY = hPosRange.y
      let vPosRangeTopY = vPosRange.topY
      let vPosRangeX = vPosRange.x

      let imgsArrangeTopArr = []
      let topImgNum = Math.floor(Math.random() * 2) // 取一个或者不取
      let topImgSpliceIndex = 0

      let imgsArrangeCenterArr = imgsArrangeArr.splice(centerIndex, 1)

      // 首先居中 centerIndex 的图片, 居中的 centerIndex 的图片不需要旋转
      imgsArrangeCenterArr[0] = {
        ...imgsArrangeCenterArr[0],
        pos: centerPos,
        rotate: 0,
        isCenter: true,
        isInverse: false
      }

      // 取出要布局上侧的图片的状态信息
      topImgSpliceIndex = Math.ceil(Math.random() * (imgsArrangeArr.length - topImgNum))
      imgsArrangeTopArr = imgsArrangeArr.splice(topImgSpliceIndex, topImgNum)
      // console.log(imgsArrangeArr.splice(topImgSpliceIndex, topImgNum))

      // 布局位于上侧的图片
      imgsArrangeTopArr.forEach(function (value, index) {
        imgsArrangeTopArr[index] = {
          ...imgsArrangeTopArr[index],
          pos: {
            top: getRangeRandom(vPosRangeTopY[0], vPosRangeTopY[1]),
            left: getRangeRandom(vPosRangeX[0], vPosRangeX[1])
          },
          rotate: get30DegRandom(),
          isCenter: false,
          isInverse: false
        }
      })

      // 布局左右两侧的图片
      for (let i = 0, j = imgsArrangeArr.length, k = j / 2; i < j; i++) {
        let hPosRangeLORX = null
        // 前半部分布局左边， 右半部分布局右边
        if (i < k) {
          hPosRangeLORX = hPosRangeLeftSecX
        } else {
          hPosRangeLORX = hPosRangeRightSecX
        }

        imgsArrangeArr[i] = {
          ...imgsArrangeArr[i],
          pos: {
            top: getRangeRandom(hPosRangeY[0], hPosRangeY[1]),
            left: getRangeRandom(hPosRangeLORX[0], hPosRangeLORX[1])
          },
          rotate: get30DegRandom(),
          isCenter: false,
          isInverse: false
        }
      }
      if (imgsArrangeTopArr && imgsArrangeTopArr[0]) {
        imgsArrangeArr.splice(topImgSpliceIndex, 0, imgsArrangeTopArr[0])
      }

      imgsArrangeArr.splice(centerIndex, 0, imgsArrangeCenterArr[0])
      this.imgsArrangeArr = imgsArrangeArr
      console.log(this.imgsArrangeArr)
    }
  }
}
</script>

<style>
@font-face {
  font-family: "icons-turn-arrow";
  src: url("../fonts/icons/turn-arrow.eot") format("embedded-opentype"), url("../fonts/icons/turn-arrow.woff") format("woff"), url("../fonts/icons/turn-arrow.ttf") format("truetype"), url("../fonts/icons/turn-arrow.svg") format("svg");
}
.index-wrap {
  width: 100%;
  height: 100%;
}
.image-wrap {
  position: relative;
  width: 100%;
  height: 680px;
  overflow: hidden;
  background-color: #ddd;
}
.image-item {
  width: 320px;
  height: 360px;
  margin: 0;
  cursor: pointer;
  position: absolute;
  text-align: center;
  transform-origin: 0 50% 0;
  transform-style: preserve-3d;
  transition: transform .6s ease-in-out, left .6s ease-in-out, top .6s ease-in-out;
  perspective: 500;
}
.img-back{
  width: 100%;
  height: 100%;
  padding: 40px;
  background-color: #fff;
  box-sizing: border-box;
  transform: rotateY(0deg);
}
.img-back p{
  font-size: 20px;
  color: #aaa;
  text-align: start;
}
.img{
  width: 100%;
  height: 100%;
  padding: 40px;
  box-sizing: border-box;
  background-color: #fff;
}
.img p{
  color: #aaa;
}
.img,.img-back{
  backface-visibility: hidden;
  transition: 0.6s;
  position: absolute;
  top: 0px;
  left: 0px;
}
.controller-nav {
  z-index: 101;
  width: 100%;
  text-align: center;
  margin-top: 30px;
}
.controller-unit {
  display: inline-block;
  margin: 0 5px;
  width: 30px;
  height: 30px;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  background-color: #aaa;
  border-radius: 50%;
  transform: scale(.5);
  transition: transform .6s ease-in-out, background-color .3s;
}
.dos-center {
  background-color: #888;
  transform: scale(1);
}
.dos-center::after {
  color: #fff;
  font-family: "icons-turn-arrow";
  font-size: 80%;
  line-height: 30px;
  content: "\e600";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.dos-inverse {
  background-color: #555;
  transform: rotateY(180deg);
}

</style>
