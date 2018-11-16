<template>
  <div class="carcontrol">
    <transition name="moveout">
      <div class="car-decrease icon-remove_circle_outline"
         v-show="food.count>0" @click.stop.prevent="desCar"></div>
    </transition>
    <div class="car-count" v-show="food.count>0">{{food.count}}</div>
    <transition name="movein">
      <div class="car-add icon-add_circle" @click.stop.prevent="addCar"></div>
    </transition>
  </div>
</template>
<script>
  import Vue from 'vue';
  export default {
    props: {
      food: {
        type: Object
      }
    },
    methods: {
      addCar(event) {
        if (!this.food.count) {
          Vue.set(this.food, 'count', 1);
        } else {
          this.food.count += 1;
        };
        this.$emit('cart-add', event.target);
      },
      desCar() {
        if (this.food.count) {
          this.food.count -= 1;
        }
      }
    }
  };
</script>
<style lang="stylus">
  .carcontrol{
    font-size 0
    .moveout-enter,.moveout-leave-to{
      &.car-decrease{
        transform rotate(180deg) translate3d(-24px,0,0)
        opacity 0
      }
    }
    .car-decrease, .car-add{
      transition all 0.4s linear
      display inline-block
      line-height 24px
      font-size 20px
      padding 6px
      color rgb(0,160,220)
    }
    .car-decrease{
      opacity 1
      transform rotate(0deg)
    }
    .car-count{
      display inline-block
      vertical-align top
      width 12px
      padding-top 6px
      line-height 24px
      text-align center
      font-size 10px
      color rgb(147,153,159)
    }
  }
</style>
