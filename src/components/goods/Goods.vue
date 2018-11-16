<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li class="menu-item" v-for="(item, index) in goods" :key="index"
        :class="{'current': currentIndex === index}" @click="selectMenu(index)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="goods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="index" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="_selectFood(food, $event)" v-for="(food, index) in item.foods" :key="index" class="food-item border-1px">
              <div class="icon">
                <img width="57" height="57" :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="des">{{food.description}}</p>
                <div class="extra">
                  <span>月售{{food.sellCount}}</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="car-control-wrapper">
                  <car-control :food="food" v-on:cart-add="cartAdd"></car-control>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shop-car :delivery-price="seller.deliveryPrice"
              :min-price="seller.minPrice"
              :select-goods="selectFoods"
              ref="shopcar"></shop-car>
    <select-food @carAdd="cartAdd" :food="selectedFood" ref="food"></select-food>
  </div>
</template>
<script>
import BScroll from 'better-scroll';
import shopCar from './../shopcar/Shopcar';
import carControl from './../cartcontrol/CarControl';
import selectFood from './../food/Food';
const ERR_OK = 0;

export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    };
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
      this.goods.forEach(good => {
        good.foods.forEach(food => {
          if (food.count) {
            foods.push(food);
          }
        });
      });
      return foods;
    }
  },
  created() {
    this.classMap = ['decrease', 'discount', 'guarantee', 'invoice'];
    this.$http.get('./api/goods').then(res => {
      res = res.body;
      if (res.erron === ERR_OK) {
        this.goods = res.data;
        this.$nextTick(() => {
          this._initScroll();
          this._calculateHeight();
        });
      }
    });
  },
  methods: {
    cartAdd(event) {
      // 体验优化  异步执行动画
      this.$nextTick(() => {
        this.$refs.shopcar.drop(event);
      });
    },
    selectMenu(index) {
      let foodList = this.$refs.foodsWrapper.querySelectorAll('.food-list-hook');
      let el = foodList[index];
      this.foodScroll.scrollToElement(el, 300);
    },
    _initScroll() {
      this.meunScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      });
      this.foodScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y));
      });
    },
    _calculateHeight() {
      let foodList = this.$refs.foodsWrapper.querySelectorAll('.food-list-hook');
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      };
    },
    _selectFood(food, event) {
      this.selectedFood = food;
      this.$refs.food.show();
    }
  },
  components: {
    shopCar,
    carControl,
    selectFood
  }
};
</script>
<style lang="stylus">
@import './../../common/stylus/mixin';

.goods {
  display: flex;
  position: absolute;
  top: 174px;
  bottom: 46px;
  width: 100%;
  overflow: hidden;

  .menu-wrapper {
    flex: 0 0 80px;
    width: 80px;
    background: #f3f5f7;

    .menu-item {
      display: table;
      height: 54px;
      width: 56px;
      line-height: 14px;
      padding: 0 12px;
      &.current {
        position: relative;
        z-index: 10;
        margin-top: -1px;
        background: #ffffff;
        font-weight: 700;
        .text{
          border-none()
        }
      }
      .icon {
        vertical-align: top;
        display: inline-block;
        width: 12px;
        height: 12px;
        margin-right: 2px;
        background-size: 12px 12px;
        background-repeat: no-repeat;

        &.decrease {
          bg-image('decrease_3');
        }

        &.discount {
          bg-image('discount_3');
        }

        &.guarantee {
          bg-image('guarantee_3');
        }

        &.invoice {
          bg-image('invoice_3');
        }
      }

      .text {
        font-size: 12px;
        display: table-cell;
        width: 56px;
        vertical-align: middle;
        border-1px(rgba(7, 17, 27, 0.1));
      }
    }
  }

  .goods-wrapper {
    flex: 1;

    .title {
      padding-left: 14px;
      height: 26px;
      line-height: 26px;
      border-left: 2px solid #d9dde1;
      font-size: 12px;
      color: rgb(147, 153, 159);
      background: #f3f5f7;
    }

    .food-item {
      display: flex;
      margin: 18px;
      padding-bottom: 18px;
      border-1px(rgba(7,17,27,0.1))
      &:last-child {
        border-none()
        padding-bottom: 0;
      }
      .icon {
        flex 0 0 57px;
        margin-right: 10px;
      }
      .content {
        flex: 1;
        .name {
          margin: 2px 0 8px 0;
          height: 14px;
          line-height: 14px;
          font-size: 14px;
          color: rgb(7,17,27);
        }
        .des, .extra {
          font-size: 10px;
          color: rgb(147,153,159);
        }
        .des{
          margin-bottom: 8px;
          line-height 12px;
        }
        .extra {
          line-height: 10px;
          .count {
            margin-right: 12px;
          }
        }
        .price{
          font-weight: 700;
          line-height: 24px;
          .now{
            margin-right: 8px;
            font-size: 14px;
            color: rgb(240,20,20);
          }
          .old {
            font-size: 10px;
            text-decoration: line-through;
            color: rgb(147,153,159);
          }
        }
        .car-control-wrapper {
          position absolute
          right 0
          bottom 12px
        }
      }
    }
  }
}
</style>
