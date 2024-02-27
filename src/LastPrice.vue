<template>
  <div class="container" :class="currentClass">
    <h2>
      <thousand-text v-if="firstMessage" :amount="amount" />
      <span v-else>-</span>
    </h2>
    <ArrowIcon v-if="showIcon" class="icon" />
  </div>
</template>

<script>
import ThousandText from './ThousandText.vue'
import ArrowIcon from './assets/IconArrowDown.svg?component'

const DEFAULT_TEXT = 'default'
const RED_TEXT = 'red'
const GREEN_TEXT = 'green'

export default {
  name: 'LastPrice',

  components: { ThousandText, ArrowIcon },

  data() {
    return {
      socket: null,

      currentClass: DEFAULT_TEXT,
      amount: 0,
      firstMessage: false,
    }
  },

  computed: {
    showIcon() {
      return this.firstMessage && this.currentClass !== DEFAULT_TEXT
    },
  },

  mounted() {
    this.initLastPriceSocket()
  },

  methods: {
    initLastPriceSocket() {
      const socket = new WebSocket('wss://ws.btse.com/ws/futures')
      this.socket = socket

      socket.addEventListener('open', () => {
        socket.send(JSON.stringify({ op: 'subscribe', args: ['tradeHistoryApi:BTCPFC'] }))
      })

      socket.addEventListener('message', (event) => {
        this.handleSocketMessage(event)
      })
    },

    // Call by parent component also
    handleSocketMessage(event) {
      this.firstMessage = true

      const { topic, data } = JSON.parse(event.data)
      if (topic !== 'tradeHistoryApi') return // 會有一個 { event: String, channel: String[] } 的 message

      const { price } = data.sort((a, b) => b.timestamp - a.timestamp)[0]

      switch (true) {
        case this.amount === price:
          this.currentClass = DEFAULT_TEXT
          break

        case this.amount < price:
          this.currentClass = GREEN_TEXT
          break

        case this.amount > price:
          this.currentClass = RED_TEXT
          break
      }

      this.amount = price
    },

    // Call by parent component
    stopSocket() {
      if (!(this.socket instanceof WebSocket)) {
        alert(`[${this.$options.name}]: socket is not an instance of WebSocket!`)
        return
      }
      this.socket.close()
    },

    // 測試用 function
    updateLastPrice() {
      const randomNum = Math.floor(Math.random() * 3)
      console.log('randomNum:', randomNum)
      let price
      switch (randomNum) {
        case 0:
          price = Number(this.amount) + Math.round(Math.random() * 100 + 100)
          break

        case 1:
          price = Number(this.amount) - Math.round(Math.random() * 100)
          break

        case 2:
          price = this.amount
          break
      }

      const fakeSocketEvent = {
        data: JSON.stringify({
          topic: 'tradeHistoryApi',
          data: [{ price }],
        }),
      }
      this.handleSocketMessage(fakeSocketEvent)
    },
  },
}
</script>

<style lang="scss" scoped>
.container {
  margin: 6px 0px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon {
  margin-left: 6px;
}

.default {
  background-color: var(--last-price-default-background-color);
  color: var(--default-text-color);
}

// 紅色
.red {
  background-color: var(--sell-quote-accumulative-total-size-bar-color);
  color: var(--sell-quote-price-text-color);
}

// 綠色
.green {
  background-color: var(--buy-quote-accumulative-total-size-bar-color);
  color: var(--buy-quote-price-text-color);

  .icon {
    transform: rotate(180deg);
  }
}
</style>

<!-- reference -->
<!-- https://medium.com/@alekswebnet/import-svg-to-vite-vue-projects-8168826a76c1 -->
<!-- https://developer.mozilla.org/en-US/docs/Web/API/WebSocket -->
