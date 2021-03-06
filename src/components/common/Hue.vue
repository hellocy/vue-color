<template>
  <div class="c-hue {{direction}}">
    <div class="vc-container" v-el:container
      @mousedown="handleMouseDown"
      @touchmove="handleChange"
      @touchstart="handleChange">
      <div class="pointer" :style="{top: pointerTop, left: pointerLeft}">
        <slot><div class="picker"></div></slot>
      </div>  
    </div>
  </div>
</template>

<script>
export default {
  name: 'Hue',
  props: {
    colors: Object,
    onChange: Function,
    direction: {
      type: String,
      // [horizontal | vertical]
      default: 'horizontal'
    }
  },
  computed: {
    pointerTop () {
      if (this.direction === 'vertical') {
        return -((this.colors.hsl.h * 100) / 360) + 100 + '%'
      } else {
        return 0
      }
    },
    pointerLeft () {
      if (this.direction === 'vertical') {
        return 0
      } else {
        return (this.colors.hsl.h * 100) / 360 + '%'
      }
    }
  },
  methods: {
    handleChange (e, skip) {
      !skip && e.preventDefault()

      var container = this.$els.container
      var containerWidth = container.clientWidth
      var containerHeight = container.clientHeight
      var left = (e.pageX || e.touches[0].pageX) - (container.getBoundingClientRect().left + window.pageXOffset)
      var top = (e.pageY || e.touches[0].pageY) - (container.getBoundingClientRect().top + window.pageYOffset)
      var h
      var percent

      if (this.direction === 'vertical') {
        if (top < 0) {
          h = 359
        } else if (top > containerHeight) {
          h = 0
        } else {
          percent = -(top * 100 / containerHeight) + 100
          h = (360 * percent / 100)
        }

        if (this.colors.hsl.h !== h) {
          this.onChange({
            h: h,
            s: this.colors.hsl.s,
            l: this.colors.hsl.l,
            a: this.colors.hsl.a,
            source: 'hsl'
          })
        }
      } else {
        if (left < 0) {
          h = 0
        } else if (left > containerWidth) {
          h = 359
        } else {
          percent = left * 100 / containerWidth
          h = (360 * percent / 100)
        }

        if (this.colors.hsl.h !== h) {
          this.onChange({
            h: h,
            s: this.colors.hsl.s,
            l: this.colors.hsl.l,
            a: this.colors.hsl.a,
            source: 'hsl'
          })
        }
      }
    },
    handleMouseDown (e) {
      this.handleChange(e, true)
      window.addEventListener('mousemove', this.handleChange)
      window.addEventListener('mouseup', this.handleMouseUp)
    },
    handleMouseUp (e) {
      this.unbindEventListeners()
    },
    unbindEventListeners () {
      window.removeEventListener('mousemove', this.handleChange)
      window.removeEventListener('mouseup', this.handleMouseUp)
    }
  }
}
</script>

<style lang="stylus">
.c-hue
  position absolute
  top 0px
  right 0px
  bottom 0px
  left 0px
  border-radius 2px
  &.horizontal
    background linear-gradient(to right, #f00 0%, #ff0 17%, #0f0 33%, #0ff 50%, #00f 67%, #f0f 83%, #f00 100%)
  &.vertical
    background linear-gradient(to top, #f00 0%, #ff0 17%, #0f0 33%, #0ff 50%, #00f 67%, #f0f 83%, #f00 100%)
  .vc-container
    margin 0 2px
    position relative
    height 100%
  .pointer
    z-index 2
    position absolute
  .picker
    margin-top 1px
    width 4px
    border-radius 1px
    height 8px
    box-shadow 0 0 2px rgba(0, 0, 0, .6)
    background #fff
    transform translateX(-2px)
</style>