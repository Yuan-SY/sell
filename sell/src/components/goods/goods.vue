<template lang="html">
  <div class="goods">
    <div class="menus-wrapper" ref="menuWrapper">
      <ul class="menus-items">
        <li v-for="(item,index) in goods" :key="index" class="menus-item"
         :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="(item,index) in goods" class="food-list food-list-hook" :key="index">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="(food,index) in item.foods" class="food-item border-1px" :key="index" @click="selectFood(food,$event)">
              <div class="icon">
                <img :src="food.icon" :alt="food.name" width="57" height="57">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="description">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span class="rating">好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now-price">¥{{food.price}}</span><span v-show="food.oldPrice" class="old-price"><del>¥{{food.oldPrice}}</del></span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food" @add="addFood"/>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"
      :selectFoods="selectFoods" ref="shopcart"/>
    <food :food="selectedFood" ref="food" @add="addFood"></food>
  </div>
</template>

<script>
import shopcart from 'components/shopcart/shopcart'
import cartcontrol from 'components/cartcontrol/cartcontrol'
import food from 'components/food/food'

import BScroll from 'better-scroll'
const ERR_OK = 0;
export default {
  data() {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    };
  },
  components: {
    shopcart: shopcart,
    cartcontrol: cartcontrol,
    food: food
  },
  computed: {
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i;
        }
      }
      return 0;
    },
    selectFoods() {
      let foods = [];
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food);
          }
        });
      });

      return foods;
    }
  },
  props: {
    seller: {
      type: Object
    }
  },
  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    this.$http.get('api/goods').then((response) => {
      response = response.body;
      if (response.errno === ERR_OK) {
        this.goods = response.data;
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      }
    });
  },
  methods: {
    _initScroll() {
      this.meunScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });

      this.foodScroll = new BScroll(this.$refs.foodsWrapper, {
        probeType: 3,
        click: true
      });

      this.foodScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y));
      })
    },
    _calculateHeight() {
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    },
    selectMenu(index, event) {
      if (!event._constructed) {
        return;
      }
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
      let el = foodList[index];
      this.foodScroll.scrollToElement(el, 300);
    },
    addFood(target) {
      this._drop(target);
    },
    _drop(target) {
      // 体验优化，异步执行
      this.$nextTick(() => {
        this.$refs.shopcart.drop(target);
      });
    },
    selectFood(food, event) {
      if (!event._constructed) {
        return;
      }
      this.selectedFood = food;
      this.$refs.food.show();
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
@import "../../common/stylus/mixin.styl"

  .goods
    display: flex
    position: absolute
    width: 100%
    top: 174px
    bottom: 46px
    overflow: hidden
    .menus-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menus-item
        display: table
        height: 54px
        width: 56px
        line-height: 14px
        font-size: 0
        padding: 0 12px
        &.current
          position: relative
          margin-top: -1px
          z-index: 10
          background: #FFF
          font-weight: 700
          .text
            border-none()
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
          border-1px(rgba(7, 17, 27, 0.1))
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-repeat: no-repeat
          background-size: 12px 12px
          &.decrease
            bg-img("decrease_3")
          &.discount
            bg-img("discount_3")
          &.special
            bg-img("special_3")
          &.invoice
            bg-img("invoice_3")
    .foods-wrapper
      flex: 1
      .title
        height: 26px
        padding-left: 14px
        vertical-align: top
        line-height: 26px
        background: #f3f5f7
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153 159)
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            font-size: 14px
            color: rgb(7, 17, 27)
            height: 14px
            line-height: 14px
            margin: 2px 0 8px 0
          .description, .extra
            font-size: 10px
            color: rgb(147, 153, 159)
            line-height: 10px
          .description
            margin: 8px 0
            line-height: 12px
          .extra
            line-height: 10px
            .count
              display: inline-block
              margin-right: 12px
            .rating
              display: inline-block
          .price
            font-weight: 700
            line-height: 24px
            .now-price
              font-size: 14px
              color: rgb(240, 20, 20)
              margin-right: 8px
            .old-price
              font-size: 10px
              color: rgb(147, 153, 159);
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 10px


</style>
