<template lang="html">
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'heightlight':totalCount > 0}">
            <i class="icon-shopping_cart" :class="{'heightlight':totalCount > 0}"></i>
          </div>
          <div class="num" v-show="totalCount > 0">
            {{totalCount}}
          </div>
        </div>
        <div class="price" :class="{'heightlight':totalPrice > 0}">
          ¥{{totalPrice}}
        </div>
        <div class="desc">
          另需配送费¥{{deliveryPrice}}元
        </div>
      </div>
      <div class="content-right" @click.stop.prevent="pay">
        <div class="pay" :class="pagClass">
          {{payDesc}}
        </div>
      </div>
      <div class="ball-container">
        <div v-for="ball in balls" :key="ball.id">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook  icon-add_circle"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click.stop.prevent="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="(list,index) in selectFoods" :key="index">
                <span class="name">{{list.name}}</span>
                <span class="price">¥{{list.count * list.price}}</span>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="list" @add="addFood"/>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>

    </div>
    <transition name="fade">
      <div class="lsit-mask" v-show="listShow" @click="hideList"></div>
    </transition>
  </div>
</template>

<script>
import cartcontrol from 'components/cartcontrol/cartcontrol'
import BScroll from 'better-scroll'

export default {
  data() {
    return {
      fold: true,
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: []
    }
  },
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [];
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  computed: {
    totalPrice() {
      let total = 0;
      this.selectFoods.forEach((food) => {
        total += food.price * food.count;
      });
      return total;
    },
    totalCount() {
      let count = 0;
      this.selectFoods.forEach((food) => {
        count += food.count;
      });
      return count;
    },
    payDesc() {
      if (this.totalCount === 0) {
        return '¥' + this.minPrice + '元起送';
      } else if (this.minPrice > this.totalPrice) {
        return '还差¥' + (this.minPrice - this.totalPrice) + '起送';
      } else {
        return '去结算';
      }
    },
    pagClass() {
      if (this.minPrice > this.totalPrice) {
        return 'not-enough';
      } else {
        return 'enough';
      }
    },
    listShow() {
      if (!this.totalCount) {
        this.fold = true;
        return false;
      }
      let show = !this.fold;
      if (show) {
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.listContent, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      }
      return show;
    }
  },
  methods: {
    toggleList() {
      if (!this.totalCount) {
        return;
      } else {
        this.fold = !this.fold;
      }
    },
    drop(el) {
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i];
        if (!ball.show) {
          ball.show = true;
          ball.el = el;
          this.dropBalls.push(ball);
          return;
        }
      }
    },
    beforeDrop(el) {
      let count = this.balls.length;
      while (count--) {
        let ball = this.balls[count];
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect();
          let x = rect.left - 32;
          let y = -(window.innerHeight - rect.top - 22);
          el.style.display = '';
          el.style.webkitTransform = 'translate3d(0,' + y + 'px,0)';
          el.style.transform = 'translate3d(0,' + y + 'px,0)'
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(' + x + 'px,0,0)';
          inner.style.transform = 'translate3d(' + x + 'px,0,0)';
        }
      }
    },
    dropping(el) {
      /* eslint-disable no-unused-vars */
      let re = el.offsetHeight;
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0,0,0)';
        el.style.transform = 'translate3d(0,0,0)'
        let inner = el.getElementsByClassName('inner-hook')[0];
        inner.style.webkitTransform = 'translate3d(0,0,0)';
        inner.style.transform = 'translate3d(0,0,0)';
      });
    },
    afterDrop(el) {
      let ball = this.dropBalls.shift();
      if (ball) {
        ball.show = false;
        el.style.display = 'none';
      }
    },
    addFood(target) {
      this.drop(target);
    },
    empty() {
      this.selectFoods.forEach ((food) => {
        food.count = 0;
      })
    },
    hideList() {
      this.fold = true
    },
    pay() {
      if (this.totalPrice < this.minPrice) {
        return;
      }
      window.alert('需支付支付¥' + this.totalPrice + '元');
    }
  },

  components: {
    cartcontrol: cartcontrol
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
@import "../../common/stylus/mixin.styl"

  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      background: #141d27
      color:rgba(255, 255, 255, 0.4)
      .content-left
        flex: 1
        font-size: 0
        .logo-wrapper
          display: inline-block
          position: relative
          top: -10px
          margin: 0 18px
          padding: 6px
          width: 56px
          height: 56px
          box-sizing: border-box
          border-radius: 50%
          vertical-align: top
          background: #141d27
          .logo
            font-size: 27px
            width: 100%
            height: 100%
            border-radius: 50%
            background: #2b343c
            text-align: center
            &.heightlight
              background: rgb(0, 160, 220)
            .icon-shopping_cart
              font-size: 24px
              line-height: 44px
              font-weight: 700
              &.heightlight
                color: rgb(255, 255, 255);
         .num
           position: absolute
           top: 0
           right: 0
           width: 24px
           height: 16px
           line-height: 16px
           border-radius: 12px
           text-align: center
           font-size: 9px
           font-weight: 700
           color: rgb(255, 255, 255)
           background: rgb(240, 20, 20)
           box-shadow: 0 4px 8px 0px rgba(0, 0, 0, 0.4)

        .price
          display: inline-block
          vertical-align: top
          font-weight: 700
          font-size: 16px
          line-height: 24px
          margin: 12px 0
          padding-right: 12px
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          &.heightlight
            color: #FFF
        .desc
          display: inline-block
          vertical-align: top
          font-weight: 400
          font-size: 10px
          line-height: 24px
          margin: 12px 0
          padding-left: 12px
      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          height: 48px
          line-height: 48px
          text-align: center
          font-weight: 700
          font-size: 12px
          &.not-enough
            background: #2b343c
          &.enough
            background: #00b43c
            color: #FFF
      .ball-container
        .ball
          position: fixed
          left: 32px
          bottom: 22px
          z-index: 200
          transition: all .5s cubic-bezier(0.46, -0.41, 0.83, 0.67)
          .inner
            width: 16px
            height: 16px
            border-radius: 50%
            font-size: 16px
            background: rgb(0, 160, 220)
            transition: all .5s linear
      .shopcart-list
        position: absolute
        top: 0
        left: 0
        width: 100%
        z-index: -1
        background: rgb(7, 17, 27, 0.6)
        transform: translate3d(0, -100%, 0)
        &.fold-enter-active, &.fold-leave-active
          transition: all 0.5s
        &.fold-enter, &.fold-leave-to
          transform: translate3d(0, 0, 0)
        .list-header
          height: 40px
          line-height: 40px
          padding: 0 18px
          background: #f3f5f7
          border-bottom: 1px solid  rgba(7, 17, 27, 0.1)
          .title
            display: inline-block
            line-height: 40px
            font-weight: 200
            font-size: 14px
            color: rgb(7, 17, 27)
          .empty
            float: right
            line-height: 40px
            font-size: 12px
            color: rgb(0, 160, 220)
        .list-content
          background: #fff
          padding: 0 18px
          overflow: hidden
          max-height: 217px
          .food
            position: relative
            padding: 12px 0
            box-sizing: border-box
            border-1px(rgba(7, 17, 27, 0.1))
            .name
              line-height: 24px
              font-size: 14px
              color: rgb(7, 17, 27)
            .price
              position: absolute
              right: 90px
              bottom: 12px
              line-height: 24px
              font-weight: 700
              font-size: 14px
              color: rgb(240, 20, 20)
            .cartcontrol-wrapper
              position: absolute
              right: 0
              bottom: 6px
    .lsit-mask
      position: fixed
      top: 0
      left: 0
      z-index: -3
      width: 100%
      height: 100%
      color: rgb(7, 17, 27, 0.6)
      backdrop-filter: blur(10px)
      &.fade-enter-active, &.fade-leave-active
        transition: all 0.5
      &.fade-enter, &.fade-leave-to
        opacity: 0
        color: rgb(7, 17, 27, 0)

</style>
