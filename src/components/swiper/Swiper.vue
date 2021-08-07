<template>
  <div id="swiper-container" ref="swiper" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
    <div id="swiper">
      <slot name="slide"></slot>
    </div>
    <div class="indicator">
      <slot name="indicator" v-if="showIndicator && slideCount>1">
        <div
            v-for="(item,index) in slideCount"
            class="point-item"
            :class="{ active: index === currentIndex-1}" ></div>
      </slot>
    </div>
  </div>
</template>

<script>


export default {
  name: "Swiper",
  props:{
    showIndicator: {
      type: Boolean,
      default: true,
    },   //
    moveRatio: {
      type: Number,
      default: .25,
    },
    animDuration: {
      type: Number,
      default: 1500,
    },
    interval: {
      type: Number,
      default: 3000
    }
  },
  data(){
    return {
      slideCount: 0,    //轮播图包含的图片总数
      currentIndex: 1,   //当前的图片下标
      totalWidth: 0,  //轮播图总宽度
      swiperStyle: {},   //轮播图样式
      scrolling: false,  //当前正在滚动
    }
  },
  methods:{
    setTransform(position){
      this.swiperStyle.transform = `translate(${position}px,0)`;
    },
    handleDom(){
      let swiperEl = document.querySelector('#swiper')
      let slideEls = swiperEl.getElementsByClassName('slide')
      this.slideCount = slideEls.length
      if(this.slideCount>1){
        let firstCloneSlideEl = slideEls[0].cloneNode(true)
        let lastCloneSlideEl = slideEls[this.slideCount-1].cloneNode(true)

        swiperEl.appendChild(firstCloneSlideEl)
        swiperEl.insertBefore(lastCloneSlideEl,slideEls[0])

        this.totalWidth = swiperEl.offsetWidth
        this.swiperStyle = swiperEl.style
      }

      this.setTransform(-this.currentIndex*this.totalWidth)
    },
    startTimer(){
      this.playTimer = setInterval(()=>{
        this.currentIndex++;
        this.scrollContent(-this.currentIndex*this.totalWidth)
      },this.interval)
    },
    stopTimer(){
      clearInterval(this.playTimer )
    },
    scrollContent(currentPosition){
      this.scrolling = true;
      this.swiperStyle.transition = 'transform ' + this.animDuration+'ms'
      this.setTransform(currentPosition)
      this.checkIndex()
      this.scrolling = false;
    },
    checkIndex(){
      setTimeout(()=>{
        this.swiperStyle.transition = '0ms';
        if (this.currentIndex >= this.slideCount + 1) {
          this.currentIndex = 1;
          this.setTransform(-this.currentIndex * this.totalWidth);
        }else if (this.currentIndex <= 0) {
            this.currentIndex = this.slideCount;
            this.setTransform(-this.currentIndex * this.totalWidth);
        }
      },this.animDuration)

    },
    touchStart(e){
      if (this.scrolling) return;
      this.stopTimer()
      this.startPositionX = e.touches[0].pageX
    },
    touchMove(e){
      this.currentPositionX = e.touches[0].pageX;
      this.deltaX = this.currentPositionX - this.startPositionX;
      let currentPosition = -this.currentIndex * this.totalWidth;
      let moveDistance = this.deltaX + currentPosition;
      this.setTransform(moveDistance);
    },
    touchEnd(e){
      // 1.获取移动的距离
      let movedPosition = Math.abs(this.deltaX);
      // 2.判断最终的距离
      if (this.deltaX === 0) {
        return
      } else if (this.deltaX > 0 && movedPosition > this.totalWidth * this.moveRatio) { // 右边移动超过0.5
        this.currentIndex--
      } else if (this.deltaX < 0 && movedPosition > this.totalWidth * this.moveRatio) { // 向左移动超过0.5
        this.currentIndex++
      }
      // 3.移动到正确的位置
      this.scrollContent(-this.currentIndex * this.totalWidth);

      this.startTimer()
    }
  },
  mounted(){
    this.handleDom()
    this.startTimer()
  }
}
</script>


<style scoped>
  #swiper-container {
    display: flex;
    position: relative;
    overflow: hidden;
  }
  #swiper{
    display: flex;
  }
  .indicator{
    display: flex;
    width: 100%;
    justify-content: center;
    position:absolute;
    bottom: 8px;
  }
  .point-item{
    box-sizing: border-box;
    width: 8px;
    height: 8px;
    border-radius: 4px;
    background-color: #fff;
    line-height: 8px;
    text-align: center;
    font-size: 12px;
    margin: 0 5px;
  }
  .active{
    background-color: rgba(212,62,46,1.0);
  }
</style>