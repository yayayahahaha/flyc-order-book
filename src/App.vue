<template>
  <div class="fade-out-red">
    <h3 class="title">Order Book</h3>

    <!-- 紅色 -->
    <order-table :data="sellData" :total="sellTotal" type="sell" />

    <!-- 中間的那個數字 -->
    <last-price />

    <!-- 綠色 -->
    <order-table :data="buyData" :total="buyTotal" type="buy" />
  </div>
</template>

<script>
import LastPrice from './LastPrice.vue'
import OrderTable from './OrderTable.vue'

const FADE_OUT_GREEN_CLASS = 'fade-out-green'
const FADE_OUT_RED_CLASS = 'fade-out-red'
const SELL_TEXT = 'sell'
const BUY_TEXT = 'buy'

export default {
  components: { LastPrice, OrderTable },

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
      const neededItem = this.sellData[0]
      if (neededItem == null) return 0

      return neededItem.total
    },

    buyTotal() {
      const neededItem = this.buyData.slice(-1)[0]
      if (neededItem == null) return 0

      return neededItem.total
    },

    sellOriMap() {
      return Object.fromEntries(this.sellOriData.map((item) => [item.key, item]))
    },
    sellMap() {
      return Object.fromEntries(this.sellData.map((item) => [item.key, item]))
    },

    buyOriMap() {
      return Object.fromEntries(this.buyOriData.map((item) => [item.key, item]))
    },
    buyMap() {
      return Object.fromEntries(this.buyData.map((item) => [item.key, item]))
    },
  },

  mounted() {
    this.initOrderbookSocket()

    // NEXT
    // 1. 把 socket 停下、然後寫一些 function 去模擬收到各種類型的 socket 時會有的情況
    // 看這些 function 要不要留下來可以做測試之類的
    // 2. 把 task 裡面寫道的那些要重新 subscript 的場景模擬看看，跟那個什麼 seqNum 有關係

    // TODO TESTING CODES
    window.vm = this
  },

  methods: {
    handleSnapshot(data, type) {
      const { bids, asks } = data

      this.buyOriData = _oriPart(bids, this.buyOriMap, this.buyOriData)
      this.sellOriData = _oriPart(asks, this.sellOriMap, this.sellOriData)

      const newSellData = _handleTotal(this.sellOriData.slice(-8)).reverse()
      const newBuyData = _handleTotal(this.buyOriData.slice(0, 8))

      // 如果是 snapshot 的話，代表是新的、直接指定上去就好
      if (type === 'snapshot') {
        this.sellData = newSellData
        this.buyData = newBuyData
        return
      }

      // 這是 delta 的部分
      this.sellData = combineUpdate(newSellData, this.sellMap, 'sell')
      this.buyData = combineUpdate(newBuyData, this.buyMap, 'buy')

      function combineUpdate(newList, currentMap, type) {
        return newList.reduce((list, item) => {
          const { key, total, size } = item
          const currentItem = currentMap[key]
          if (currentItem == null) {
            const className = type === SELL_TEXT ? FADE_OUT_RED_CLASS : type === BUY_TEXT ? FADE_OUT_GREEN_CLASS : ''
            list.push({ ...item, class: className })
          } else {
            // total 直接覆寫
            currentItem.total = total

            // size 的閃爍處理
            switch (true) {
              case currentItem.size < size:
                currentItem.sizeClass = FADE_OUT_GREEN_CLASS
                break

              case currentItem.size > size:
                currentItem.sizeClass = FADE_OUT_RED_CLASS
                break
            }
            currentItem.size = size

            // reset class: TODO 在連續修正的時候可能會有問題
            setTimeout(() => {
              currentItem.class = ''
            }, 350)

            list.push(currentItem)
          }

          return list
        }, [])
      }

      function _handleTotal(list) {
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

      function _oriPart(delta, oriMap, oriList) {
        delta.forEach((item) => {
          const [oriPrice, oriSize] = item
          const [price, size] = [Number(oriPrice), Number(oriSize)]
          const key = `key-${price}`
          const oriItem = oriMap[key]

          if (oriItem == null) {
            oriList.push({ key, price, size, total: 0 })
            oriList.sort((a, b) => a.price - b.price)
          } else {
            Object.assign(oriItem, { price, size })
          }
        })

        return oriList.filter((item) => item.size !== 0)
      }
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
            this.handleSnapshot(data, 'snapshot')
            break

          case 'delta':
            this.handleSnapshot(data, 'delta')
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
</style>

<!-- reference -->
<!-- https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations -->
