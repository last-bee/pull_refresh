<template>
  <div class="pull-refresh">
    <div 
      class="pull-refresh-track"
      :style="style"
      @touchstart="onTouchStart"
      @touchmove="onTouchMove"
      @touchend="onTouchEnd"
      @touchcancel="onTouchEnd"
    >
    <div class='list'>

    </div>
    </div>
  </div>
</template>

<script>
  export default {
    props:{
      headHeight: {
        type: Number,
        default: 50
      },
      animationDuration: {
        type: Number,
        default: 300
      }
    },
    data(){
      return{
        status: 'normal',
        height: 0,
        duration: 0
      }
    },
    computed:{
      style() {
        return {
          transition: `${this.duration}ms`,
          transform: `translate3d(0,${this.height}px, 0)`
        };
      }
    },
    mounted(){
       this.scrollEl = this.getScrollEventTarget(this.$el);
    },
    methods:{
      touchStart(event) {
        this.direction = '';
        this.deltaX = 0;
        this.deltaY = 0;
        this.offsetX = 0;
        this.offsetY = 0;
        this.startX = event.touches[0].clientX;
        this.startY = event.touches[0].clientY;
      },
      touchMove(event) {
        const touch = event.touches[0];
        this.deltaX = touch.clientX - this.startX;
        this.deltaY = touch.clientY - this.startY;
        this.offsetX = Math.abs(this.deltaX);
        this.offsetY = Math.abs(this.deltaY);
        this.direction = this.offsetX > this.offsetY ? 'horizontal' : this.offsetX < this.offsetY ? 'vertical' : '';
      },
      onTouchStart(event){
        console.log(this.getCeiling())
        if(this.getCeiling()){
          this.direction = 0;
          this.touchStart(event);
        }
      },
      onTouchMove(event){
        this.touchMove(event);
        if (!this.ceiling && this.getCeiling()) {
          this.duration = 0;
          this.startY = event.touches[0].clientY;
          this.deltaY = 0;
        }
        console.log(this.deltaY )
        if (this.ceiling && this.deltaY >= 0) {
          console.log(this.direction)
          if (this.direction === 'vertical') {
            // console.log(this.ease(this.deltaY))
            this.getStatus(this.ease(this.deltaY));
            event.preventDefault();
          }
        }
      },
      ease(height) {
        const  headHeight  = this.headHeight;
        return height < headHeight
          ? height
          : height < headHeight * 2
            ? Math.round(headHeight + (height - headHeight) / 2)
            : Math.round(headHeight * 1.5 + (height - headHeight * 2) / 4);
      },
      getStatus(height, isLoading) {
        this.height = height;

        const status = isLoading
          ? 'loading' : height === 0
            ? 'normal' : height < this.headHeight
              ? 'pulling' : 'loosing';

        if (status !== this.status) {
          this.status = status;
        }
      },
      onTouchEnd(){
        if (this.ceiling && this.deltaY) {
          this.duration = this.animationDuration;
          if (this.status === 'loosing') {
            this.getStatus(this.headHeight, true);
          } else {
            this.getStatus(0);
          }
        }
      },
      getCeiling() {
        this.ceiling = this.getScrollTop(this.scrollEl) === 0;
        return this.ceiling;
      },
      getScrollEventTarget(element, rootParent = window) {
        let currentNode = element;
        while (currentNode && currentNode.tagName !== 'HTML' && currentNode.tagName !== 'BODY' && currentNode.nodeType === 1 && currentNode !== rootParent) {
          const overflowY = window.getComputedStyle(currentNode).overflowY;
          if (overflowY === 'scroll' || overflowY === 'auto') {
            return currentNode;
          }
          currentNode = currentNode.parentNode;
        }
        return rootParent;
      },
      getScrollTop(element) {
          return 'scrollTop' in element ? element.scrollTop : element.pageYOffset;
      },
    }
  }
</script>

<style scoped lang='stylus'>
.pull-refresh-track
  position relative
  .list
    width:100%
    height 200px
    background #dddddd
</style>