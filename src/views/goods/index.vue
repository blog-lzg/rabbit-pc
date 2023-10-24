<template>
  <div class='xtx-goods-page'>
    <!-- 如果商品加载完成并且商品存在，显示商品信息 -->
    <div class="container" v-if="!loading && goods">
      <!-- 面包屑导航 -->
      <XtxBread>
        <XtxBreadItem to="/">首页</XtxBreadItem>
        <XtxBreadItem :to="`/category/${goods.categories[1].id}`">{{goods.categories[1].name}}</XtxBreadItem>
        <XtxBreadItem :to="`/category/sub/${goods.categories[0].id}`">{{goods.categories[0].name}}</XtxBreadItem>
        <XtxBreadItem>{{goods.name}}</XtxBreadItem>
      </XtxBread>
      <!-- 商品信息区域 -->
      <div class="goods-info">
        <div class="media">
          <!-- 商品图片 -->
          <GoodsImage :images="goods.mainPictures" />
          <!-- 商品销售信息 -->
          <GoodsSales />
        </div>
        <div class="spec">
          <!-- 商品名称 -->
          <GoodsName :goods="goods" />
          <!-- 商品规格选择 -->
          <GoodsSku :goods="goods" @change="changeSku" />
          <!-- 商品数量选择 -->
          <XtxNumbox label="数量" v-model="num" :max="goods.inventory" />
          <!-- 加入购物车按钮 -->
          <XtxButton @click="insertCart()" type="primary" style="margin-top:20px">加入购物车</XtxButton>
        </div>
      </div>
      <!-- 商品推荐区域 -->
      <GoodsRelevant :goodsId="goods.id" />
      <!-- 商品详情区域 -->
      <div class="goods-footer">
        <div class="goods-article">
          <!-- 商品评价 -->
          <GoodsTabs />
          <!-- 商品注意事项 -->
          <GoodsWarn />
        </div>
        <!-- 热销商品区域 -->
        <div class="goods-aside">
          <GoodsHot />
          <GoodsHot :type="2" />
        </div>
      </div>
    </div>
    <!-- 如果商品正在加载或者商品不存在，显示加载中 -->
    <div v-else class="loading"></div>
  </div>
</template>

<script>
import GoodsRelevant from './components/goods-relevant'
import GoodsImage from './components/goods-image'
import GoodsSales from './components/goods-sales'
import GoodsName from './components/goods-name'
import GoodsSku from './components/goods-sku'
import GoodsTabs from './components/goods-tabs'
import GoodsHot from './components/goods-hot'
import GoodsWarn from './components/goods-warn'
import { nextTick, provide, ref, watch } from 'vue'
import { findGoods } from '@/api/product'
import { useRoute } from 'vue-router'
import { useStore } from 'vuex'
import Message from '@/components/library/Message'

export default {
  name: 'XtxGoodsPage',
  components: { GoodsRelevant, GoodsImage, GoodsSales, GoodsName, GoodsSku, GoodsTabs, GoodsHot, GoodsWarn },
  setup () {
    // 使用商品信息
    const { goods, loading } = useGoods()

    // 修改商品规格
    const changeSku = (sku) => {
      // 如果sku存在，修改商品的价格和库存信息
      if (sku.skuId) {
        goods.value.price = sku.price
        goods.value.oldPrice = sku.oldPrice
        goods.value.inventory = sku.inventory
      }
      // 记录选择的sku
      currSku.value = sku
    }

    // 提供商品数据给子组件使用
    provide('goods', goods)

    // 选择的商品数量
    const num = ref(1)

    // 加入购物车操作
    const store = useStore()
    const currSku = ref(null)
    const insertCart = () => {
      // 如果选择了完整的规格，将商品添加到购物车
      if (currSku.value && currSku.value.skuId) {
        const { skuId, specsText: attrsText, inventory: stock } = currSku.value
        const { id, name, price, mainPictures } = goods.value
        store.dispatch('cart/insertCart', {
          skuId,
          attrsText,
          stock,
          id,
          name,
          price,
          nowPrice: price,
          picture: mainPictures[0],
          selected: true,
          isEffective: true,
          count: num.value
        }).then(() => {
          Message({ type: 'success', text: '加入购物车成功' })
        })
      } else {
        // 如果没有选择完整的规格，提示用户
        Message({ text: '请选择完整规格' })
      }
    }

    return { goods, loading, changeSku, num, insertCart }
  }
}

// 获取商品详情
const useGoods = () => {
  // 商品信息
  const goods = ref(null)
  const route = useRoute()
  // 商品加载状态
  const loading = ref(false)

  // 监听路由变化，如果商品ID变化，重新获取商品信息
  watch(() => route.params.id, (newVal) => {
    if (newVal && `/product/${newVal}` === route.path) {
      loading.value = true
      findGoods(route.params.id).then(data => {
        goods.value = null
        nextTick(() => {
          goods.value = data.result
          loading.value = false
        })
      })
    }
  }, { immediate: true })

  return { goods, loading }
}
</script>

<style scoped lang='less'>
.loading {
  height: 580px;
  width: 1240px;
  margin: 72px auto 20px;
  background: rgba(255,255,255,.9) url(../../assets/images/loading.gif) no-repeat center;
}
.goods-info {
  min-height: 600px;
  background: #fff;
  display: flex;
  .media {
    width: 580px;
    height: 600px;
    padding: 30px 50px;
  }
  .spec {
    flex: 1;
    padding: 30px 30px 30px 0;
  }
}
.goods-footer {
  display: flex;
  margin-top: 20px;
  .goods-article {
    width: 940px;
    margin-right: 20px;
  }
  .goods-aside {
    width: 280px;
    min-height: 1000px;
  }
}
// .goods-tabs {
//   min-height: 600px;
//   background: #fff;
// }
// .goods-warn {
//   min-height: 600px;
//   background: #fff;
//   margin-top: 20px;
// }
</style>
