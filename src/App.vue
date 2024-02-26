<template>
  <div>
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
        <tr v-for="item in sellData" :key="item.id /* TODO 這個還不確定 pk 是什麼 */">
          <td class="price">
            <thousand-text :amount="item.price" />
          </td>
          <td class="size">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="total___" type="sell" />
          </td>
        </tr>
      </tbody>
    </table>

    <!-- 中間的那個數字 -->
    <last-price />

    <!-- 綠色 -->
    <table class="buy">
      <tbody>
        <tr v-for="item in buyData" :key="item.id /* TODO 這個還不確定 pk 是什麼 */">
          <td class="price">
            <thousand-text :amount="item.price" />
          </td>
          <td class="size">
            <thousand-text :amount="item.size" />
          </td>
          <td class="total">
            <total-bar :value="item.total" :total="total___" type="buy" />
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
      amount: 123123,
      total___: 123123123,

      sellData: [],
      buyData: [],
    }
  },

  mounted() {
    this.initOrderbookSocket()
  },

  methods: {
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
        if (data.type !== 'snapshot') return

        const { bids, asks } = data

        this.sellData = asks.map(_mapFunc).slice(-8)
        this.buyData = bids.map(_mapFunc).slice(0, 8)
      })

      function _mapFunc(item) {
        const [price, size] = item
        return { price, size, key: `key-${price}`, total: 123 }
      }
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
</style>
