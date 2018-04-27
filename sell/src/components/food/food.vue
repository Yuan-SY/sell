<template lang="html">
  <transition name="move">
    <div class="food" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="img-header">
          <img :src="food.image" :alt="food.name" width="">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="tilte">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">¥{{food.price}}</span><span class="old-price" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
          </div>

        </div>
        <div class="cartcontrol-wrapper">
          <cartcontrol :food="food" @add="addFood"></cartcontrol>
        </div>
        <transition name="fade">
          <div class="buy" v-show="!food.count || food.count ===0" @click.stop.pravent="addFirest">
            加入购物车
          </div>
        </transition>
      </div>
    </div>
  </transition>
</template>

<script>
import BScroll from 'better-scroll'
import cartcontrol from 'components/cartcontrol/cartcontrol'
import Vue from 'vue'


export default {
  data () {
    return {
      showFlag: false
    }
  },
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    show() {
      this.showFlag = true;
      this.$nextTick(() => {
        if(!this.scroll) {
          this.scroll = new BScroll(this.$refs.food, {
            click: true
          })
        } else {
          this.scroll.refresh();
        }
      });
    },
    hide() {
      this.showFlag = false;
    },
    addFirest(event) {
      if (!event._constructed) {
        return;
      }
      this.$emit('add', event.target);
      Vue.set(this.food, 'count', 1);
    },
    addFood(target) {
      this.$emit('add', target);
    }
  },
  components: {
    cartcontrol: cartcontrol
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
  .food
    position: fixed
    top: 0
    left: 0
    bottom: 48px
    z-index: 30
    width: 100%
    height: 100%
    background: #fff
    &.move-enter-active,&.move-leave-active
      transition: all 0.4s linear
    &.move-enter,&.move-leave-to
      transform: translate3d(100%,0,0)
    .img-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100%
      img
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
      .back
        position: absolute
        top: 10px
        left: 0
        .icon-arrow_lift
          display: block
          padding: 10px
          font-size: 20px
          color: #CCC
    .content
      padding: 18px
      .tilte
        margin-bottom: 8px
        font-weight: 700
        font-size: 14px
        line-height: 14px
        color: rgb(7, 17, 27)
      .detail
        margin-bottom: 18px
        line-height: 10px
        height: 10px
        font-size: 0
        .sell-count, .rating
          display: inline-block
          line-height: 10px
          font-size: 10px
          color: rgb(147, 153, 159)
        .rating
          margin-left: 12px
      .price
        margin-top: 18px
        .now
          margin-right: 8px
          line-height: 24px
          font-weight: 700
          font-size: 14px
          color: rgb(240, 20, 20)
        .old-price
          line-height: 24px
          font-weight: normal
          font-size: 10px
          color: rgb(147, 153, 159)
          text-decoration: line-through
    .cartcontrol-wrapper
      position: absolute
      right: 12px
      bottom: 12px
    .buy
      position: absolute
      right: 18px
      bottom: 20px
      z-index: 10
      line-height: 24px
      height: 24px
      width: 79px
      padding: 0 12px
      text-align: center
      box-sizing: border-box
      border-radius: 12px
      font-size: 10px
      color: rgb(255, 255, 255)
      background: rgb(0, 160, 220)
      &.fade-enter-active, &.fade-leave-active
        transition: all 0.2s
      &.fade-enter, &.fade-leave-to
        opacity: 0

</style>
