<template>
  <div id="app">
    <el-row>
      <el-col :span='8' class="pos-order" id="order-list">
        <el-tabs style="margin: 10px;">
          <el-tab-pane label="点餐">
            <!-- show-summary -->
            <el-table :data="goodsList" border style="width: 100%">
              <el-table-column label="商品名称" prop="goodsName"></el-table-column>
              <el-table-column label="数量" prop="count" width="70"></el-table-column>
              <el-table-column label="价格" prop="price" width="70"></el-table-column>
              <el-table-column label="操作" fixed="right" width="100">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <span style="margin: 10px">数量:{{totalCount}}</span>
              <span style="margin:10px">总价: ¥{{totalMoney}}元</span>
            </div>
            <div class="order-btns">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods()">清空</el-button>
              <el-button type="success" @click="checkout()">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">挂单</el-tab-pane>
          <el-tab-pane label="外卖">外卖</el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span='16'>
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="goods in oftenGoods" :key="goods.goodsName" @click="addOrderList(goods)">
                <span class="o-price">{{goods.goodsName}}</span>
                <span>¥{{goods.price}}元</span>
              </li>
            </ul>
          </div>
        </div>
        <el-tabs class="goods-type">
          <el-tab-pane label="汉堡">
            <ul class='cookList'>
              <li v-for="item in type0Goods" :key="item.goodsId" @click="addOrderList(item)">
                <span class="foodImg">
                  <img :src="item.goodsImg" width="100%">
                </span>
                <span class="foodName">{{item.goodsName}}</span>
                <span class="foodPrice">￥{{item.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="小食">
            <ul class='cookList'>
              <li v-for="item in type1Goods" :key="item.goodsId" @click="addOrderList(item)">
                <span class="foodImg">
                  <img :src="item.goodsImg" width="100%">
                </span>
                <span class="foodName">{{item.goodsName}}</span>
                <span class="foodPrice">￥{{item.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="饮料">
            <ul class='cookList'>
              <li v-for="item in type2Goods" :key="item.goodsId" @click="addOrderList(item)">
                <span class="foodImg">
                  <img :src="item.goodsImg" width="100%">
                </span>
                <span class="foodName">{{item.goodsName}}</span>
                <span class="foodPrice">￥{{item.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
          <el-tab-pane label="套餐">
            <ul class='cookList'>
              <li v-for="item in type3Goods" :key="item.goodsId">
                <span class="foodImg">
                  <img :src="item.goodsImg" width="100%">
                </span>
                <span class="foodName">{{item.goodsName}}</span>
                <span class="foodPrice">￥{{item.price}}元</span>
              </li>
            </ul>
          </el-tab-pane>
        </el-tabs>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data () {
    return {
      msg: 'data',
      totalCount: 0,
      totalMoney: 0,
      goodsList: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: []
    }
  },
  created () {
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
      .then(response => {
        // console.log(response)
        this.oftenGoods = response.data
        // alert('常用商品数据请求成功!')
      })
      .catch(error => {
        console.log(error)
        alert('网络错误，不能访问')
      })
    axios.get('http://jspang.com/DemoApi/typeGoods.php')
      .then(response => {
        console.log(response)
        this.type0Goods = response.data[0]
        this.type1Goods = response.data[1]
        this.type2Goods = response.data[2]
        this.type3Goods = response.data[3]
      })
      .catch(error => {
        console.log(error)
      })
  },
  mounted () {
    var orderHeight = document.body.clientHeight
    console.log(orderHeight)
    document.getElementById('order-list').style.height = orderHeight + 'px'
  },
  methods: {
    // 添加订单列表的方法
    addOrderList (goods) {
      this.totalCount = 0 // 汇总数量清0
      this.totalMoney = 0
      let isHave = false
      // 判断是否这个商品已经存在于订单列表
      for (let i = 0; i < this.goodsList.length; i++) {
        console.log(this.goodsList[i].goodsId)
        if (this.goodsList[i].goodsId === goods.goodsId) {
          isHave = true // 存在
        }
      }
      // 根据isHave的值判断订单列表中是否已经有此商品
      if (isHave) {
        // 存在就进行数量添加
        let arr = this.goodsList.filter(o => o.goodsId === goods.goodsId)
        arr[0].count++
        // console.log(arr);
      } else {
        // 不存在就推入数组
        let newGoods = { goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1 }
        this.goodsList.push(newGoods)
      }

      // 进行数量和价格的汇总计算
      this.getAllMoney()
    },
    // 删除单个商品
    delSingleGoods (goods) {
      console.log(goods)
      this.goodsList = this.goodsList.filter(o => o.goodsId !== goods.goodsId)
      // 进行数量和价格的汇总计算
      this.getAllMoney()
    },
    // 汇总数量和金额
    getAllMoney () {
      this.totalCount = 0
      this.totalMoney = 0
      if (this.goodsList) {
        this.goodsList.forEach((element) => {
          this.totalCount += element.count
          this.totalMoney = this.totalMoney + (element.price * element.count)
        })
      }
    },
    // 删除所有商品
    delAllGoods () {
      this.goodsList = []
      this.totalCount = 0
      this.totalMoney = 0
    },
    // 结账
    checkout () {
      if (this.totalCount > 0) {
        this.delAllGoods()
        this.$message({
          message: '结账成功!',
          type: 'success'
        })
      } else {
        this.$message.error('加油,再拿一单!')
      }
    }
  }
}

</script>

<style scoped>
.pos-order {
    background-color: beige;
    border-right: 1px solid #c0ccda;
}

.order-btns {
    margin-top: 10px;
    text-align: center;
}

.title {
    height: 20px;
    border-bottom: 1px solid #d3dce6;
    background-color: #f9fafc;
    padding: 10px;
}

.often-goods-list ul li {
    list-style: none;
    float: left;
    border: 1px solid #e5e9f2;
    padding: 10px;
    margin: 10px;
    background-color: #fff;
    cursor: pointer;
}

.o-price {
    color: #58b7ff;
}

.goods-type {
    clear: both;
    margin: 10px;
}

.cookList li {
    list-style: none;
    width: 23%;
    border: 1px solid #e5e9f2;
    height: auot;
    overflow: hidden;
    background-color: #fff;
    padding: 2px;
    float: left;
    margin: 2px;
    cursor: pointer;
}
.cookList li span {
    display: block;
    float: left;
}
.foodImg {
    width: 40%;
}
.foodName {
    font-family: Georgia, "Times New Roman", Times, serif;
    font-size: 15px;
    padding-left: 10px;
    color: brown;
}
.foodPrice {
    font-size: 16px;
    padding-left: 10px;
    padding-top: 10px;
}

.totalDiv {
    padding: 10px;
    background-color: white;
    text-align: center;
}
</style>
