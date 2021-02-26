<template>
<div @mousemove="wrapperMousemove" class="wrapper" @mouseup="wrapperMouseup">
  <div class="sidebar">
    <div v-for="ele in elementsData" :key="ele.id" class="image-wrapper">
      <img :src="ele.src" @click="addElm(ele)"/>
    </div>
    <button @click="download">下载</button>
    <button @click="save">保存</button>
  </div>
  <div class="container" @click="wrapperClick" id="my-node">
    <div v-for="ele in elmList" :key="ele.ref">
      <img v-if="ele.type === 'image'" class="hover" :ref="ele.ref"
      :data-ref="ele.ref" data-type="image" draggable="false"
      :src="ele.src"
      :style="ele.style"
      @mousedown="mousedown($event, ele)"
      @mousemove="mousemove"
      @mouseup="mouseup"/>
      <div v-if="ele.type === 'text'" class="hover text-wrapper"
      :ref="ele.ref" :data-ref="ele.ref" data-type="text"
      draggable="false"
      :style="ele.style"
      @mousedown="mousedown($event, ele)"
      @mousemove="mousemove"
      @mouseup="mouseup" @dblclick="dblclick(ele)">
      <div>
        <span class="content" spellcheck="false"
        :contenteditable="ele.contenteditable" @blur="blur(ele)">{{ ele.content }}</span>
      </div>

      </div>
    </div>
    <!-- <img class="comp hover"
    ref="comp"
    data-ref="comp"
    draggable="false"
    src="../assets/logo.png"
    @mousedown="mousedown"
    @mousemove="mousemove"
    @mouseup="mouseup"/>
    <div class="comp hover"
    ref="comp1"
    data-ref="comp1"
    @mousedown="mousedown"
    @mousemove="mousemove"
    @mouseup="mouseup">移动一下试试</div> -->
  </div>
  <div draggable="false" v-for="(dot, index) in dots" :key="index" class="dot"
    v-show="showDots"
    :style="dot"
    @mousedown="dotMousedown($event, index)"
    ></div>
</div>
</template>
<script>
import elementsData from './elements'
import html2canvas from 'html2canvas'
const DOT_WIDTH = 8
export default {
  data() {
    return {
      dots: Array(8).fill(''),
      showResize: false,
      x: 0,
      y: 0,
      scaleX: 1,
      scaleY: 1,
      resizeDraggable: false,
      width: 0,
      height: 0,
      left: 0,
      top: 0,
      draggable: false,
      showDots: false,
      elementsData: elementsData,
      elmRefs: [],
      elmList: [],
      content: '',
      curEle: null
    }
  },
  mounted() {
    this.dotElms = document.getElementsByClassName('dot')
  },
  methods: {
    download() {
      html2canvas(document.getElementById('my-node'))
      .then(function (canvas) {
        var link = document.createElement('a');
        link.download = 'my-image-name.jpeg';
        link.href = canvas.toDataURL();
        link.click();
      });
    },
    save() {
      let result = []
      let node = document.getElementById('my-node')
      let children1 = node.children
      for (let i = 0; i < children1.length; i++) {
        let target = children1[i].children[0]
        result.push({
          ref: target.dataset.ref,
          type: target.dataset.type,
          content: target.innerText || null,
          style: target.attributes.style.value,
          src: target.dataset.type === 'image' ? target.attributes.src.value : null
        })
      }
      console.log(result)
    },
    addElm(elm) {
      let refName = this.randomStr()
      while (this.elmRefs.indexOf(refName) > -1) {
        refName = this.randomStr()
      }
      this.elmRefs.push(refName)
      this.elmList.push({
        ...elm,
        ref: refName,
        style: `${elm.style}position: absolute; left: 0; top: 0;`
      })
      refName =  this.randomStr()
      this.elmRefs.push(refName)
      this.elmList.push({
        type: 'text',
        content: 'agfdagfdagd',
        ref: refName,
        style: `position: absolute; left: 0; top: 0;width: 50px;font-size: 16px;`
      })
    },
    randomStr() {
      return Math.random().toString(36).slice(-6);
    },
    dblclick(ele) {
      console.log('dblclick-----')
      ele.contenteditable = true
      this.contenteditable = true
      this.showDots = false
    },
    blur(ele) {
      ele.contenteditable = false
      this.contenteditable = false
    },
    mousedown(e, ele) {
      console.log('aaaaa', e)
      let refName = e.target.dataset.ref
      if (!refName && ele.type === 'text') {
        refName = e.path[1].dataset.ref || e.path[2].dataset.ref
      }
      this.isDragging = true
      console.log(this.$refs)
      this.elm = this.$refs[refName][0]
      if (ele.type === 'text') {
        this.fontSize = Number(this.elm.style.fontSize.substring(0, this.elm.style.fontSize.length - 2))
      }
      this.curEle = ele
      this.resetDots()
      this.x = e.x
      this.y = e.y
      this.showDots = true
    },
    mousemove(e) {
      if (!this.isDragging) {
        return
      }
      this.showDots = false
      let left = e.x - this.x
      let top = e.y - this.y
      this.elm.style.left = `${this.elmLeft + left}px`
      this.elm.style.top = `${this.elmTop + top}px`
    },
    mouseup() {
      console.log('mouseup-----')
      // setTimeout(() => {
      //   if (this.contenteditable) {
      //     return
      //   }
      //   this.showDots = true
      // }, 100)
      this.isDragging = false
      this.isDotDragging = false
      if (this.elm) {
        this.resetDots()
      }
      this.showDots = true
    },
    wrapperClick(e) {
      console.log('wrapperClick', e)
      if (!e.target.dataset.ref && !e.target.offsetParent.dataset.ref) {
        this.showDots = false
      }
    },
    wrapperMouseup() {
      this.isDotDragging = false
      this.isDragging = false
      if (this.elm) {
        this.resetDots()
      }
    },
    dotMouseup() {
      this.isDotDragging = false
      this.resetDots()
    },
    wrapperMousemove(e) {
      if (!this.isDotDragging) {
        return
      }
      this.deltaX = e.x - this.dotInitX
      this.deltaY = e.y - this.dotInitY
      if (this.curEle.type === 'text') {
        if (this.dotIndex === 3 || this.dotIndex === 4) {
          this.height = this.elm.children[0].offsetHeight
        }
      }
      if (this.dotIndex === 0) {
        this.scaleValue = (this.width - this.deltaX) / this.width
        this.deltaX = this.width - this.width * this.scaleValue
        this.deltaY = this.height - this.height * this.scaleValue
        this.elm.style.fontSize = `${this.fontSize * this.scaleValue}px`
      }
      else if (this.dotIndex === 2) {
        this.scaleValue = (this.width + this.deltaX) / this.width
        this.deltaX = this.width * this.scaleValue - this.width
        this.deltaY = this.height - this.height * this.scaleValue
        this.elm.style.fontSize = `${this.fontSize * this.scaleValue}px`
      } else if (this.dotIndex === 5) {
        this.scaleValue = (this.width - this.deltaX) / this.width
        this.deltaX = this.width - this.width * this.scaleValue
        this.deltaY = this.height * this.scaleValue - this.height
        this.elm.style.fontSize = `${this.fontSize * this.scaleValue}px`
      } else if (this.dotIndex === 7) {
        this.scaleValue = ((this.width + this.deltaX) / this.width).toFixed(2)
        this.deltaX = this.width * this.scaleValue - this.width
        this.deltaY = this.height * this.scaleValue - this.height
        this.elm.style.fontSize = `${this.fontSize * this.scaleValue}px`
      }
      this.changeDot()
    },
    dotMousedown(e, index) {
      this.isDotDragging = true
      this.dotInitX = e.x
      this.dotInitY = e.y
      this.dotIndex = index
      this.deltaX = 0
      this.deltaY = 0
      this.scaleValue = 1
      if (this.curEle.type === 'text') {
        this.fontSize = Number(this.elm.style.fontSize.substring(0, this.elm.style.fontSize.length - 2))
      }
    },
    resetDots() {
      const elm = this.elm
      this.elmLeft = elm.offsetLeft
      this.elmTop = elm.offsetTop
      let height = this.height = elm.offsetHeight
      let width = this.width = elm.offsetWidth
      const left = this.left = elm.offsetParent.offsetLeft + this.elmLeft
      const top = this.top = elm.offsetParent.offsetTop + this.elmTop
      this.dots = [
        `left: ${left - DOT_WIDTH / 2}px; top: ${top - DOT_WIDTH / 2}px;cursor: nw-resize;`,
        `left: ${left + (width - DOT_WIDTH) / 2}px; top: ${top - DOT_WIDTH / 2}px; cursor: n-resize;`,
        `left: ${left + width - DOT_WIDTH / 2}px; top: ${top - DOT_WIDTH / 2}px; cursor: ne-resize;`,
        `left: ${left - DOT_WIDTH / 2}px; top: ${top + (height - DOT_WIDTH)/2}px; cursor: w-resize;`,
        `left: ${left + width - DOT_WIDTH / 2}px; top: ${top + (height - DOT_WIDTH)/2}px; cursor: e-resize;`,
        `left: ${left - DOT_WIDTH / 2}px; top: ${top + height - DOT_WIDTH / 2}px; cursor: sw-resize;`,
        `left: ${left + (width - DOT_WIDTH) / 2}px; top: ${top + height - DOT_WIDTH / 2}px; cursor: s-resize;`,
        `left: ${left + width - DOT_WIDTH / 2}px; top: ${top + height - DOT_WIDTH / 2}px; cursor: se-resize;`,
      ]
    },
    changeDot() {
      const width = this.width
      const height = this.height
      const deltaX = this.deltaX
      let deltaY = this.deltaY
      if (this.curEle.type === 'text') {
        if (this.dotIndex === 3 ||  this.dotIndex === 4) {
          deltaY = this.elm.children[0].offsetHeight - this.height
        }
      }
      switch(this.dotIndex) {
        case 0:
          this.elm.style.width = `${width - deltaX}px`
          this.elm.style.height = `${height - deltaY}px`
          this.elm.style.left = `${this.elmLeft + deltaX}px`
          this.elm.style.top = `${this.elmTop + deltaY}px`
          this.dotElms[0].style.left =
          this.dotElms[3].style.left =
          this.dotElms[5].style.left = `${this.left + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[0].style.top =
          this.dotElms[1].style.top =
          this.dotElms[2].style.top = `${this.top + deltaY - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          break
        case 1:
          this.elm.style.height = `${height - deltaY}px`
          this.elm.style.top = `${this.elmTop + deltaY}px`
          this.dotElms[0].style.top =
          this.dotElms[1].style.top =
          this.dotElms[2].style.top = `${this.top + deltaY - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          break
        case 2:
          this.elm.style.width = `${width + deltaX}px`
          this.elm.style.height = `${height - deltaY}px`
          this.elm.style.top = `${this.elmTop + deltaY}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[2].style.left =
          this.dotElms[4].style.left =
          this.dotElms[7].style.left = `${this.left + width + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[0].style.top =
          this.dotElms[1].style.top =
          this.dotElms[2].style.top = `${this.top + deltaY - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          break
        case 3:
          this.elm.style.width = `${width - deltaX}px`
          this.elm.style.height = `${height + deltaY}px`
          this.elm.style.left = `${this.elmLeft + deltaX}px`
          this.dotElms[0].style.left =
          this.dotElms[3].style.left =
          this.dotElms[5].style.left = `${this.left + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + height / 2 - DOT_WIDTH / 2}px`
          this.dotElms[5].style.top =
          this.dotElms[6].style.top =
          this.dotElms[7].style.top = `${this.top + height - DOT_WIDTH / 2}px`
          break
        case 4:
          this.elm.style.width = `${width + deltaX}px`
          this.elm.style.height = `${height + deltaY}px`
          this.dotElms[2].style.left =
          this.dotElms[4].style.left =
          this.dotElms[7].style.left = `${this.left + width + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + height / 2 - DOT_WIDTH / 2}px`
          this.dotElms[5].style.top =
          this.dotElms[6].style.top =
          this.dotElms[7].style.top = `${this.top + height - DOT_WIDTH / 2}px`
          break
        case 5:
          this.elm.style.width = `${width - deltaX}px`
          this.elm.style.height = `${height + deltaY}px`
          this.elm.style.left = `${this.elmLeft + deltaX}px`
          this.dotElms[0].style.left =
          this.dotElms[3].style.left =
          this.dotElms[5].style.left = `${this.left + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[5].style.top =
          this.dotElms[6].style.top =
          this.dotElms[7].style.top = `${this.top + height + deltaY - DOT_WIDTH / 2}px`
          break
        case 6:
          this.elm.style.height = `${height + deltaY}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[5].style.top =
          this.dotElms[6].style.top =
          this.dotElms[7].style.top = `${this.top + height + deltaY - DOT_WIDTH / 2}px`
          break
        case 7:
          this.elm.style.width = `${width + deltaX}px`
          this.elm.style.height = `${height + deltaY}px`
          this.dotElms[1].style.left =
          this.dotElms[6].style.left = `${this.left + (width + deltaX) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[2].style.left =
          this.dotElms[4].style.left =
          this.dotElms[7].style.left = `${this.left + width + deltaX - DOT_WIDTH / 2}px`
          this.dotElms[3].style.top =
          this.dotElms[4].style.top = `${this.top + (height + deltaY) / 2 - DOT_WIDTH / 2}px`
          this.dotElms[5].style.top =
          this.dotElms[6].style.top =
          this.dotElms[7].style.top = `${this.top + height + deltaY - DOT_WIDTH / 2}px`
          break
        default:
          break
      }
    },
  }
}
</script>
<style lang="scss" scoped>
.content {
  background-color: transparent;
  color: white;
  border: none;
  outline: none;
}
.text-wrapper {
  text-align: center;
  overflow-wrap: break-word;
  span {
    outline: none;
    user-select: text;
    white-space: pre-wrap;
    overflow-wrap: break-word;
    word-break: break-all;
  }
}
.sidebar {
  width: 200px;
  height: 100vh;
  position: absolute;
  left: 0;
  top: 0;
}
.image-wrapper {
  width: 100%;
  height: 80px;
  margin-top: 10px;
  img {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
}
.wrapper {
  width: 100vw;
  height: 100vh;
  background-color: burlywood;
}
.container {
  position: absolute;
  width: 500px;
  height: 500px;
  background-color: cyan;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  overflow: hidden;
}
.resize {
  // width: 200px;
  // height: 200px;
  position: absolute;

  // background-color: blue;

  &:hover {
    cursor: move;
  }
}
.hover {
  border: 1px solid transparent;
  box-sizing: border-box;
  &:hover {
    border: 1px solid blue;
  }
}
.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: red;
  position: absolute;
}
* {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
img {
  object-fit: fill;
}
</style>