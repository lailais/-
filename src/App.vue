<template>
  <div id="app">
    <EleHeader  :seller="seller"></EleHeader>
    <div class="tab">
      <div class="tab-item">
        <router-link to="/">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评价</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <keep-alive>
      <router-view  :goodsList="goodsList"
                    :selectMenuFoods="selectMenuFoods"
                    @refreshAppFood="refreshAppFood"
                    :seller="seller"
      ></router-view>
    </keep-alive>
    <div class="shopCar">
      <shopCar ref="shopCar" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice" :selectFoodList="selectFoodList" @refreshAppFood="refreshAppFood"></shopCar>
    </div>
  </div>
</template>

<script>
  import EleHeader from './components/EleHeader.vue'
  import shopCar from './components/shopCar.vue'
export default {
  name: 'App',
  data () {
    return {
      seller: {},
      goodsList: [],
      selectFoodList: [],
      selectMenuFoods: []
    }
  },
  components: {
    EleHeader,
    shopCar
  },
  created () {
    this.obtainData()
    this.getData()
  },
  methods: {
    obtainData () { // 网络请求获取元数据
      this.axios.get('/api/seller').then(response => {
        let res = response.data
        if (res && res.errCode === 0) {
          this.seller = res.data
        }
      }).catch(err => {
        console.log('error:' + err)
      })
    },
    getData () {
      this.axios.get('api/goods').then(res => {
        res = res.data
        if (res && res.errCode === 0) {
          this.goodsList = res.data // 网络请求数据成功更新data,此时ui还未渲染
//          console.log(res.data)
        }
      }).catch((err) => {
        console.log(err)
      })
    },
    refreshAppFood (obj) {
//            console.log(obj)
      let targetFood = obj.food
      let foods = [] // 被添加的,无重复,有数量的商品
      let menuCount = [] // 左边菜单栏对应每组的商品个数
      this.goodsList.forEach(good => { // 遍历所有商品组
        let count = 0
        good.foods.forEach(food => { // 遍历每组的整组商品
          // 将同名食物更新count
          if (food.name === targetFood.name && food !== targetFood) { // 同名食物但不是同一个对象(即不是本身)
            if (!food.count) {
              this.$set(food, 'count', targetFood.count)
            } else {
              food.count = targetFood.count
            }
          } else if (food.count) { // 避免同名商品重复添加到购物车foods数组中
            for (var i = 0; i < foods.length; i++) {
              if (food.name === foods[i].name) {
                foods.splice(i, 1)
              }
            }
            foods.push(food)
          }
          if (food.count) { // 将每一组menu的数量计算清楚添加到menuCount数组中
            count += food.count
          }
        })
        menuCount.push(count)
      })
      this.selectFoodList = foods
      this.selectMenuFoods = menuCount

      if (obj.el) { // 出现小球
        this.$refs.shopCar.drop(obj.el)
      }
    }
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "./common/sass/mixin";
  #app {
    .tab {
      display: flex;
      width: 100%;
      height: 40px;
      line-height: 40px;
      border-bottom: 1px solid rgba(7, 17, 27, 0.1);
    }
    .tab-item {
      flex: 1;
      text-align: center;
      a {
        display: block;
        font-size: 14px;
        color: rgb(77, 85, 93);
        &.active {
          color: rgb(240, 20, 20);
        }
      }
    }
    .shopCar{
      position: fixed;
      bottom: 0px;
      left: 0px;
    }
  }

</style>
