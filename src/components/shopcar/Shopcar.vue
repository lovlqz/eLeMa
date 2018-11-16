<template>
  <div class="shopcar">
    <div class="content">
      <div class="content-left" @click="toggleList">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <span :class="{'highlight':totalCount>0}" class="icon-shopping_cart"></span>
          </div>
          <div class="num" v-show="totalCount>0">{{totalCount}}</div>
        </div>
        <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
        <div class="des">另需配送费￥{{deliveryPrice}}元 </div>
      </div>
      <div class="content-right" @click="goPay">
        <div class="pay" :class="payClass">{{payDesc}}</div>
      </div>
      <div class="ball-container">
        <transition-group name="ball"
              v-on:before-enter="beforeEnter"
              v-on:enter="enter"
              v-on:after-enter="afterEnter">
          <div class="ball" v-for="(ball,index) in balls" :key="index" v-show="ball.show">
            <div class="inner inner-ball"></div>
          </div>
        </transition-group>
      </div>
      <transition name="fold">
        <div class="shopcar-list" v-show="listShow">
          <div class="list-header">
            <div class="title">购物车</div>
            <span class="empty" @click="emptyCart">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="(food,index) in selectGoods" :key="index">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price*food.count}}</span>
                </div>
                <div class="carcontrol-wrapper">
                  <car-control :food="food"></car-control>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" @click="hideListMask" v-show="listShow"></div>
    </transition>
  </div>
</template>
<script>
import carControl from './../cartcontrol/CarControl';
import BScroll from 'better-scroll';
  export default {
    props: {
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      },
      selectGoods: {
        type: Array,
        default() {
          return [];
        }
      }
    },
    data() {
      return {
        balls: [
          {show: false},
          {show: false},
          {show: false},
          {show: false},
          {show: false}
        ],
        dropBalls: [],
        fold: true
      };
    },
    computed: {
      totalPrice() {
        let total = 0;
        this.selectGoods.forEach(food => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectGoods.forEach(food => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return `￥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差￥${diff}元起送`;
        } else {
          return '去结算';
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },
      listShow(e) {
        let self = this;
        if (!this.totalCount) {
          self.fold = true;
          return false;
        }
        let show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              var listContentDom = this.$refs.listContent;
              self.scroll = new BScroll(listContentDom, {
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
          return false;
        }
        this.fold = !this.fold;
      },
      drop(el) {
        for (let i = 0; i < this.balls.length; i++) {
            let ball = this.balls[i];
            if (!ball.show) {
                ball.show = true;
                ball.el = el;
                this.dropBalls.push(ball);
                return false;
            }
        }
      },
      beforeEnter(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let inner = el.querySelector('.inner-ball');
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 48);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
            el.style.transform = `translate3d(0,${y}px,0)`;
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
            inner.style.transform = `translate3d(${x}px,0,0)`;
          }
        }
      },
      enter(el) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.querySelector('.inner-ball');
          inner.style.webkitTransform = 'translate3d(28px,0,0)';
          inner.style.transform = 'translate3d(28px,0,0)';
        });
      },
      afterEnter(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      },
      emptyCart() {
        this.selectGoods.forEach((food) => {
          food.count = 0;
        });
      },
      hideListMask() {
        this.fold = true;
      },
      goPay() {
        if (this.totalPrice < this.minPrice) {
          return false;
        }
        window.alert('支付成功');
      }
    },
    components: {
      carControl
    }
  };
</script>
<style lang="stylus">
@import './../../common/stylus/mixin';
  .shopcar {
    position fixed
    left 0
    bottom 0
    width 100%
    height 48px
    z-index 50
    .content {
      display flex
      font-size 0
      color rgba(255,255,255,0.4)
      .content-left {
        flex 1
        z-index 60
        background #141d27
        .logo-wrapper {
          display inline-block
          position relative
          top -10px
          margin 0 12px
          padding 6px
          width 56px
          height 56px
          border-radius 50%
          box-sizing border-box
          vertical-align top
          background #141d27
          .logo {
            width 100%
            height 100%
            border-radius 50%
            background #2b343c
            text-align center
            &.highlight {
              background rgb(0,120,220)
            }
            .icon-shopping_cart {
              color #80858a
              font-size 24px
              line-height 44px
              &.highlight {
                color rgb(255,255,255)
              }
            }
          }
          .num {
            position absolute
            top 0
            right 0
            width 24px
            height 16px
            line-height 16px
            text-align center
            border-radius 16px
            font-size 9px
            font-weight 700
            color #ffffff
            background rgb(240,20,20)
            box-shadow 0 4px 8px 0 rgba(0,0,0,0.4)
          }
        }
        .price {
          display inline-block
          vertical-align top
          line-height 24px
          margin-top 12px
          box-sizing border-box
          padding-right 12px
          border-right 1px solid rgba(255,255,255,0.1)
          font-size 16px
          font-weight 700
          &.highlight {
            color rgb(255,255,255)
          }
        }
        .des{
          display inline-block
          vertical-align top
          line-height 24px
          margin 12px 0 0 12px
          font-size 10px
        }
      }
      .content-right {
        flex 0 0 80px
        width 80px
        z-index 60
        background #2b333b
        .pay {
          font-size 12px
          height 48px
          line-height 48px
          text-align center
          font-weight 700
          background #2b333b
          &.not-enough {
            background #2b333b
          }
          &.enough {
            background #00b43c
            color rgb(255,255,255)
          }
        }
      }
      .ball-container {
        position relative
        .ball,.inner {
          position fixed
          left 32px
          bottom 22px
          z-index 200
          width 16px
          height 16px
          border-radius 50%
        }
        .inner {
          left 0
          bottom 0
        }
        .ball-enter-active, .ball-leave-active{
          transition: all 0.4s linear;
          .inner{
            background rgb(0,160,220)
            transition: all 0.7s cubic-bezier(.07,.45,.25,1.45);
          }
        }
      }
      .shopcar-list {
        position absolute
        top 0
        left 0
        z-index 10
        opacity 1
        width 100%
        transform translate3d(0,-100%,0)
        .list-header {
          height 40px
          line-height 40px
          padding  0 18px
          background #f3f5f7
          border-bottom 1px solid rgba(7,17,27,0.1)
          .title {
            float left
            font-size 14px
            color rgb(7,17,27)
          }
          .empty {
            float right
            font-size 12px
            color rgb(0,160,220)
          }
        }
        .list-content {
          padding  0 18px
          max-height  217px
          overflow hidden
          background #ffffff
          .food {
            position relative
            padding 12px 0
            box-sizing border-box
            border-1px(rgba(7,17,27,0.1))
            .name {
              line-height 24px
              font-size 14px
              color rgb(7,17,27)
            }
            .price {
              position absolute
              right 90px
              bottom 12px
              line-height 24px
              font-size 14px
              font-weight 700
              color rgb(240,20,20)
            }
            .carcontrol-wrapper {
              position absolute
              right 0
              bottom 6px
            }
          }
        }
        &.fold-enter {
          transform translate3d(0,0,0)
        }
        &.fold-enter-active, &.fold-leave-active {
            transition: all 0.6s
            z-index 10
            opacity 0
        }
        &.fold-enter-to {
          opacity 1
          transform translate3d(0,-100%,0)
        }
        &.fold-leave-to {
          opacity 0
          transform translate3d(0,0,0)
        }
      }
    }
    .list-mask {
      position fixed
      left 0
      top 0
      width 100%
      height 100%
      z-index -1
      backdrop-filter blur(10px)
      background rgba(7, 17, 27, 0.6)
      &.fade-transition {
        opacity 1
        background rgba(7, 17, 27, 0.6)
      }
      &.fade-enter {
        transform translate3d(0, 0, 0)
      }
      &.fade-enter-active, &.fade-leave-active {
          background rgba(7, 17, 27, 0)
          opacity 0
      }
    }
  }
</style>
