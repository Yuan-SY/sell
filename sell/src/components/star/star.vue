<template lang="html">
  <div class="star" :class="starType">
    <span v-for="(itemClass,index) in itemClasses" :key="index" :class="itemClass" class="starItem"></span>
  </div>
</template>

<script>

const LENGTH = 5;
const CLS_ON = 'on';
const CLS_HALF = 'half';
const CLS_OFF = 'off';

export default {
  props: {
    size: {
      type: Number
    },
    score: {
      type: Number
    }
  },
  computed: {
    starType() {
      return 'star-' + this.size;
    },
    itemClasses() {
      let result = [];
      let score = Math.floor(this.score * 2) / 2;
      let hasDecimal = score % 1 !== 0;
      let integer = Math.floor(score);
      for (let i = 0; i < integer; i++) {
        result.push(CLS_ON);
      }
      if (hasDecimal) {
        result.push(CLS_HALF);
      }
      while (result.length < LENGTH) {
        result.push(CLS_OFF);
      }
      return result;
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet" scoped>
@import "../../common/stylus/mixin.styl"

  .star
    font-size: 0
    .starItem
      display: inline-block;
      background-repeat: no-repeat;
    &.star-48
      .starItem
        width: 20px
        height: 20px
        margin-right: 22px
        background-size: 20px 20px
        &:last-child
          margin-right: 0
        &.on
          bg-img("star48_on")
        &.half
          bg-img("star48_half")
        &.off
          bg-img("star48_off")
    &.star-36
      .starItem
        width: 15px
        height: 15px
        margin-right: 16px
        background-size: 15px 15px
        &:last-child
          margin-right: 0
        &.on
          bg-img("star36_on")
        &.half
          bg-img("star36_half")
        &.off
          bg-img("star36_off")
    &.star-24
      .starItem
        width: 10px
        height: 10px
        margin-right: 3px
        background-size: 10px 10px
        &:last-child
          margin-right: 0
        &.on
          bg-img("star24_on")
        &.half
          bg-img("star24_half")
        &.off
          bg-img("star24_off")

</style>
