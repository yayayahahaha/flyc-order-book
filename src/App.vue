<template>
  <div class="fade-out-red">
    <h3 class="title">Order Book</h3>

    <!-- 紅色 -->
    <table class="sell">
      <thead>
        <tr>
          <th class="table-header price">Price (USD)</th>
          <th class="table-header size">Size</th>
          <th class="table-header total">Total</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sellData" :key="item.key">
          <td class="price">
            <thousand-text :amount="item.price" />
          </td>
          <td class="size" :class="item.class">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="sellTotal" type="sell" />
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 中間的那個數字 -->
    <last-price />

    <!-- 綠色 -->
    <table class="buy">
      <tbody>
        <tr v-for="item in buyData" :key="item.key">
          <td class="price">
            <thousand-text :amount="item.price" />
          </td>
          <td class="size">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="buyTotal" type="buy" />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import LastPrice from './LastPrice.vue'
import ThousandText from './ThousandText.vue'
import TotalBar from './TotalBar.vue'

export default {
  components: { LastPrice, ThousandText, TotalBar },

  data() {
    return {
      sellOriData: [],
      sellData: [],

      buyOriData: [],
      buyData: [],
    }
  },

  computed: {
    sellTotal() {
      return this.sellData[0].total
    },

    buyTotal() {
      return this.buyData.slice(-1)[0].total
    },

    sellOriMap() {
      return Object.fromEntries(this.sellOriData.map((item) => [item.price, item]))
    },
    sellMap() {
      return Object.fromEntries(this.sellData.map((item) => [item.price, item]))
    },

    buyOriMap() {
      return Object.fromEntries(this.buyOriData.map((item) => [item.price, item]))
    },
    buyMap() {
      return Object.fromEntries(this.buyData.map((item) => [item.price, item]))
    },
  },

  mounted() {
    this.initOrderbookSocket()

    window.vm = this
  },

  methods: {
    handleSnapshot(data) {
      const { bids, asks } = data

      function oriPart(delta, oriMap, oriList) {
        delta.forEach((item) => {
          const [price, size] = item
          const oriItem = oriMap[price]
          if (oriItem == null) {
            oriList.push({ price, size })
            oriList.sort((a, b) => Number(a.price) - Number(b.price))
          } else {
            Object.assign(oriItem, { price, size })
          }
        })

        return oriList.filter((item) => Number(item.size) !== 0)
      }

      this.buyOriData = oriPart(bids, this.buyOriMap, this.buyOriData)
      this.sellOriData = oriPart(asks, this.sellOriMap, this.sellOriData)

      // TODO 這邊以下要處理一下新增和閃爍的那部分

      this.sellData = handleTotal(this.sellOriData.slice(-8).map(_mapFunc).reverse()).reverse()
      this.buyData = handleTotal(this.buyOriData.slice(0, 8).map(_mapFunc))

      function handleTotal(list) {
        return list.reduce(
          (payload, item) => {
            const { list, total } = payload
            const nextTotal = total + item.size
            list.push({ ...item, total: nextTotal })

            return { list, total: nextTotal }
          },
          { list: [], total: 0 }
        ).list
      }

      function _mapFunc(item) {
        const { price, size } = item
        return { price, size: Number(size), key: `key-${price}`, total: 0 }
      }
    },

    handleDelta(data) {
      const { bids, asks } = data

      function handleList(deltaList, infoMap) {
        deltaList.forEach((item) => {
          const [price, size] = item

          if (infoMap[price] != null) {
            // 已經存在
            switch (true) {
              case infoMap[price].size < size:
                infoMap[price].class = 'fade-out-green'
                break

              case infoMap[price].size > size:
                infoMap[price].class = 'fade-out-red'
                break
            }
            infoMap[price].size = size

            // reset class: TODO 在連續修正的時候可能會有問題
            setTimeout(() => {
              infoMap[price].class = ''
            }, 350)
          } else {
            // 新的
            // currentList.push({ price, size, total: 123 })
          }
        })
      }

      handleList(asks, this.sellMap)

      handleList(bids, this.buyMap)
    },

    initOrderbookSocket() {
      const TOPIC_TEXT = 'update:BTCPFC'
      const orderbookSocket = new WebSocket('wss://ws.btse.com/ws/oss/futures')

      // Connection opened
      orderbookSocket.addEventListener('open', () => {
        orderbookSocket.send(JSON.stringify({ op: 'subscribe', args: [TOPIC_TEXT] }))
      })

      // Listen for messages
      orderbookSocket.addEventListener('message', (event) => {
        const { topic, data } = JSON.parse(event.data)

        if (topic !== TOPIC_TEXT) return
        switch (data.type) {
          case 'snapshot':
            this.handleSnapshot(data)
            break

          case 'delta':
            console.log('receive delta')
            this.handleSnapshot(data)
            break
        }
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.title {
  color: var(--default-text-color);
  padding: 12px 12px;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: var(--default-text-color-for-line);
}

table {
  padding: 0px 6px;
  width: 100%;

  .table-header {
    font-weight: 400;
    color: var(--quote-table-head-text-color);
  }

  tbody {
    font-weight: bold;
    color: var(--default-text-color);
  }

  .price {
    width: 30%;
    text-align: left;
  }
  .size {
    width: 20%;
    text-align: right;
  }
  .total {
    width: 50%;
    text-align: right;
  }

  &.sell {
    tbody {
      .price {
        color: var(--sell-quote-price-text-color);
      }
    }
  }

  &.buy {
    tbody {
      .price {
        color: var(--buy-quote-price-text-color);
      }
    }
  }
}

.fade-out-red {
  animation-duration: 0.3s;
  animation-name: fade_out_red;
  animation-fill-mode: forwards;
}
.fade-out-green {
  animation-duration: 0.3s;
  animation-name: fade_out_green;
  animation-fill-mode: forwards;
}

@keyframes fade_out_red {
  0% {
    background-color: var(--animation-flash-red-background-color);
  }
  100% {
    background-color: transparent;
  }
}

@keyframes fade_out_green {
  0% {
    background-color: var(--animation-flash-green-background-color);
  }
  100% {
    background-color: transparent;
  }
}
</style>

<!-- reference -->
<!-- https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations -->
